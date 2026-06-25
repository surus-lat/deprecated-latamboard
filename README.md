# LatamBoard

Simple, stylized leaderboard for evaluating LLMs on Spanish and Portuguese tasks in LATAM. Built with React, Vite, and TailwindCSS.

## Updating leaderboard data (read this first)

**The data is not in this repo.** LatamBoard fetches all leaderboard rows and task content at runtime from the public Hugging Face dataset:

> https://huggingface.co/datasets/LatamBoard/leaderboard-results

To change a score, add a model, edit a task description, or fix a group:

1. Clone the dataset: `git clone https://huggingface.co/datasets/LatamBoard/leaderboard-results`
2. Edit one of three files:
   - `leaderboard_table.json` — rows on the landing page (model name + scores)
   - `tasks_groups.json` — group cards on `/tests`; column groupings on landing
   - `tasks_list.json` — individual task cards inside each group on `/tests`
3. `git add`, `git commit`, `git push` to `main`
4. Reload latamboard in your browser — changes appear immediately (`cache: 'no-store'` is set)

**Do not** edit `public/*.json` (the files don't exist), run `npm run fetch:data` (script was removed), or change scores in `src/`. The HF dataset is the only source.

For the full how-to, schema crib sheet, troubleshooting, and common mistakes, see `.claude/skills/update-data/SKILL.md` — it loads automatically for AI agents working in this repo.

## Features

- Landing page with hero and a sortable, filterable results table
  - Default visible aggregates: `overall_latam_score`, `spanish_score`, `portuguese_score`
  - Column toggles grouped by task categories. Aggregates and per-language tasks are visually distinguished
  - Columns order is derived from `tasks_groups.json` (fetched from HF)
- Tasks page with expandable group cards and per-task cards
  - Group cards: click to expand the long description (supports inline Markdown-style links)
  - Task cards: click to expand long description and dataset link
- About page with context on who/why/how
- Submit page to suggest models (model name, precision, revision, email)
- Footer links to website, Discord, and email

## Stack

- React + Vite
- TailwindCSS (brand tokens in `src/index.css`)
- No backend; data fetched directly from Hugging Face on page load

## Getting Started

Requirements:
- Node.js 18+

Install and run:

```bash
npm i
npm run dev
```

Open the printed Local URL. Both pages fetch `leaderboard_table.json`, `tasks_groups.json`, and `tasks_list.json` directly from Hugging Face — there is no `public/` data step.

## Data Source

See [Updating leaderboard data](#updating-leaderboard-data-read-this-first) at the top. If HF is unreachable at runtime, the pages display a "Failed to load" error; there is no bundled fallback.

## Configuration

- Branding colors and fonts: `src/index.css` (CSS variables)
- Table behavior (defaults/groups/order): `src/pages/Landing.tsx`
- Tasks page grouping logic: `src/pages/Tests.tsx`

## Project Structure

```
latam-leaderboard/
  public/
    latam_map.png           # static image asset only
  src/
    pages/
      Landing.tsx           # hero + leaderboard table (fetches from HF)
      Tests.tsx             # task groups + task cards (fetches from HF)
      About.tsx             # who/why/how
      Submit.tsx            # suggestion form (client-only)
    index.css               # Tailwind + brand tokens
    App.tsx                 # routes + layout + footer
```

## Deployment

Build:

```bash
npm run build
```

Preview locally:

```bash
npm run preview
```

Any static host (e.g., Netlify, Vercel, GitHub Pages, S3) can serve the `dist/` folder. No data step is required at build or deploy time — the app fetches from HF in the browser.

## Data Source & Community

- Dataset: https://huggingface.co/datasets/LatamBoard/leaderboard-results
- Discord: https://discord.com/invite/yGCCUhqtpS
- Website: https://surus.dev

## Notes

- The table groups columns by category using the keys/subtasks in `tasks_groups.json`. Aggregates are highlighted and component task columns are styled to match their group.
- The Submit page is client-only (no backend); submissions are logged in the console as a placeholder.
