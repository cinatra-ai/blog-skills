# Codebase Structure

**Analysis Date:** 2026-06-09

## Directory Layout

```
blog-skills/
├── skills/                          # All skill definitions (one subdirectory per skill)
│   ├── generate-blog-ideas/
│   │   └── SKILL.md                 # Blog-idea brainstorming system prompt
│   ├── generate-blog-post-draft/
│   │   └── SKILL.md                 # Full blog post draft system prompt
│   └── generate-linkedin-post/
│       └── SKILL.md                 # LinkedIn promotional post system prompt
├── .github/
│   └── workflows/
│       ├── ci.yml                   # PR/push validation pipeline
│       └── release.yml              # npm release pipeline
├── .planning/
│   └── codebase/                    # GSD codebase analysis documents
├── package.json                     # npm manifest + Cinatra skill registration
├── tsconfig.json                    # Boilerplate TS config (no src/ exists today)
├── .npmrc                           # npm registry configuration
├── LICENSE                          # Apache-2.0
└── README.md                        # Human-readable bundle overview
```

## Directory Purposes

**`skills/`:**
- Purpose: Houses every skill this package exposes; each immediate subdirectory is one skill
- Contains: One `SKILL.md` per skill
- Key files: `skills/generate-blog-ideas/SKILL.md`, `skills/generate-blog-post-draft/SKILL.md`, `skills/generate-linkedin-post/SKILL.md`

**`.github/workflows/`:**
- Purpose: CI/CD automation for validation and publishing
- Contains: `ci.yml` (build gate), `release.yml` (publish gate)

**`.planning/codebase/`:**
- Purpose: GSD planning documents written by codebase-mapper agents
- Generated: Yes (by tooling)
- Committed: Yes

## Key File Locations

**Entry Points:**
- `package.json`: Cinatra skill registration — `cinatra.capabilities` maps capability keys to skill subdirectories

**Skill Prompts:**
- `skills/generate-blog-ideas/SKILL.md`: System prompt for idea brainstorming from transcripts
- `skills/generate-blog-post-draft/SKILL.md`: System prompt for full blog post drafting
- `skills/generate-linkedin-post/SKILL.md`: System prompt for LinkedIn post generation

**Configuration:**
- `tsconfig.json`: TypeScript compiler config (boilerplate; no `src/` exists)
- `.npmrc`: npm registry settings
- `.github/workflows/ci.yml`: Build and validation pipeline

**Documentation:**
- `README.md`: Human overview of the bundle's capabilities and compatible agents

## Naming Conventions

**Files:**
- Skill prompt files are always named `SKILL.md` (uppercase, no variation)
- Workflow files use lowercase kebab-case: `ci.yml`, `release.yml`

**Directories:**
- Skill directories use lowercase kebab-case matching the `name` field in `SKILL.md` front-matter: `generate-blog-ideas`, `generate-blog-post-draft`, `generate-linkedin-post`
- Capability keys in `package.json` use dot-namespaced lowercase: `blog.generate-ideas`, `blog.generate-post-draft`, `blog.generate-linkedin-post`

## Where to Add New Code

**New Skill:**
- Create a new subdirectory under `skills/` using lowercase kebab-case: `skills/<new-skill-name>/`
- Add `SKILL.md` with YAML front-matter (`name`, `description`, `match_when`) and a plain-text system prompt body
- Register the new capability in `package.json` under `cinatra.capabilities`: `"blog.<capability-key>": "<new-skill-name>"`

**Updated Prompt:**
- Edit the relevant `skills/<skill-name>/SKILL.md` prompt body directly; no code changes required

**New Agent Targeting:**
- Add a `match_when` entry to the relevant `SKILL.md` front-matter specifying the new `agent_id`

**TypeScript Source (if ever needed):**
- Place under `src/` (referenced by `tsconfig.json`)
- Note: Adding source files changes the repo from a "source mirror" to a "standalone" repo in CI terms — review `.github/workflows/ci.yml` first-party dep logic before doing so

## Special Directories

**`skills/`:**
- Purpose: Skill prompt content consumed by the Cinatra agent platform
- Generated: No (hand-authored)
- Committed: Yes

**`dist/`:**
- Purpose: TypeScript compile output (referenced by `tsconfig.json` `outDir`)
- Generated: Yes (by tsc if TypeScript sources are ever added)
- Committed: No (not present today)

**`.planning/`:**
- Purpose: Tooling-generated planning and analysis documents
- Generated: Yes
- Committed: Yes

---

*Structure analysis: 2026-06-09*
