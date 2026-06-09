# Coding Conventions

**Analysis Date:** 2026-06-09

## Repository Type

This is a **content-only skill extension** — a Cinatra Marketplace package containing only SKILL.md prompt files (no TypeScript source). There is no `src/` directory and no compiled code. Conventions below reflect the authoring and structural rules enforced across this repo.

## Skill File Conventions

**Format:** Each skill is a YAML-frontmatter Markdown file named `SKILL.md`, located in its own directory under `skills/`.

**Frontmatter fields:**
- `name`: kebab-case skill identifier matching the directory name (e.g., `generate-blog-ideas`)
- `description`: Single sentence describing the skill's purpose
- `match_when`: List of agent scope conditions (e.g., `agent_id: "@cinatra-ai/wordpress-agent"`)

**Body:** Plain English system prompt instructions with no markdown headings. Terse, imperative voice.

**Example pattern:**
```
skills/
  generate-blog-ideas/SKILL.md
  generate-blog-post-draft/SKILL.md
  generate-linkedin-post/SKILL.md
```

## Naming Patterns

**Directories:**
- Skill directories: `kebab-case` matching the `name` frontmatter field exactly (e.g., `skills/generate-blog-ideas/`)

**Files:**
- Skill prompt files: always `SKILL.md` (uppercase)
- Workflow configs: `release.yml`, `ci.yml` (lowercase kebab-case)

**Package naming:**
- npm scope: `@cinatra-ai/` prefix
- Package name: `@cinatra-ai/blog-skills` — scoped, kebab-case

**Capability keys:**
- Dot-namespaced, domain-prefixed: `blog.generate-ideas`, `blog.generate-post-draft`, `blog.generate-linkedin-post` (defined in `package.json` under `cinatra.capabilities`)

## Package Manifest Conventions

**`package.json` structure:**
- `"type": "module"` — ESM
- `cinatra` block is required with: `apiVersion`, `kind`, `dependencies`, `capabilities`
- `cinatra.kind` must be `"skill"` for this package type
- `cinatra.capabilities` maps dot-namespaced capability keys to directory names under `skills/`
- First-party `@cinatra-ai/*` packages MUST NOT appear in `dependencies`, `devDependencies`, or `optionalDependencies` — only as optional `peerDependencies` with `peerDependenciesMeta[pkg].optional = true`

## TypeScript Config Conventions

`tsconfig.json` is present but targets a (currently empty) `src/` directory. Settings establish the baseline for any future TS additions:
- `"strict": true` with `"noImplicitAny": false`
- `"verbatimModuleSyntax": true`
- `"moduleResolution": "bundler"`, `"module": "ESNext"`, `"target": "ES2023"`
- `"isolatedModules": true`

## Prompt Authoring Style

**Voice:** Imperative ("Write a...", "Do not...", "Return..."), no hedging language.

**Anonymization rule:** Every skill prompt explicitly instructs the model not to surface transcript origins, speaker names, or third-party company names in output. This is a consistent cross-skill constraint enforced in prose.

**Output format guidance:** Included inline in the prompt body when output shape matters (e.g., `generate-blog-post-draft` specifies markdown headings, numbered lists, teaser excerpt before draft).

## CI/CD Conventions

**Dependency shape gate:** CI (`ci.yml`) enforces that no first-party `@cinatra-ai/*` packages appear in non-peer dependency fields. Violations exit with code 2 (hard failure).

**Content-only skip logic:** Because this repo has no TypeScript sources, the CI typecheck and test steps self-detect and skip gracefully using `git ls-files '*.ts'` checks.

**Pack validation:** `npm pack --dry-run` runs on every CI build to validate package shape and publish payload.

**Release:** Published via the reusable `cinatra-ai/.github` workflow triggered by a GitHub Release tag matching `v<package.json.version>`. Direct Verdaccio publish is explicitly prohibited.

## Error Handling

Not applicable — no executable code in this repository.

## Logging

Not applicable — no executable code in this repository.

## Comments

**In CI YAML:** Extensive inline comments explain skip logic, ordering rationale, and architectural decisions. This is the primary documentation medium for operational behavior.

**In `tsconfig.json`:** A `"//"` comment key documents the standalone config rationale.

---

*Convention analysis: 2026-06-09*
