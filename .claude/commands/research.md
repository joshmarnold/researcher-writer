---
description: Research → write → review → glue for the active draft (confirm outline.md at start)
---

## Implementation

On start, show a preview/diff of `outline.md` and ask to confirm. Proceed only on confirmation.

### Algorithm (per workflow.orchestrator.md):

1. **Context Resolution**:

   - Read active project from `.active-project`
   - Read active draft from `projects/{project}/.active-draft`
   - Load `project.config.yml` and `project.brief.md`

2. **Build Work Queue**:

- Parse `outline.md` for headings/bullets, IDs (assigned in-memory), and markers
- Apply mode defaults (narrative=headings, list=bullets, hybrid=both)
- Build work queue in outline order (respect `[research]`/`[skip]` markers)

3. **For Each Research Unit**:

   - **Researcher Agent**: Use Task tool with researcher prompt from `agents/researcher.prompt.md`
     - Input: global brief, project config, section micro-brief, markers
     - Output: Save to `research/N.md`
   - **Writer Agent**: Use Task tool with section-writer prompt
     - Input: rolling packet (brief, synopsis ≤350 words, last 2 sections, research notes)
     - Output: Save to `sections/N.md` + extract synopsis to `synopses/N.txt`
   - **Reviewer Agent**: Use Task tool with reviewer prompt
     - Input: section content
     - Output: Edit `sections/N.md` in place

4. **Final Compilation**:
   - **Editor-Glue Agent**: Use Task tool with editor-glue prompt
     - Input: all synopses, first/last paragraphs of each section
     - Output: `FINAL.md`

### Rolling Context Rules:

- Synopsis: 3-5 sentences per section, ≤350 words total
- Context: Last 2 finalized sections verbatim
- Research notes: Full notes for current section only

This is the complete research-to-draft pipeline using Claude Code's sub-agent system.
