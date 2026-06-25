---
name: update-data
description: Use when adding, removing, or changing leaderboard rows, task groups, task descriptions, scores, or any content shown on LatamBoard. The data is NOT in this repo — it lives on Hugging Face and is fetched at runtime. Triggers on phrases like "add a model", "fix this score", "update task description", "change leaderboard data", "edit tasks_groups", "the page shows wrong data".
---

# Update LatamBoard data

## TL;DR

The data lives on Hugging Face, not in this repo. Edit and push to the HF dataset, reload the page, done. `cache: 'no-store'` is already set, so changes appear immediately.

## Where the data lives

LatamBoard fetches all leaderboard and task content at runtime from the public HF dataset:

- **Repo:** `LatamBoard/leaderboard-results`
- **URL:** https://huggingface.co/datasets/LatamBoard/leaderboard-results
- **Resolve base:** `https://huggingface.co/datasets/LatamBoard/leaderboard-results/resolve/main/`

There are no JSON data files in this repo. `public/` only holds static image assets.

## The three files

| File | Controls |
|---|---|
| `leaderboard_table.json` | Every row on the landing page (model name + scores per task column) |
| `tasks_groups.json` | Group cards on `/tests`; column groupings and ordering on `/landing` |
| `tasks_list.json` | Individual task cards inside each group on `/tests` |

## How to update

1. Clone the dataset (first time only):
   ```bash
   git clone https://huggingface.co/datasets/LatamBoard/leaderboard-results
   ```
   For private/large repos, install `huggingface_hub` or use an HF token.
2. Edit the JSON file(s).
3. Commit and push to `main`:
   ```bash
   cd leaderboard-results
   git add <file>.json
   git commit -m "Add <model> / fix <task> / update <group>"
   git push
   ```
4. Reload latamboard in the browser. Done.

## Verifying the change is live

Open the page you touched (landing for scores, `/tests` for task content):

- The change should be visible after a normal reload — no hard-reload needed.
- If it isn't, check the browser's network tab for the request to `huggingface.co/datasets/LatamBoard/leaderboard-results/resolve/main/<file>.json` and confirm the response body contains your edit.
- If the page shows "Failed to load", your JSON is malformed. Validate with `jq . <file>.json` before pushing.

## Common mistakes

| Mistake | Why it doesn't work |
|---|---|
| Editing `public/leaderboard_table.json` locally | This file does not exist. The bundled JSON was removed. |
| Running `npm run fetch:data` | Script was removed. Data is fetched in the browser, not at build time. |
| Editing `src/pages/Landing.tsx` to change a score | Source code does not contain data. Only the HF dataset does. |
| Forgetting to push the HF commit | A local clone change doesn't surface until pushed to `main` on huggingface.co. |
| Restoring an HF-first /public fallback in `fetchHFJson` | The fallback would silently bypass HF whenever you tested locally. See `learnings.jsonl` → `no-hf-first-public-fallback`. |

## Schema crib sheet

Fetch the current file before editing so you match its shape:

```bash
curl -s https://huggingface.co/datasets/LatamBoard/leaderboard-results/resolve/main/tasks_groups.json | jq .
curl -s https://huggingface.co/datasets/LatamBoard/leaderboard-results/resolve/main/tasks_list.json  | jq .
curl -s https://huggingface.co/datasets/LatamBoard/leaderboard-results/resolve/main/leaderboard_table.json | jq '.[0]'
```

Group keys in `tasks_groups.json` map to column prefixes in `leaderboard_table.json` via the `groupPrefixMap` in `src/pages/Landing.tsx`:

| Group key | Column prefix |
|---|---|
| `latam_es` | `spanish_` |
| `latam_pr` | `portuguese_` |
| `transcription` | `transcription_` |

Adding a new group? Add it to `tasks_groups.json` AND update `groupPrefixMap` in `src/pages/Landing.tsx` to wire it to its column prefix.

## Where the fetch happens (for code-side debugging)

- `src/pages/Landing.tsx` — `fetchHFJson` helper, called for all three files in `useEffect`
- `src/pages/Tests.tsx` — `fetchJson` helper, called for `tasks_groups.json` and `tasks_list.json`

Both use `cache: 'no-store'` so changes are picked up on reload without CDN delay.
