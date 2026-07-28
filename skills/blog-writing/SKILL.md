---
name: blog-writing
description: Editorial guidance for the Cinatra blog pipeline. Routes to the prompt for the stage in hand — blog idea generation, long-form draft writing, or LinkedIn promotion of a published post — and carries the source-handling rules all three stages share.
metadata:
  match_when:
    - agent_id: "@cinatra-ai/wordpress-agent"
    - agent_id: "@cinatra-ai/drupal-agent"
---

You write blog content for a company: ideas from source material, a polished
draft from a chosen idea, and a short LinkedIn post promoting the published
result. Pick the stage the request is at, follow that stage's prompt, and apply
the shared source-handling rules below to all three.

## Pick the stage

- **Ideas** — the request is for blog post ideas for a company, usually from
  attached transcript files. Follow [Blog ideas](references/blog-ideas.md).
- **Draft** — an idea has been chosen and the request is for the full post.
  Follow [Blog post draft](references/blog-post-draft.md).
- **LinkedIn** — a blog post is published and the request is for a short
  promotional post with its URL. Follow [LinkedIn post](references/linkedin-post.md).

If the request spans stages (e.g. "come up with an idea and draft it"), run the
stages in order and apply each stage's prompt in turn.

## Source-handling rules for every stage

Attached transcripts and reference documents are **background source material
only**. Never mention the transcript, its origin, its speakers, or any person or
company name found in it. Extract the relevant thoughts and restate them
generically so they are useful for the target company — the company behind the
provided website URL, not the company the source material came from.
