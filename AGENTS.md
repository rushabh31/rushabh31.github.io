# Agent instructions

Full instructions live in [CLAUDE.md](CLAUDE.md). Read it before changing anything. The editing
workflow is in [.claude/skills/portfolio-editing/SKILL.md](.claude/skills/portfolio-editing/SKILL.md).

The short version:

- `index.html` is the whole homepage in one self-contained file (inline CSS and JS, no build).
  Pushing to `main` deploys it to https://rushabh.info through GitHub Pages.
- Content lives in JS data objects (`NODES`, `STEPS`, `ROUTES`, `PAPERS`, `REVIEWS`, `OBS`).
  Every fact is real; never invent or alter content, quotes or links.
- Keep lines 20 to 49 of `index.html` (the color tokens) where they are; the résumé build reads them.
- Every change must work in dark and light, at phone and desktop width, with reduced motion.
- Never put an email, phone number or home address on a page or in any repo file.
- `resume.html` and `resume.pdf` are generated; change them only for an explicit résumé request.
- Don't touch `CNAME` or `old-index.html`.
