# Testing Patterns

**Analysis Date:** 2026-06-09

## Test Framework

**Runner:** Not applicable — no test runner is configured.

**Config:** No `jest.config.*`, `vitest.config.*`, or equivalent file is present.

**Run Commands:**
```bash
# CI runs: corepack pnpm test --if-present
# No test script is defined in package.json — the step exits 0 silently.
```

## Test File Organization

No test files exist in this repository. This is a content-only skill extension consisting entirely of SKILL.md prompt files. There is no source code to unit test.

## What CI Does Instead

The CI pipeline (`.github/workflows/ci.yml`) enforces correctness through structural validation gates rather than unit tests:

**Dependency shape gate:**
- Validates that no `@cinatra-ai/*` packages appear in `dependencies`, `devDependencies`, or `optionalDependencies`
- Validates that all first-party peers are marked `peerDependenciesMeta[pkg].optional = true`
- Implemented as an inline `node -e` script in the `ci.yml` `build` job
- Hard failure (exit 2) on violation

**Pack dry-run:**
- `npm pack --dry-run` validates the publish payload and package shape on every CI run
- Catches missing files, malformed `package.json`, and manifest issues before release

**Kind-specific gate:**
- The `kind-gates` job runs after `build`; for `skill` kind, no additional gate is configured today (placeholder step echoes "No kind-specific gate")

**Typecheck:**
- `tsconfig.json` is present but CI self-detects that no `.ts` files are tracked via `git ls-files '*.ts' '*.tsx' '*.mts' '*.cts'` and skips the typecheck step cleanly

## Coverage

**Requirements:** Not applicable — no tests exist.

**Coverage tooling:** Not detected.

## Test Types

**Unit Tests:** Not present.

**Integration Tests:** Not present.

**E2E Tests:** Not present.

**Structural/Manifest Validation:** Performed inline in CI via `node -e` scripts in `.github/workflows/ci.yml`.

## Skill Prompt Validation

There is no automated linter or schema validator for the SKILL.md frontmatter fields. Correctness of:
- `name` matching the directory name
- `match_when` agent IDs being valid
- `cinatra.capabilities` keys in `package.json` pointing to real `skills/` subdirectories

...is enforced only by convention and human review.

## Future Testing Guidance

If TypeScript source is added to `src/`, the CI pipeline will automatically:
1. Install dependencies (`corepack pnpm install --no-frozen-lockfile`)
2. Run typecheck (via `pnpm run typecheck` script, local `tsc --noEmit`, or ephemeral `npx tsc`)
3. Run `pnpm test --if-present`

Add a `test` script to `package.json` and a test framework (e.g., vitest) to activate test execution. The CI `Test` step is already wired and waiting.

---

*Testing analysis: 2026-06-09*
