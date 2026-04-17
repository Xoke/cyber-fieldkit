# Cyber Dashboard High-Value Additions — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add EPSS scores, CISA Advisories, Abuse.ch threat feeds, Ransomware activity, and Shodan InternetDB to `cyber-dashboard.html`.

**Architecture:** Single-file HTML app — all changes are in `cyber-dashboard.html`. No build step. Data sources are public APIs with CORS support. New JS functions follow existing `loadXxx()` / `setLoading()` / `setError()` patterns. Threats tab uses lazy loading (first tab visit triggers fetch; subsequent visits do not re-fetch).

**Tech Stack:** Vanilla JS, CSS custom properties, Fetch API, `rss2json` proxy (already in use), abuse.ch APIs, ransomware.live API, Shodan InternetDB, FIRST.org EPSS API.

**Spec:** `docs/superpowers/specs/2026-04-17-cyber-dashboard-additions-design.md`

---

## Task 1: Add CSS for EPSS badges and InternetDB block

**Files:**
- Modify: `cyber-dashboard.html` — add CSS after line 165 (after `.badge-low` rule)

- [ ] **Step 1: Insert CSS after the `.badge-low` rule**

Find this line (around line 165):
```css
.badge-low { background: var(--low); }
```

Add immediately after it:
```css
/* EPSS badge */
.epss-badge { display:inline-block; padding:1px 5px; border-radius:3px; font-size:11px; font-weight:600; }
.epss-high { background:color-mix(in srgb,var(--critical) 20%,transparent); color:var(--critical); }
.epss-med  { background:color-mix(in srgb,var(--medium)   20%,transparent); color:var(--medium); }
.epss-low  { background:color-mix(in srgb,var(--text-muted) 15%,transparent); color:var(--text-muted); }

/* Shodan InternetDB result block */
#idb-result { margin-top:12px; padding:10px 12px; background:var(--bg-hover); border-radius:6px; font-size:12px; }
.idb-row { margin-bottom:6px; color:var(--text); }
.idb-label { color:var(--text-muted); margin-right:6px; font-size:11px; }
.idb-tag { display:inline-block; padding:1px 6px; border-radius:3px; background:var(--accent-light); color:var(--accent); font-size:11px; margin:1px; }
```

- [ ] **Step 2: Verify in browser**

Open `cyber-dashboard.html` in browser. Open DevTools console and run:
```javascript
document.body.insertAdjacentHTML('beforeend','<span class="epss-badge epss-high">EPSS 94%</span> <span class="epss-badge epss-med">EPSS 23%</span> <span class="epss-badge epss-low">EPSS 2%</span>');
```
Expected: Three styled badges appear at bottom of page with red/yellow/gray coloring.

- [ ] **Step 3: Commit**

```bash
git add cyber-dashboard.html
git commit -m "style: add EPSS badge and InternetDB CSS"
```

---

## Task 2: EPSS overlay on KEV panel

**Files:**
- Modify: `cyber-dashboard.html` — `renderKEVList()` function (~line 1112) and add `loadEPSS()` function

- [ ] **Step 1: Add `data-cve` attribute and `epss-slot` span to KEV list items**

In `renderKEVList()`, find this block (around line 1128):
```javascript
    const li = document.createElement('li');
    li.className = 'feed-item' + (swHit ? ' sw-match' : '');
    li.innerHTML = `
      <div style="font-weight:600;margin-bottom:2px;">${swHit ? '⚡ ' : ''}${escHtml(productLabel || v.vulnerabilityName)}</div>
      ${productLabel ? `<div style="font-size:12px;color:var(--text);margin-top:2px;">${escHtml(truncate(v.vulnerabilityName, 90))}</div>` : ''}
      <div style="font-size:12px;color:var(--text-muted);display:flex;gap:10px;flex-wrap:wrap;margin-top:3px;">
        <a class="feed-item-title" href="https://nvd.nist.gov/vuln/detail/${escHtml(v.cveID)}" target="_blank" rel="noopener">${escHtml(v.cveID)}</a>
        <span>Added: ${escHtml(v.dateAdded)}</span>
        <span class="${dueCls}">${escHtml(dueLabel)}</span>
        ${sourceNote ? `<span style="font-style:italic;">${escHtml(sourceNote)}</span>` : ''}
      </div>`;
```

Replace with:
```javascript
    const li = document.createElement('li');
    li.className = 'feed-item' + (swHit ? ' sw-match' : '');
    li.dataset.cve = v.cveID;
    li.innerHTML = `
      <div style="font-weight:600;margin-bottom:2px;">${swHit ? '⚡ ' : ''}${escHtml(productLabel || v.vulnerabilityName)}</div>
      ${productLabel ? `<div style="font-size:12px;color:var(--text);margin-top:2px;">${escHtml(truncate(v.vulnerabilityName, 90))}</div>` : ''}
      <div style="font-size:12px;color:var(--text-muted);display:flex;gap:10px;flex-wrap:wrap;margin-top:3px;">
        <a class="feed-item-title" href="https://nvd.nist.gov/vuln/detail/${escHtml(v.cveID)}" target="_blank" rel="noopener">${escHtml(v.cveID)}</a>
        <span>Added: ${escHtml(v.dateAdded)}</span>
        <span class="${dueCls}">${escHtml(dueLabel)}</span>
        <span class="epss-slot"></span>
        ${sourceNote ? `<span style="font-style:italic;">${escHtml(sourceNote)}</span>` : ''}
      </div>`;
```

- [ ] **Step 2: Call `loadEPSS()` after KEV list is rendered**

In `renderKEVList()`, find these two lines at the end:
```javascript
  document.getElementById('kev-body').innerHTML = '';
  document.getElementById('kev-body').appendChild(ul);
```

Replace with:
```javascript
  document.getElementById('kev-body').innerHTML = '';
  document.getElementById('kev-body').appendChild(ul);
  loadEPSS(vulns.map(v => v.cveID), 'kev-body');
```

- [ ] **Step 3: Add `loadEPSS()` function**

Add this function after the `// ─── Helpers` section (around line 1060, before `// ─── Panel 1: CISA KEV`):

```javascript
// ─── EPSS Score Overlay ───────────────────────────────────────────────────────
async function loadEPSS(cveIds, panelId) {
  if (!cveIds.length) return;
  try {
    const resp = await fetch('https://api.first.org/data/v1/epss?cve=' + cveIds.join(','));
    if (!resp.ok) return;
    const data = await resp.json();
    (data.data || []).forEach(item => {
      const pct = Math.round(parseFloat(item.epss) * 100);
      const cls = pct >= 50 ? 'epss-high' : pct >= 10 ? 'epss-med' : 'epss-low';
      const slot = document.querySelector(`[data-cve="${item.cve}"] .epss-slot`);
      if (slot) slot.innerHTML = `<span class="epss-badge ${cls}">EPSS ${pct}%</span>`;
    });
  } catch(_) {}
}
```

- [ ] **Step 4: Verify in browser**

Reload `cyber-dashboard.html`. Wait for KEV panel to load. Expected: within ~1 second, small EPSS badges appear next to each CVE entry in the KEV panel (red for high, yellow for medium, gray for low probability).

- [ ] **Step 5: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: add EPSS score overlay to KEV panel"
```

---

## Task 3: EPSS overlay on NVD panel

**Files:**
- Modify: `cyber-dashboard.html` — `loadNVD()` function (~line 1188)

- [ ] **Step 1: Add `data-cve` attribute and `epss-slot` to NVD list items**

In `loadNVD()`, find this block (around line 1211):
```javascript
      const li = document.createElement('li');
      li.className = 'feed-item' + (swHitN ? ' sw-match' : '');
      li.innerHTML = `
        <div style="font-weight:600;margin-bottom:2px;">${swHitN ? '⚡ ' : ''}${escHtml(truncate(desc, 120))}</div>
        <div style="font-size:12px;color:var(--text-muted);display:flex;gap:8px;flex-wrap:wrap;align-items:center;margin-top:3px;">
          <a class="feed-item-title" href="https://nvd.nist.gov/vuln/detail/${escHtml(cve.id)}" target="_blank" rel="noopener">${escHtml(cve.id)}</a>
          <span class="cvss-badge ${cvssBadgeClass(score)}">${score.toFixed(1)}</span>
          <span>${timeAgo(cve.published)}</span>
        </div>`;
```

Replace with:
```javascript
      const li = document.createElement('li');
      li.className = 'feed-item' + (swHitN ? ' sw-match' : '');
      li.dataset.cve = cve.id;
      li.innerHTML = `
        <div style="font-weight:600;margin-bottom:2px;">${swHitN ? '⚡ ' : ''}${escHtml(truncate(desc, 120))}</div>
        <div style="font-size:12px;color:var(--text-muted);display:flex;gap:8px;flex-wrap:wrap;align-items:center;margin-top:3px;">
          <a class="feed-item-title" href="https://nvd.nist.gov/vuln/detail/${escHtml(cve.id)}" target="_blank" rel="noopener">${escHtml(cve.id)}</a>
          <span class="cvss-badge ${cvssBadgeClass(score)}">${score.toFixed(1)}</span>
          <span>${timeAgo(cve.published)}</span>
          <span class="epss-slot"></span>
        </div>`;
```

- [ ] **Step 2: Call `loadEPSS()` after NVD list is rendered**

In `loadNVD()`, find these two lines:
```javascript
    document.getElementById('nvd-body').innerHTML = '';
    document.getElementById('nvd-body').appendChild(ul);
```

Replace with:
```javascript
    document.getElementById('nvd-body').innerHTML = '';
    document.getElementById('nvd-body').appendChild(ul);
    loadEPSS(items.map(i => i.cve.id), 'nvd-body');
```

- [ ] **Step 3: Verify in browser**

Reload the file. Wait for NVD panel. Expected: EPSS badges appear next to each CVE in the NVD panel alongside the CVSS score badge.

- [ ] **Step 4: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: add EPSS score overlay to NVD panel"
```

---

## Task 4: CISA Advisories panel on Briefing tab

**Files:**
- Modify: `cyber-dashboard.html` — add HTML panel (~line 431), add `loadCISAAdvisories()` function, wire into `refreshAll()` and `applySettings()`

- [ ] **Step 1: Add CISA Advisories panel HTML to Briefing tab**

In the Briefing tab's `dashboard-grid`, find this comment and opening div (around line 423):
```html
        <div class="panel" id="panel-reddit">
```

Add the following new panel immediately BEFORE the reddit panel opening div (so it appears between News and Reddit panels in the 2-column grid):
```html
        <!-- CISA Advisories panel -->
        <div class="panel" id="panel-cisa">
          <div class="panel-header">
            <span class="panel-title">CISA Advisories</span>
            <div class="panel-header-links">
              <a class="panel-link" href="https://www.cisa.gov/news-events/cybersecurity-advisories" target="_blank" rel="noopener">View all ↗</a>
              <button class="panel-refresh" onclick="loadCISAAdvisories()">↻ Refresh</button>
            </div>
          </div>
          <div class="panel-body" id="cisa-body"><div class="loading-state"><span class="spinner"></span>Loading…</div></div>
        </div>

```

- [ ] **Step 2: Add `loadCISAAdvisories()` function**

Add this function after `loadReddit()` and before the `// ─── Panel 5: Threat Intel Lookup` comment:

```javascript
// ─── Panel: CISA Advisories ───────────────────────────────────────────────────
async function loadCISAAdvisories() {
  setLoading('cisa-body');
  try {
    const resp = await fetch(RSS2JSON_BASE + encodeURIComponent('https://www.cisa.gov/cybersecurity-advisories/all.xml'), { cache: 'default' });
    if (!resp.ok) throw new Error('HTTP ' + resp.status);
    const data = await resp.json();
    if (data.status !== 'ok') throw new Error('feed error');
    const items = (data.items || []).slice(0, 10);
    if (!items.length) { document.getElementById('cisa-body').innerHTML = '<div class="loading-state">No advisories found.</div>'; return; }
    const kw = getIndustryKW();
    const ul = document.createElement('ul');
    ul.className = 'feed-list';
    items.forEach(item => {
      const title = item.title || '';
      const typeMatch = title.match(/^(AA-\d{2}|ICSA-\d{2}-\d+|ICS-CERT|TA\d{2})/);
      const typeTag = typeMatch ? typeMatch[1] : '';
      const relevant = kw.test(title);
      const li = document.createElement('li');
      li.className = 'feed-item' + (relevant ? ' industry-relevant' : '');
      li.innerHTML = `
        <a class="feed-item-title" href="${escHtml(item.link || item.guid || '#')}" target="_blank" rel="noopener">${escHtml(title)}</a>
        <div class="feed-item-meta">
          ${typeTag ? `<span style="color:var(--accent);font-weight:600;">${escHtml(typeTag)}</span>` : ''}
          <span>CISA</span>
          <span>${item.pubDate ? timeAgo(item.pubDate) : ''}</span>
          ${relevant ? '<span style="color:var(--accent);">★ relevant</span>' : ''}
        </div>`;
      ul.appendChild(li);
    });
    document.getElementById('cisa-body').innerHTML = '';
    document.getElementById('cisa-body').appendChild(ul);
  } catch(e) {
    setError('cisa-body', 'https://www.cisa.gov/news-events/cybersecurity-advisories');
  }
}
```

- [ ] **Step 3: Wire `loadCISAAdvisories()` into `refreshAll()` and `applySettings()`**

In `refreshAll()`, find:
```javascript
function refreshAll() {
  loadKEV();
  loadNVD();
  loadNewsRSS();
  loadReddit();
  updateTimestamp();
}
```

Replace with:
```javascript
function refreshAll() {
  loadKEV();
  loadNVD();
  loadNewsRSS();
  loadReddit();
  loadCISAAdvisories();
  updateTimestamp();
}
```

In `applySettings()`, find:
```javascript
  updateBriefingContext();
  renderNews();
  loadNewsRSS();
  renderRegulatory();
  loadKEV();
  loadNVD();
  loadReddit();
```

Replace with:
```javascript
  updateBriefingContext();
  renderNews();
  loadNewsRSS();
  renderRegulatory();
  loadKEV();
  loadNVD();
  loadReddit();
  loadCISAAdvisories();
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: CISA Advisories panel appears on Briefing tab between News and Reddit panels. Items load with titles linking to advisory pages. Industry-relevant items are highlighted. Type tags like `AA-24` shown in accent color if present.

- [ ] **Step 5: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: add CISA Advisories panel to Briefing tab"
```

---

## Task 5: Add "Threats" tab nav button and HTML skeleton

**Files:**
- Modify: `cyber-dashboard.html` — add nav button (~line 356) and tab panel HTML (~line 540)

- [ ] **Step 1: Add "Threats" nav button**

Find the Intel nav button (around line 356):
```html
      <button class="nav-item" data-tab="intel" onclick="switchTab('intel')">
        <span class="nav-icon">🔍</span><span class="nav-label">Intel</span>
      </button>
```

Add immediately after it:
```html
      <button class="nav-item" data-tab="threats" onclick="switchTab('threats')">
        <span class="nav-icon">⚠</span><span class="nav-label">Threats</span>
      </button>
```

- [ ] **Step 2: Add Threats tab panel HTML**

Find the closing `</main>` tag. The Intel tab ends just before `</main>`. Find the closing `</div>` of `tab-intel` (after the CVE lookup panel). Add the new tab immediately after:

```html
    <div id="tab-threats" class="tab-panel">
      <div class="dashboard-grid">
        <!-- Abuse.ch Active Threats -->
        <div class="panel" id="panel-abuse">
          <div class="panel-header">
            <span class="panel-title">Active Threats (Abuse.ch)</span>
            <div class="panel-header-links">
              <a class="panel-link" href="https://abuse.ch" target="_blank" rel="noopener">abuse.ch ↗</a>
              <button class="panel-refresh" onclick="loadAbuse()">↻ Refresh</button>
            </div>
          </div>
          <div class="panel-body" id="abuse-body"><div class="loading-state">Loading on first visit…</div></div>
        </div>

        <!-- Ransomware Activity -->
        <div class="panel" id="panel-ransomware">
          <div class="panel-header">
            <span class="panel-title">Ransomware Activity</span>
            <div class="panel-header-links">
              <a class="panel-link" href="https://www.ransomware.live" target="_blank" rel="noopener">ransomware.live ↗</a>
              <button class="panel-refresh" onclick="loadRansomware()">↻ Refresh</button>
            </div>
          </div>
          <div class="panel-body" id="ransomware-body"><div class="loading-state">Loading on first visit…</div></div>
        </div>
      </div>
    </div>
```

- [ ] **Step 3: Verify in browser**

Reload. Expected: "Threats" tab appears in nav sidebar. Clicking it switches to the (empty) Threats tab with two panels showing placeholder text. No JS errors in console.

- [ ] **Step 4: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: add Threats tab nav and HTML skeleton"
```

---

## Task 6: Implement `loadAbuse()` (Feodo C2 + ThreatFox IOCs)

**Files:**
- Modify: `cyber-dashboard.html` — add `loadAbuse()` function after the `loadCISAAdvisories()` block

- [ ] **Step 1: Add `loadAbuse()` function**

Add after `loadCISAAdvisories()` and before the `// ─── Panel 5: Threat Intel Lookup` comment:

```javascript
// ─── Threats Tab: Abuse.ch Active Threats ────────────────────────────────────
async function loadAbuse() {
  setLoading('abuse-body');
  const [feodoResult, threatfoxResult] = await Promise.allSettled([
    fetch('https://feodotracker.abuse.ch/downloads/ipblocklist.json')
      .then(r => { if (!r.ok) throw new Error('HTTP ' + r.status); return r.json(); }),
    fetch('https://threatfox-api.abuse.ch/api/v1/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query: 'get_iocs', days: 1 })
    }).then(r => { if (!r.ok) throw new Error('HTTP ' + r.status); return r.json(); })
  ]);

  const entries = [];

  if (feodoResult.status === 'fulfilled') {
    const items = Array.isArray(feodoResult.value) ? feodoResult.value : [];
    items
      .sort((a, b) => (b.status === 'online' ? 1 : 0) - (a.status === 'online' ? 1 : 0))
      .slice(0, 15)
      .forEach(item => entries.push({
        malware: item.malware || 'Unknown',
        ioc: `${item.ip_address || ''}:${item.port || ''}`,
        type: 'C2 IP',
        country: item.country || '',
        status: item.status || '',
        source: 'Feodo',
        confidence: null,
      }));
  }

  if (threatfoxResult.status === 'fulfilled') {
    const tfData = threatfoxResult.value?.data;
    if (Array.isArray(tfData)) {
      tfData
        .sort((a, b) => (b.confidence_level || 0) - (a.confidence_level || 0))
        .slice(0, 15)
        .forEach(item => entries.push({
          malware: item.malware_printable || item.malware || 'Unknown',
          ioc: item.ioc || '',
          type: item.ioc_type || '',
          country: '',
          status: '',
          source: 'ThreatFox',
          confidence: item.confidence_level != null ? item.confidence_level : null,
        }));
    }
  }

  if (!entries.length) {
    setError('abuse-body', 'https://abuse.ch');
    return;
  }

  const ul = document.createElement('ul');
  ul.className = 'feed-list';
  entries.forEach(e => {
    const swHit = softwareMatches(e.malware);
    const li = document.createElement('li');
    li.className = 'feed-item' + (swHit ? ' sw-match' : '');
    li.innerHTML = `
      <div style="font-weight:600;margin-bottom:2px;">${swHit ? '⚡ ' : ''}${escHtml(e.malware)}</div>
      <div style="font-size:12px;color:var(--text);font-family:monospace;margin-bottom:3px;">${escHtml(truncate(e.ioc, 60))}</div>
      <div class="feed-item-meta">
        <span>${escHtml(e.source)}</span>
        <span>${escHtml(e.type)}</span>
        ${e.country ? `<span>${escHtml(e.country)}</span>` : ''}
        ${e.status === 'online' ? '<span style="color:var(--critical);font-weight:600;">● ONLINE</span>' : ''}
        ${e.confidence != null ? `<span>conf: ${e.confidence}%</span>` : ''}
      </div>`;
    ul.appendChild(li);
  });
  document.getElementById('abuse-body').innerHTML = '';
  document.getElementById('abuse-body').appendChild(ul);
}
```

- [ ] **Step 2: Verify in browser**

Click the Threats tab. Expected: Abuse.ch panel loads C2 IPs from Feodo Tracker (format `1.2.3.4:443`) and IOCs from ThreatFox. Online C2s show red "● ONLINE" status. Software chip matches show ⚡ prefix.

- [ ] **Step 3: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: implement Abuse.ch active threats panel (Feodo + ThreatFox)"
```

---

## Task 7: Implement `loadRansomware()`

**Files:**
- Modify: `cyber-dashboard.html` — add `loadRansomware()` function after `loadAbuse()`

- [ ] **Step 1: Add `loadRansomware()` function**

Add after `loadAbuse()` and before the `// ─── Panel 5: Threat Intel Lookup` comment:

```javascript
// ─── Threats Tab: Ransomware Activity ────────────────────────────────────────
async function loadRansomware() {
  setLoading('ransomware-body');
  try {
    const resp = await fetch('https://api.ransomware.live/recentvictims');
    if (!resp.ok) throw new Error('HTTP ' + resp.status);
    const data = await resp.json();
    const victims = (Array.isArray(data) ? data : [])
      .sort((a, b) => new Date(b.discovered || 0) - new Date(a.discovered || 0))
      .slice(0, 15);
    if (!victims.length) {
      document.getElementById('ransomware-body').innerHTML = '<div class="loading-state">No recent victims.</div>';
      return;
    }
    const kw = getIndustryKW();
    const ul = document.createElement('ul');
    ul.className = 'feed-list';
    victims.forEach(v => {
      const activity = v.activity || '';
      const relevant = kw.test((v.post_title || '') + ' ' + activity);
      const li = document.createElement('li');
      li.className = 'feed-item' + (relevant ? ' industry-relevant' : '');
      li.innerHTML = `
        <div style="font-weight:600;margin-bottom:2px;">${escHtml(v.post_title || 'Unknown')}</div>
        <div class="feed-item-meta">
          <span style="color:var(--critical);">${escHtml(v.group_name || '?')}</span>
          ${v.country ? `<span>${escHtml(v.country)}</span>` : ''}
          ${activity ? `<span>${escHtml(truncate(activity, 40))}</span>` : ''}
          <span>${v.discovered ? timeAgo(v.discovered) : ''}</span>
          ${relevant ? '<span style="color:var(--accent);">★ relevant</span>' : ''}
        </div>`;
      ul.appendChild(li);
    });
    document.getElementById('ransomware-body').innerHTML = '';
    document.getElementById('ransomware-body').appendChild(ul);
  } catch(e) {
    setError('ransomware-body', 'https://www.ransomware.live');
  }
}
```

- [ ] **Step 2: Verify in browser**

Click Threats tab. Expected: Ransomware panel loads recent victims. Each entry shows victim name, group name in red, country, and time ago. Industry-relevant entries highlighted.

If `api.ransomware.live` fails CORS: error state shows with link to ransomware.live — this is expected behavior per spec.

- [ ] **Step 3: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: implement Ransomware Activity panel"
```

---

## Task 8: Wire lazy loading for Threats tab

**Files:**
- Modify: `cyber-dashboard.html` — add `_threatsLoaded` flag, modify `switchTab()`

- [ ] **Step 1: Add `_threatsLoaded` flag and modify `switchTab()`**

Find `switchTab()` (around line 596):
```javascript
function switchTab(name) {
  document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.nav-item[data-tab]').forEach(b => b.classList.remove('active'));
  document.getElementById('tab-' + name).classList.add('active');
  const btn = document.querySelector('.nav-item[data-tab="' + name + '"]');
  if (btn) btn.classList.add('active');
}
```

Replace with:
```javascript
let _threatsLoaded = false;
function switchTab(name) {
  document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.nav-item[data-tab]').forEach(b => b.classList.remove('active'));
  document.getElementById('tab-' + name).classList.add('active');
  const btn = document.querySelector('.nav-item[data-tab="' + name + '"]');
  if (btn) btn.classList.add('active');
  if (name === 'threats' && !_threatsLoaded) {
    _threatsLoaded = true;
    loadAbuse();
    loadRansomware();
  }
}
```

- [ ] **Step 2: Reset flag when Refresh buttons are used**

The Refresh buttons on the Threats panels call `loadAbuse()` and `loadRansomware()` directly — no flag change needed. The flag only prevents re-fetching on subsequent tab visits, not on explicit refresh. This is correct behavior — no change required.

- [ ] **Step 3: Verify in browser**

1. Load dashboard on Briefing tab — no Threats API calls in Network tab.
2. Click Threats tab — both panels load.
3. Click Briefing tab then Threats tab again — no duplicate network calls (check Network tab).
4. Click ↻ Refresh on Abuse panel — reloads just that panel.

- [ ] **Step 4: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: lazy-load Threats tab on first visit"
```

---

## Task 9: Shodan InternetDB inline result in Threat Intel Lookup

**Files:**
- Modify: `cyber-dashboard.html` — add `#idb-result` div to TI panel HTML, add `loadInternetDB()`, modify `tiDetect()` and `tiRender()`

- [ ] **Step 1: Add `#idb-result` div to TI Lookup panel**

In the Intel tab, find `<div id="ti-services"></div>` (around line 521):
```html
            <div id="ti-services"></div>
```

Replace with:
```html
            <div id="ti-services"></div>
            <div id="idb-result" style="display:none;"></div>
```

- [ ] **Step 2: Add `loadInternetDB()` function**

Add after `tiOpenAll()` and before the `tiRender()` call at the bottom of the TI section:

```javascript
async function loadInternetDB(ip) {
  const el = document.getElementById('idb-result');
  if (!el) return;
  el.style.display = 'none';
  el.innerHTML = '';
  try {
    const resp = await fetch('https://internetdb.shodan.io/' + encodeURIComponent(ip));
    if (!resp.ok) return;
    const d = await resp.json();
    if (d.detail) return;
    const parts = [];
    if (d.ports?.length)     parts.push(`<div class="idb-row"><span class="idb-label">Ports</span>${escHtml(d.ports.join(', '))}</div>`);
    if (d.hostnames?.length) parts.push(`<div class="idb-row"><span class="idb-label">Hostnames</span>${escHtml(d.hostnames.join(', '))}</div>`);
    if (d.tags?.length)      parts.push(`<div class="idb-row"><span class="idb-label">Tags</span>${d.tags.map(t => `<span class="idb-tag">${escHtml(t)}</span>`).join(' ')}</div>`);
    if (d.vulns?.length)     parts.push(`<div class="idb-row"><span class="idb-label">Vulns</span>${d.vulns.map(cve => `<a class="feed-item-title" href="https://nvd.nist.gov/vuln/detail/${escHtml(cve)}" target="_blank" rel="noopener">${escHtml(cve)}</a>`).join(' ')}</div>`);
    if (d.cpes?.length)      parts.push(`<div class="idb-row"><span class="idb-label">CPEs</span>${escHtml(truncate(d.cpes.join(', '), 120))}</div>`);
    if (!parts.length) return;
    el.innerHTML = `<div style="font-size:11px;color:var(--text-muted);margin-bottom:6px;font-weight:600;">Shodan InternetDB</div>${parts.join('')}`;
    el.style.display = 'block';
  } catch(_) {}
}
```

- [ ] **Step 3: Modify `tiDetect()` to call `loadInternetDB()` for IP type**

Find `tiDetect()` (around line 1334):
```javascript
function tiDetect() {
  const val = document.getElementById('ti-val').value.trim();
  if (val) document.getElementById('ti-type').value = tiDetectType(val);
  tiRender();
}
```

Replace with:
```javascript
function tiDetect() {
  const val = document.getElementById('ti-val').value.trim();
  if (val) document.getElementById('ti-type').value = tiDetectType(val);
  tiRender();
  const type = document.getElementById('ti-type').value;
  if (type === 'ip' && val) {
    loadInternetDB(val);
  } else {
    const el = document.getElementById('idb-result');
    if (el) { el.style.display = 'none'; el.innerHTML = ''; }
  }
}
```

- [ ] **Step 4: Modify `tiRender()` to clear InternetDB result when type is not IP**

Find `tiRender()` (around line 1340):
```javascript
function tiRender() {
  const val = document.getElementById('ti-val').value.trim();
  const type = document.getElementById('ti-type').value;
  const svcs = TI_SERVICES[type] || [];
```

Replace the first three lines with:
```javascript
function tiRender() {
  const val = document.getElementById('ti-val').value.trim();
  const type = document.getElementById('ti-type').value;
  if (type !== 'ip') {
    const el = document.getElementById('idb-result');
    if (el) { el.style.display = 'none'; el.innerHTML = ''; }
  }
  const svcs = TI_SERVICES[type] || [];
```

- [ ] **Step 5: Verify in browser**

1. Go to Intel tab.
2. Type a known IP (e.g. `8.8.8.8`) into the TI Lookup input and click Detect.
   Expected: Service buttons appear AND a "Shodan InternetDB" block appears below showing ports, hostnames, tags, etc.
3. Type a domain (e.g. `google.com`) and click Detect.
   Expected: InternetDB block disappears, service buttons change to domain services.
4. Type a private IP (e.g. `192.168.1.1`) and click Detect.
   Expected: Service buttons appear, InternetDB block stays hidden (404 = no data).

- [ ] **Step 6: Commit**

```bash
git add cyber-dashboard.html
git commit -m "feat: add Shodan InternetDB inline result to Threat Intel Lookup"
```

---

## Self-Review Notes

Spec requirements checked:
- [x] EPSS async overlay on KEV — Task 2
- [x] EPSS async overlay on NVD — Task 3
- [x] `loadEPSS()` function with EPSS slot injection — Task 2
- [x] CISA Advisories panel on Briefing tab — Task 4
- [x] CISA wired into refreshAll + applySettings — Task 4
- [x] Threats tab nav + HTML — Task 5
- [x] Abuse.ch (Feodo + ThreatFox) combined panel — Task 6
- [x] Ransomware.live panel with CORS fallback — Task 7
- [x] Lazy loading with `_threatsLoaded` flag — Task 8
- [x] Shodan InternetDB inline in TI Lookup — Task 9
- [x] CSS for EPSS badges and InternetDB block — Task 1
- [x] Software chip matching on Abuse.ch malware names — Task 6
- [x] Industry keyword highlighting on Ransomware + CISA — Tasks 4, 7
- [x] `setError` fallback on all new loaders — Tasks 4, 6, 7
