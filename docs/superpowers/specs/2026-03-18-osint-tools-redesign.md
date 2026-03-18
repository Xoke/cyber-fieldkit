# OSINT Tools Redesign — Design Spec
*Date: 2026-03-18*

## Overview

Redesign `osint-tools.html` to support an investigation workflow: search once, step through services in an embedded panel, and take temporary notes — without tab-switching.

## Problem

The current workflow requires the user to:
1. Enter search terms
2. Click a service button → new tab opens
3. View results, mentally note findings
4. Return to the tool tab
5. Click the next service → repeat

This tab-switching is tedious for real investigation work where you need to compare results across multiple sources.

## Goals

- Results load inline (no leaving the page)
- Notes scratch-pad for cross-referencing during an investigation
- Smooth fallback when a site blocks iframe embedding

## Non-Goals

- Pulling structured data from external sites (CORS-blocked, no public APIs)
- Persisting notes between sessions
- Replacing the "Open in new tab" pattern for sites that block iframes

## Layout

Two-column split below the existing header and tab bar:

**Left panel (~35% width, independently scrollable)**
- Input card (unchanged — same fields, same validation/shake behavior)
- Service buttons grouped by category (unchanged)
- "Investigation Notes" section at the bottom of the panel

**Right panel (~65% width, fills viewport height)**
- Iframe that loads service results when a button is clicked
- Small header bar above the iframe showing:
  - Currently loaded service name
  - "Open in new tab" button (always present as fallback)
- Placeholder state when nothing is loaded: *"Click a service to load results here"*

The existing header (app name + theme switcher) and tab bar (Names, Phone, Email, Username) are unchanged.

## Behavior Changes

### Service buttons
- Clicking a button loads the URL into the right-panel iframe instead of `window.open(..., '_blank')`
- The iframe header updates to show the service name
- The previously active button gets a subtle "active" highlight so the user knows what's loaded

### Blocked iframe detection
- When a site refuses to embed (X-Frame-Options / CSP), the iframe will appear blank
- Reliable auto-detection is not possible: `iframe.contentDocument` always throws a cross-origin security error regardless of whether the site blocked embedding, so there is no way to distinguish a blocked embed from a successfully loaded cross-origin page
- Instead: make the "Open in new tab" button in the panel header prominent and always visible; add a static helper note below the iframe: *"If the page above is blank, the site blocks embedding — use the button above."*
- No overlay, no timeout logic — the user handles it manually via the always-present button

### "Open All" button
- Removed. No longer useful in the single-panel investigation workflow.

### "Clear" button
- Behavior unchanged — clears input fields only

## Investigation Notes

- Location: below service buttons in the left panel
- A `<textarea>` with label "Investigation Notes"
- Vertically resizable (CSS `resize: vertical`)
- A small "Clear Notes" button resets it to empty (distinct label from the input card's "Clear" button)
- Session-only — no persistence, wiped on page close/refresh
- Placeholder text: *"Paste addresses, phone numbers, notes to cross-reference..."*

## Technical Notes

- All changes are within the single `osint-tools.html` file — no new files
- Layout uses CSS flexbox (`flex-direction: row` on the content area)
- Left panel: `overflow-y: auto`, fixed width or flex basis ~35% — fixed, not user-resizable (no drag handle)
- Right panel: `flex: 1`, iframe is `width: 100%; height: 100%; border: none`
- Minimum viable layout width: ~900px. No mobile/narrow viewport support required — this is a desktop investigation tool. No responsive collapse behavior needed.
- Active button highlight (the service currently loaded in the iframe) clears when the user switches tabs, since switching tabs changes the search context; the iframe retains whatever URL was last loaded (does not reset to placeholder on tab switch)
- No changes to theme system, tab switching logic, input validation, shake animation, or `getArgs()` / `renderServices()` functions
- `openAll()` function and its button are removed

## Out of Scope (Future)

- Vehicle / Plate tab (tracked in `osint-tools-todo.md`)
- Additional services per tab
- Inline API-backed results (would require backend proxy)
