# Bookmarks App — Design Spec
**Date:** 2026-04-04

## Overview

A standalone single-file HTML bookmark manager styled after `notes.html`. No build step, no dependencies. Bookmarks are organized into folders, each bookmark has a title, URL, description, and tags. Clicking a bookmark opens it in a new tab. An edit mode allows adding and removing bookmarks and folders.

---

## Architecture

- **Single HTML file** (`bookmarks.html`) — open directly in a browser, no server required.
- **No external dependencies** — no marked.js needed (no markdown rendering).
- **Data persistence:** TiddlyWiki-style — data embedded in a `<script id="tiddlers-data" type="application/json">` tag. On save, the app fetches its own HTML, replaces the script tag content with current JSON, and writes back via the File System Access API (`showSaveFilePicker`). Falls back to `localStorage` when no file handle is available.
- **Load priority:** embedded JSON → localStorage → empty state.
- **4 themes:** `light` (default), `sepia`, `dusk`, `dark` — implemented via CSS custom properties on `html[data-theme="..."]`. Theme saved to `localStorage` key `bookmarks-theme`.

---

## Data Model

Stored in `tiddlers-data` and `localStorage` key `bookmarks-app-data`:

```json
{
  "folders": [
    {
      "id": "abc123",
      "name": "Work",
      "bookmarks": [
        {
          "id": "def456",
          "title": "Example Site",
          "url": "https://example.com",
          "description": "Optional notes about this bookmark",
          "tags": ["research", "tools"],
          "created": "2026-04-04T12:00:00.000Z"
        }
      ]
    }
  ]
}
```

---

## Layout

Two-panel layout matching notes.html: `display: grid; grid-template-columns: 260px 1fr`.

### Sidebar (260px)

Top to bottom:
1. **Header row** — app name ("Bookmarks"), save status indicator, Save button
2. **Theme row** — 4 color-dot buttons for theme switching
3. **Search input** — filters bookmarks across all folders by title, URL, description, and tags; folder list shows match counts
4. **Tag chips panel** — aggregated tags from all bookmarks; clicking a chip filters to bookmarks with that tag across all folders
5. **Folder list** — each folder shows its name and bookmark count; active folder is highlighted; clicking selects it and loads its bookmarks in the main panel
6. **"+ New Folder" button** — prompts inline for a folder name and creates it

### Main Panel

- **Empty state** when no folder is selected: "Select a folder or create a new one"
- **Toolbar** when a folder is selected:
  - Folder name (left)
  - "Edit" toggle button (right) — enters/exits edit mode for this folder
- **Card grid** — responsive CSS grid of bookmark cards, each showing:
  - Title (bold)
  - URL (truncated, muted)
  - Description snippet (up to ~80 chars, muted)
  - Tag chips (small, styled like sidebar chips)
- **Click behavior (normal mode):** clicking anywhere on a card opens the URL in a new tab (`window.open(url, '_blank')`)

---

## Edit Mode

Toggled by the "Edit" button in the main toolbar. State is per-session (not persisted).

When active:
- Each card gains a red × button in the top-right corner; clicking it removes the bookmark (with no confirmation — immediate)
- A dashed "+ Add Bookmark" card appears at the end of the grid
- Clicking the "+ Add Bookmark" card expands an inline form below the grid (not a modal) with fields:
  - Title (required)
  - URL (required)
  - Description (optional)
  - Tags (comma-separated, optional)
  - Confirm / Cancel buttons
- Confirming adds the bookmark to the active folder and re-renders the grid; canceling dismisses the form
- Folder rename: in edit mode, the folder name in the toolbar becomes an editable `<input>`; changes apply on blur or Enter
- Folder delete: a trash icon appears next to the folder name in the toolbar; clicking it removes the folder and all its bookmarks (no confirmation), selects the first remaining folder or shows empty state

---

## Search & Tag Filtering

- **Search** (sidebar input): filters bookmarks across all folders. The folder list shows each folder's match count in parentheses; folders with 0 matches are dimmed. The main panel shows only matching cards for the selected folder.
- **Tag filter** (sidebar chip): clicking a tag shows only bookmarks with that tag across all folders, with folder match counts updated. Clicking the active tag again clears the filter.
- Search and tag filter compose (both active simultaneously narrows results further).

---

## Theming

CSS custom properties, identical palette and variable names to `notes.html`:

| Variable | Purpose |
|---|---|
| `--bg` | Main background |
| `--bg-alt` | Main panel background |
| `--bg-sidebar` | Sidebar background |
| `--bg-ctrl` | Button/input backgrounds |
| `--bg-hover` | Hover states |
| `--bg-code` | (unused, kept for consistency) |
| `--border` | Borders and dividers |
| `--text` | Primary text |
| `--text-muted` | Secondary text |
| `--accent` | Buttons, active states, links |
| `--accent-hover` | Hover on accent |

---

## File Save Flow

Identical to notes.html:
1. On load, `cacheOriginalHtml()` fetches `location.href` with `cache: 'no-store'` to store the original HTML.
2. `buildUpdatedHtml()` finds the `tiddlers-data` script tag in the cached HTML and replaces its content with the current JSON.
3. `saveFile()` calls `showSaveFilePicker` (first save) or reuses the existing `fileHandle`. Falls back to `triggerFallbackDownload()` on browsers without File System Access API support.
4. Save status indicator cycles through: `Saved` / `Unsaved` / `Saving…` / `Error!` / `Downloaded`.
5. `Ctrl+S` triggers save.

---

## Out of Scope

- Bookmark import/export (browser bookmarks format)
- Favicon fetching
- Drag-and-drop reordering
- Nested folders
