# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Manifest V3 Chrome extension that injects a dark-theme stylesheet into Synology DS Audio's web UI (an ExtJS 3.x app — expect `.x-*` class selectors everywhere). There is no JavaScript in the extension; the payload is a single compiled CSS file declared as a `content_scripts.css` entry in `src/manifest.json`.

## Commands

- `pnpm i --frozen-lockfile` — install (Node version pinned in `.nvmrc` to v18.7.0)
- `pnpm run start` — full build: cleans `dist/`, compiles `src/theme.scss` → `dist/theme.css` via `sass`, then copies `ds.png` and `manifest.json` into `dist/`. There is no watch script; re-run `pnpm run start` after edits.
- Load `dist/` as an unpacked extension at `about://extensions` to test.

There are no tests, linters, or type checks configured.

## Architecture

`src/theme.scss` is the single entry point. It `@import`s partials in this order — order matters because `_variables.scss` and `_mixins.scss` must be available to every downstream partial:

1. `_variables.scss` — color palette, ExtJS image URL variables (`images/_1x/...` paths served by DS Audio itself), and base64 SVG gradient data URIs
2. `_mixins.scss` — reusable mixins (`linear-gradient`, `box-shadow`, `user-select`, `button-states`, `grid-header-styles`, `panel-background`, `window-header-gradient`, `grid-row-states`, `tree-node-leaf-states`)
3. Feature partials: `_base`, `_windows`, `_controls`, `_grid`, `_miniplayer`, `_search`, `_components`

The feature-partial split came from commit `683db8d` (refactor/split). As of this writing only `_base.scss` and `_windows.scss` contain real rules — the other partials (`_controls`, `_grid`, `_miniplayer`, `_search`, `_components`) are empty stubs waiting for content to be migrated out of `src/theme-backup.scss`, which is the pre-split monolithic stylesheet kept for reference. When adding styles, put them in the appropriate partial rather than growing one file.

## Manifest content_scripts.matches

`src/manifest.json` lists the hosts where the CSS is injected. These are user-specific (currently local Synology IPs and `*solvill.is-by.us`). Uncommitted edits to this file are expected on the working branch (`feat/matching-url` at time of writing) — do not overwrite a user's match patterns unless explicitly asked.

## Debug tip (from README)

DS Audio installs a custom right-click menu that blocks devtools inspection. Paste `document.body = document.body.cloneNode(true)` into the devtools console to strip it.
