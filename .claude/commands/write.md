---
description: "Run the full pipeline: research → draft → review → compile for the active draft"
---

# Full Writing Pipeline

This command orchestrates the entire workflow for producing a completed draft.  
It coordinates four sub-agents — **Researcher**, **Writer**, **Reviewer**, and **Editor** — through a sequential pipeline.


## Algorithm

### Resolve context
- Read active project name from `.active-project`.
- Read active draft name from `projects/{active-project}/.active-draft`.
- Abort if either is missing.

**Path base**
- {project} = !`cat .active-project`
- {draft}   = !`cat projects/{project}/.active-draft`
- {base}    = projects/{project}/drafts/{draft}`

**Ensure required dirs exist**
- !`mkdir -p {base}/research {base}/sections {base}/synopses`

### Validate inputs
- Ensure `project.brief.md`, `draft.config.yml`, and `outline.md` exist and are non-empty.
- Ensure every outline node has `[research]` or `[skip]`.
- Abort if validation fails.

### Build work queue
- Parse `outline.md` into ordered tasks.
- Only nodes marked `[research]` are added to the queue.
- Assign IDs in memory (`S1, S2…` or `L1, L2…`).
- Process tasks in outline order.

### For each task N in the queue
- If `[media]` is present, run a **media hunt**:
  - The Researcher agent searches for relevant YouTube videos, X/Truth/Bluesky posts, or images.  
  - Save any findings to the `MEDIA` section of `research/N.md` (include canonical URLs, timestamps, or image links).  
  - Do not embed directly; the media postprocessor will handle rendering later.

**Researcher step**  
- **Purpose**: Gather 3–5 concrete, date-stamped, verifiable pieces of evidence with citations.  
- **Inputs**: project brief, draft config, the section’s micro-brief, markers, and any user questions.  
- **Outputs**: Output file MUST be `{base}/research/{ID}.md` (no other path). Containing sources, key findings, optional media links, and a short narrative summary.  
- **Notes**:  
  - `[light]` → 2–3 facts, short.  
  - `[deep]` → more sources, counterpoints, timeline.  
  - `[media]` → capture links or screenshots of posts/videos/images.

**Writer step**  
- **Purpose**: Expand the section into prose, using both evidence and rolling context for smooth local flow.  
- **Inputs**:  
  - Project brief  
  - Draft config  
  - Outline (to understand full structure and upcoming sections)  
  - Synopsis of prior sections (≤350 words)  
  - Last 2 finalized sections verbatim  
  - Current section’s micro-brief  
  - `research/N.md`  
- **Outputs**:  
  - Output files MUST be `{base}/sections/{ID}.md` and `{base}/synopses/{ID}.txt` (3–5 sentence summary of this section)  
- **Notes**: Each section writer should both connect back to what just came before *and* anticipate what’s coming next using the outline.

**Reviewer step**  
- **Purpose**: Fact-check, polish, and enforce consistency without overwriting.  
- **Inputs**: `sections/N.md`  
- **Outputs**: Edits applied in place.  
- **Notes**: Reviewer ensures clarity, accuracy, formatting, and tone match the project brief.

**Validate outputs**
- Research: !`test -f {base}/research/{ID}.md || (f=$(find . -maxdepth 3 -name "{ID}.md" -type f | head -n1); if [ -n "$f" ]; then mkdir -p {base}/research && mv "$f" "{base}/research/{ID}.md"; fi)`
- Section:  !`test -f {base}/sections/{ID}.md  || (f=$(find . -maxdepth 3 -name "{ID}.md" -type f | head -n1); if [ -n "$f" ]; then mkdir -p {base}/sections && mv "$f" "{base}/sections/{ID}.md"; fi)`
- Synopsis: !`test -f {base}/synopses/{ID}.txt || (f=$(find . -maxdepth 3 -name "{ID}.txt" -type f | head -n1); if [ -n "$f" ]; then mkdir -p {base}/synopses && mv "$f" "{base}/synopses/{ID}.txt"; fi)`

### After all tasks complete
**Editor-glue step**  
- **Purpose**: Perform a light global pass to ensure the entire draft reads smoothly as one coherent piece.  
- **Inputs**: project brief, draft config, the full outline, all synopses, and the first/last paragraphs of each section.  
- **Outputs**:  
  - `FINAL.md`  
  - `FINAL.html` if `[media]` nodes were used  
- **Notes**: The editor-glue agent does not rewrite sections. It smooths transitions across larger boundaries, normalizes headings, and harmonizes tone/voice across the whole piece.

