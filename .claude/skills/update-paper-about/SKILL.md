---
name: update-paper-about
description: Use when changing the About page content — title, authors, abstract, body, keywords, date, or citation. The About page is NOT a React component. It is a Quarto-rendered standalone HTML file at public/about.html, ~1.9MB, and the .qmd source is not checked in. Triggers on phrases like "update the abstract", "add an author to the paper", "change the about page", "edit the manuscript", "update the paper".
---

# Update the LatamBoard About page (the paper)

## TL;DR

The About page is a Quarto-rendered standalone HTML file. There is **no .qmd source** in this repo — the rendered artifact `public/about.html` is the source of truth, and every edit is a surgical text edit on that file. The route `/about` (React) just redirects to `/about.html`.

Source content for the paper lives at `manuscript.md` at the repo root (markdown). When the manuscript changes, mirror the change into `public/about.html` at the seven specific locations listed below.

## Why it works this way (and what's brittle)

- Original setup (on the `marian-redesign` branch, then merged into `main`): Quarto + AGU template → `public/about.html` + `public/nav-injector.js` (client-side React-nav graft) + `scripts/inject-nav.js` (prebuild hook).
- The `.qmd` source was never committed. **The rendered HTML is the source.** Updating means editing the HTML directly.
- Reconstructing the `.qmd` and re-rendering would technically be cleaner, but requires the Quarto CLI + AGU template + matching the existing visual exactly. Not worth doing on a per-edit basis — only if the layout itself needs to change.

## The architecture in 30 seconds

```
manuscript.md  (the writeable source of truth — abstract content)
    │
    │  (manual mirror — this skill is the workflow)
    ▼
public/about.html  (1.9MB Quarto-rendered AGU paper)
    │
    │  (prebuild: scripts/inject-nav.js inserts a <script> tag)
    ▼
public/nav-injector.js  (runtime: prepends a vanilla-CSS navbar to match React app)
    │
    │  ( /about route redirects here )
    ▼
src/pages/About.tsx  (`window.location.href = '/about.html'`)
```

## The seven edit locations in `public/about.html`

When you change the manuscript, update all of these in one pass. Use the `Edit` tool with unique surrounding context — line numbers shift as soon as one edit lands.

| # | What | Where | Notes |
|---|---|---|---|
| 1 | HTML head meta — authors | `<meta name="author" content="…">` block near top | One `<meta>` per author. Add/remove as needed. |
| 2 | HTML head meta — date | `<meta name="dcterms.date" content="YYYY-MM-DD">` | ISO date. |
| 3 | HTML head meta — keywords | `<meta name="keywords" content="…">` | Comma-separated. |
| 4 | HTML `<title>` | `<title>LatamBoard: …</title>` | Keep the `LatamBoard: ` prefix for SEO. |
| 5 | Citation meta block | `<meta name="citation_title">`, multiple `<meta name="citation_author">` | Mirror authors from #1; title from #4. |
| 6 | Visible header — `h1.title`, author/affiliation cards, published date, abstract, keywords | Inside `<header id="title-block-header">…</header>` | See "Author card structure" below. |
| 7 | Body `<main>` sections + TOC + Citation appendix | After `<main class="content…">` and inside `<nav id="TOC">` | One `<section class="level2">` per body section. The TOC has one `<li>` per section. The citation appendix at the bottom has the BibTeX + citation string. |

Plus: every edit should update **the TOC sidebar** (`<nav id="TOC">`) AND **the citation block** (`<div id="quarto-appendix">`) to match.

## Workflow

1. **Read `manuscript.md`** — that's the source of truth for the abstract & body content.
2. **Open `public/about.html`** and locate the 7 edit zones with grep:
   ```bash
   awk 'length<800' public/about.html | grep -nE 'class="title"|class="author"|class="affiliation"|class="abstract"|class="keywords"|class="date"|<title>|h2 class="anchored"|id="quarto-citation"'
   ```
   The `awk 'length<800'` filter is **important** — about.html contains multiple 100KB+ embedded encoded Bootstrap CSS lines that swamp grep output.
3. **Edit in this order** (so failed matches don't cascade):
   1. Title (#1 #3 #4 #5 title meta + h1)
   2. Authors (#1 author meta + #5 citation_author meta + #6 author cards)
   3. Date (#2 + #6 published)
   4. Keywords (#3 + #6 keywords)
   5. Abstract (#6)
   6. Body sections + TOC (#7)
   7. Citation appendix (#7)
4. **Verify**:
   ```bash
   # tag balance
   python3 -c "
   import re
   c = open('public/about.html').read()
   for tag in ['div', 'section', 'p']:
       o = len(re.findall(rf'<{tag}\b', c))
       cl = len(re.findall(rf'</{tag}>', c))
       print(f'{tag}: open={o} close={cl} diff={o-cl}')
   "
   # author + section count
   grep -c "Francis Perelman" public/about.html  # expect: 4 (meta, citation_author, p.author, citation block)
   awk 'length<800' public/about.html | grep -c '<h2 class="anchored"'  # expect: # of body sections
   ```
5. **Run the app and visit `/about`**:
   ```bash
   npm run dev
   # open http://localhost:8080/about → redirects to /about.html → should render the paper with React navbar grafted on top
   ```

## Author card structure (copy-paste template)

Each author has TWO `<div class="quarto-title-meta-contents">` blocks — one for the author name, one for the affiliation. They sit inside `<div class="quarto-title-meta-author">`. A minimal author with no email/orcid looks like:

```html
                      <div class="quarto-title-meta-contents">
            <p class="author">First Last</p>
          </div>
                <div class="quarto-title-meta-contents">
                    <p class="affiliation">
                        Institution Name
                      </p>
                  </div>
```

An author WITH email + orcid (Quarto's default render) has a long `<p class="author">…<a class="quarto-title-author-email"…><a class="quarto-title-author-orcid"…><img…base64-png…></a></p>` line. To add an author to an existing block, copy one of the existing minimal-style blocks (Francis Perelman's, after we added him).

## What NOT to do

| Don't | Why |
|---|---|
| Edit `public/about.html` by hand-typing large blocks | The embedded CSS / base64 PNGs trip up most editors. Use the `Edit` tool with unique anchor strings. |
| Try to re-render with `quarto render` | No `.qmd` source is checked in. Even with one, the AGU template + custom navbar injection are fiddly. |
| Switch the page to a React component | The Quarto paper look is the deliberate aesthetic. If you want React rendering, that's a separate, intentional redesign — discuss with user first. |
| Forget to also update the TOC (`<nav id="TOC">`) when adding/removing body sections | The sidebar will drift from the body and look broken. |
| Forget to update the Citation appendix at the bottom | The BibTeX is what people copy. It must match authors/title/date. |
| Remove the prebuild hook (`scripts/inject-nav.js`) | It's what makes `<script src="/nav-injector.js">` exist in the deployed HTML. Without it, the React-style navbar disappears. |

## Files in play

| File | Role |
|---|---|
| `manuscript.md` | Source of truth for paper content (markdown). |
| `public/about.html` | The Quarto-rendered standalone HTML (1.9MB). **Edit this directly.** |
| `public/nav-injector.js` | Client-side script that grafts a React-app-styled navbar onto the Quarto HTML. |
| `scripts/inject-nav.js` | Build script that inserts `<script src="/nav-injector.js">` before `</body>` in about.html. Runs via `prebuild`. |
| `src/pages/About.tsx` | Redirect-only React component. Sends `/about` → `/about.html`. |
| `NAVIGATION_INJECTION_PLAN.md` | Original design doc for the nav-injection approach. Read for context only. |

## Common mistakes that waste a debugging cycle

| Symptom | Cause | Fix |
|---|---|---|
| `Edit` says "old_string not found" | You included one of the giant embedded CSS lines (>10KB) in your match string | Use shorter, more specific anchor strings — avoid lines >300 chars |
| TOC sidebar shows old section names after body edit | Forgot to update `<nav id="TOC">` | Re-grep for `<a href="#sec-` to find TOC entries and update |
| Author shows in some places but not others | Updated `<meta name="author">` but forgot `<meta name="citation_author">` or `<div class="quarto-title-meta-author">` cards | Use grep `-c "Author Name"` — expect ≥4 hits |
| BibTeX still shows old title/year after edit | Forgot to update `id="quarto-citation"` appendix block at line ~2380 | Search for `@online{` to find the BibTeX block |
| Deployed change doesn't show | Edits land in `public/about.html` but Cloudflare cache | Hard-reload (`Cmd-Shift-R`); Cloudflare serves `cache: 'no-store'` for the React app but `/about.html` is a static asset and may be cached longer |
| `/about` URL shows React spinner forever | `About.tsx` redirect broken, or `/about.html` returns 404 | Check that `public/about.html` exists and `About.tsx` still has `window.location.href = '/about.html'` |
