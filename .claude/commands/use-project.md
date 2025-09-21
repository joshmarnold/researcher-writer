---
description: Set the active project and align active draft within it
argument-hint: [project name]
---

Switch the active project and ensure a sensible active draft is selected.

1. Validate that `projects/{project}/` exists. If not, show available projects.
2. Write `{project}` to `.active-project`.
3. Determine the draft to activate:
   - If there is an existing active draft name recorded in the previous project (from `projects/{prevProject}/.active-draft`) and a draft with that same name exists in `projects/{project}/drafts/`, keep that draft name.
   - Otherwise, select the first draft in `projects/{project}/drafts/` (sorted lexicographically) if any exist.
4. If a draft was determined in step 3, write it to `projects/{project}/.active-draft`. If no drafts exist, omit `.active-draft` and report that the project has no drafts yet.

Notes

- This behavior avoids landing in a project with a non-existent draft reference.
- Users can always override by running `/use-draft <draft-name>` afterward.
