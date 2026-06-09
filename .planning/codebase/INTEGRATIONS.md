# External Integrations

**Analysis Date:** 2026-06-09

## APIs & External Services

**Cinatra Agent Platform:**
- Cinatra WordPress Agent (`@cinatra-ai/wordpress-agent`) — skills `generate-blog-ideas` and `generate-blog-post-draft` are matched to this agent via `match_when` in their `SKILL.md` front-matter
- Cinatra Drupal Agent (`@cinatra-ai/drupal-agent`) — same skills also matched to this agent
- LinkedIn — `generate-linkedin-post` skill targets LinkedIn publishing (no `match_when` agent restriction; platform-agnostic)

**Cinatra Marketplace:**
- Registry: `registry.cinatra.ai`
- Submission mechanism: marketplace MCP proxy (`extension-submit-for-review` → approve → promotion saga)
- Publish flow defined in `.github/workflows/release.yml` via reusable workflow `cinatra-ai/.github/.github/workflows/reusable-extension-release.yml@main`
- Auth: `CINATRA_MARKETPLACE_VENDOR_TOKEN` org secret (inherited by release workflow)

## Data Storage

**Databases:**
- Not applicable — content-only skill extension; no database access

**File Storage:**
- Not applicable

**Caching:**
- Not applicable

## Authentication & Identity

**Auth Provider:**
- GitHub OIDC — release workflow requests `id-token: write` for build-provenance attestation
- `CINATRA_MARKETPLACE_VENDOR_TOKEN` — org-level GitHub Actions secret used during marketplace submission

## Monitoring & Observability

**Error Tracking:**
- Not detected

**Logs:**
- CI step output only (GitHub Actions console logs)

## CI/CD & Deployment

**Hosting:**
- Cinatra Marketplace (`registry.cinatra.ai`)

**CI Pipeline:**
- GitHub Actions
  - `.github/workflows/ci.yml` — runs on push/PR to `main`; jobs: `build` (checkout, Node 24, corepack, dep-shape validation, install, typecheck, test, pack dry-run) and `kind-gates` (no extra gate for `skill` kind)
  - `.github/workflows/release.yml` — triggered on published GitHub Release or manual `workflow_dispatch` against a tag; delegates entirely to `cinatra-ai/.github` reusable release workflow

## Environment Configuration

**Required env vars:**
- None at runtime (content-only extension)
- `CINATRA_MARKETPLACE_VENDOR_TOKEN` — required in GitHub org secrets for release publishing

**Secrets location:**
- GitHub org-level secrets (not stored in repo)

## Webhooks & Callbacks

**Incoming:**
- Not applicable

**Outgoing:**
- Not applicable — skill prompts are consumed by Cinatra agents at inference time; no outgoing HTTP calls originate from this package

---

*Integration audit: 2026-06-09*
