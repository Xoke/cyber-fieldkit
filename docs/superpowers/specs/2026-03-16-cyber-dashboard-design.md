# Cyber Dashboard Redesign — Design Spec
**Date:** 2026-03-16
**Status:** Approved

---

## Overview

Redesign `cyber-dashboard.html` from a static-link-heavy grid into a live "morning pulse check" briefing tool. The user opens it each morning to quickly assess what's happening in their industry and jurisdiction — active threats, breaking news, community chatter, and applicable regulations — all in one view.

---

## Layout & Navigation

Replace the current flat panel grid with a **left sidebar + main content area** structure.

### Sidebar (always visible, ~160px wide)
- App name / logo at top
- Nav items (icon + label):
  - 📰 **Briefing** (default active tab)
  - 🔗 **Resources**
  - 🔍 **Intel**
- Bottom of sidebar: ⚙ Settings button + theme colour pickers

### Header bar
Remove the current sticky top header. Replace with a slim context line inside the Briefing content area: `"Morning Briefing · [Industry] · [States]"` + Refresh All button.

---

## Briefing Tab (default)

The primary view. All panels load live data on page load and on "Refresh All". Two-column grid, regulations full-width at bottom.

### Panel 1 — CISA KEV *(unchanged logic, keep as-is)*
- 12 most recent entries, sorted by date added
- Software match highlighting (⚡ prefix + yellow border)
- Due-date colouring (overdue = red, ≤7 days = orange)

### Panel 2 — Critical CVEs / NVD *(unchanged logic, keep as-is)*
- NVD API, CVSS ≥ 9.0 in last 30 days
- Software match highlighting

### Panel 3 — Security News (RSS)
- Fetch RSS from 3–4 industry-specific feeds via `https://api.rss2json.com/v1/api.json?rss_url=ENCODED_URL`
- No API key required for light use (free tier, ~1 req/10s per IP)
- Show up to 12 stories: headline, 1-sentence excerpt, source name, age
- Filter/highlight stories matching industry `kwPattern`
- Sort by date descending
- **Fallback**: if rss2json request fails or returns error, silently render a link-tile grid using the industry's `news` link data from `PROFILE_DATA.industries[industry].news` (the same data that populates the Resources tab). The fallback reads from the same in-memory data structure — no duplication needed. No error message is shown to the user.
- RSS sources per industry (exact feed URLs to hardcode):
  - **Banking**:
    - `https://www.bleepingcomputer.com/feed/` (BleepingComputer)
    - `https://krebsonsecurity.com/feed/` (Krebs on Security)
    - `https://www.bankinfosecurity.com/rss/news` (BankInfoSecurity)
    - `https://www.darkreading.com/rss/all.xml` (Dark Reading)
  - **Healthcare**:
    - `https://www.healthcareinfosecurity.com/rss/news` (HealthcareInfoSecurity)
    - `https://www.hipaajournal.com/feed/` (HIPAA Journal)
    - `https://www.bleepingcomputer.com/feed/` (BleepingComputer)
    - `https://krebsonsecurity.com/feed/` (Krebs on Security)
  - **Retail**:
    - `https://www.scmagazine.com/feed/` (SC Magazine)
    - `https://www.bleepingcomputer.com/feed/` (BleepingComputer)
    - `https://krebsonsecurity.com/feed/` (Krebs on Security)
    - `https://www.darkreading.com/rss/all.xml` (Dark Reading)
  - **Technology**:
    - `https://feeds.feedburner.com/TheHackersNews` (The Hacker News)
    - `https://www.bleepingcomputer.com/feed/` (BleepingComputer)
    - `https://krebsonsecurity.com/feed/` (Krebs on Security)
    - `https://isc.sans.edu/rssfeed.xml` (SANS ISC)
  - **Legal**:
    - `https://www.darkreading.com/rss/all.xml` (Dark Reading)
    - `https://www.bleepingcomputer.com/feed/` (BleepingComputer)
    - `https://krebsonsecurity.com/feed/` (Krebs on Security)
    - `https://www.law.com/legaltechnews/feed/` (Law.com Legal Technology)
  - **Government**:
    - `https://fedscoop.com/feed/` (FedScoop)
    - `https://statescoop.com/feed/` (StateScoop)
    - `https://www.bleepingcomputer.com/feed/` (BleepingComputer)
    - `https://krebsonsecurity.com/feed/` (Krebs on Security)

### Panel 4 — Community Pulse (Reddit)
- Replace current "latest posts from subreddit" with Reddit search API:
  `https://www.reddit.com/search.json?q=QUERY&sort=top&t=week&limit=25`
- Make **two separate API calls**, then merge and re-sort by score:
  1. Subreddit-scoped: `q=(subreddit:netsec OR subreddit:cybersecurity)&sort=top&t=week&limit=15`
  2. Industry keyword search: `q=INDUSTRY_KEYWORDS&sort=top&t=week&limit=15` using the pre-defined keyword strings below (do NOT attempt to derive these from `kwPattern` regex at runtime)
- Pre-defined `INDUSTRY_KEYWORDS` strings (hardcoded per industry key):
  - `banking`: `"bank fraud ransomware phishing breach payment wire swift GLBA"`
  - `healthcare`: `"healthcare HIPAA ransomware patient breach medical PHI EHR"`
  - `retail`: `"retail ecommerce PCI payment skimming magecart breach card"`
  - `technology`: `"cloud vulnerability zero-day exploit ransomware API supply chain"`
  - `legal`: `"law firm ransomware breach attorney privilege client data"`
  - `government`: `"government CISA federal ransomware nation-state breach critical infrastructure FISMA"`
- Deduplicate by post URL, merge arrays, sort by score descending, take top 10
- Show each post: upvote score, title (linked to post), subreddit badge, age
- Industry-relevant posts (matching `kwPattern`) get blue left-border highlight (existing `banking-relevant` CSS class, rename to `industry-relevant`)
- **On fetch failure** (CORS error or network error): show standard error state with a link to r/netsec — do not collapse the panel
- Panel title: "Community Pulse"

### Panel 5 — Regulations (full width)
- Unchanged logic: renders selected states' breach notification laws + industry's federal regs
- Title updates dynamically: `"Regulations · CA / NY + Federal Banking"`

---

## Resources Tab

Static reference content moved here from the main page:
- Industry news link tiles (the current "Banking Security News" link grid)
- Vendor/patch advisory links (current vendor links panel)
- Patch Tuesday countdown panel

No live API calls in this tab. Renders instantly.

---

## Intel Tab

Unchanged from current implementation:
- Threat Intel lookup (IP / domain / hash → external services)
- CVE Lookup (NVD API search by CVE ID)

---

## Settings

### Drawer (unchanged UX, expanded content)
- Industry dropdown — 6 options (unchanged)
- States checkboxes — expanded from 8 to **12 states**:
  - Keep: CA, NY, TX, FL, CO, WA, IL, NV
  - Add: VA (VCDPA), CT (CTDPA), OR (OCPA), GA (breach notification)
- Software/systems chip input — unchanged
- Save & Apply button triggers re-render of Briefing tab panels

### State data additions
New entries required in `PROFILE_DATA.states`:

| Abbr | Law | Key detail |
|------|-----|-----------|
| VA | Virginia CDPA (VCDPA) | Consumer rights + data protection obligations; eff. Jan 1 2023 |
| CT | Connecticut TDPA (CTDPA) | Opt-out rights; eff. July 1 2023 |
| OR | Oregon Consumer Privacy Act (OCPA) | eff. July 1 2024 |
| GA | Georgia Computer Systems Protection Act | Breach notification; no specific timeline |

---

## First-Run Behaviour

1. On page load, check `localStorage` for `cyber-dash-settings`
2. If key is absent (first run):
   - Display a dismissible yellow banner at the top of the Briefing content area:
     *"Welcome — select your industry and states to personalise this briefing."*
   - Auto-open the Settings drawer
   - Pre-select defaults: Banking / CA + NV
3. Banner is stored as seen via a separate `localStorage` key (`cyber-dash-welcomed`); set to `"1"` immediately when Save & Apply is clicked (before the drawer closes)
4. Banner and drawer both dismiss synchronously on Save & Apply
5. On subsequent loads the banner does not appear

---

## Data Flow

```
Page load
  ├── loadSettings() → localStorage
  ├── if no settings → show banner + open drawer
  ├── applyTheme()
  ├── renderSidebar() → set active tab = Briefing
  ├── renderBriefingTab()
  │     ├── loadKEV()          → CISA → NVD fallback
  │     ├── loadNVD()          → NVD API
  │     ├── loadNewsRSS()      → rss2json.com → link-tile fallback
  │     ├── loadReddit()       → Reddit search API (industry keywords)
  │     └── renderRegulatory() → PROFILE_DATA (synchronous)
  └── renderResourcesTab()     → PROFILE_DATA (synchronous, not visible)

"Refresh All" button → re-runs loadKEV, loadNVD, loadNewsRSS, loadReddit
Settings "Save & Apply" → saveSettings() → re-runs all Briefing renders
```

---

## What Is Not Changing

- Single-file HTML architecture (no build step, no npm)
- Data persistence model (localStorage only; no File System Access API — this file has no embedded data to save)
- Theme system (4 themes via CSS custom properties)
- marked.js dependency (not used in this file — N/A)
- CORS constraint handling (only `Access-Control-Allow-Origin: *` APIs used)
- KEV and NVD panel logic
- Threat Intel and CVE lookup tools
- Software chip matching logic

---

## Out of Scope

- Pulling full article body text (not feasible from `file://` without a backend)
- Notifications or background refresh
- Additional industries or states beyond the 12 listed
- Cross-site trending score calculation (Reddit upvotes serve as proxy)
