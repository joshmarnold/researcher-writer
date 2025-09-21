# Research Writer

_Give it an outline, get back a researched article._

---

**Research Writer** is a multi-project workflow for producing complete articles as well as evidence-based list reports. It coordinates specialized agents—**researcher, writer, reviewer, and editor**—through a simple pipeline:

**Outline → Research → Write → Review → Compile**

---

### What It Does

Research Writer is designed to be **generic and flexible**. Give it an outline (or let it propose one), and it will:

- **Research**: Gather up-to-date, cited evidence from authoritative sources
- **Write**: Expand each outline item into a polished section or evidence card
- **Review**: Fact-check, tighten structure, and ensure tone consistency
- **Compile**: Assemble everything into a clean draft (Markdown, or HTML with media)

### What You Can Create

- **Narrative articles**: Multi-section essays, op-eds, or explainers written in any style or voice
- **Evidence lists**: Catalogs of examples, timelines, or grouped case studies (e.g. “10 cases of crony capitalism this year”)
- **Hybrid pieces**: Articles that include both flowing narrative and structured lists of evidence

### Flexible Options

- **Any subject, any audience**: Adaptable to your topic, style, and reader
- **Light or deep research**: Quick fact collection or comprehensive sourcing with multiple viewpoints
- **Media-aware**: Automatically embeds YouTube videos, screenshots of posts (X, Bluesky, Truth Social), or images when relevant
- **Multi-project setup**: Keep different topics organized, with multiple drafts and outlines tracked

## How It Works (High-level)

1. **Create Project** → Set up research brief and project structure — see [/new-project](#create-a-project)
2. **Create Draft** → Create a draft folder (e.g., draft-one, draft-two, or date-based) — see [/new-draft](#new-draft) and [Draft Config](#draft-config-draftconfigyml)
3. **Create Outline** → Draft an outline for your piece (headings or bullets) — see [Outline](#outline)
4. **Run Write** — see [Write](#write) → research → write → review → compile (confirm outline.md at start)

## Slash Commands at a Glance

- [`/new-project`](#create-a-project) `<project-name>` `<brief-content>` `[audience]` `[voice]` `[citation]`
- [`/new-draft`](#draft-config-draftconfigyml) `[project]` `[draft-name]` `[mode]`
- `/use-project` `<project-name>` — sets active project (`.active-project`) and aligns active draft (keeps same draft name if it exists in the project, else selects the first draft)
- `/use-draft` `<draft-name>` — sets active draft (`projects/{project}/.active-draft`)
- [`/outline`](#outline) `[input]`
- [`/write`](#write)

## Example Workflow

```bash
# Create project
/new-project
Create "Policy Analysis" project with brief: Analyzing regulatory impacts...

# Create first draft (adds under drafts/; rerun to add more — picks next available ID)
/new-draft

# Run outline — auto-generates if missing; normalizes input (uses conventions in templates/outline.example.md)
/outline

# Run write pipeline
/write
```

## Create a Project

Use `/new-project` to create a new project folder with a research brief and drafts directory, and set it as active.

- What it does:

  - Validates inputs: if no brief content is provided (at least a Working Thesis line/paragraph), it stops and makes no changes; a project name is required (either provided explicitly or inferable from the thesis).
  - Parses optional parameters from the command: `audience="..."`, `voice="..."`, `citation="..."`
  - Reads defaults from `templates/project.brief.example.md`
  - Extracts or infers a concise, relevant project name from your message/brief (e.g., "Policy Analysis") and sanitizes it for a folder name
  - Creates `projects/{project}/`
  - Writes a complete brief to `projects/{project}/project.brief.md` using your provided thesis plus defaults/overrides
  - Ensures `projects/{project}/drafts/` exists
  - Sets the project as active by writing its name to `.active-project`

- Basic usage:

  ```
  /new-project

  Create "Policy Analysis" project with this brief:

  ## Working Thesis
  Analyzing recent policy impacts on healthcare reform and regulatory changes.
  ```

- With parameters:

  ```
  /new-project audience="Academic researchers" voice="Formal academic tone with detailed citations" citation="APA format with numbered references"

  Create "Policy Analysis" project with this brief:

  ## Working Thesis
  Analyzing recent policy impacts on healthcare reform and regulatory changes.
  ```

- Natural language (no key=value):

  ```
  /new-project

  Create "Policy Analysis" project. The audience is academic researchers, use a formal academic voice, and please cite sources in APA style.

  ## Working Thesis
  Analyzing recent policy impacts on healthcare reform and regulatory changes.
  ```

- Files created:

  - `projects/{project}/project.brief.md` — your brief plus defaults (audience, voice, citation)
  - `projects/{project}/drafts/` — empty drafts folder ready for `/new-draft`
  - `.active-project` — sets this as the active project

## Draft Config (`draft.config.yml`)

When you create a new draft, you’ll see a `draft.config.yml` file in that draft folder (`projects/{project}/drafts/{draft}/draft.config.yml`). This file controls the draft’s defaults and research behavior.

Think of it as the draft’s “house rules”: how structured the writing should be, how much to chase media, and what kinds of sources to lean on. You can keep it simple at first and refine as you go—changes affect future runs of the pipeline.

Quick tips:

- Writing a list-style piece? Set `mode: list` and write a one-sentence `item_definition` describing what counts as an item.
- Prefer timelines? Use `list_defaults.grouping: chronology` and add a `date_window` (e.g., “last 12 months”).
- Want fewer images/clips? Keep media opt-in via `[media]` and leave `media_aggressiveness: low` or `normal`.
- Tighten sourcing fast: add trusted domains under `sources.prefer` and noisy ones under `sources.avoid`.
- Not sure yet? Leave most fields as-is—the defaults are sensible and you can tweak later.

- mode: narrative | list | hybrid — pick the shape of the piece (default: `hybrid`)
  - narrative: A traditional article with sections/paragraphs; treats headings as primary units
  - list: A list of items (e.g., cases or examples) with detailed write-ups for each
  - hybrid: A mix of narrative sections and structured lists when appropriate
  - technical defaults: narrative → H2/H3 default to `[research]`; list → top-level bullets default to `[research]`; hybrid → both apply
- media_aggressiveness: low | normal | high — how aggressively to look for media when a node has `[media]` (default: `normal`)
  - Current media scope: images, and URLs from X.com and YouTube
- list_defaults: applies when writing evidence lists or in list/hybrid mode
  - item_definition: a one-sentence rule describing what qualifies as an “item” and what fields it should include
    - Examples:
      - "Each item is a dated event that includes who/what/when/where and 1–2 citations."
      - "Each item is a case summary with three bullets: what happened, why it matters, and a source link."
  - inclusion / exclusion: optional keyword filters to bias candidate selection
  - max_items: cap on number of items in a list (default: 25)
  - grouping: none | theme | chronology — how to organize items (default: theme)
  - date_window: optional time window (e.g., "last 12 months" or ISO range)
- sources:
  - prefer: prioritized source classes or specific publishers/domains (e.g., `official`, `reputable_journalism`, `academic`, `nytimes.com`, `reuters.com`)
  - avoid: sources to de-prioritize or skip (use classes or domains, e.g., `example-clickbait.com`)

Example:

```yaml
mode: list
media_aggressiveness: normal
list_defaults:
  item_definition: "Each item is a dated event that includes who/what/when/where and 1–2 citations."
  inclusion: ["ethics", "conflict of interest"]
  exclusion: ["press release", "opinion"]
  max_items: 25
  grouping: chronology
  date_window: "last 5 years"
sources:
  prefer:
    [official, reputable_journalism, academic, "nytimes.com", "reuters.com"]
  avoid: ["example-clickbait.com"]
```

You can edit `draft.config.yml` at any time; changes apply to subsequent steps in the pipeline.

## Outline

The outline is a single file per draft: `projects/{project}/drafts/{draft}/outline.md`. It’s the source of truth for the draft’s structure.

- Create/open: Run `/outline`. If `outline.md` doesn’t exist, it seeds from `templates/outline.example.md` and your `project.brief.md`.
- Inputs accepted:
  - A short topic line to seed a first-pass outline
  - A bullet list of sections
  - A full pasted outline
- What it does:
  - Normalizes input (heading levels, list markers, marker syntax) and preserves your hierarchy/order
  - Assigns/updates consistent numbering where used (e.g., S1/S2, L1/L2)
  - De-duplicates obviously repeated sections
  - Assigns markers where missing:
    - Writes default `[research]` markers based on `mode` (narrative/list/hybrid)
    - Infers `[light]` vs `[deep]` from section intent; adds `[media]` when likely relevant; `[skip]` only for obvious placeholders
    - Never overwrites markers you’ve already added — it only fills gaps
- Defaults by mode: If you omit markers, nodes default to research based on `draft.config.yml` mode. See [Default behavior](#default-behavior).
- Markers: Use `[research]`, `[skip]`, `[light]`, `[deep]`, `[media]` sparingly; details in [Outline Markers](#outline-markers).

---

### Outline Markers

Control research behavior by adding markers to headings or bullets. See example in `templates/outline.example.md`.
Details by marker:

### Default behavior

If you don’t add markers, defaults follow the draft’s `mode` (see `draft.config.yml`) and items are included in the final draft unless skipped:

- Narrative: H2/H3 headings default to `[research]` unless `[skip]`.
- List: top-level list items default to `[research]` unless `[skip]`.
- Hybrid: both rules apply.
  During processing, these implicit defaults are treated the same as explicit `[research]` markers.

### Marker: `[research]`

Forces a research-and-write unit for this node. Use on sections or list items that need dedicated sourcing and a standalone written section.

### `[skip]`

- Purpose: Excludes this node from the research and writing queue.
- Output: No `research/N.md`, no `sections/N.md`, and no separate section in the final draft.
- Roll-up: If under a researched parent, treat as guidance; writer may fold points into the parent’s prose.
- Inheritance: Applies to children unless a child explicitly adds `[research]`.
- Precedence: Suppresses other markers on the same node (e.g., `[media]`, `[light]`, `[deep]`).
- Numbering: Skipped nodes don’t receive queue IDs and don’t create files.
- Examples:
  - Background `[research]` → child Timeline `[skip]`: Only Background runs; no separate Timeline section.
  - Appendix `[skip]` → child Dataset summary `[research]`: Only Dataset summary runs; Appendix itself doesn’t create a section.

### `[light]` / `[deep]`

Hints the research depth for a node marked (or defaulted) to `[research]`. Light favors quick validation and fewer sources; deep favors comprehensive sourcing and multi-viewpoint coverage. Children can override the parent.

### `[media]`

Runs a media pass for this node (opt-in). Current scope: image assets, and URLs from X.com and YouTube. Media is attached to the research for use at compile time.

## Write

- **Command:** `/write`
- **Purpose:** Runs the full pipeline from outline → research → writing → review → compilation.
- **Active context:** Uses the active project (`.active-project`) and active draft (`projects/{project}/.active-draft`).
- **Inputs:**
  - `projects/{project}/project.brief.md` (tone, sourcing rules)
  - `projects/{project}/drafts/{draft}/draft.config.yml` (draft-specific settings)
  - `outline.md` (+ markers)

### How It Works

The `/research` command walks through your outline **sequentially, section by section**. For each section (or list item), it does three things:

1. **Research:**  
   A dedicated _researcher_ agent gathers concrete facts, citations, and media (if `[media]` is marked). Results are written to `research/N.md`.

2. **Writing with rolling context:**  
   A _writer_ agent expands that section into prose using:

   - the global brief,
   - the section’s research notes,
   - a short synopsis of prior sections (≤ 350 words),
   - and the last two finalized sections verbatim.  
     This rolling context ensures continuity and flow without overwhelming tokens.  
     Draft is saved to `sections/N.md` and a 3–5 sentence synopsis is saved to `synopses/N.txt`.

3. **Review:**  
   A _reviewer_ agent fact-checks and polishes `sections/N.md` in place.

After all sections are processed, an _editor_ agent stitches the work together:

- Reads all synopses + the first and last paragraph of each section.
- Adjusts transitions, voice, and headings.
- Produces the final compiled draft (`FINAL.md`).
- If `[media]` markers are present, a media postprocessor generates an HTML version with embeds.

### Sequential Flow (not parallel)

This process is **not parallelized**. It intentionally moves **one section at a time**, carrying forward only concise summaries and the last two completed sections. This keeps each step grounded while preventing context bloat. The trade-off is that it’s slower than a parallel run, but produces more coherent results.

### Files Produced

- `research/N.md` — raw research notes per section
- `sections/N.md` — written section drafts
- `synopses/N.txt` — compact summaries of each section
- `FINAL.md` — compiled draft (Markdown)
- `FINAL.html` — compiled draft with media (only if `[media]` is used)

## Project Structure

```
projects/project-name/
├── project.brief.md           # research brief
└── drafts/
  └── draft-one/
    ├── draft.config.yml   # draft-level settings
    ├── outline.md         # single outline file
    ├── research/          # per-section notes
    ├── sections/          # written sections
    ├── synopses/          # per-section summaries (N.txt)
    ├── FINAL.md           # compiled output (Markdown)
    └── FINAL.html         # compiled with media (if any)
```
