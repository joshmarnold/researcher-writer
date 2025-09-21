# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-agent research workflow system designed for creating well-researched, fact-checked content with a systematic approach: outline → research → write → review → compile. The workflow uses separate agents for each phase to maintain context isolation and quality control.

## Key Workflow Commands

Since Claude Code doesn't support VS Code slash commands, execute these workflows manually:

### Project and Draft Management

- **Create new project**: Create `projects/{project}/` with `project.config.yml` and `project.brief.md`
- **Create new draft**: Create a draft folder under `projects/{project}/drafts/{draft-id}/` with `draft.config.yml` (e.g., `mode: narrative|list|hybrid`)
- **Set active context**: Track via `.active-project` and `.active-draft` files

### Research Pipeline Execution

On start of a run, show a preview/diff of `outline.md` and ask to confirm before proceeding.

Use Claude Code's Task tool to spawn sub-agents:

1. **Researcher agent**: Reads from approved outline, writes to `research/N.md`
2. **Writer agent**: Uses research + rolling context, writes to `sections/N.md`
3. **Reviewer agent**: Edits sections in place
4. **Editor-glue agent**: Compiles final `FINAL.md`

## Project Structure

```
projects/{project}/
├── project.config.yml          # optional org-wide defaults (media, sources)
├── project.brief.md            # global research brief
├── .active-draft              # tracks current draft
└── drafts/{draft-id}/
   ├── draft.config.yml         # draft-level settings (e.g., mode: narrative|list|hybrid)
   ├── outline.md              # single outline file
    ├── research/               # N.md files per section
    ├── sections/               # N.md draft sections
    ├── synopses/               # N.txt section summaries
    └── FINAL.md                # final compiled output
```

## Outline Markers System

Markers control research behavior per section/item:

- `[research]` - force research unit
- `[skip]` - exclude from research
- `[light]` / `[deep]` - research depth
- `[media]` - run a media pass (opt-in media hunt)

Default behavior by mode:

- **narrative**: H2/H3 sections default to `[research]`
- **list**: list items default to `[research]`
- **hybrid**: both rules apply

When normalizing the outline, also infer markers where missing while preserving any explicit ones:

- Depth: add `[light]` for overviews/context/logistics; `[deep]` for key claims or dense evidence
- Media: add `[media]` for sections likely to include posts/videos/images
- Skip: add `[skip]` only for non-content placeholders (e.g., "TBD", "Notes")

## Agent Prompts Location

- Researcher: `agents/researcher.md`
- Writer: `agents/section-writer.md`
- Reviewer: `agents/reviewer.md`
- Editor: `agents/editor-glue.md`
- Orchestrator logic: See "Research Pipeline Algorithm" below

## Using with Claude Code's Task Tool

To run the research pipeline with proper agent isolation:

```python
# Example Task tool invocation pattern
Task(
    subagent_type="general-purpose",
    description="Research section X",
    prompt=f"You are the Researcher. {researcher_prompt} Input: {outline_item}"
)
```

Each agent should be spawned with:

1. Its specific prompt from `agents/`
2. Required inputs (brief, outline, prior sections)
3. Clear output expectations (format, location)

## Critical Workflow Rules

1. **Outline Confirm**: At run start, confirm `outline.md` contents
2. **Rolling Context**: Writer receives max 350 words synopsis + last 2 sections
3. **Section IDs**: S1, S2... for narrative; L1, L2... for lists
4. **File Outputs**: Each phase writes to designated folders
5. **Model Usage**: Can adapt models in `agentpack.json` but maintain agent separation
6. **No-op on missing required inputs**: If a command lacks required content, stop with a short message and make no changes. For `/new-project`, require a brief body (e.g., a Working Thesis) and a meaningful project name. If a name is not explicitly provided, infer a short relevant name from the brief; if no clear name can be inferred, stop and ask the user for one. Do not use generic timestamped names.

## Research Pipeline Algorithm

### Inputs

- Active project: read from `.active-project`
- Active draft: read from `projects/{project}/.active-draft`
- Project config: `projects/{project}/project.config.yml` (optional global defaults: media/sources)
- Draft config: `projects/{project}/drafts/{draft}/draft.config.yml` (mode and draft-level settings)
- Global brief: `projects/{project}/project.brief.md`
- Outline file (per draft): `projects/{project}/drafts/{draft}/outline.md`

### Algorithm

0. **Resolve context**:

   - Read active project from `.active-project`
   - Read active draft from `projects/{project}/.active-draft`
   - If either is missing, ABORT and instruct user to run `/new-project` and `/new-draft`

1. **Read configs and brief**:

   - Load `draft.config.yml` for draft settings (mode, media, list defaults, sources)
   - If present, load `project.config.yml` for optional global defaults
   - Load `project.brief.md` for voice and sourcing rules

2. **Outline confirm**:

   - Show `outline.md` preview/diff; proceed only on user confirmation

3. **Build marker-aware work queue from outline.md**:

   - Marker syntax (inline, appended to headings/bullets): `[research]`, `[skip]`, `[light|deep]`, `[media]`
   - Defaults by mode:
     - narrative: every top-level section (H2/H3) is `[research]` unless `[skip]`
     - list: every list item is `[research]` unless `[skip]`
     - hybrid: apply both rules where applicable
   - Inheritance: parent markers apply to children unless overridden
   - Assign IDs in-memory during queue build:
     - Narrative sections: `S1`, `S2`, … (subsections `S1.1`, `S1.2`, …)
     - List items: `L1`, `L2`, …
   - Build tasks: include nodes marked (or defaulted) as `[research]` minus `[skip]`; process in outline order

4. **For each task N in the queue**:

   - Only run a media hunt when a node is marked `[media]`
   - **Call researcher** with:
     - Global brief (full)
     - Project config (mode/media defaults)
       - Section-N micro-brief (from outline) and markers (`light|deep`, `media`)
     - Any specific questions
   - Save notes to `projects/{project}/drafts/{draft}/research/N.md`
   - **Build rolling packet for writing**:
     - Global brief (full)
     - Project config (mode/media defaults)
     - Running synopsis of prior sections (≤ 350 words total; bullets OK)
     - The last 2 finalized sections (verbatim)
     - Section-N micro-brief (from outline)
     - Research notes from `research/N.md`
   - **Call section-writer** with packet → save to `sections/N.md`
   - **Extract 3–5 sentence section synopsis** → save to `synopses/N.txt`
   - **Call reviewer** on `sections/N.md` and apply fixes in place

5. **After all sections complete, call editor-glue** with:
   - Global brief
   - Project config
   - All section synopses
   - First and last paragraph of each section
   - Write compiled result to `FINAL.md`

### Guardrails

- Preserve specifics, dates, and citations; prefer primary/reputable sources
- Explicit dates (e.g., "March 12, 2025")
- Token budgets: synopsis ≤ 350 words; only last 2 sections verbatim in packet
- Glue pass adjusts transitions/headings/voice; avoid full rewrites

### Files Produced

- Research notes: `projects/{project}/drafts/{draft}/research/N.md`
- Draft sections: `projects/{project}/drafts/{draft}/sections/N.md`
- Synopses: `projects/{project}/drafts/{draft}/synopses/N.txt`
- Compiled draft: `projects/{project}/drafts/{draft}/FINAL.md`

### Mode Influence

- **narrative**: standard prose with connective tissue
- **list**: tighter bullets with brief transitions
- **hybrid**: prose with occasional bullet blocks for dense facts
