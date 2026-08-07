# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static, single-page "digital typewriter" webpage that renders a printable receipt-style daily summary (calendar events + to-do list). No build tooling, no package manager, no framework — plain `index.html` + `script.js` + `styles.css`. Per the README, the intended end state is for the receipt content to be synced automatically each day from calendars and Notion via an external AI agent; the data in `index.html` today is placeholder/manually-edited markup, not fetched at runtime.

Tutorial reference (context only, not something to fetch or rely on for code behavior): https://www.notion.so/Digital-Typewriter-Tutorial-35b988f3a140807e9d1ae5329f42f6b8

## Running it

There is no build/lint/test tooling in this repo (no `package.json`). To preview:
- Open `index.html` directly in a browser, or
- Serve the directory locally, e.g. `python3 -m http.server` from the repo root, then visit `localhost:8000`.

Changes to `index.html`, `styles.css`, or `script.js` are visible on a page refresh — no compilation step.

## Architecture

**Single page, three files, state driven by CSS classes on `#stage`.**

- `index.html` — static markup for the receipt. The `.events` and `.todos` `<ul>` lists are hand-authored placeholder entries (`Work Event 1`, `Personal Task 1`, etc.) meant to be replaced by synced data; when editing sample content, follow the existing per-`<li>` structure (`event-icon`/`todo-icon` span with `event-work`/`event-personal`/`todo-work`/`todo-personal` class, a `.label` span, optionally a `.time` span or a link).
- `styles.css` — all animation/layout timing is centralized in `:root` custom properties (`--print-duration`, `--move-duration`, `--zoom`, `--slot-y`, `--receipt-w`, etc.). The print sequence is implemented as CSS transitions gated by two state classes toggled on `#stage`: `.is-printing` (receipt slides out of the printer) and `.is-printing.is-done` (receipt settles into final resting position, printer slides away). Adjust animation feel by tuning these custom properties rather than editing individual transition rules.
- `script.js` — vanilla DOM code, no modules/bundler, organized as sequential top-level sections (each preceded by a `// --- Section ---` comment):
  1. Receipt date stamp.
  2. Typewriter heading effect (`typeString`/`deleteChars`/`loopWord` — types/deletes text into the `<h1>` around a blinking caret span).
  3. Printer idle/printing status text, cycled via `setInterval` frame arrays (`IDLE_FRAMES`, `PRINTING_FRAMES`); idle frames adapt based on `.events li` count.
  4. To-do click handling: toggles `.done` and spawns a radial burst of sparkle spans (`spawnSparkles`) positioned via inline styles and CSS custom properties (`--dx`/`--dy`) consumed by a keyframe animation.
  5. Receipt click → wiggle animation (`wiggleReceipt`), re-triggered by removing/re-adding the `.is-wiggle` class (forces reflow via `receipt.offsetWidth`).
  6. Print button flow: adds `.is-printing`/`.is-done` to `#stage` in sequence, chained via `transitionend` listeners keyed on `e.propertyName === 'transform'` (not `setTimeout`), so this logic is coupled to the transition durations/properties defined in `styles.css`.
  7. Background sparkle field: generates fixed-count decorative `<span>` elements with randomized position/timing via inline styles + CSS custom properties.
  8. Drifting decor images: a small custom physics loop (`requestAnimationFrame`-driven `tickDecors`) animating floating PNGs from `assets/background-decors/`. Two behavior modes per `DECOR_IMAGES` entry: `wander: true` items drift and wrap around viewport edges with no collision; non-wander items bounce off viewport edges and each other (`resolveCollision` does simple elastic circle-circle collision) and get an outward "kick" velocity on click (`KICK_SPEED`) with a squash/stretch bounce scale. When adding new decor images, add an entry to `DECOR_IMAGES` with `src`/`size` (and `wander` if it shouldn't collide).

There are no JS modules, no imports/exports, and no external JS dependencies — everything in `script.js` runs top-to-bottom on page load via the `defer` attribute on the `<script>` tag.

## Assets

`assets/` holds PNGs referenced directly by `index.html`/`styles.css`/`script.js` (printer, button, event/todo icon labels, heart, favicon) plus `assets/background-decors/` for the floating decor images enumerated in `script.js`'s `DECOR_IMAGES`. Fonts are loaded from Google Fonts (Pixelify Sans) via `<link>` tags in `index.html`, not self-hosted.
