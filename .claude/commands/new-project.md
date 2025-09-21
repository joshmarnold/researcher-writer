---
description: Create projects/{project}/ with project brief; ensure drafts/ subfolder; set active
argument-hint:
  ["project-name", "brief-content", "audience", "voice", "citation"]
---

Create a new research project with the specified name and brief content.

**Usage Examples:**

**Basic usage (uses defaults from templates/research-brief.sample.md):**

```
/new-project

Create "Policy Analysis" project with this brief:

## Working Thesis
Analyzing recent policy impacts on healthcare reform and regulatory changes.
```

**Advanced usage with custom parameters:**

```
/new-project audience="Academic researchers" voice="Formal academic tone with detailed citations" citation="APA format with numbered references"

Create "Policy Analysis" project with this brief:

## Working Thesis
Analyzing recent policy impacts on healthcare reform and regulatory changes.
```

**Process:**

0. Validate inputs:

- If no brief content is provided (no Working Thesis or equivalent), STOP with a short message and make no changes.
- Project name is required. If not explicitly provided, attempt to infer a short, relevant name from the brief’s thesis (e.g., key nouns/phrases, ≤ 5 words). If a clear name cannot be inferred, STOP and ask the user to provide one. Do not use generic timestamped names.

1. Parse optional parameters from command arguments: `audience="..."`, `voice="..."`, `citation="..."`
2. Read default values from `templates/research-brief.sample.md`
3. Parse or derive the project name from the command content/brief (e.g., "Policy Analysis"); sanitize for folder
4. Sanitize project name for directory (e.g., "policy-analysis")
5. Create `projects/{sanitized-project-name}/` directory
6. Generate complete project brief:
   - Extract Working Thesis from user content
   - Use custom audience parameter OR default: "Policy-curious general audience; assumes basic civics literacy."
   - Use custom voice parameter OR default: "Clear, neutral, specific. Avoid loaded language; explain contested claims..."
   - Use custom citation parameter OR default: "Use inline bracketed citations like [Author/Source, Year] or numbered footnotes."
7. Write complete project brief → `projects/{sanitized-project-name}/project.brief.md`
8. Ensure `projects/{sanitized-project-name}/drafts/` subdirectory exists
9. Write the sanitized project name to `.active-project` file to set as active

**Arguments:**

- Required: Brief content (at minimum a Working Thesis line or paragraph)
- Project name: Required (explicit or inferable from your brief). If not inferable, the command halts and asks for a name.
- Optional parameters: `audience="..."`, `voice="..."`, `citation="..."`

The project becomes the active project for subsequent operations.
