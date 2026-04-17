# Cyber Dashboard — High-Value Additions Design

**Date:** 2026-04-17  
**File:** `cyber-dashboard.html`  
**Status:** Approved

## Overview

Add five high-value features to the existing single-file cybersecurity dashboard:

1. EPSS score overlay on KEV and NVD panels
2. CISA Advisories panel on Briefing tab
3. New "Threats" tab with Abuse.ch IOCs and Ransomware activity panels
4. Shodan InternetDB inline result in Threat Intel Lookup

Layout decision: CISA Advisories → Briefing tab. Abuse.ch + Ransomware → new "Threats" tab.

---

## Feature 1: EPSS Score Overlay

**Where:** Enhances existing `renderKEVList()` and `loadNVD()` — no new panel.

**API:** `https://api.first.org/data/v1/epss?cve=CVE-1,CVE-2,...` (GET, CORS-enabled)

**Response shape:**
```json
{ "data": [{ "cve": "CVE-2024-1234", "epss": "0.94321", "percentile": "0.99876", "date": "..." }] }
```

**Implementation:**
- `renderKEVList()` and the NVD render block both add `data-cve="CVE-XXXX"` attributes to each `<li>`
- After rendering, call `loadEPSS(cveIds[])` — a separate async function that fires one batch fetch
- On response, iterate results and inject an EPSS badge into the meta row (the flex div containing CVSS score and time) of matching `<li>` elements via `querySelector('[data-cve="..."] .epss-slot')`; each `<li>` includes an empty `<span class="epss-slot"></span>` in its meta row for this purpose
- EPSS badge: `<span class="epss-badge epss-{low|med|high}">EPSS X%</span>` where high ≥ 50%, med ≥ 10%, low < 10%
- If EPSS fetch fails, silently skip — existing panel content unaffected

**CSS:** Three EPSS badge variants using existing color variables: `--low` / `--medium` / `--critical`.

---

## Feature 2: CISA Advisories Panel

**Where:** Briefing tab, half-width panel added after the existing 4 panels (before Regulatory).

**API:** rss2json proxy → `https://www.cisa.gov/cybersecurity-advisories/all.xml`  
Uses existing `RSS2JSON_BASE` constant — same pattern as `loadNewsRSS()`.

**Panel HTML:** `id="panel-cisa"`, body `id="cisa-body"`, title "CISA Advisories", refresh button calls `loadCISAAdvisories()`.

**Rendering:**
- Up to 10 items using `feed-item` / `feed-item-title` / `feed-item-meta` CSS classes
- Title prefix detection: if title starts with "AA-", "ICS-CERT", "ICSA-" → extract and display as a type tag in meta row
- Industry keyword highlighting: apply `industry-relevant` class when `getIndustryKW()` matches title
- Time ago from pubDate
- `setLoading` / `setError` pattern with fallback link to `https://www.cisa.gov/news-events/cybersecurity-advisories`

**Init:** Called in `refreshAll()` and `applySettings()`.

---

## Feature 3: New "Threats" Tab

**Nav:** Add `<button class="nav-item" data-tab="threats" onclick="switchTab('threats')">Threats</button>` to the nav bar.

**HTML:** New `<div id="tab-threats" class="tab-panel">` with a `dashboard-grid` containing two half-width panels.

### Panel A: Abuse.ch Active Threats

**Panel:** `id="panel-abuse"`, body `id="abuse-body"`, title "Active Threats (Abuse.ch)", refresh calls `loadAbuse()`.

**Sources (parallel fetch):**

1. **Feodo C2 Blocklist** — `GET https://feodotracker.abuse.ch/downloads/ipblocklist.json`  
   Response: array of `{ ip_address, port, status, malware, country, ... }`  
   Show: top 15 by last_seen, filter to `status === "online"` first

2. **ThreatFox Recent IOCs** — `POST https://threatfox-api.abuse.ch/api/v1/`  
   Body: `{"query":"get_iocs","days":1}`  
   Response: `{ data: [{ id, ioc, ioc_type, malware, malware_printable, confidence_level, ... }] }`  
   Show: top 15 by confidence_level

**Rendering:** Unified feed list. Each entry shows:
- Malware family name (bold)
- IOC value (IP:port, domain, or URL) — truncated at 60 chars
- Meta: source (Feodo/ThreatFox), country (Feodo only), status badge for online C2s
- Software chip match: if malware name matches a chip → `⚡ sw-match` class

Both fetch in parallel via `Promise.allSettled`. Wait for both to settle, then combine results from whichever succeeded and render once. If both fail, show standard error state.

### Panel B: Ransomware Activity

**Panel:** `id="panel-ransomware"`, body `id="ransomware-body"`, title "Ransomware Activity", refresh calls `loadRansomware()`.

**API:** `GET https://api.ransomware.live/recentvictims`  
Response: array of `{ post_title, group_name, discovered, country, activity, ... }`

**Rendering:**
- Up to 15 most recent victims (sort by `discovered` desc)
- Each item: victim name (bold), group name, country, date
- Industry keyword match on `activity` field → `industry-relevant` class
- On CORS/network failure: standard `setError` with link to `https://www.ransomware.live`

**Init:** `loadAbuse()` and `loadRansomware()` called lazily on first visit to the Threats tab. `switchTab()` is modified to detect `name === 'threats'` and call both loaders if a `_threatsLoaded` flag is false, then set the flag. NOT included in `refreshAll()` to avoid background network calls while user is on Briefing tab. Refresh buttons on each panel trigger their loader directly and reset the flag.

---

## Feature 4: Shodan InternetDB in Threat Intel Lookup

**Where:** Intel tab, existing Threat Intel Lookup panel (`id="panel-ti"`).

**Trigger:** When `tiDetect()` identifies type as `ip`, additionally call `loadInternetDB(ip)` after rendering service buttons.

**API:** `GET https://internetdb.shodan.io/{ip}` (no auth, CORS-enabled)  
Response: `{ ip, ports, hostnames, cpes, tags, vulns }` or `{ detail: "No information available" }` (404)

**Rendering:** A `<div id="idb-result">` block injected below `#ti-services`:
- Ports: comma-separated list
- Hostnames: comma-separated
- Tags: each as a small badge (e.g. `honeypot`, `self-signed`)
- Vulns: CVE IDs as clickable links to `https://nvd.nist.gov/vuln/detail/{cve}`
- CPEs: comma-separated, truncated

**Edge cases:**
- 404 / `detail` field present → hide `#idb-result` entirely (no data is normal for private/clean IPs)
- Network/CORS failure → silently suppress, service buttons unaffected
- Non-IP type selected → clear/hide `#idb-result`
- `#idb-result` cleared on each new `tiDetect()` call before fetching

---

## Unchanged

- All existing panels, data sources, and CSS classes remain as-is
- Settings (industry, states, software chips) apply to all new panels via existing `getIndustryKW()` and `softwareMatches()` helpers
- `escHtml()`, `timeAgo()`, `truncate()` helpers reused throughout
- `refreshAll()` gains `loadCISAAdvisories()` but NOT the Threats tab loaders (lazy)
- `applySettings()` gains `loadCISAAdvisories()`
