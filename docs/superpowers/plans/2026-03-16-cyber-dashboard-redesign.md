# Cyber Dashboard Redesign Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign `cyber-dashboard.html` from a static-link grid into a morning pulse-check briefing tool with sidebar nav, live RSS news stories, improved Reddit filtering, expanded state coverage, and first-run onboarding.

**Architecture:** Replace the flat panel grid with a sidebar-nav + tabbed main content area. The Briefing tab holds all live-data panels (KEV, CVEs, RSS news, Reddit, Regulations). Resources tab holds static link panels. Intel tab holds lookup tools. All changes are in a single HTML file — no build step.

**Tech Stack:** Vanilla HTML/CSS/JS, localStorage, Reddit JSON API (CORS-native), rss2json.com free tier (CORS proxy for RSS), NVD API, CISA KEV JSON.

**Spec:** `docs/superpowers/specs/2026-03-16-cyber-dashboard-design.md`

---

## File Map

| File | Action | What changes |
|------|--------|-------------|
| `cyber-dashboard.html` | Modify | All changes — CSS, HTML structure, JS logic |

---

## Task 1: App Shell — Sidebar + Tab CSS and HTML Skeleton

Replace the existing `#header` + `#content` flat layout with a sidebar-nav shell and three tab panels.

**Files:**
- Modify: `cyber-dashboard.html` (CSS ~line 8–317, HTML ~line 320–332)

- [ ] **Step 1: Add app shell CSS**

  In `<style>`, remove the `/* Header */` and `/* Grid */` blocks (lines ~43–82) and replace with:

  ```css
  /* App Shell */
  .app-shell { display: flex; height: 100vh; overflow: hidden; }

  /* Sidebar */
  .sidebar {
    width: 160px; flex-shrink: 0;
    background: var(--bg-alt); border-right: 1px solid var(--border);
    display: flex; flex-direction: column; overflow: hidden;
  }
  .sidebar-brand {
    padding: 14px 14px 12px; font-size: 14px; font-weight: 700;
    border-bottom: 1px solid var(--border); flex-shrink: 0;
  }
  .sidebar-nav { padding: 8px 6px; display: flex; flex-direction: column; gap: 2px; flex: 1; }
  .nav-item {
    display: flex; align-items: center; gap: 8px;
    padding: 8px 10px; background: none; border: none; border-radius: 5px;
    color: var(--text-muted); font-size: 13px; font-family: inherit;
    cursor: pointer; text-align: left; width: 100%; transition: background 0.12s, color 0.12s;
  }
  .nav-item:hover { background: var(--bg-hover); color: var(--text); }
  .nav-item.active { background: var(--accent-light); color: var(--accent); font-weight: 600; }
  .nav-icon { font-size: 15px; flex-shrink: 0; }
  .sidebar-bottom { padding: 8px 6px 12px; border-top: 1px solid var(--border); flex-shrink: 0; }
  .theme-row { display: flex; gap: 6px; padding: 6px 10px; }

  /* Main content */
  .main-content { flex: 1; display: flex; flex-direction: column; overflow: hidden; }

  /* Tab panels */
  .tab-panel { display: none; flex: 1; overflow-y: auto; padding: 16px 20px; }
  .tab-panel.active { display: block; }

  /* Dashboard grid (inside tabs) */
  .dashboard-grid {
    display: grid; grid-template-columns: 1fr 1fr; gap: 16px;
  }
  .col-span-2 { grid-column: 1 / -1; }
  @media (max-width: 860px) {
    .app-shell { flex-direction: column; }
    .sidebar { width: 100%; height: auto; flex-direction: row; flex-wrap: wrap; }
    .sidebar-nav { flex-direction: row; flex: none; }
    .sidebar-bottom { margin-left: auto; }
    .dashboard-grid { grid-template-columns: 1fr; }
    .col-span-2 { grid-column: 1; }
  }
  ```

- [ ] **Step 2: Replace HTML shell**

  Replace the entire `<div id="header">...</div>` and opening `<div id="content">` (lines ~321–333) with:

  ```html
  <div class="app-shell">
    <nav class="sidebar">
      <div class="sidebar-brand">⚡ Cyber Dash</div>
      <div class="sidebar-nav">
        <button class="nav-item active" data-tab="briefing" onclick="switchTab('briefing')">
          <span class="nav-icon">📰</span><span class="nav-label">Briefing</span>
        </button>
        <button class="nav-item" data-tab="resources" onclick="switchTab('resources')">
          <span class="nav-icon">🔗</span><span class="nav-label">Resources</span>
        </button>
        <button class="nav-item" data-tab="intel" onclick="switchTab('intel')">
          <span class="nav-icon">🔍</span><span class="nav-label">Intel</span>
        </button>
      </div>
      <div class="sidebar-bottom">
        <button class="nav-item" onclick="openSettings()">
          <span class="nav-icon">⚙</span><span class="nav-label">Settings</span>
        </button>
        <div class="theme-row">
          <button class="theme-btn" data-t="light" title="Light"></button>
          <button class="theme-btn" data-t="sepia" title="Sepia"></button>
          <button class="theme-btn" data-t="dusk" title="Dusk"></button>
          <button class="theme-btn" data-t="dark" title="Dark"></button>
        </div>
      </div>
    </nav>
    <main class="main-content">
      <div id="tab-briefing" class="tab-panel active"></div>
      <div id="tab-resources" class="tab-panel"></div>
      <div id="tab-intel" class="tab-panel"></div>
    </main>
  </div>
  ```

  Also remove the old closing `</div><!-- #content -->` and `</div><!-- .dashboard-grid -->` at the bottom of the body (they will be replaced in Task 2).

- [ ] **Step 3: Add switchTab JS function**

  Add to the `<script>` block (before the `// ─── Theme` section):

  ```javascript
  function switchTab(name) {
    document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
    document.querySelectorAll('.nav-item[data-tab]').forEach(b => b.classList.remove('active'));
    document.getElementById('tab-' + name).classList.add('active');
    const btn = document.querySelector('.nav-item[data-tab="' + name + '"]');
    if (btn) btn.classList.add('active');
  }
  ```

- [ ] **Step 4: Verify in browser**

  Open `cyber-dashboard.html` in a browser. Confirm:
  - Sidebar visible on left with three nav items (Briefing, Resources, Intel) + Settings + theme dots
  - Clicking each nav item highlights it (active state)
  - Tab panels switch (empty for now — content comes in Task 2)
  - Theme dots still change the colour scheme

- [ ] **Step 5: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: add sidebar nav + tab shell layout"
  ```

---

## Task 2: Distribute Existing Panels into Tabs

Move all existing panel HTML into the correct tab. Briefing gets the live data panels; Resources gets static link panels; Intel gets the lookup tools.

**Files:**
- Modify: `cyber-dashboard.html` (HTML body, ~lines 335–466)

- [ ] **Step 1: Populate the Briefing tab**

  Inside `<div id="tab-briefing" class="tab-panel active">`, add:

  ```html
  <!-- Briefing header -->
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:14px;flex-wrap:wrap;gap:8px;">
    <span id="briefing-context" style="font-size:12px;color:var(--text-muted);">Morning Briefing</span>
    <button id="refresh-all-btn" onclick="refreshAll()" style="padding:5px 12px;background:var(--accent);color:#fff;border:none;border-radius:5px;font-size:12px;font-weight:500;cursor:pointer;">↻ Refresh All</button>
  </div>
  <div id="last-updated" style="font-size:11px;color:var(--text-muted);margin-bottom:12px;margin-top:-10px;"></div>

  <div class="dashboard-grid">
    <!-- KEV panel (unchanged) -->
    <div class="panel" id="panel-kev"> ... </div>

    <!-- NVD panel (unchanged) -->
    <div class="panel" id="panel-nvd"> ... </div>

    <!-- News RSS panel (new — body populated by loadNewsRSS in Task 5) -->
    <div class="panel" id="panel-news">
      <div class="panel-header">
        <span class="panel-title" id="news-title">Security News</span>
        <div class="panel-header-links">
          <span id="news-source-label" style="font-size:11px;color:var(--text-muted);">RSS</span>
          <button class="panel-refresh" onclick="loadNewsRSS()">↻ Refresh</button>
        </div>
      </div>
      <div class="panel-body" id="news-body"><div class="loading-state"><span class="spinner"></span>Loading…</div></div>
    </div>

    <!-- Reddit / Community Pulse panel (body unchanged for now — updated in Task 6) -->
    <div class="panel" id="panel-reddit">
      <div class="panel-header">
        <span class="panel-title">Community Pulse <span id="reddit-kw-label" style="font-weight:400;color:var(--accent);font-size:11px;">★ = industry relevant</span></span>
        <div class="panel-header-links">
          <a class="panel-link" href="https://www.reddit.com/r/netsec/" target="_blank" rel="noopener">r/netsec ↗</a>
          <button class="panel-refresh" onclick="loadReddit()">↻ Refresh</button>
        </div>
      </div>
      <div class="panel-body" id="reddit-body"><div class="loading-state"><span class="spinner"></span>Loading…</div></div>
    </div>

    <!-- Regulations panel (unchanged, full width) -->
    <div class="panel col-span-2" id="panel-refs">
      <div class="panel-header">
        <span class="panel-title" id="refs-title">Regulatory Quick Reference</span>
      </div>
      <div class="panel-body no-scroll" id="refs-body"></div>
    </div>
  </div>
  ```

  Copy the actual KEV and NVD panel HTML from the old grid into the correct positions above (preserve `id` attributes exactly).

- [ ] **Step 2: Populate the Resources tab**

  Inside `<div id="tab-resources" class="tab-panel">`, add:

  ```html
  <div class="dashboard-grid">
    <!-- Industry news links (moved from old panel-news) -->
    <div class="panel" id="panel-news-links">
      <div class="panel-header">
        <span class="panel-title" id="news-links-title">Security News Links</span>
      </div>
      <div class="panel-body no-scroll" id="news-links-body"></div>
    </div>

    <!-- Vendor patch pages (moved from old panel-vendor) -->
    <div class="panel" id="panel-vendor">
      ... (copy existing vendor panel HTML unchanged) ...
    </div>

    <!-- Patch Tuesday (moved from old panel-pt) -->
    <div class="panel" id="panel-pt">
      <div class="panel-header">
        <span class="panel-title">Patch Tuesday Countdown</span>
      </div>
      <div class="panel-body no-scroll" id="pt-body"></div>
    </div>
  </div>
  ```

- [ ] **Step 3: Populate the Intel tab**

  Inside `<div id="tab-intel" class="tab-panel">`, add:

  ```html
  <div class="dashboard-grid">
    <!-- Threat Intel Lookup (moved, full width) -->
    <div class="panel col-span-2" id="panel-ti">
      ... (copy existing threat intel panel HTML unchanged) ...
    </div>

    <!-- CVE Lookup (moved) -->
    <div class="panel col-span-2" id="panel-cve">
      ... (copy existing CVE lookup panel HTML unchanged) ...
    </div>
  </div>
  ```

- [ ] **Step 4: Update renderNews() to target news-links-body** ⚠️ Do this before or at the same time as Step 1–3 above — if `renderNews()` still targets `#news-body` when the page loads, it will overwrite the RSS panel body.

  The existing `renderNews()` function writes to `#news-body`. Now that `#news-body` is the RSS panel and the static link tiles are in `#news-links-body`, update `renderNews()` to write to `news-links-body` and update the title element:

  ```javascript
  function renderNews() {
    const s = loadSettings();
    const ind = PROFILE_DATA.industries[s.industry] || PROFILE_DATA.industries.banking;
    document.getElementById('news-links-title').textContent = ind.newsTitle;
    let html = '';
    ind.news.forEach(sec => {
      html += `<div class="link-section"><div class="link-section-label">${escHtml(sec.label)}</div><div class="link-grid">`;
      sec.links.forEach(l => { html += `<a class="link-tile" href="${escHtml(l.url)}" target="_blank" rel="noopener">${escHtml(l.name)}</a>`; });
      html += '</div></div>';
    });
    document.getElementById('news-links-body').innerHTML = html;
  }
  ```

- [ ] **Step 5: Update init to render Patch Tuesday and regulatory**

  Find the init block at the bottom of `<script>` (~line 1282) and update:

  ```javascript
  renderPatchTuesday();
  renderNews();       // now targets news-links-body in Resources tab
  renderRegulatory();
  refreshAll();
  ```

- [ ] **Step 6: Verify in browser**

  Open the file. Confirm:
  - Briefing tab shows: KEV, NVD, empty news panel, Reddit, Regulations
  - Resources tab shows: news link tiles, vendor links, Patch Tuesday countdown
  - Intel tab shows: Threat Intel and CVE Lookup
  - All panels function (KEV and NVD load, Reddit loads, regulatory renders)

- [ ] **Step 7: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: distribute panels into Briefing/Resources/Intel tabs"
  ```

---

## Task 3: First-Run Behaviour

Show a welcome banner and auto-open settings when no settings exist in localStorage.

**Files:**
- Modify: `cyber-dashboard.html`

- [ ] **Step 1: Add welcome banner CSS**

  Add to `<style>`:

  ```css
  .welcome-banner {
    background: #7a5c00; color: #ffe082; border: 1px solid #a07800;
    border-radius: 6px; padding: 10px 14px; margin-bottom: 14px;
    font-size: 13px; display: flex; align-items: center; gap: 10px; flex-wrap: wrap;
  }
  html[data-theme="light"] .welcome-banner { background: #fff8e1; color: #7a5c00; border-color: #f0c000; }
  html[data-theme="sepia"] .welcome-banner { background: #f5edd6; color: #6b4c00; border-color: #c8a040; }
  .welcome-banner button {
    padding: 4px 10px; background: var(--accent); color: #fff; border: none;
    border-radius: 4px; font-size: 12px; cursor: pointer; white-space: nowrap;
  }
  ```

- [ ] **Step 2: Add banner HTML inside the Briefing tab**

  As the first element inside `<div id="tab-briefing" ...>`, before the briefing header div:

  ```html
  <div id="welcome-banner" class="welcome-banner" style="display:none;">
    <span>Welcome — select your industry and states to personalise this briefing.</span>
    <button onclick="openSettings()">Open Settings ⚙</button>
  </div>
  ```

- [ ] **Step 3: Add WELCOMED_KEY constant and first-run check**

  Add `WELCOMED_KEY` at module scope alongside `SETTINGS_KEY` (around line 528):

  ```javascript
  const WELCOMED_KEY = 'cyber-dash-welcomed';
  ```

  Then add the `checkFirstRun` function to the `<script>` block (before the init block):

  ```javascript
  function checkFirstRun() {
    const hasSettings = !!localStorage.getItem(SETTINGS_KEY);
    const hasWelcomed = !!localStorage.getItem(WELCOMED_KEY);
    if (!hasSettings && !hasWelcomed) {
      document.getElementById('welcome-banner').style.display = 'flex';
      openSettings();
    }
  }
  ```

  Note: use `&&` (both absent) so the banner only shows on a truly fresh load, not if the user has settings but hasn't seen the welcome.

- [ ] **Step 4: Mark welcomed on save**

  In `applySettings()`, add at the top of the function (before `saveSettings(...)`):

  ```javascript
  localStorage.setItem(WELCOMED_KEY, '1');
  document.getElementById('welcome-banner').style.display = 'none';
  ```

- [ ] **Step 5: Call checkFirstRun() in init**

  Add `checkFirstRun();` to the init block just before `refreshAll()`.

- [ ] **Step 6: Verify in browser**

  1. Open the file normally (settings already exist from previous use) — no banner, no drawer auto-open.
  2. Open DevTools → Application → LocalStorage → delete `cyber-dash-settings` and `cyber-dash-welcomed` → refresh. Confirm banner appears and Settings drawer auto-opens.
  3. Fill in settings, click Save & Apply. Confirm banner disappears and does not reappear on next reload.

- [ ] **Step 7: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: add first-run banner and auto-open settings on new load"
  ```

---

## Task 4: Expand State List (VA, CT, OR, GA)

Add four new states to the settings data and checkboxes.

**Files:**
- Modify: `cyber-dashboard.html` (PROFILE_DATA.states ~line 719, settings HTML ~line 489)

- [ ] **Step 1: Add state data to PROFILE_DATA.states**

  After the `IL` entry (closing `}` of IL block at ~line 772), add:

  ```javascript
  VA: { label: 'Virginia Consumer Data Protection Act (VCDPA)',
    html: `<strong style="color:var(--text);">VCDPA (Va. Code §59.1-575 et seq.)</strong> — eff. Jan 1, 2023. Applies to controllers processing data of 100,000+ VA consumers, or 25,000+ if >50% revenue from data sales. Breach notification (Va. Code §18.2-186.6): notify without unreasonable delay.`,
    links: [
      { name: 'VA AG — Privacy', url: 'https://www.oag.state.va.us/consumer-protection/index.php/privacy-and-security' },
      { name: 'VCDPA Text', url: 'https://law.lis.virginia.gov/vacode/title59.1/chapter53/' },
    ]
  },
  CT: { label: 'Connecticut Data Privacy Act (CTDPA)',
    html: `<strong style="color:var(--text);">CTDPA (Pub. Act 22-15)</strong> — eff. July 1, 2023. Applies to controllers processing data of 100,000+ CT consumers, or 25,000+ if >25% revenue from data sales. Breach notification (Conn. Gen. Stat. §36a-701b): notify within <strong style="color:var(--critical);">60 days</strong> of discovery.`,
    links: [
      { name: 'CT AG — Data Privacy', url: 'https://portal.ct.gov/AG/Sections/Privacy/The-Privacy-Section' },
    ]
  },
  OR: { label: 'Oregon Consumer Privacy Act (OCPA)',
    html: `<strong style="color:var(--text);">OCPA (ORS 646A.570 et seq.)</strong> — eff. July 1, 2024. Applies to controllers processing data of 100,000+ OR consumers, or 25,000+ if >25% revenue from data sales. Breach notification (ORS 646A.604): notify within <strong style="color:var(--critical);">30 days</strong> of discovery.`,
    links: [
      { name: 'OR DOJ — Data Breach', url: 'https://www.doj.state.or.us/consumer-protection/data-breach-information-for-businesses/' },
    ]
  },
  GA: { label: 'Georgia Breach Notification',
    html: `<strong style="color:var(--text);">O.C.G.A. §10-1-910 et seq.</strong> — notify affected GA residents "in the most expedient time possible." No specific day limit. No AG notification requirement under current law.`,
    links: [
      { name: 'GA AG — Consumer Protection', url: 'https://consumer.georgia.gov/data-breaches' },
    ]
  }
  ```

- [ ] **Step 2: Add checkboxes to settings HTML**

  In the `<div class="check-grid" id="s-states">` block (~line 490), add four new labels after the IL checkbox:

  ```html
  <label class="check-item"><input type="checkbox" value="VA"> Virginia</label>
  <label class="check-item"><input type="checkbox" value="CT"> Connecticut</label>
  <label class="check-item"><input type="checkbox" value="OR"> Oregon</label>
  <label class="check-item"><input type="checkbox" value="GA"> Georgia</label>
  ```

- [ ] **Step 3: Verify in browser**

  Open Settings. Confirm 12 state checkboxes appear. Select VA and OR. Click Save & Apply. Confirm the Regulations panel shows VA VCDPA and OR OCPA entries alongside the federal regs.

- [ ] **Step 4: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: expand state list to 12 (add VA, CT, OR, GA)"
  ```

---

## Task 5: RSS News Panel

Replace the static link tiles in the Briefing news panel with live RSS stories via rss2json.com, falling back to link tiles on failure.

**Files:**
- Modify: `cyber-dashboard.html` (PROFILE_DATA, JS loadNewsRSS function)

- [ ] **Step 1: Add rssFeeds arrays to each industry in PROFILE_DATA**

  For each industry in `PROFILE_DATA.industries`, add an `rssFeeds` property (a list of RSS URL strings). Edit each industry object:

  ```javascript
  // banking — add after the federalRegs property:
  rssFeeds: [
    'https://www.bleepingcomputer.com/feed/',
    'https://krebsonsecurity.com/feed/',
    'https://www.bankinfosecurity.com/rss/news',
    'https://www.darkreading.com/rss/all.xml',
  ],

  // healthcare:
  rssFeeds: [
    'https://www.healthcareinfosecurity.com/rss/news',
    'https://www.hipaajournal.com/feed/',
    'https://www.bleepingcomputer.com/feed/',
    'https://krebsonsecurity.com/feed/',
  ],

  // retail:
  rssFeeds: [
    'https://www.scmagazine.com/feed/',
    'https://www.bleepingcomputer.com/feed/',
    'https://krebsonsecurity.com/feed/',
    'https://www.darkreading.com/rss/all.xml',
  ],

  // technology:
  rssFeeds: [
    'https://feeds.feedburner.com/TheHackersNews',
    'https://www.bleepingcomputer.com/feed/',
    'https://krebsonsecurity.com/feed/',
    'https://isc.sans.edu/rssfeed.xml',
  ],

  // legal:
  rssFeeds: [
    'https://www.darkreading.com/rss/all.xml',
    'https://www.bleepingcomputer.com/feed/',
    'https://krebsonsecurity.com/feed/',
    'https://www.law.com/legaltechnews/feed/',
  ],

  // government:
  rssFeeds: [
    'https://fedscoop.com/feed/',
    'https://statescoop.com/feed/',
    'https://www.bleepingcomputer.com/feed/',
    'https://krebsonsecurity.com/feed/',
  ],
  ```

- [ ] **Step 2: Add loadNewsRSS() function**

  Add after the `renderRegulatory()` function (~line 897):

  ```javascript
  const RSS2JSON_BASE = 'https://api.rss2json.com/v1/api.json?rss_url=';

  async function loadNewsRSS() {
    const s = loadSettings();
    const ind = PROFILE_DATA.industries[s.industry] || PROFILE_DATA.industries.banking;
    const titleEl = document.getElementById('news-title');
    if (titleEl) titleEl.textContent = ind.newsTitle;
    setLoading('news-body');
    try {
      const feeds = ind.rssFeeds || [];
      const results = await Promise.allSettled(
        feeds.map(url =>
          fetch(RSS2JSON_BASE + encodeURIComponent(url), { cache: 'default' })
            .then(r => { if (!r.ok) throw new Error('HTTP ' + r.status); return r.json(); })
        )
      );
      const items = [];
      results.forEach(r => {
        if (r.status !== 'fulfilled' || r.value.status !== 'ok') return;
        const feedTitle = r.value.feed?.title || '';
        (r.value.items || []).slice(0, 6).forEach(item => {
          items.push({
            title: item.title || '',
            link: item.link || item.guid || '#',
            description: (item.description || item.content || '').replace(/<[^>]+>/g, '').trim().slice(0, 130),
            pubDate: item.pubDate || '',
            source: feedTitle,
          });
        });
      });
      if (!items.length) throw new Error('no items');
      items.sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));
      const kw = getIndustryKW();
      const ul = document.createElement('ul');
      ul.className = 'feed-list';
      items.slice(0, 12).forEach(item => {
        const relevant = kw.test(item.title + ' ' + item.description);
        const li = document.createElement('li');
        li.className = 'feed-item' + (relevant ? ' industry-relevant' : '');
        li.innerHTML = `
          <a class="feed-item-title" href="${escHtml(item.link)}" target="_blank" rel="noopener">${escHtml(item.title)}</a>
          <div class="feed-item-meta">
            <span>${escHtml(item.source)}</span>
            <span>${timeAgo(item.pubDate)}</span>
            ${relevant ? '<span style="color:var(--accent);">★ relevant</span>' : ''}
          </div>`;
        ul.appendChild(li);
      });
      document.getElementById('news-body').innerHTML = '';
      document.getElementById('news-body').appendChild(ul);
    } catch(e) {
      // Fallback: render the static link-tile grid in the news panel
      const s2 = loadSettings();
      const ind2 = PROFILE_DATA.industries[s2.industry] || PROFILE_DATA.industries.banking;
      let html = '';
      ind2.news.forEach(sec => {
        html += `<div class="link-section"><div class="link-section-label">${escHtml(sec.label)}</div><div class="link-grid">`;
        sec.links.forEach(l => { html += `<a class="link-tile" href="${escHtml(l.url)}" target="_blank" rel="noopener">${escHtml(l.name)}</a>`; });
        html += '</div></div>';
      });
      document.getElementById('news-body').innerHTML = html || '<div class="loading-state">Could not load news.</div>';
    }
  }
  ```

- [ ] **Step 3: Add industry-relevant CSS class**

  In `<style>`, add (alongside the existing `.feed-item.banking-relevant` rule ~line 129):

  ```css
  .feed-item.industry-relevant { background: var(--accent-light); border-left: 3px solid var(--accent); padding-left: 11px; margin-left: -14px; }
  ```

  Keep the existing `.feed-item.banking-relevant` rule for backwards compatibility for now (will be cleaned up in Task 6).

- [ ] **Step 4: Wire loadNewsRSS into refreshAll() and applySettings()**

  Update `refreshAll()`:
  ```javascript
  function refreshAll() {
    loadKEV();
    loadNVD();
    loadNewsRSS();
    loadReddit();
    updateTimestamp();
  }
  ```

  In `applySettings()`, the existing `renderNews()` call should remain (it targets `news-links-body` in Resources tab). Also add `loadNewsRSS()` after `renderNews()`:
  ```javascript
  renderNews();
  loadNewsRSS();
  ```

- [ ] **Step 5: Add loadNewsRSS() to init block**

  The init block already calls `refreshAll()` which now includes `loadNewsRSS()`. No extra call needed.

- [ ] **Step 6: Verify in browser**

  Open Briefing tab. Confirm the Security News panel loads article headlines with source names and timestamps. Confirm industry-relevant articles show the `★ relevant` label and blue left border.

  To test the fallback: open DevTools → Network tab → block `api.rss2json.com` (right-click a rss2json request → "Block request domain") → click ↻ Refresh on the panel. Confirm it degrades to link tiles without an error message.

- [ ] **Step 7: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: add RSS news panel with rss2json fetch and link-tile fallback"
  ```

---

## Task 6: Community Pulse — Improved Reddit Panel

Replace the "hot posts from subreddits" approach with two targeted searches: subreddit-scoped and industry-keyword-filtered. Rename the CSS highlight class.

**Files:**
- Modify: `cyber-dashboard.html` (JS `loadReddit()` ~line 1075, CSS `banking-relevant` ~line 129)

- [ ] **Step 1: Add REDDIT_KEYWORDS constant**

  Add just before the `loadReddit` function definition (~line 1074):

  ```javascript
  const REDDIT_KEYWORDS = {
    banking:    'bank fraud ransomware phishing breach payment wire swift GLBA',
    healthcare: 'healthcare HIPAA ransomware patient breach medical PHI EHR',
    retail:     'retail ecommerce PCI payment skimming magecart breach card',
    technology: 'cloud vulnerability zero-day exploit ransomware API supply chain',
    legal:      'law firm ransomware breach attorney privilege client data',
    government: 'government CISA federal ransomware nation-state breach critical infrastructure FISMA',
  };
  ```

- [ ] **Step 2: Replace loadReddit() entirely**

  Replace the existing `loadReddit()` function (~lines 1075–1114) with:

  ```javascript
  async function loadReddit() {
    setLoading('reddit-body');
    const s = loadSettings();
    const kw = getIndustryKW();
    const keywords = REDDIT_KEYWORDS[s.industry] || REDDIT_KEYWORDS.banking;
    const base = 'https://www.reddit.com/search.json?sort=top&t=week&limit=15&raw_json=1';
    try {
      const [r1, r2] = await Promise.all([
        fetch(`${base}&q=${encodeURIComponent('(subreddit:netsec OR subreddit:cybersecurity)')}`)
          .then(r => { if (!r.ok) throw new Error(); return r.json(); }),
        fetch(`${base}&q=${encodeURIComponent(keywords)}`)
          .then(r => { if (!r.ok) throw new Error(); return r.json(); }),
      ]);
      const seen = new Set();
      const posts = [];
      [...(r1.data?.children || []), ...(r2.data?.children || [])].forEach(c => {
        const p = c.data;
        const key = p.url || p.id;
        if (!seen.has(key)) { seen.add(key); posts.push(p); }
      });
      posts.sort((a, b) => (b.score || 0) - (a.score || 0));
      const top10 = posts.slice(0, 10);
      if (!top10.length) {
        document.getElementById('reddit-body').innerHTML = '<div class="loading-state">No posts found.</div>';
        return;
      }
      const ul = document.createElement('ul');
      ul.className = 'feed-list';
      top10.forEach(p => {
        const postUrl = 'https://www.reddit.com' + (p.permalink || '');
        const relevant = kw.test(p.title + ' ' + (p.selftext || ''));
        const sub = p.subreddit || 'unknown';
        const li = document.createElement('li');
        li.className = 'feed-item' + (relevant ? ' industry-relevant' : '');
        li.innerHTML = `
          <a class="feed-item-title" href="${escHtml(p.url || postUrl)}" target="_blank" rel="noopener">${relevant ? '★ ' : ''}${escHtml(p.title)}</a>
          <div class="feed-item-meta">
            <span class="reddit-sub">r/${escHtml(sub)}</span>
            <span>↑ ${(p.score || 0).toLocaleString()}</span>
            <a href="${escHtml(postUrl)}" target="_blank" rel="noopener">${p.num_comments || 0} comments</a>
            <span>${timeAgo(new Date(p.created_utc * 1000).toISOString())}</span>
          </div>`;
        ul.appendChild(li);
      });
      document.getElementById('reddit-body').innerHTML = '';
      document.getElementById('reddit-body').appendChild(ul);
    } catch(e) {
      setError('reddit-body', 'https://www.reddit.com/r/netsec/');
    }
  }
  ```

- [ ] **Step 3: Remove the old banking-relevant CSS rule**

  Find and remove (or replace) the `.feed-item.banking-relevant` CSS rule (~line 129). The `.feed-item.industry-relevant` rule added in Task 5 covers this. Search for `banking-relevant` in the entire file and replace all occurrences with `industry-relevant`.

- [ ] **Step 4: Verify in browser**

  Open Briefing tab. The Community Pulse panel should show posts relevant to the selected industry (e.g. banking-related posts with ★ highlights when Banking is selected). Change industry to Healthcare in Settings → Save → confirm posts shift toward healthcare topics.

- [ ] **Step 5: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: improve Community Pulse with industry-keyword Reddit search"
  ```

---

## Task 7: Briefing Context Line + Final Wiring

Update the briefing header context line to show current industry/state selections, and clean up any remaining references to the old layout.

**Files:**
- Modify: `cyber-dashboard.html`

- [ ] **Step 1: Add updateBriefingContext() function**

  Add after `renderRegulatory()`:

  ```javascript
  function updateBriefingContext() {
    const s = loadSettings();
    const ind = PROFILE_DATA.industries[s.industry] || PROFILE_DATA.industries.banking;
    const stateLabels = (s.states || []).map(a => a); // use abbreviations for brevity
    const ctx = document.getElementById('briefing-context');
    if (!ctx) return;
    ctx.textContent = 'Morning Briefing · ' + ind.label + (stateLabels.length ? ' · ' + stateLabels.join(', ') : '');
  }
  ```

- [ ] **Step 2: Call updateBriefingContext() from applySettings() and init**

  In `applySettings()`, add `updateBriefingContext();` after `renderRegulatory();`.

  In the init block, add `updateBriefingContext();` just before `checkFirstRun();`.

- [ ] **Step 3: Verify timestamps still update**

  `updateTimestamp()` writes to `#last-updated`. Confirm that element is still in the DOM (it was moved to inside `tab-briefing` in Task 2) and that it updates when Refresh All is clicked.

- [ ] **Step 4: Remove dead #header CSS rules**

  Search for `#header`, `#app-name`, `#last-updated` (as a top-level rule), `#refresh-all-btn`, `#settings-btn` CSS rules in the `<style>` block and remove them — these IDs no longer exist in the HTML.

  Also remove the old `#settings-btn` HTML `<button>` if it still exists anywhere in the file.

- [ ] **Step 5: Final smoke test in browser**

  Run through the full checklist:
  - [ ] First run: clear localStorage, reload → banner + Settings drawer auto-open
  - [ ] Save settings with Banking / CA + NY → context line shows correct industry/states
  - [ ] Briefing tab: KEV, NVD, RSS news, Community Pulse all load
  - [ ] Resources tab: news link tiles, vendor links, Patch Tuesday countdown all render
  - [ ] Intel tab: Threat Intel lookup and CVE Lookup work
  - [ ] Settings: change to Healthcare / WA → news panel, Reddit, and regulations all update
  - [ ] Refresh All button refreshes KEV, NVD, news, and Reddit
  - [ ] Theme switcher still works from sidebar
  - [ ] Selecting VA, CT, OR, GA shows their regulatory content
  - [ ] Fallback: block rss2json in DevTools → news panel falls back to link tiles silently

- [ ] **Step 6: Commit**

  ```bash
  git add cyber-dashboard.html
  git commit -m "feat: add briefing context line and complete dashboard redesign"
  ```

---

## Done

The redesigned dashboard is complete. All live-data panels are in the Briefing tab, static reference links are in Resources, and lookup tools are in Intel. The RSS news and improved Reddit feeds provide the morning pulse-check experience. First-run onboarding guides new users to configure their industry and states.
