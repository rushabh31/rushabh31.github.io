# rushabh.info: agent guide

This repo is Rushabh Patel's personal site, served at https://rushabh.info by GitHub Pages.
Read this file before you change anything. It is written so an agent with no other context can
keep developing the site safely. `AGENTS.md` points here, and `.claude/skills/portfolio-editing/`
holds the step-by-step editing workflow.

## What the site is

The homepage tells Rushabh's education and career as a **live AI agent loop**. Visitors pick a
route (education or experience) and an "agent harness" runs it stop by stop: plan, think, call a
tool, check the result, remember it. The look follows the original rushabh.info style: a deep navy
night sky in dark mode, a warm paper palette in light mode, hand-lettered headings and a
handwritten signature.

Everything on the page is real: dates, employers, degrees, papers and the LinkedIn
recommendations come from Rushabh's résumé and profile. Never invent content (see Rules).

## File map

| Path | What it is |
| --- | --- |
| `index.html` | The homepage. One self-contained file: inline CSS in a single `<style>`, inline JS in a single IIFE `<script>`. This file is the source of truth; there is no build step for it. |
| `resume.html` | Interactive résumé page in the same design tokens. Generated (see "Résumé"). |
| `resume.pdf` | ATS-friendly one-column PDF of the résumé, printed with headless Chromium. Generated. |
| `old-index.html` | The previous homepage, kept as a backup. Don't edit it, don't link to it from the new page. |
| `CNAME` | `rushabh.info`. Never change or delete it, or the custom domain breaks. |
| `css/Chasing.ttf`, `css/BeautyDemo.ttf` | The two display fonts used by `index.html` and `resume.html`. |
| `images/logo/avatar.png` | Favicon. |
| `tailwind.css`, `js/`, the rest of `css/` and `images/` | Assets used only by `old-index.html`. Leave them in place. |

## Design system

### Fonts

| Token | Font | Use |
| --- | --- | --- |
| `--f-head` | `chasing` (local `css/Chasing.ttf`) | Section headings, the "Hi, I'm Rushabh" greeting, big hand-lettered moments |
| `--f-sig` | `BeautyDemo` (local `css/BeautyDemo.ttf`) | The signature only |
| `--f-body` | Inter (Google Fonts) | Body copy, UI |
| `--f-mono` | JetBrains Mono (Google Fonts) | Code, the agent console, tool calls, keycaps, small labels |

Google Fonts is the only external stylesheet. No other CDNs are used.

### Color tokens

Defined at the top of the `<style>` in `index.html`, **lines 20 to 49**:

1. `:root{ ... color-scheme:dark; }` is the dark palette (the default).
2. `@media (prefers-color-scheme: light){ :root:not([data-theme="dark"]){ ... } }` is the light palette for the OS setting.
3. `:root[data-theme="light"]{ ... }` is the same light palette when the visitor toggles it.

**Keep those 30 lines exactly where they are and in that shape.** The résumé build reads lines
20 to 49 of `index.html` verbatim and asserts that line 20 starts with `:root{` and line 49 contains
`color-scheme:light }`. If you add CSS above them or reformat them, the résumé build fails. Add new
tokens inside the existing blocks (in all three) rather than adding lines before them.

Main tokens: `--bg`, `--sky-top`, `--sky-bot`, `--card`, `--card-2`, `--line`, `--code`, `--ink`,
`--muted`, accents `--pink --mint --lime --wave --blue`, and one color per industry used for routes
and stops: `--t-edu` (education), `--t-hc` (healthcare), `--t-ins` (insurance), `--t-fin`
(payments), `--t-tel` (telecom), `--t-bank` (banking). `--ease` is the shared easing curve.

Never hard-code a color in new CSS or SVG; use these tokens so both themes work. JS reads the
current values through `readVars()` when it needs them in canvas or SVG.

### Theme

The toggle button `#themeBtn` sets `data-theme` on `<html>` and saves it in `localStorage` under
`rp-theme`. With nothing saved, the OS setting wins. Every change must look right in both themes.

### Motion

The page is heavily animated, so motion has a strict rule: **everything must respect
`prefers-reduced-motion: reduce`**. CSS has a reduced-motion block near the end of the `<style>`;
JS reads it into `RM` at the top of the script and listens for changes. With reduced motion,
show the final state at once: no typing, no packets, no drawing animations. Animations also pause
when the tab is hidden.

### Layout and responsiveness

Two breakpoints matter in practice: phone (test at 390 px) and desktop (test at 1366 px). There
must be no horizontal scroll at any width. SVG diagrams measure their container and rebuild on
resize (ResizeObserver), and several switch to a vertical layout on narrow screens.

## Page structure (`index.html`)

Top to bottom:

1. `nav.nav`: logo, section links, Résumé link (`resume.html`), theme toggle.
2. `header.hero`: the greeting "Hi, I'm Rushabh" with an ink underline, the headline "I build AI
   that regulated industries can trust.", the lede, the **runline**, the stats row (10+ years in ML, 3 degrees,
   10 stops at work, 9 papers) and the agent card.
   - The runline is a fake command `› agent.route("education")`. A small script types the argument,
     shows "planning route…", prints an observation from `OBS` (for example "10 stops · Allstate →
     Jvion → CHOP → AIG → Citi"), holds, deletes and swaps to the other route. Hover or focus
     pauses it. Clicking a token opens that route in the console.
3. `section#roadmap`: the **console** (`#console`), the heart of the page. It has two views,
   switched by `data-view`:
   - `#vRouter` (`data-view="router"`): the **branch router**. One root (the agent) forks into two
     git-style branches, education and experience, drawn as SVG lanes with every stop as a node,
     moving packets on the lanes, and both branches merging at "today". Hovering a branch dims the
     other one and the readout `p(education) / p(experience)` swings toward it, like a router
     choosing. Clicking a branch, a stop or a keycap button opens the route at that stop. Keys
     **E** and **X** open a route while the console is on screen. Built by `buildFork()`;
     horizontal above 760 px wide, vertical below.
   - `#vRoute` (`data-view="route"`): the **agent harness** running one route. Five modules:
     `mPlan` (planner), `mCore` (the model, with a small neural-net animation), `mTool` (tool call
     with arguments), `mEval` (eval / observation), `mMem` (memory of skills learned). Wires carry
     packets between modules. `nav.trace` is the rail of steps on the route. `render()` draws a
     step, `openRoute(r, j)` opens route `r` at step `j`, `showRouter()` goes back.
   - Clicking a stop also opens a details drawer (`openNode()`), styled as a tool-call trace.
4. `section#now`: "Where the run landed": the current role.
5. `section#evals`: "14 people reviewed the run": the LinkedIn recommendations as an orbit of
   reviewers, with a selected quote and signal tags (`SIG`).
6. `section#research`: "Nine papers along the way": publications from `PAPERS`.
7. `section#contact`: "Let's build what's next": links only (LinkedIn, GitHub, Stack Overflow,
   X/Twitter). No email, no phone, no form.
8. `footer`: signature.

### Data objects in the script

Content lives in plain JS objects near the top of the IIFE. Edit content here, not in the markup.

| Object | Holds |
| --- | --- |
| `DOMAIN`, `IND` | Industry key → label: `edu hc ins fin tel bank` |
| `NODES` | Every stop by id (`sym`, `allstate`, `paytooth`, `uis`, `jvion`, `vns1`, `chop`, `phd`, `att`, `eiq`, `vns2`, `aig`, `citi`): `short`, `title`, `org`, `place`, `when`, `d` (industry), `pts` (bullets) |
| `STEPS` | One agent-loop step per stop: route `k`, `nodes`, tool `fn`, `args`, `year`, `place`, `thought`, `obs`, `learned`, `tok` |
| `ROUTES` | `edu` and `exp`: name, color token, stop `ids`, and the `end` card shown when a route completes |
| `FORK_ORDER`, `FK_SHORT` | Chronological order of stops on the branch router, and short labels |
| `OBS` | The hero runline's typed words and observations |
| `PAPERS` | `[year, title, venue]` for each paper |
| `ORGS` | Organization names |
| `REVIEWS` | The 14 LinkedIn recommendations, quoted verbatim: name `n`, `role`, `org`, group `g` (`edu`/`exp`), context `ctx`, quote `q` |
| `REV_CTX`, `SIG` | Review context labels, and keyword → signal tag rules |
| `DOMS`, `MODS` | Industry order, harness module ids |

Counts appear in copy as well as in data. If you add or remove a stop, paper or review, update every
place that states the count: hero stats, the agent card, the `.fk-meta` lines ("3 degrees",
"10 stops"), `OBS`, `ROUTES.*.end`, section headings ("14 people", "Nine papers") and the
`<meta name="description">`. Search the file for the old number before you commit.

## Content sources and facts

- Facts come from Rushabh's résumé. Experience is stated as **10+ years** everywhere.
- Current role: GenAI engineering lead for risk and controls at Citi.
- Recommendations are real LinkedIn recommendations, quoted word for word. Never write, edit,
  paraphrase or "improve" a quote, and never add a recommendation that Rushabh hasn't given you.
- Links: only real profile links already in the file. Never guess a URL.

## Résumé (`resume.html`, `resume.pdf`)

Both are generated outside this repo and committed as output:

- `resume.html` is built from a source template by a small script that injects the color tokens
  from `index.html` (lines 20 to 49, see above), the fonts and the favicon. It shares the site's
  look.
- `resume.pdf` is a plain one-column ATS version printed from that content with headless
  Chromium (puppeteer), US Letter, Liberation Sans.

Don't hand-edit or regenerate them as part of homepage work. Change them only when Rushabh asks
for a résumé change, and keep both in sync when you do. The homepage links to `resume.html`; the
résumé page links to the PDF.

## Deploy

- GitHub Pages serves the `main` branch root. There is no build: pushing to `main` deploys.
- Push with `git push -u origin main`. Each push starts a "pages build and deployment" run in
  Actions. Check it finished with
  `gh api repos/rushabh31/rushabh31.github.io/actions/runs --jq '.workflow_runs[:2][]|.name+" "+.status+" "+(.conclusion//"")+" "+.head_sha[:7]'`.
- Then open `https://rushabh.info/?v=<short sha>` in a real browser to bust the cache and confirm
  the change is live.
- Everything in this repo is public on the web, including this file. Never commit anything private.

## Testing a change

There is no test suite. Before every push:

1. Open `index.html` in headless Chromium (puppeteer or Playwright) from a folder that also holds
   `css/` and `images/`, so the fonts load.
2. Screenshot at 1366 px and 390 px wide, in dark and light (set `data-theme` on `<html>`). Look at
   the hero, the router, a running route, the drawer, evals and contact.
3. Collect console errors and `pageerror` events. There must be none.
4. Check there is no horizontal scroll at 390 px (`document.documentElement.scrollWidth <= 390`).
5. Exercise the interactions you touched: click a branch, a stop and a keycap button, press E and
   X, go back to the router, toggle the theme.
6. Emulate `prefers-reduced-motion: reduce` and confirm everything shows its final state.

## Rules

Do:
- Keep `index.html` a single self-contained file with inline CSS and JS.
- Use the design tokens and the four fonts above; match the hand-lettered plus mono "agent" voice.
- Keep every interactive element reachable by keyboard, with a visible focus ring and an
  `aria-label` on SVG controls.
- Respect reduced motion and both themes in every change.
- Write copy in first person, in plain warm sentences, in Rushabh's voice. Avoid em dashes in
  copy; en dashes are fine for date ranges.

Don't:
- Put an email address, phone number or home address on any web page or in any repo file. The
  contact section uses profile links only.
- Add a phone number to the PDF unless Rushabh confirms it.
- Invent or alter recommendations, employers, dates, papers, metrics or links.
- Move or reformat lines 20 to 49 of `index.html` (the résumé build reads them).
- Touch `CNAME`, delete `old-index.html`, or remove the legacy assets it uses.
- Add trackers, analytics or third-party scripts.
- Rewrite history on `main` or force-push.
