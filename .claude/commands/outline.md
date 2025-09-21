---
description: Create/open the single outline file for the active draft; supports seeding and input normalization.
argument-hint: ["input"]
---

## Editorial intent

The outline is the blueprint for the final piece. Build an intelligent outline that fits the genre and audience implied by the project brief:

- Genre-aware: a magazine-style feature favors narrative flow, scenes, and transitions; an academic-style piece prefers structured sections; a quick brief can be tight bullets.
- Coherent flow: introduce context, develop evidence/analysis, surface counterpoints, and land conclusions/next steps.

## Implementation

1. Resolve context: `projects/{active-project}/drafts/{active-draft}/`
2. If `outline.md` is missing or empty:
   - Seed from `templates/outline.example.md` and `projects/{project}/project.brief.md` to create a starter outline.
3. If `input` is provided:
   - Parse and normalize (topic | bullets | full outline) into `outline.md`
   - Input must follow the conventions demonstrated in `templates/outline.example.md` (headings, markers, numbering)
4. Save edits to `outline.md`
5. Assign markers where missing:
   - Start with defaults by `mode` and write them explicitly, but **only for leaf nodes** (headings or list items with no sub-sections):
     - narrative: leaf H2/H3 → `[research]` unless `[skip]`
     - list: leaf top-level list items → `[research]` unless `[skip]`
     - hybrid: both rules apply
   - Never assign `[research]` to container/parent headings that have children; leave parents unmarked unless the user explicitly marks them.
   - Then infer section-specific markers from titles/content and the project brief (again, only for leaves):
     - `[light]` / `[deep]`: favor `[light]` for overview/context/logistics; use `[deep]` for key claims, analysis, or dense evidence blocks
     - `[media]`: add when sections likely reference posts/videos/images (e.g., X/Twitter, YouTube, screenshot, press conference)
     - `[skip]` (rare): only for obvious placeholders or non-content scaffolding (e.g., "TBD", "Notes", "Parking Lot")
   - Never overwrite explicit markers the user already provided; only add where absent.
