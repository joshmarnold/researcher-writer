# /use-draft

Switch the active draft for the (given or active) project.

## Usage
- Required `draft` parameter for the draft ID
- Optional `project` parameter (uses active project if not specified)

## Implementation
1. Determine target project (parameter or active from `.active-project`)
2. Validate that `projects/{project}/drafts/{draft}/` exists
3. If valid, write draft ID to `projects/{project}/.active-draft`
4. If invalid, show available drafts in the project