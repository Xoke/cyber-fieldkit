# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This directory contains four standalone single-file HTML applications — no build system, no package manager, no dependencies to install. Open directly in a browser.

- **`0-CPE Tracker.html`** — CISSP CPE (Continuing Professional Education) activity tracker
- **`notes.html`** — Markdown notes app with tagging and search
- **`cyber-dashboard.html`** — Cybersecurity awareness dashboard with live data feeds and settings
- **`osint-tools.html`** — OSINT lookup tool that opens external search sites with pre-filled queries

## Architecture

Both files share the same self-contained architecture pattern:

### Data Persistence (TiddlyWiki-style)
Data is stored inside the HTML file itself within a `<script id="tiddlers-data" type="application/json">` tag. On save, the app fetches its own file (`fetch(location.href)`), replaces the content of that script tag with the current JSON state, and writes the result back using the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API) (`showSaveFilePicker` / `FileSystemWritableFileStream`). localStorage is used as a fallback when the file hasn't been explicitly saved via the file picker.

Load priority: embedded JSON in `tiddlers-data` → localStorage → empty state.

### Structure of Each File
```
<head>
  <script>  ← marked.js v15.0.12 (minified, inlined — do not edit)
  <style>   ← all CSS with CSS custom properties for 4 themes
</head>
<body>
  HTML layout
  <script id="tiddlers-data" type="application/json">{...}</script>
  <script>  ← all application JavaScript
</body>
```

### Theming
Four themes implemented via CSS custom properties on `html[data-theme="..."]`: `light` (default), `sepia`, `dusk`, `dark`. Theme preference saved to localStorage.

### CPE Tracker specifics
- Entries have: title, date, CPE hours, type (Group A / Group B), CISSP domains covered, summary (markdown), proof file references, transcript checkbox
- CISSP requires 120 CPE total (≥40 Group A) per 3-year cycle
- Cycle dates are configurable; progress shown in sidebar

### Notes app specifics
- Notes have: title, body (markdown, edit/preview toggle), tags
- Sidebar has full-text search and tag filtering
- Editor uses monospace font; viewer renders markdown via marked.js

### cyber-dashboard.html specifics
- **No self-save** — no embedded data, no File System Access API. State stored only in localStorage.
- **Live data panels** (fetch on load + Refresh All button): CISA KEV, NVD Critical CVEs, Reddit (r/netsec + r/cybersecurity)
- **CORS constraint**: Only APIs with `Access-Control-Allow-Origin: *` work from `file://`. CISA KEV JSON lacks this header; `loadKEV()` tries CISA first, then falls back to NVD's `cisaExploitAdd` fields. NVD requires **both** `pubStartDate` and `pubEndDate` when either is used — omitting one returns HTTP 404.
- **Settings drawer** (⚙ button): industry dropdown + state checkboxes + software chip list, all saved to `localStorage` key `cyber-dash-settings`. Drives: news panel links, regulatory panel content, Reddit keyword highlights (`banking-relevant` CSS class), and KEV/NVD software match highlights (`sw-match` CSS class, ⚡ prefix).
- **PROFILE_DATA** JS object holds all industry configs (keywords regex, news links, federal regs) and state breach law summaries. Default: banking / CA+NV.
- Theme saved to `localStorage` key `cyber-dash-theme`; default is `dark`.

### osint-tools.html specifics
- **No data persistence** — purely a query-builder that opens external sites in new tabs.
- Four tabs: Names, Phone, Email, Username. Each tab has input fields; search buttons open the target site with the query pre-filled in the URL.
- All search targets defined as JS data structures with `url` functions — no hardcoded `<a>` tags.
- **Open All** does a popup-blocker test first (`window.open('about:blank','_blank')`); if blocked, shows a step-by-step alert instead of silently failing.
- Names tab uses a `STATE_NAMES` lookup object (abbrev → full name) because some sites (Spokeo, Intelius, ZabaSearch) require full state names in URL paths. URL patterns differ per site: some use query params, some use path segments, some need slugified (hyphenated lowercase) values.
- Social Media category has `note: '* login required'` rendered in the category header for TikTok and Bluesky.
- Theme saved to `localStorage` key `osint-theme`.

## Development

No build step. Edit the HTML files directly, then refresh the browser. The minified marked.js block at the top of each file should not be modified — it's a vendored dependency.

To test changes, open the file in a browser. Data changes are only persisted if you use the "Save" button (which triggers the File System Access API picker on first save).
