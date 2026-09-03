# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal portfolio site (Mj Spencer Almodiel — Lead Full-Stack AI Engineer), deployed to GitHub Pages. There is **no build step, no package manager, no test suite, and no dependencies to install**. `index.html` is the entire application: HTML, CSS (one `<style>` block), and JS (one `<script>` block) live in that one ~1800-line file. Everything else is static assets.

## Commands

- **Preview locally:** open `index.html` directly in a browser, or `python -m http.server 8000` from the repo root (needed if you want hash routing to behave like production).
- **Deploy:** push to `main`. `.github/workflows/deploy.yml` uploads the repo root as-is to GitHub Pages — no build. `.nojekyll` keeps Pages from filtering files.

## Architecture

### Two-view SPA with hash routing (no framework)

`index.html` contains two mutually exclusive view layers, toggled by plain JS:

- `#work-listing-view` — hero, work card grid, capabilities arch, contact section.
- Four `.case-study-overlay` divs (`#study-bb`, `#study-cheercpt`, `#study-fff`, `#study-nexus`) — full deep-dive pages, hidden until activated.

`openCaseStudy(id)` hides the listing and adds `.active` to one overlay; `showAllWork()` reverses it. The URL hash is the source of truth: `handleHashChange()` runs on `hashchange` and `DOMContentLoaded`, matching against the `CASE_STUDY_IDS` array. **Adding a case study means adding its id to `CASE_STUDY_IDS`** — otherwise a deep link to it silently falls back to the listing view.

`scrollToSection('capabilities'|'contact')` is a special case: it must first restore the listing view (`showAllWork(false)`) and force-reveal the target before scrolling, since those sections may be `display: none` when a case study is open.

### Sticky-scroll capabilities arch

`#capabilities` is an `.arch-scroll-track` with `height: 280vh` — a scroll runway. Inside, `.arch-sticky-viewport` pins while the page scrolls through it. `updateStickyArchProgress()` (scroll listener) converts `-rect.top / (offsetHeight - innerHeight)` into 0→1 progress, then:

- interpolates `stroke-dashoffset` on `#arch-overlay-path` from `1376` → `344` (dasharray `1720`) to draw the arc,
- buckets progress into 4 quarters and swaps `.active` between `#cap-card-0..3` and `.arch-dot-lit` between `#arch-dot-0..3`.

The four SVG dot coordinates on the `M 0,350 A 848,848 0 0,1 1440,350` path are hand-computed to sit on the curve. If the arc path or viewBox changes, the dot `cx`/`cy` values and the dashoffset endpoints must be recomputed together. Changing the card count means updating the quarter thresholds, the dashoffset range, the dot geometry, and `scrollToCapStep`'s `stepIndex / 3` divisor.

`body`/`html` use `overflow-x: clip` rather than `hidden` — `hidden` breaks `position: sticky` here. Don't "fix" it back.

### Reveal animations

Elements carry `.reveal-on-scroll` (opacity 0 + translateY) and get `.is-visible` from `checkReveal()` on scroll/load. Because it's a class toggle rather than IntersectionObserver, code that reveals a hidden section programmatically must add `.is-visible` to the section *and* its descendants (see `scrollToSection`).

## Conventions

- All styling flows from the CSS custom properties in `:root` (`--color-surface-*`, `--color-text-*`, accents, `--radius-*`, `--ease-out-expo`). Use the tokens; don't hardcode hexes. The design deliberately tracks mariajoaoabrantes.work — dark surfaces, Plus Jakarta Sans + IBM Plex Mono from Google Fonts.
- Interactivity is wired with inline `onclick` handlers calling global functions. Match that pattern rather than introducing listeners or modules.
- Case studies share a fixed section vocabulary: `.study-hero-grid` → `.outcomes-row`/`.outcome-card` → `.project-meta-grid` (My Role / Stage & Access / Primary Stack / AI Tooling) → `.study-main-visual` → `.section-headline` + `.features-list-grid` → `.gallery-grid`. Clone an existing overlay when adding one.
- Screenshots live in `assets/` with a per-project prefix (`bb-`, `cheercpt-`, `fff-`, `sienvi-nexus-`). All `<img>` use `./assets/...` relative paths and `loading="lazy"` so the site works under a project subpath as well as at a domain root.
- `Spencer_AI_Automator_Case_Studies.pdf` is a committed deliverable at the repo root; the current `index.html` does not link to it. `.gitignore` excludes the local `generate*.py` / `build_deck.py` scripts that produced it, so those are not in the repo.
