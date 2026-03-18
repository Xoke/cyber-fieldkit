# OSINT Tools Split-Pane Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Redesign `osint-tools.html` into a split-pane layout where service buttons load results in an inline iframe (left panel: search + notes, right panel: results), replacing the current "open in new tab" workflow.

**Architecture:** Single-file HTML app — all changes are in `osint-tools.html`. The content area becomes a flex-row split: a scrollable left panel (inputs, service buttons, notes) and a fixed-height right panel (iframe + header). Service button clicks set `iframe.src` instead of calling `window.open()`. No backend, no build step.

**Tech Stack:** Vanilla HTML/CSS/JS, no dependencies, no build system. Open directly in browser to test.

> **Note on testing:** This project has no test framework. All verification steps are browser checks — open `osint-tools.html` directly in a browser and confirm the described behavior.

---

## File Map

| File | Action | What changes |
|------|--------|--------------|
| `osint-tools.html` | Modify | CSS, HTML structure, JavaScript — all in one file |

---

### Task 1: Add split-pane CSS

**Files:**
- Modify: `osint-tools.html` — `<style>` block (after line 134, before `</style>`)

Add CSS for the split layout, right panel, iframe container, and notes section. The existing `#content` styles stay; we add new rules beneath them.

- [ ] **Step 1: Add CSS rules to the `<style>` block**

Find the line:
```css
#content { flex: 1; overflow-y: auto; padding: 20px 24px; }
```

Replace it with:
```css
#content { flex: 1; display: flex; overflow: hidden; }

/* Split layout */
.left-panel {
  width: 36%; min-width: 280px; overflow-y: auto;
  padding: 20px 16px 20px 24px; flex-shrink: 0;
  border-right: 1px solid var(--border);
}
.right-panel {
  flex: 1; display: flex; flex-direction: column;
  overflow: hidden; background: var(--bg-alt);
}

/* Iframe panel */
.iframe-header {
  display: flex; align-items: center; gap: 8px;
  padding: 8px 12px; background: var(--bg-alt);
  border-bottom: 1px solid var(--border); flex-shrink: 0;
}
.iframe-service-name {
  font-size: 12px; font-weight: 600; color: var(--text-muted);
  flex: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
#result-frame {
  flex: 1; border: none; background: var(--bg);
}
.iframe-hint {
  font-size: 11px; color: var(--text-muted); padding: 6px 12px;
  flex-shrink: 0; border-top: 1px solid var(--border);
}
.iframe-placeholder {
  flex: 1; display: flex; align-items: center; justify-content: center;
  color: var(--text-muted); font-size: 13px;
}

/* Notes section */
.notes-section { margin-top: 20px; }
.notes-section h2 {
  font-size: 13px; font-weight: 600; color: var(--text-muted);
  text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 8px;
}
.notes-section textarea {
  width: 100%; min-height: 120px; padding: 8px 10px;
  background: var(--bg-ctrl); border: 1px solid var(--border);
  border-radius: 5px; color: var(--text); font-size: 12px;
  font-family: inherit; resize: vertical; outline: none;
  transition: border-color 0.15s;
}
.notes-section textarea:focus { border-color: var(--accent); }
.notes-actions { display: flex; justify-content: flex-end; margin-top: 6px; }

/* Active service button */
.svc-btn.active {
  background: var(--accent); color: #fff; border-color: var(--accent);
}
```

- [ ] **Step 2: Verify CSS loads without errors**

Open `osint-tools.html` in a browser. Open DevTools console — no CSS errors. The layout will look broken (no split yet) because the HTML hasn't been restructured. That's expected.

- [ ] **Step 3: Commit**

```bash
git add osint-tools.html
git commit -m "feat: add split-pane CSS (layout, iframe panel, notes section)"
```

---

### Task 2: Restructure HTML — split layout + right panel

**Files:**
- Modify: `osint-tools.html` — `<div id="content">` section

Wrap all tab panels in a `.left-panel` div. Add a `.right-panel` div after them containing the iframe header, iframe, and hint text.

- [ ] **Step 1: Wrap tab panels in `.left-panel`**

Find:
```html
<div id="content">

  <!-- NAMES TAB -->
```

Replace with:
```html
<div id="content">
<div class="left-panel">

  <!-- NAMES TAB -->
```

Then find the closing `</div>` of the content block — it's the `</div>` that closes `id="content"` (after the username tab panel's closing `</div>`). Insert `</div>` before it to close `.left-panel`:

```html
  </div><!-- end tab-username -->

</div><!-- end left-panel -->

</div><!-- end content -->
```

- [ ] **Step 2: Add the right panel**

Between `</div><!-- end left-panel -->` and `</div><!-- end content -->`, insert:

```html
<div class="right-panel">
  <div class="iframe-header">
    <span class="iframe-service-name" id="iframe-service-label">No service loaded</span>
    <button class="btn-secondary" id="iframe-newtab-btn" style="display:none;" onclick="openCurrentInNewTab()">Open in new tab ↗</button>
  </div>
  <div class="iframe-placeholder" id="iframe-placeholder">
    Click a service to load results here
  </div>
  <iframe id="result-frame" style="display:none;" src="about:blank"></iframe>
  <p class="iframe-hint" id="iframe-hint" style="display:none;">
    If the page above is blank, the site blocks embedding — use the "Open in new tab" button above.
  </p>
</div>
```

- [ ] **Step 3: Verify layout in browser**

Open `osint-tools.html`. You should see:
- Left column (~36%): input card + service buttons
- Right column: gray area with "Click a service to load results here" centered
- Theme switcher and tab bar unchanged
- All four themes still work

- [ ] **Step 4: Commit**

```bash
git add osint-tools.html
git commit -m "feat: restructure HTML into split-pane layout with iframe right panel"
```

---

### Task 3: Remove "Open All" button from all tabs

**Files:**
- Modify: `osint-tools.html` — four `input-card-actions` blocks + `<script>` block

The `openAll()` function and its button are removed. The "Clear" button and tip text stay.

- [ ] **Step 1: Remove "Open All" buttons from HTML**

In each of the four `input-card-actions` blocks (names, phone, email, username), remove:
```html
<button class="btn-primary" onclick="openAll('names')">Open All</button>
```
(with the appropriate tab name in each). Leave the `btn-secondary` Clear button and `tip-text` span in place.

After removing all four, also remove the `tip-text` span (it told users to allow popups — no longer needed):
```html
<span class="tip-text">Allow popups in your browser</span>
```

- [ ] **Step 2: Remove `openAll()` function and Enter key wiring from JS**

In the `<script>` block, delete the entire `openAll()` function (lines starting with `// ── Open all` through the closing `}`):

```js
// ── Open all ──────────────────────────────────────────────────────────────
function openAll(tab) {
  ...
}
```

Also delete the `addEnterKey` helper function and the four lines at the bottom of the script that call it:

```js
// ── Enter key triggers Open All ───────────────────────────────────────────
function addEnterKey(inputId, tab) {
  const el = document.getElementById(inputId);
  if (el) el.addEventListener('keydown', e => { if (e.key === 'Enter') openAll(tab); });
}
['n-first','n-last','n-city','n-state'].forEach(id => addEnterKey(id, 'names'));
addEnterKey('p-phone', 'phone');
addEnterKey('e-email', 'email');
addEnterKey('u-user', 'username');
```

> If you leave these in place and `openAll` is removed, pressing Enter in any input will throw an uncaught TypeError.

- [ ] **Step 3: Verify in browser**

Open `osint-tools.html`. The Names tab input card should show only the "Clear" button. No "Open All" button on any tab. No console errors.

- [ ] **Step 4: Commit**

```bash
git add osint-tools.html
git commit -m "feat: remove Open All button — replaced by iframe workflow"
```

---

### Task 4: Wire service buttons to load in iframe

**Files:**
- Modify: `osint-tools.html` — `<script>` block, `renderServices()` function and new helpers

Service button clicks now call `loadService(url, label)` instead of `window.open()`. A new `currentServiceUrl` variable tracks the loaded URL for the "Open in new tab" fallback.

- [ ] **Step 1: Add `loadService()` and `openCurrentInNewTab()` functions**

In the `<script>` block, after the `// ── Service definitions` comment and before `const SERVICES = {`, add:

```js
// ── State ──────────────────────────────────────────────────────────────────
let currentServiceUrl = null;
let activeServiceBtn = null;

// ── Load service in iframe ─────────────────────────────────────────────────
function loadService(url, label, btn) {
  currentServiceUrl = url;

  // Update iframe
  const frame = document.getElementById('result-frame');
  const placeholder = document.getElementById('iframe-placeholder');
  const hint = document.getElementById('iframe-hint');
  frame.src = url;
  frame.style.display = 'block';
  placeholder.style.display = 'none';
  hint.style.display = 'block';

  // Update header
  document.getElementById('iframe-service-label').textContent = label;
  document.getElementById('iframe-newtab-btn').style.display = 'inline-block';

  // Update active button highlight
  if (activeServiceBtn) activeServiceBtn.classList.remove('active');
  activeServiceBtn = btn;
  btn.classList.add('active');
}

function openCurrentInNewTab() {
  if (currentServiceUrl) window.open(currentServiceUrl, '_blank');
}
```

- [ ] **Step 2: Update `renderServices()` to call `loadService()`**

Find the button click listener inside `renderServices()`:
```js
btn.addEventListener('click', () => {
  const args = getArgs(tab);
  if (!args) { shakeCard(tab); return; }
  const url = item.url(...args);
  window.open(url, '_blank');
});
```

Replace with:
```js
btn.addEventListener('click', () => {
  const args = getArgs(tab);
  if (!args) { shakeCard(tab); return; }
  const url = item.url(...args);
  loadService(url, item.label, btn);
});
```

- [ ] **Step 3: Verify in browser**

Open `osint-tools.html`. Enter "Jane Doe" in the Names tab.
- Click "TruePeopleSearch" → right panel should show the site loading, header label updates to "TruePeopleSearch", "Open in new tab ↗" button appears
- Click "Google" → right panel loads Google search, header updates
- Click "Open in new tab ↗" → opens current service in a new tab
- If a site blocks the iframe (right panel is blank) → hint text is visible at the bottom

- [ ] **Step 4: Commit**

```bash
git add osint-tools.html
git commit -m "feat: service buttons load in iframe with active highlight and new-tab fallback"
```

---

### Task 5: Clear active button on tab switch

**Files:**
- Modify: `osint-tools.html` — `<script>` block, tab switching logic

When the user switches tabs, the active service button highlight clears (the iframe retains its current content).

- [ ] **Step 1: Update the tab switching event listener**

Find the tab click listener:
```js
document.querySelectorAll('.tab-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
    btn.classList.add('active');
    document.getElementById('tab-' + btn.dataset.tab).classList.add('active');
  });
});
```

Replace with:
```js
document.querySelectorAll('.tab-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
    btn.classList.add('active');
    document.getElementById('tab-' + btn.dataset.tab).classList.add('active');
    // Clear active service button highlight on tab switch
    if (activeServiceBtn) { activeServiceBtn.classList.remove('active'); activeServiceBtn = null; }
  });
});
```

- [ ] **Step 2: Verify in browser**

Open `osint-tools.html`. Enter "Jane Doe", click "Google" in Names tab → Google button highlights. Switch to Phone tab → Google button highlight is gone. Switch back to Names tab → still no highlight (iframe still shows last loaded page).

- [ ] **Step 3: Commit**

```bash
git add osint-tools.html
git commit -m "feat: clear active service button highlight on tab switch"
```

---

### Task 6: Add Investigation Notes section

**Files:**
- Modify: `osint-tools.html` — HTML (inside `.left-panel`, after the last tab panel) + `<script>` block

The notes textarea is rendered once, always visible in the left panel regardless of which tab is active.

- [ ] **Step 1: Add notes HTML**

Find `</div><!-- end left-panel -->` (added in Task 2). Insert this immediately before it:

```html
  <!-- INVESTIGATION NOTES -->
  <div class="notes-section">
    <h2>Investigation Notes</h2>
    <textarea id="investigation-notes" placeholder="Paste addresses, phone numbers, notes to cross-reference..."></textarea>
    <div class="notes-actions">
      <button class="btn-secondary" onclick="clearNotes()">Clear Notes</button>
    </div>
  </div>
```

- [ ] **Step 2: Add `clearNotes()` to JS**

In the `<script>` block, after the `clearTab()` function, add:

```js
// ── Clear notes ────────────────────────────────────────────────────────────
function clearNotes() {
  document.getElementById('investigation-notes').value = '';
}
```

- [ ] **Step 3: Verify in browser**

Open `osint-tools.html`.
- Notes section is visible below the service buttons on all tabs
- Typing in the textarea works
- "Clear Notes" button empties it
- Switching tabs does not clear the notes
- Notes are gone after page refresh (session-only, expected)

- [ ] **Step 4: Commit**

```bash
git add osint-tools.html
git commit -m "feat: add session-only Investigation Notes section to left panel"
```

---

### Task 7: Final smoke test

- [ ] **Step 1: Full browser walkthrough**

Open `osint-tools.html` and verify:

1. **Layout**: Two-column split renders correctly; left panel scrolls independently; right panel fills remaining width
2. **Names tab**: Enter first + last name, click a service → loads in right panel with correct label and "Open in new tab" button
3. **Phone tab**: Enter a 10-digit number, click a service → loads in iframe
4. **Email tab**: Enter an email, click a service → loads in iframe
5. **Username tab**: Enter a username, click a service → loads in iframe
6. **Active highlight**: Clicked button highlights; clicking another button moves highlight; switching tab clears highlight
7. **Open in new tab**: Clicking "Open in new tab ↗" opens the current service URL in a new browser tab
8. **Blocked sites**: For any site that shows blank in the iframe, hint text is visible at the bottom
9. **Notes**: Type notes, switch tabs, notes persist; Clear Notes empties the textarea
10. **Themes**: All four themes (light/sepia/dusk/dark) render correctly with new layout
11. **Validation**: Clicking a service with empty required fields triggers shake animation
12. **Enter key**: Pressing Enter in any input field does NOT trigger `openAll` (removed) — it should do nothing now (that's fine)
13. **No console errors**: DevTools console is clean

- [ ] **Step 2: Fix any issues found**

Fix inline, then commit with descriptive message.

- [ ] **Step 3: Final commit if needed**

```bash
git add osint-tools.html
git commit -m "fix: [describe issue]"
```
