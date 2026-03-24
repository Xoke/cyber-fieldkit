# cyber-fieldkit

Four standalone, single-file HTML tools for security practitioners. No server, no build step, no dependencies — open directly in a browser.

## Tools

### OSINT Lookup (`osint-tools.html`)
Query builder that opens external OSINT sites with pre-filled searches. Covers names, phone numbers, emails, and usernames across dozens of sources. Includes an **Open All** function for bulk searches. Based entirely on [Michael Bazzell's IntelTechniques Search Tools](https://inteltechniques.com/tools/index.html).

### Cybersecurity Dashboard (`cyber-dashboard.html`)
Live threat intelligence dashboard pulling from CISA KEV, NVD, and Reddit (r/netsec, r/cybersecurity). Configurable by industry and state — highlights relevant KEVs and news based on your profile. Data fetched fresh on load.

### CISSP CPE Tracker (`0-CPE Tracker.html`)
Tracks Continuing Professional Education hours for CISSP recertification. Logs activities with domain coverage, CPE type (Group A/B), proof attachments, and markdown summaries. Shows progress toward the 120-hour / 3-year cycle requirement.

### Notes (`notes.html`)
Minimal markdown notes app with tagging and full-text search. Edit in plain text, preview rendered markdown. Tags shown in sidebar for quick filtering.

## Usage

Download the file(s) you want and open them in any modern browser. That's it.

**Saving data:** The CPE Tracker and Notes app store data inside the HTML file itself using the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API). On first save you'll be prompted to choose a location — save over the same file to persist your data. localStorage is used as a fallback between saves.

**Themes:** All tools support light, sepia, dusk, and dark themes via a toggle in the UI.

## Compatibility

Tested in Chrome and Edge. Firefox has limited support for the File System Access API — saving will fall back to localStorage only.

## License

[Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/)

You are free to share and adapt these tools for any purpose, including commercially, as long as you give appropriate credit and distribute any derivatives under the same license.
