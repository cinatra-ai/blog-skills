# Cinatra Blog Writing Skill

Editorial guidance for the Cinatra blog agents. This bundle is one router covering blog idea brainstorming, full long-form drafting, and short LinkedIn promotion — keeping voice, structure, and source-handling rules consistent so generated posts stay on-brand for the target company and never leak the underlying source material.

Install this extension from the Cinatra marketplace and attach it to a WordPress or Drupal blog agent. The agent receives a transcript file and a target website URL; the router guides it to extract relevant themes, draft a polished post that matches the site's editorial style, and produce a LinkedIn teaser with a canonical link. No transcript origin, speaker name, or third-party company name surfaces in the output.

Configuration is minimal: no credentials are required by the skill itself. The host agent must already be connected to your CMS (WordPress or Drupal) through its own connector. To develop or extend it, edit the router at skills/blog-writing/SKILL.md or the per-stage prompt it points at under skills/blog-writing/references/, re-run the extension kind gate (node extension-kind-gate.mjs --package-root .), and open a pull request. If generated content repeats a transcript speaker's name, verify the agent is using the latest version of this bundle and that no older skill cache is active.

## Works with

- Cinatra WordPress blog agent
- Cinatra Drupal blog agent
- LinkedIn

## Capabilities

- Brainstorm blog post ideas tailored to a target company from attached transcripts
- Match the tone and structure of the company's existing published posts
- Produce a polished long-form draft with a concise teaser and clean markdown headings
- Keep generated content anonymized — no transcript origins, speakers, or third-party names surface in the output
- Write a short, hook-led LinkedIn post to promote a published blog
- Hold a consistent editorial format across ideation, drafting, and promotion
