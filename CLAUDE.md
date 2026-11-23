# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately minimal to-do app: one file, `index.html`, with inline CSS and JS. No frameworks, no build step, no package manager, no tests, no linter. Keep it that way unless asked otherwise.

Run it by opening `index.html` in a browser (or the Browser pane). `mockup/index.html` is a standalone device-frame mock-up page with base64-embedded screenshots; it is not part of the app and goes stale when the UI changes.

## Architecture (all in `index.html`)

- **State**: a `todos` array of `{ id, text, category, due, done }` (`due` is `YYYY-MM-DD` or `""`), persisted to `localStorage["todos"]`. Theme is stored in `localStorage["theme"]`. All storage access is wrapped in try/catch because it throws in `data:` URL previews and private windows; the app must work without storage.
- **Rendering**: `render()` rebuilds the whole `<ul id="list">` from `todos` on every change, applying the active category `filter` and updating the "N items left" counter (`#count`, in the header, counts only visible items). Always call `save()` then `render()` after mutating `todos`.
- **Events**: a single delegated click handler on `#list` handles edit, delete and checkbox; `keydown` (Enter saves, Escape cancels) and `focusout` handlers implement inline editing. Edit mode is driven by `editingId`; `finishEdit()` reads the `.editing .edit` fields in order (text, category, date), so keep that order if you change the edit UI.
- **Categories**: fixed list `CATEGORIES = ["Work","Personal","Others"]`. `normCat()` maps any stored or typed value onto it case-insensitively (unknown -> "Others") and is applied to loaded data, so legacy data keeps working.
- **Theming**: CSS custom properties on `:root`. Dark values are defined twice: once in `@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])`, and once in `:root[data-theme="dark"]`. Change both when editing the palette. The header icon button toggles `data-theme` on `<html>` and shows the mode you would switch *to* (moon in light, sun in dark).
- **Layout**: the header is a 3-column grid (`1fr auto 1fr`: counter, centred title, theme icon). Single column, `max-width: 480px`, built to work at ~360-393px wide with 16px inputs (avoids iOS zoom) and ~44px tap targets.

## Gotchas

- Todo ids are `Date.now() + Math.random()` (same-millisecond adds otherwise collide). The click handler reads ids back via `Number(li.dataset.id)`.
- Text is inserted with `textContent` via the `el()` helper, never `innerHTML` (only trusted SVG strings use `innerHTML`). Keep user text out of `innerHTML`.
- Overdue is a string comparison of `due` against the local-date `todayStr()`.
- Icons are inline SVG with `pointer-events: none` so clicks land on the button; the click handler distinguishes buttons by `className`/`tagName`.
