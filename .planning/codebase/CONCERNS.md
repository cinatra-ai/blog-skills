# Codebase Concerns

**Analysis Date:** 2026-06-09

## Tech Debt

**No lockfile committed:**
- Issue: `package.json` exists but no `pnpm-lock.yaml` is committed. CI explicitly uses `--no-frozen-lockfile`, meaning dependency resolution is non-deterministic between runs.
- Files: `package.json`, `.github/workflows/ci.yml` (line 81)
- Impact: A transitive dependency could change between CI runs and a release without any signal. Reproducing a past build is impossible.
- Fix approach: Commit a lockfile. If the repo is truly a "source mirror" that cannot install standalone, document that explicitly and consider a CI step that fails if a lockfile is accidentally added.

**`tsconfig.json` targets a non-existent `src/` directory:**
- Issue: `tsconfig.json` sets `rootDir: "src"` and `include: ["src/**/*.ts", "src/**/*.tsx"]`, but no `src/` directory exists. The repo contains only SKILL.md prompt files under `skills/`.
- Files: `tsconfig.json`
- Impact: Any CI step that attempts a real `tsc` compile will fail with TS18003 ("No inputs were found"). The CI workflow works around this by detecting "no tracked TS" and skipping typecheck, but the tsconfig is misleading boilerplate and could cause confusion when someone tries to add real TypeScript code without realizing the config is misaligned.
- Fix approach: Either remove `tsconfig.json` entirely (this is a content-only skill repo with no TypeScript) or update `include`/`rootDir` to match actual source locations if TypeScript is ever introduced.

**`noImplicitAny: false` with `strict: true`:**
- Issue: `tsconfig.json` enables `strict` mode but then explicitly disables `noImplicitAny`. These two settings partially contradict each other since `strict` sets `noImplicitAny: true`.
- Files: `tsconfig.json` (lines 8–9)
- Impact: Any future TypeScript added to this repo will silently allow implicit `any`, removing a key type-safety check despite the appearance of strict mode.
- Fix approach: Remove `noImplicitAny: false` to let `strict` take full effect, or explicitly acknowledge the exception with a comment explaining why implicit `any` is acceptable here.

## Known Bugs

**Not detected** — the repo contains only prompt text in SKILL.md files and no executable code, so no runtime bugs are observable.

## Security Considerations

**Prompt injection via transcript inputs:**
- Risk: The skills explicitly accept "attached transcript files" as input and instruct the model to anonymize them. A malicious or crafted transcript could contain prompt-injection content designed to override the anonymization instructions or leak the system prompt.
- Files: `skills/generate-blog-ideas/SKILL.md`, `skills/generate-blog-post-draft/SKILL.md`
- Current mitigation: The prompts instruct the model not to surface transcript origins, speakers, or third-party names, but there is no structural sandboxing or input validation beyond LLM instruction-following.
- Recommendations: Consider prefixing user-supplied transcript content with a clear delimiter and restating the anonymization rule after the transcript block to reduce injection surface.

**`.npmrc` present with `auto-install-peers=false`:**
- Risk: The `.npmrc` file is committed. If it ever contains registry credentials or auth tokens, those would be exposed in the public repo.
- Files: `.npmrc`
- Current mitigation: Current contents are benign (`auto-install-peers=false` only). No credentials present.
- Recommendations: Add `.npmrc` to `.gitignore` or maintain a policy check that it never contains auth tokens.

**Release workflow uses `secrets: inherit` with `id-token: write`:**
- Risk: The release workflow grants `id-token: write` (OIDC token) and inherits all org secrets. If the reusable workflow at `cinatra-ai/.github` is ever compromised or its `@main` ref is poisoned, the publishing token and OIDC identity are exposed.
- Files: `.github/workflows/release.yml` (lines 25–30)
- Current mitigation: The reusable workflow is pinned to `@main` (a branch, not a SHA), so any push to that branch affects all consuming repos.
- Recommendations: Pin the reusable workflow to a specific commit SHA rather than `@main` to prevent supply-chain drift.

## Performance Bottlenecks

**Not applicable** — the repo contains only static prompt text. There are no runtime execution paths or data processing code.

## Fragile Areas

**`generate-linkedin-post` skill has no `match_when` agent scope:**
- Files: `skills/generate-linkedin-post/SKILL.md`
- Why fragile: The other two skills (`generate-blog-ideas`, `generate-blog-post-draft`) restrict themselves to `@cinatra-ai/wordpress-agent` and `@cinatra-ai/drupal-agent` via `match_when`. The LinkedIn skill has no `match_when` at all, meaning it will be matched by any agent in the skills catalog. This could cause it to activate unexpectedly in unrelated agent contexts.
- Safe modification: Add a `match_when` block constraining the LinkedIn skill to appropriate agent IDs.
- Test coverage: No tests exist for skill routing behavior.

**Skill prompts contain no versioning or change-history markers:**
- Files: `skills/generate-blog-ideas/SKILL.md`, `skills/generate-blog-post-draft/SKILL.md`, `skills/generate-linkedin-post/SKILL.md`
- Why fragile: Prompt wording changes directly alter LLM output quality and brand safety. There is no mechanism to A/B test prompt changes, roll back a bad edit, or signal to consuming agents that a breaking prompt change occurred.
- Safe modification: Treat prompt edits as semver changes; bump `package.json` version on any meaningful prompt edit.
- Test coverage: No golden-output or regression tests exist.

**CI kind-gates job has only a placeholder step:**
- Files: `.github/workflows/ci.yml` (lines 129–141)
- Why fragile: The `kind-gates` job runs `echo "No kind-specific gate for this extension kind."` — it performs no actual validation of the skill manifest structure, capability keys, or SKILL.md frontmatter. A malformed skill registration could pass CI silently.
- Safe modification: Add a validation step that parses each `SKILL.md` frontmatter and verifies `name`, `description`, and optionally `match_when` fields are present and non-empty, and that all `capabilities` keys in `package.json` have a corresponding `skills/` subdirectory with a `SKILL.md`.

## Scaling Limits

**Not applicable** — this is a static prompt/skill content repo with no runtime infrastructure.

## Dependencies at Risk

**No runtime dependencies declared:**
- The `package.json` declares no `dependencies`, `devDependencies`, or `peerDependencies`. This means no dependency drift risk for the package itself.
- Risk: The CI pipeline uses `actions/checkout@v4`, `actions/setup-node@v4`, and `corepack pnpm` without pinning to specific SHAs. A tag could be force-pushed to point at different code.
- Impact: CI environment drift without a visible signal.
- Migration plan: Pin all GitHub Actions to full commit SHAs in `.github/workflows/ci.yml` and `.github/workflows/release.yml`.

## Missing Critical Features

**No skill manifest validation in CI:**
- Problem: There is no automated check that `package.json` `cinatra.capabilities` entries map 1:1 to `skills/` subdirectories, or that each SKILL.md frontmatter is valid YAML with required fields.
- Blocks: A mismatch between declared capabilities and actual skill files would only be caught at runtime by the consuming agent, not at PR time.

**No output quality tests:**
- Problem: There are no tests that run the prompts against a mock LLM and assert output structure (e.g., that a blog draft contains a teaser excerpt, uses markdown headings, avoids repeating the title as the opening line as required by `generate-blog-post-draft`).
- Blocks: Prompt regressions introduced by edits are undetectable without human review.

## Test Coverage Gaps

**No tests of any kind:**
- What's not tested: Skill frontmatter validity, capability-to-directory mapping, prompt instruction completeness, and LinkedIn post `match_when` absence.
- Files: entire `skills/` directory
- Risk: Structural regressions (missing fields, broken capability keys, prompt text deletions) pass CI undetected.
- Priority: Medium — the repo is small and static, but prompt correctness is brand-critical for the consuming agents.

---

*Concerns audit: 2026-06-09*
