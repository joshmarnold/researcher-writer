---
description: Create a new draft under the (given or active) project and set it active
argument-hint: ["project", "draft-name", "mode"]
---

Create a new draft under the (given or active) project and set it active.

**Usage:**

- Optional `project` parameter (uses active project if not specified)
- Optional `draft-name` parameter for custom draft ID (auto-generates if not specified)
- Optional `mode` parameter to set draft mode: `narrative`, `list`, or `hybrid` (defaults to `hybrid` if omitted)

**Process:**

1. Determine target project (parameter or active from `.active-project`)
2. Generate draft ID if not provided:
   - Check existing drafts in `projects/{project}/drafts/`
   - Suggest a new name based on your convention (e.g., `draft-one`, `draft-two`, or a date-based ID like `2025-03-14`)
   - If using ordinal names, pick the next available; if date-based, use today's date (append `-b` if a duplicate exists)
3. Create `projects/{project}/drafts/{draft-id}/` with subdirectories:
   - `research/`
   - `sections/`
   - `synopses/`
4. Create `draft.config.yml` in the draft folder with at least:
   - `mode: <narrative|list|hybrid>` (from parameter or default `hybrid`)
5. Seed `outline.md` (empty)
6. Create empty `FINAL.md`
7. Write draft ID to `projects/{project}/.active-draft`
