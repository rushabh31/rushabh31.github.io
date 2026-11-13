---
name: portfolio-editing
description: Use when changing rushabh.info (index.html, its copy, data, design, animations or the résumé links). Gives the workflow to edit the single-file homepage safely, test it in both themes at phone and desktop width, and deploy it through GitHub Pages.
---

# Editing rushabh.info

Read `CLAUDE.md` first for the design system, page structure and rules. This skill is the workflow.

## 1. Orient

- `git pull` on `main`, then read the part of `index.html` you'll change. The file is long (about
  1,900 lines); find things by searching, for example `id="vRouter"`, `function buildFork`,
  `const NODES`, `const REVIEWS`, `.fk-` or `prefers-reduced-motion`.
- Decide where the change belongs:
  - Copy or facts: the JS data objects (`NODES`, `STEPS`, `ROUTES`, `OBS`, `PAPERS`, `REVIEWS`),
    or the static markup for headings and the hero.
  - Look: CSS in the single `<style>`. Use tokens (`var(--ink)`, `var(--t-edu)` …), never raw colors.
  - Behaviour: the IIFE `<script>`. Keep helpers local to it; don't add globals or libraries.

## 2. Edit

- Keep `index.html` self-contained. Fonts come from `css/*.ttf` and Google Fonts only.
- Don't add lines above line 50 or reformat lines 20 to 49 (the token blocks).
- New CSS that animates needs a matching rule in the reduced-motion block. New JS animation must
  check `RM` and render the final state when it's true.
- New SVG controls need `tabindex="0"`, `role`, an `aria-label`, Enter/Space handling and a focus style.
- If a count changes (stops, degrees, papers, reviews, years), update every place that states it.
  Search the file for the old number.
- Contact stays links only: no email, no phone, no address, anywhere in the repo.
- Quotes in `REVIEWS` are verbatim LinkedIn recommendations. Don't edit them; add one only when
  Rushabh supplies its exact text.

## 3. Test

Use headless Chromium. A minimal puppeteer loop:

```js
const puppeteer = require('puppeteer');
(async () => {
  const b = await puppeteer.launch();
  for (const [w, h] of [[1366, 900], [390, 844]]) for (const theme of ['dark', 'light']) {
    const p = await b.newPage();
    const errs = [];
    p.on('pageerror', e => errs.push(e.message));
    p.on('console', m => m.type() === 'error' && errs.push(m.text()));
    await p.setViewport({ width: w, height: h });
    await p.goto('file://' + process.cwd() + '/index.html');
    await p.evaluate(t => document.documentElement.setAttribute('data-theme', t), theme);
    await new Promise(r => setTimeout(r, 1500));
    await p.screenshot({ path: `shot-${w}-${theme}.png`, fullPage: true });
    const sw = await p.evaluate(() => document.documentElement.scrollWidth);
    console.log(w, theme, 'scrollWidth', sw, 'errors', errs);
    await p.close();
  }
  await b.close();
})();
```

Run it from the repo root so `css/` and `images/` resolve. Then:

- Look at every screenshot: hero, branch router, a running route, drawer, evals, contact.
- Click through what you changed (a branch, a stop, a keycap; keys E and X; back to the router).
- Repeat once with `await p.emulateMediaFeatures([{ name: 'prefers-reduced-motion', value: 'reduce' }])`.
- Errors must be empty and `scrollWidth` must not exceed the viewport width.

Don't commit screenshots or test scripts.

## 4. Deploy

- Commit only the files you meant to change. Write a plain message saying what a visitor will see.
- Push only when Rushabh has asked for the change to go live: `git push -u origin main`.
- Watch the Pages run:
  `gh api repos/rushabh31/rushabh31.github.io/actions/runs --jq '.workflow_runs[:2][]|.name+" "+.status+" "+(.conclusion//"")+" "+.head_sha[:7]'`
- Open `https://rushabh.info/?v=<short sha>` in a browser and confirm the change shows.

## Common changes

- **New job or degree:** add a `NODES` entry, a `STEPS` entry, its id in `ROUTES.<r>.ids` and in
  `FORK_ORDER` (chronological), then update counts in copy, `OBS` and `ROUTES.<r>.end`.
- **New paper:** add `[year, title, venue]` to `PAPERS` (newest first) and update "Nine papers".
- **New recommendation:** add it to `REVIEWS` verbatim with its group `g`, then update "14 people"
  and "All 14".
- **Copy tweak in the hero:** edit the markup in `header.hero`; the runline words live in `OBS`.
