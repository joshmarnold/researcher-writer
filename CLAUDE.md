# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-agent research and writing workflow system. It produces full drafts through a structured pipeline:

**outline → research → write → review → compile**

The pipeline is orchestrated by `/write` (see `.claude/commands/write.md`), which coordinates specialized agents for each phase.

## Key Files

- **Orchestrator**: `.claude/commands/write.md` – full step-by-step algorithm and guardrails  
- **Agents**:  
  - Researcher → `.claude/agents/researcher.md`  
  - Writer → `.claude/agents/section-writer.md`  
  - Reviewer → `.claude/agents/content-reviewer.md`  
  - Editor → `.claude/agents/editor-glue-pass.md`

## Project Structure

Active context:  
- `.active-project` (repo root)  
- `.active-draft` (inside each project folder)  

Each project has:  

```
projects/{project}/
├── project.brief.md              # global research brief
├── .active-draft                 # current draft id (optional)
├── drafts/
│   └── {draft-id}/
│       ├── draft.config.yml
│       ├── outline.md
│       ├── research/
│       ├── sections/
│       ├── synopses/
│       ├── FINAL.md
│       └── FINAL.html
```

## Outline Markers

Each node in the outline must be explicitly marked:

- `[research]` → run Researcher + Writer + Reviewer  
- `[skip]` → bypass Researcher, still run Writer + Reviewer  
- `[light]` / `[deep]` → modify research depth  
- `[media]` → run media hunt (posts, videos, images)

## Notes

- `/write` is the single source of truth for orchestration.  
- Agent files define their own methodology.  
- This file is only an overview and map of where things live.