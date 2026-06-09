<!-- refreshed: 2026-06-09 -->
# Architecture

**Analysis Date:** 2026-06-09

## System Overview

```text
┌─────────────────────────────────────────────────────────────┐
│               Cinatra Agent Platform (external)              │
│   @cinatra-ai/wordpress-agent  |  @cinatra-ai/drupal-agent  │
└──────────────────┬──────────────────┬───────────────────────┘
                   │ match_when       │ capability lookup
                   ▼                  ▼
┌─────────────────────────────────────────────────────────────┐
│                  blog-skills skill bundle                    │
│  Registered via `cinatra.capabilities` in `package.json`    │
├──────────────────┬──────────────────┬───────────────────────┤
│  generate-       │  generate-blog-  │  generate-linkedin-   │
│  blog-ideas      │  post-draft      │  post                 │
│  `skills/        │  `skills/        │  `skills/             │
│  generate-blog-  │  generate-blog-  │  generate-linkedin-   │
│  ideas/SKILL.md` │  post-draft/     │  post/SKILL.md`       │
│                  │  SKILL.md`       │                       │
└──────────────────┴──────────────────┴───────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│     LLM inference (handled by the Cinatra agent platform)   │
└─────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| Skill manifest | Declares npm package identity, Cinatra kind, and capability-to-skill directory mapping | `package.json` |
| generate-blog-ideas | System prompt for brainstorming blog ideas from attached transcripts | `skills/generate-blog-ideas/SKILL.md` |
| generate-blog-post-draft | System prompt for producing a full blog post draft with teaser excerpt | `skills/generate-blog-post-draft/SKILL.md` |
| generate-linkedin-post | System prompt for producing a LinkedIn promotional post for a published blog | `skills/generate-linkedin-post/SKILL.md` |
| CI pipeline | Validates package shape, first-party dependency hygiene, typecheck, and npm pack dry-run | `.github/workflows/ci.yml` |
| Release pipeline | Handles versioned releases of the npm package | `.github/workflows/release.yml` |

## Pattern Overview

**Overall:** Content-only skill bundle — a Cinatra extension of `kind: skill`. There is no runtime code; every skill is a plain-text system prompt stored in a `SKILL.md` file with a YAML front-matter header.

**Key Characteristics:**
- Zero TypeScript/JavaScript source files; `tsconfig.json` is a boilerplate placeholder that targets a non-existent `src/` directory
- Skills are resolved by the host platform via capability keys declared in `package.json` under `cinatra.capabilities`
- Agents opt in to skills via `match_when` YAML front matter (e.g., `agent_id: "@cinatra-ai/wordpress-agent"`)
- No runtime dependencies; no `node_modules`; no lockfile committed

## Layers

**Skill Definitions (content layer):**
- Purpose: Provide LLM system prompts that govern agent editorial behavior
- Location: `skills/*/SKILL.md`
- Contains: YAML front-matter (name, description, match_when) + plain-text system prompt body
- Depends on: Nothing — interpreted by the Cinatra agent platform at runtime
- Used by: `@cinatra-ai/wordpress-agent`, `@cinatra-ai/drupal-agent` (and any agent matching declared `match_when` predicates)

**Package Manifest (registry layer):**
- Purpose: Register the bundle as an npm-distributable Cinatra skill extension
- Location: `package.json`
- Contains: npm identity, `cinatra` metadata block with `apiVersion`, `kind`, `dependencies`, and `capabilities` map
- Depends on: Nothing (zero runtime deps)
- Used by: Cinatra monorepo workspace and npm registry consumers

**CI/CD (validation layer):**
- Purpose: Enforce dependency hygiene, package shape, and kind-specific gate checks
- Location: `.github/workflows/ci.yml`, `.github/workflows/release.yml`
- Contains: Node 24 + corepack pnpm pipeline; first-party dep shape check; npm pack dry-run; kind-gate placeholder
- Depends on: GitHub Actions, Node.js 24

## Data Flow

### Skill Resolution Path

1. Host agent platform receives a task matching a registered capability (e.g., `blog.generate-ideas`)
2. Platform resolves the capability key via `cinatra.capabilities` in `package.json` to the directory name (e.g., `generate-blog-ideas`)
3. Platform loads `skills/generate-blog-ideas/SKILL.md`, reads the YAML front-matter and prompt body
4. Platform injects the prompt body as the LLM system prompt, combined with caller-provided transcript attachments and target URL
5. LLM produces blog idea output; platform returns it to the calling agent

### Editorial Pipeline (sequential usage)

1. Agent calls `blog.generate-ideas` with transcript files → `skills/generate-blog-ideas/SKILL.md`
2. User selects an idea; agent calls `blog.generate-post-draft` with transcript + idea → `skills/generate-blog-post-draft/SKILL.md`
3. Agent publishes the post; then calls `blog.generate-linkedin-post` with post URL → `skills/generate-linkedin-post/SKILL.md`

**State Management:**
- No in-process state. Each skill invocation is stateless; context is passed by the caller at invocation time.

## Key Abstractions

**SKILL.md file:**
- Purpose: Self-contained unit of LLM behavioral guidance; combines routing metadata (YAML front-matter) with prompt content
- Examples: `skills/generate-blog-ideas/SKILL.md`, `skills/generate-blog-post-draft/SKILL.md`, `skills/generate-linkedin-post/SKILL.md`
- Pattern: Each file is `---\n<yaml front-matter>\n---\n<plain-text prompt body>`

**`cinatra.capabilities` map:**
- Purpose: Declares which capability keys this package handles and maps them to skill directories
- Examples: `"blog.generate-ideas": "generate-blog-ideas"` in `package.json`
- Pattern: dot-namespaced capability key → subdirectory name under `skills/`

## Entry Points

**npm package entry:**
- Location: `package.json` (`cinatra.capabilities`)
- Triggers: Cinatra platform skill resolution at agent runtime
- Responsibilities: Map capability keys to skill subdirectories

**Skill invocation:**
- Location: `skills/<skill-name>/SKILL.md`
- Triggers: Agent platform capability lookup
- Responsibilities: Deliver system prompt to the LLM for the requested capability

## Architectural Constraints

- **No source code:** `tsconfig.json` references `src/` which does not exist; the repo is purely content-only
- **Global state:** None
- **Circular imports:** Not applicable (no code)
- **Dependency model:** This is a "source mirror" repo — host-internal `@cinatra-ai/*` packages are optional peer dependencies resolved only by the Cinatra monorepo; standalone install/typecheck/test are skipped in CI when such peers are declared

## Anti-Patterns

### Adding TypeScript source without updating CI skip logic

**What happens:** Adding a `src/` directory triggers tsc in CI but the `@cinatra-ai/*` peer deps cannot resolve standalone.
**Why it's wrong:** CI will fail on typecheck because host-internal peers are not on any registry.
**Do this instead:** Keep all skill content in `skills/*/SKILL.md`; if code is needed, it belongs in the Cinatra monorepo package that consumes these skills.

### Listing `@cinatra-ai/*` in `dependencies` or `devDependencies`

**What happens:** CI's first-party dep shape check exits with code 2 and fails the build.
**Why it's wrong:** Host-internal packages must be `optionalPeerDependencies` so the monorepo provides them without leaking into standalone installs.
**Do this instead:** Declare host-internal packages only under `peerDependencies` with `peerDependenciesMeta.<pkg>.optional: true`.

## Error Handling

**Strategy:** Not applicable — no runtime code.

**Patterns:**
- CI validates package shape at build time; misconfigured `package.json` causes CI to fail with an explicit error message

## Cross-Cutting Concerns

**Logging:** Not applicable — content-only package
**Validation:** CI first-party dep shape check (`ci.yml` lines 48-69)
**Authentication:** Not applicable

---

*Architecture analysis: 2026-06-09*
