# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

FitSanitario (brand: **HealthTribes**) — a Spanish-language marketing/prototype site for a team fitness gamification app aimed at healthcare workers. There is **no build system, no dependencies, no tests, no package manager**. The project is a handful of self-contained HTML files deployed via **GitHub Pages**.

- [index.html](index.html) — landing page; links to the deliverables below.
- [app.html](app.html) — **"ideal" prototype**: the full-vision navigable app prototype (~1500 lines), hand-authored. 5 tabs: perfil, ejercicio, cámara, logros, social.
- [app-mvp.html](app-mvp.html) — **MVP prototype**: a pared-down 3-tab version (perfil, ejercicio, cámara). See note below — this is a *generated* bundle, not hand-authored.
- [deck.html](deck.html) — cinematic 8-slide commercial presentation (~970 lines).

`app.html` and `app-mvp.html` are independent deliverables shown side by side on the landing page (ideal vs. minimum viable); they do not share code.

All copy and identifiers are in Spanish — keep new strings in Spanish.

### app-mvp.html is a generated bundle — do not hand-edit its app logic

Unlike the other files, `app-mvp.html` is a **self-contained "bundler" export**, not authored in the inline style. A small loader script at the top decodes base64 assets from `<script type="__bundler/manifest">` (some are gzip-compressed, inflated via `DecompressionStream`) into blob URLs, then rebuilds the real document at runtime from the JSON string in `<script type="__bundler/template">`. The actual app code lives in two **minified JS blobs** inside the manifest, so editing the prototype's behavior by hand is impractical — treat this file as a build artifact and regenerate/re-export it from source rather than patching it. The newest export currently lives in `~/Downloads/FitSanitario MVP (standalone)*.html`. It also carries an edit-mode "Tweaks" panel that talks to a host via `postMessage` (inert when opened standalone).

## Running / previewing

Open the files directly in a browser, or serve locally so relative asset paths and `localStorage` behave like production:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

There is nothing to build, lint, or test. Verifying a change means opening the file in a browser and exercising the affected screen/slide.

## Architecture

**Self-contained files.** Each HTML file inlines its own CSS (`<style>`) and JS (`<script>`) — no shared stylesheet or script, no bundler. Images, avatars, and fonts are embedded as `data:` URIs (this is why the files are large; `deck.html` is ~700 KB). The only external resource is Google Fonts. There is **no framework** (no Vue/React despite incidental substring matches) — everything is vanilla JS.

**Design tokens** are duplicated per file as CSS custom properties on `:root` (e.g. `--primary:#0A6E63`, the teal accent). Changing the palette means editing each file's `:root` block; they are not shared.

### app.html — state-driven single-page prototype

A hand-rolled render loop over one global state object:

- `var S = load()` — the single source of truth. `load()`/`save()` persist `S` to `localStorage` under the key **`fitsani`**, with inline migration logic (`defaults()`, `PHOTOMAP`, `migAv`) that backfills older saved shapes. To reset the prototype, clear that key.
- `render()` rewrites `document.getElementById('app').innerHTML` from scratch on every change — there is no diffing. It draws header + scrollable body + tab bar (`chrome()`), then re-wires event handlers each render.
- **Screen routing** is `S.screen` (a string). Screens are functions in the `SCREENS` map; per-screen event wiring lives in the `WIRE` map. The tab bar is defined by the `TABS` array: `perfil`, `ejercicio`, `camara`, `logros`, `social` — plus the special `onboarding` screen rendered without chrome.
- Navigate with `go(screenName)`; helpers include `$()` (querySelector wrapper), `toast()`, and feed mutators (`prependFeed`, `refreshPerfilFeed`). Avatars are hydrated post-render by `hydrateAvatars()`.

When adding a screen: add a `SCREENS[name]` function, optionally a `WIRE[name]` handler, and an entry in `TABS` if it needs a tab.

### deck.html — commercial deck

A standalone slide deck (8 slides) with its own embedded animation/scroll logic and inlined assets. Independent from `app.html` — they share branding by convention only, not code.

## Conventions

- This is a brief-driven deliverable: it was asked to be **single-file HTML prototypes**. Do not refactor into a component library, framework, or multi-file structure without explicit approval — match the existing inline, vanilla-JS, self-contained style.
- Keep each file self-contained: prefer inlining new assets as `data:` URIs over adding external files.
