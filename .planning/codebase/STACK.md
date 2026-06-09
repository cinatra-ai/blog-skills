# Technology Stack

**Analysis Date:** 2026-06-09

## Languages

**Primary:**
- YAML/Markdown - Skill definitions in `skills/*/SKILL.md` (YAML front-matter + plain-text prompt bodies)

**Secondary:**
- TypeScript (ES2023, ESNext modules) - Declared in `tsconfig.json` as the intended compilation target; no `.ts` source files are currently tracked (content-only skill extension)

## Runtime

**Environment:**
- Node.js 24 (specified in `.github/workflows/ci.yml` via `actions/setup-node@v4` with `node-version: "24"`)

**Package Manager:**
- pnpm (via corepack) — `corepack enable` + `corepack pnpm install` used in CI
- Lockfile: not committed (CI runs `--no-frozen-lockfile`)

## Frameworks

**Core:**
- Cinatra Skills SDK (custom) - `cinatra.apiVersion: cinatra.ai/v1`, `cinatra.kind: skill` declared in `package.json`. Skills are matched to agents via `match_when` in `SKILL.md` front-matter.

**Testing:**
- Not applicable — no test framework configured; this is a content-only skill extension with no TypeScript sources

**Build/Dev:**
- TypeScript compiler (`tsc`) — configured via `tsconfig.json` targeting `outDir: dist`, `rootDir: src`; CI invokes it but skips when no `.ts` files are tracked

## Key Dependencies

**Critical:**
- No runtime dependencies declared in `package.json` (`dependencies` field absent)
- No devDependencies declared
- No peerDependencies declared (this is a standalone content-only extension with zero first-party `@cinatra-ai/*` deps)

**Infrastructure:**
- GitHub Actions — CI/CD via `.github/workflows/ci.yml` and `.github/workflows/release.yml`
- Cinatra Marketplace — publishing target via `cinatra-ai/.github` reusable workflow (`reusable-extension-release.yml`)

## Configuration

**Environment:**
- No `.env` files detected
- No runtime environment variables required (content-only extension)

**Build:**
- `tsconfig.json` — standalone strict TypeScript config; targets `src/` → `dist/`; `strict: true`, `noImplicitAny: false`, `isolatedModules: true`, `verbatimModuleSyntax: true`
- `.npmrc` — present (existence noted; contents not read)

## Platform Requirements

**Development:**
- Node.js 24+
- pnpm (via corepack)

**Production:**
- Deployed to `registry.cinatra.ai` via the Cinatra Marketplace MCP proxy
- Release triggered by a published GitHub Release whose tag matches `v<package.json.version>`

---

*Stack analysis: 2026-06-09*
