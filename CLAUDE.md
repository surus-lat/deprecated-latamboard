# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## First, read this

**If the user wants to add, change, or remove any leaderboard content (rows, scores, task descriptions, groups): the data is NOT in this repo.** It lives on Hugging Face at `LatamBoard/leaderboard-results` and is fetched at runtime.

Before editing anything in `src/` or `public/` in response to a data-change request, invoke the `update-data` skill: `.claude/skills/update-data/SKILL.md`. It contains the exact workflow, schema crib sheet, and the common-mistakes table that prevents the multi-commit "edits don't take effect" loop documented in `learnings.jsonl`.

**If the user wants to change the About page (title, authors, abstract, body, keywords, citation): the About page is a Quarto-rendered standalone HTML at `public/about.html`** (~1.9MB) — not a React component. The `.qmd` source is not checked in; every edit is a surgical text edit on the rendered HTML. The markdown source for the paper content is `manuscript.md` at the repo root.

Before touching `public/about.html`, invoke the `update-paper-about` skill: `.claude/skills/update-paper-about/SKILL.md`. It lists the seven edit zones that all need to stay in sync (head meta, citation meta, h1, author cards, abstract, TOC, body sections, BibTeX appendix).

**If the user wants to change a redirect, URL rewrite, response header, SPA fallback, or anything about how latamboard.ai routes requests: that config is NOT in this repo.** The site is on Cloudflare Pages and routing lives in the Cloudflare dashboard. Editing `src/` cannot fix a redirect that runs at the edge.

Before changing routing, invoke the `cloudflare-routing` skill: `.claude/skills/cloudflare-routing/SKILL.md`. It explains where to look in the CF dashboard, how to verify with `curl -sI`, and when to put routing in the repo (`public/_redirects`) vs leave it in the dashboard.

## Project Overview

LatamBoard is a React + Vite application for evaluating LLMs on Spanish and Portuguese tasks. It features a sortable/filterable leaderboard table, task descriptions, and data sourced from a Hugging Face dataset.

## Development Commands

### Setup & Development
- `npm install` - Install dependencies
- `npm run dev` - Start development server (runs on port 8080)

### Build & Quality
- `npm run build` - Production build
- `npm run build:dev` - Development mode build
- `npm run preview` - Preview production build locally
- `npm run lint` - Run ESLint

### Data Management
**Hugging Face is the only source of truth.** There is no `fetch:data` script and no bundled JSON files. Both Landing and Tests pages fetch directly from `https://huggingface.co/datasets/LatamBoard/leaderboard-results/resolve/main/` at runtime with `cache: 'no-store'`.

To change leaderboard or task content: edit and push the JSON files in the HF dataset repo. Reload the page to see the change. Editing files in this repo will **not** affect the data shown.

## Architecture

### Tech Stack
- **Frontend**: React 19, TypeScript, Vite
- **Styling**: TailwindCSS with custom design system
- **Routing**: React Router DOM
- **Internationalization**: Custom i18n system supporting EN/ES/PT
- **Data**: Fetched at runtime from Hugging Face (no bundled data, no backend)

### Key Directories
- `/src/pages/` - Route components (Landing, About, Tests, Submit)
- `/src/components/ui/` - Reusable UI components
- `/src/i18n/` - Internationalization system with locale detection
- `/src/content/` - Static content files
- `/public/` - Static assets only (e.g., `latam_map.png`). No JSON data.

### Design System
- Custom CSS variables defined in `src/index.css`
- Extended Tailwind config with brand colors, animations, and spacing
- Score-specific color system (excellent, good, average, poor)
- Inter font family as primary typeface

### Data Flow
1. Browser loads the page; React effect fires `fetch(HF_BASE + '/...json', {cache: 'no-store'})`
2. Landing fetches `leaderboard_table.json`, `tasks_groups.json`, `tasks_list.json`
3. Tests fetches `tasks_groups.json` and `tasks_list.json`
4. If HF is unreachable, pages show a "Failed to load" error — there is no local fallback
5. All content supports multilingual display via i18n system

### Internationalization
- Auto-detects browser language (falls back to English)
- Stores locale preference in localStorage
- Translation files organized by locale in `/src/i18n/`
- `useI18n()` hook provides `t()` function and locale switching

### Component Patterns
- Pages are route-level components handling data fetching
- UI components follow shadcn/ui patterns
- Heavy use of Tailwind utility classes
- Custom animations for smooth transitions
- Responsive design with mobile-first approach