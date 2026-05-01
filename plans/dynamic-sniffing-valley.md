# Implementation Plan: Personal Knowledge Wiki System (PKWS)

## Context

The user works across 8+ domains (fullstack dev, data science, AI/ML, cloud, marketing, 3D design, backend, system design) and is losing control of scattered knowledge. We're building a multi-domain LLM Wiki system inside Obsidian, based on Karpathy's LLM Wiki pattern, starting with a Data Science pilot wiki.

**Design Spec:** `docs/superpowers/specs/2026-04-12-llm-wiki-system-design.md`

## Implementation Steps

### Step 1: Create Vault Foundation
- Create folder structure: `_meta/`, `_templates/`, `_skills/`, `data-science/` (with subdirectories)
- Write global `CLAUDE.md` (master schema defining system-wide rules and wiki conventions)
- Write `master-index.md` stub
- Write `log.md` stub
- Write `_meta/goals.md` and `_meta/profile.md` stubs

**Critical files:** `CLAUDE.md`, `master-index.md`

### Step 2: Create Templates
- `_templates/wiki-page.md` — standard wiki page with frontmatter, sections
- `_templates/raw-source.md` — raw source metadata template
- These are Obsidian Templater-compatible files

**Critical files:** `_templates/wiki-page.md`

### Step 3: Create Data Science Wiki
- Write `data-science/CLAUDE.md` (domain schema)
- Write `data-science/index.md` stub
- Write `data-science/log.md` stub
- Create subdirectories: `raw/course-notes/`, `raw/papers/`, `raw/web-clips/`, `pages/statistics/`, `pages/machine-learning/`, `pages/math-foundations/`, `pages/sql-bigquery/`, `pages/python-data/`, `pages/projects/`

**Critical files:** `data-science/CLAUDE.md`, `data-science/index.md`

### Step 4: Create Custom Commands (Skills)
- Write core commands: `_skills/ingest.md`, `_skills/query.md`, `_skills/lint.md`
- Write thinking commands: `_skills/ideas.md`, `_skills/challenge.md`, `_skills/connect.md`, `_skills/gaps.md`
- Write management commands: `_skills/profile-update.md`, `_skills/roadmap.md`

**Critical files:** `_skills/ingest.md`, `_skills/query.md`, `_skills/lint.md`

### Step 5: First Ingest Test
- Add a sample source to `data-science/raw/` (e.g., a course note excerpt)
- Run the ingest skill to create first wiki pages
- Verify: page format correct, index updated, log written, wikilinks work

### Step 6: Obsidian Setup
- Install recommended plugins: Dataview, Templater, Web Clipper, Local Images Plus
- Configure Templater to use `_templates/` folder
- Configure Web Clipper to save to appropriate `raw/` folders
- Verify Graph View shows connections

### Step 7: Verification
- Run all verification checks from spec
- Test /query, /lint, /gaps commands
- Verify cross-references work correctly
- Confirm Obsidian visualization is functional

## Verification
1. Structure test: all folders and files exist
2. Ingest test: raw source → wiki page with correct format
3. Query test: question → answer citing wiki pages
4. Cross-reference test: second ingest creates wikilinks to existing pages
5. Obsidian test: graph view, templates, plugins working
6. Lint test: catches broken pages
7. Index test: accurate catalog of all pages
