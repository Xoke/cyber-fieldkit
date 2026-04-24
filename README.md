# Cyber Fieldkit

A collection of standalone single-file HTML tools for security professionals. No install, no build step, no dependencies — just open in a browser.

## Tools

### [CISSP CPE Tracker](0-CPE%20Tracker.html)
Track Continuing Professional Education hours toward CISSP renewal. Logs activities with domains covered, CPE type (Group A/B), hours, proof files, and markdown notes. Tracks progress against your 3-year cycle goal (120 CPE, ≥40 Group A).

### [Notes](notes.html)
Markdown note-taking app with full-text search, tag filtering, and live preview. Notes are stored inside the file itself — save once via the file picker and subsequent saves update the same file.

### [Cyber Dashboard](cyber-dashboard.html)
Morning briefing dashboard with live feeds for CISA KEV, NVD critical CVEs, Reddit security communities, and CISA advisories. Includes threat intel lookup, CVE search, and Patch Tuesday countdown. Configurable by industry and state for relevant regulatory and news content.

### [Bookmarks](bookmarks.html)
Self-contained bookmark manager with folders, tags, and search. Import directly from a browser bookmark export (HTML format). Data is stored inside the file itself — works the same way as the CPE Tracker and Notes.

### [OSINT Tools](osint-tools.html)
Query builder that opens external OSINT sources pre-filled with your search terms. Covers name, phone, email, and username lookups across 50+ sources. Includes an "Open All" mode with popup blocker detection.

## Usage

Download any file and open it directly in a browser. That's it.

**Data persistence** (CPE Tracker and Notes): On first save, the browser prompts you to choose a save location via the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API). After that, Ctrl+S saves in place. Data is stored inside the HTML file itself — back it up like any other file. Falls back to localStorage if you haven't saved via the file picker.

**Cyber Dashboard and OSINT Tools** store no personal data. Dashboard settings (industry, states, software) are saved to localStorage.

## Updates

Each tool has a **Check for Updates** button that compares your local version against the latest file on GitHub. If a newer version is available, you'll be prompted to download it.

> Note: Downloading an update replaces the file — if you use CPE Tracker or Notes, export or back up your data first, then re-import into the new version.

## Themes

All four tools support Light, Sepia, Dusk, and Dark themes. Theme preference is saved to localStorage per tool.
