# VM Symptom & Trigger Questionnaire

Single-file, patient-facing self-report tool for vestibular migraine (VM). Everything lives in `index.html` — no build step, no dependencies. Deployed via GitHub Pages at https://jakalnz.github.io/vm-q/ from the `main` branch root.

## Structure

- `PHASES` — the 4 timeframes rated (pre-ictal/ictal/post-ictal/inter-ictal), each with duration options.
- `GROUPS` / `SYMS` — symptom categories and the ~44 symptoms in them, each with per-severity (1–5) descriptive anchors.
- Overview tab: a phase × symptom severity/frequency matrix, grouped into **collapsible accordion sections per category** (collapsed by default, auto-expands when a symptom in it gets rated).
- Nine detail tabs with chip/radio follow-up questions (`sec()` helper), a Triggers tab, and a Summary tab (`renderSummary()`, `computeFlags()`) that must stay live-updated from *every* input surface, not just its own tab.
- Mobile (<640px): matrix becomes stacked cards (`renderMobileMatrix()`), nav becomes a `<select>` instead of the tab row. Desktop and mobile share the same state (`sev`, `freq`, `durSel`, `trgSel`, `detailAns`, `grpExpanded`) and both re-render on every change to stay in sync.
- No live backend yet — Summary tab offers JSON/text downloads only. `submitProfile(payload)` is a deliberate no-op stub marking the seam for a future institution-configured endpoint (Google Sheets Apps Script / REDCap / etc).

## Known gotchas (already hit once each — don't reintroduce)

- **`position:sticky` is unreliable here.** The nav bar uses `position:fixed` instead. The phase-column headers use sticky successfully, but only after removing `overflow-x:auto` from `.mat-wrap` — per the CSS overflow spec, a non-`visible` `overflow-x` forces `overflow-y` to compute as `auto` too, silently turning the wrapper into its own (non-scrolling) scroll container and breaking sticky's "nearest scrolling ancestor" resolution. If you add horizontal scroll back to that wrapper, sticky headers will silently break again.
- Any inline `el.style.position = 'relative'` set by JS popovers will permanently override a CSS `position:sticky`/`fixed` rule on that same element (inline beats any stylesheet rule). Check for this before adding new popover-anchoring code.
- Colors are chosen for people with migraine/visual sensitivity: avoid pure black/white or high-contrast/saturated combinations. Category header bars use muted `--bar-bg`/`--bar-tx` tokens, not `--text-primary`/`--surface-1`.

## Testing

No browser test runner in-repo. Verification approach used so far: a jsdom + Node scratchpad harness (dispatch DOM events, assert on resulting DOM/state) — the Chrome MCP browser tool has been unreliable across sessions (hangs on screenshot/read_page even on plain pages), so don't rely on it being available; fall back to jsdom if it doesn't respond after one retry.

When testing state changes across re-renders, always re-query elements from `document` after an action rather than holding onto a reference from before — `renderMatrix()`/`renderMobileMatrix()` fully rebuild their containers on every change, so old references go stale and give false negatives.

## Content/taxonomy notes

Symptom grouping intentionally keeps head pain + light sensitivity + nausea together (ICHD-3 diagnostic tetrad), even though light sensitivity is arguably a "sensory hypersensitivity" symptom rather than a headache one. There's a TODO comment above `GROUPS` in `index.html` proposing a future unified "Sensory hypersensitivity" group (light + smell + sound sensitivity + allodynia) — not yet actioned, flagged for a future session.

## Git

Repo has a remote (`origin` → `github.com/jakalnz/vm-q`). Push only after the user has approved committing/pushing for that turn — don't assume standing permission across sessions.
