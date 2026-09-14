---
name: run-dynamic-scheduler
description: How to open, drive, and test the single-file Dynamic Scheduler HTML tool (index.html) — no build/server needed, but exporting/CSV testing and drag-and-drop have real gotchas.
---

# Dynamic Scheduler (single-file HTML app)

The whole app is one static file: `index.html` (Traditional
Chinese / English bilingual, dark/light theme). No build step, no server,
no `package.json` for the app itself. Everything — markup, CSS, and a
single IIFE `<script>` — lives in that one file.

## Running it

Just open the file directly in a browser:

```
file:///C:/Users/tu-hs/OneDrive/文件/2022_0308_MASA/2022-0708/project_claude_dynamic_schedule/index.html
```

The filename has a space and parentheses — always build the URL with
`encodeURI(...)` (or `%20`/`%28`/`%29`) when constructing it in a script,
otherwise navigation silently fails or 404s.

State persists in the browser's `localStorage` under these keys:
`dsched_state` (tasks/resources/settings JSON), `dsched_lang` (`zh`/`en`),
`dsched_theme` (`dark`/`light`), `dsched_zoom` (Gantt zoom multiplier).
Each is written only when an action actually calls `saveState()`/
`storage.set(...)` — a fresh load with no prior localStorage returns
`null`, that's expected, not a bug.

## Driving it for screenshots / interaction tests

There's no `chromium-cli` on this machine and no project `package.json`.
The working recipe used repeatedly in this project:

```bash
cd <scratchpad dir>
npm init -y
npm install playwright
npx playwright install chromium   # first time only; can be silent on success
node your_script.js
```

Then in the script:

```js
const { chromium } = require('playwright');
const FILE = 'file://' + encodeURI('C:/.../index.html');
const browser = await chromium.launch();
const page = await browser.newPage({ viewport: { width: 1400, height: 900 } });
page.on('pageerror', e => console.log('pageerror:', e.message));
page.on('console', m => { if (m.type() === 'error') console.log('console:', m.text()); });
await page.goto(FILE);
await page.waitForSelector('#ganttRoot .taskbar');
```

Always check `pageerror`/`console --errors` after interacting — the app
has no build-time type checking, so a typo only shows up at runtime.

### Import/export is unified, with a format switch — not separate buttons

There is exactly one Export button and one Import button, both in the
header (visible on every tab). A `<select id="exportFormatSel">` next
to Export picks `json` (full state backup — importing it *replaces*
`state` wholesale) or `csv` (task list only — importing it *syncs*:
`importTasksCSV` matches rows to existing tasks by `name` and updates
them in place, adds unmatched names as new tasks, and auto-creates any
resource name it doesn't recognize). The single hidden `#importFile`
input accepts both `.json` and `.csv`; its change handler sniffs the
extension/MIME type to decide which importer to call — there is no
separate CSV import button/input anymore. If you add a third export
format, extend this same select + sniffing logic rather than adding
another button pair.

### The Gantt "expand" modal moves the real DOM, it doesn't clone it

`#expandGanttBtn` doesn't build a second Gantt — it re-parents the
*live* `#ganttPane` (which wraps the legend/controls, `.gantt-scroll`,
and the drag-hint note as one block) into `#ganttZoomModal`'s
`#ganttZoomHost`, and `closeGanttZoom()` re-parents it straight back
into `#view-gantt`. This is why zoom, filters, drag-and-drop, and
dependency arrows keep working identically inside the modal: it's the
same nodes with the same listeners, not a duplicate. The first attempt
only moved `.gantt-scroll` and left the zoom/today-jump buttons behind
in `.legend` — they were then visually and functionally blocked by the
modal overlay sitting on top of them. If you add more Gantt-view
controls, put them inside `#ganttPane`, or they'll have the same bug.

### The task/resource tables are inline-editable, not read-only rows

`renderTaskList`/`renderResList` build a plain CSS-grid table (`.et-row`
divs with matching `grid-template-columns`, not a real `<table>`) where
every cell is a live `<input>`/`<select>` bound directly to that task's
or resource's field. Each control commits on its native `change` event
(so typing in the name field doesn't trigger a re-render on every
keystroke — only on blur/selection-change), and the handler always does
`saveState(); render();` — the *whole* app re-renders on every single
field edit, same as every other mutation in this codebase. That's
intentional, not an oversight: don't try to "optimize" it into a
partial re-render, it'd be inconsistent with how the rest of the file
works and isn't needed at this data scale. When testing inline edits
with Playwright, `.fill()` alone won't commit the value — you need to
also fire blur (`el.evaluate(e => e.blur())` or `press('Tab')`) or the
`change` handler never runs and `localStorage` won't reflect the edit.

### Every state mutation must call `pushUndo()` first, or undo silently misses it

Undo/redo (`pushUndo`/`undo`/`redo`, near `saveState`) works by
snapshotting `JSON.stringify(state)` onto `undoStack` — there's no
mutation-observer or Proxy watching `state`, so it only knows about a
change if the handler explicitly calls `pushUndo()` as the *first*
line, before touching `state.tasks`/`state.resources`/anything else.
Every existing mutation site (inline table edits, drag-and-drop, the
task/resource modals, duplicate/delete, auto-schedule, CSV/JSON import,
horizon/skipWeekend changes, both sample-data reset buttons) already
does this — if you add a new way to change `state`, add `pushUndo()`
too, or that action will be silently un-undoable while everything
around it works fine. The global `Ctrl+Z`/`Ctrl+Y` keydown handler
skips this if `document.activeElement` is an `INPUT`/`TEXTAREA`/`SELECT`,
so it doesn't fight the browser's native undo while typing — keep that
guard if you touch the handler.

### Gotcha: exports use `<a download>` + Blob, not navigation

Both JSON and CSV export do `URL.createObjectURL(blob)` + a synthetic
`<a>` click. Playwright won't see this as a navigation — you must
capture it as a download:

```js
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.click('#csvExportBtn'),
]);
await download.saveAs('out.csv');
const text = require('fs').readFileSync('out.csv', 'utf8');
```

To test CSV *import*, use `page.setInputFiles('#csvImportFile', { name, mimeType, buffer })`
directly — no need to click the hidden `<input type=file>` first.

### Gotcha: CSV downloads need a UTF-8 BOM or Excel garbles Chinese text

Excel on Windows opens `.csv` files by guessing the system ANSI codepage
unless the file starts with a UTF-8 BOM (`EF BB BF`). Every CSV Blob
built for download (`exportTasksCSV`, the `csvTemplateBtn` handler)
prepends a literal `'﻿'` character before the content and uses
`type:'text/csv;charset=utf-8'`. If you add another CSV export path,
do the same — otherwise Traditional Chinese task/resource names come
out as 亂碼 (mojibake) when opened in Excel. `parseCSV` also strips a
leading BOM defensively (`FileReader.readAsText` already strips it in
practice, but don't rely on that alone if you change the read path).

### Gotcha: task bars overflow their grid cell — z-index matters

Task bars (`.taskbar`) are absolutely positioned inside their *start*
cell but their `width` can span multiple grid columns (multi-shift
tasks), visually overflowing into sibling `.daycell` elements that come
later in DOM order. Because those sibling cells are separate positioned
elements painted *after* the bar's parent in the same stacking context,
they paint on top of the bar's overflow region unless the bar has an
explicit `z-index` higher than `.daycell`'s (which is `auto`/0). This
previously broke hit-testing (clicking/dragging the middle of a wide bar
hit the grid cell underneath, not the bar). Current fix: `.taskbar` is
`z-index:2`, `.setupbar` is `z-index:1`, `.today-line` is `z-index:3`,
`.daycell`/`.daycell.over` stay at the default `auto`. If you touch this
CSS, re-verify with `document.elementFromPoint(x, y)` at the center of a
multi-slot bar — it must return the `.taskbar`, not a `.daycell`.

### Gotcha: toolbar button groups need `flex: 0 0 auto`, not `flex: 0`

Several header rows use the pattern
`<div class="row"><div>title</div><div style="flex:...">buttons</div></div>`.
`.row > div { flex:1; min-width:160px; }` is the base rule. If a button
group's inline style is just `flex:0` (i.e. `flex-basis:0%`), the flex
algorithm can collapse it down to its `min-width:160px` floor even
though its buttons need far more room — and because that div is itself
`display:flex; flex-wrap:wrap`, every button then wraps onto its own
line (looks like a vertical stack of buttons taking way too much
height). Fix: use `flex:0 0 auto` so the group sizes to its natural
content width instead of collapsing. Any new toolbar/button-group div
added to a `.row` should use `flex:0 0 auto`, not bare `flex:0`.

### Bilingual strings

UI text lives in the `STR.zh` / `STR.en` objects near the top of the
script, keyed by short camelCase names. Static text nodes are wired via
`data-i="keyName"` attributes + `applyLanguage()`'s generic
`querySelectorAll('[data-i]')` loop (sets `textContent`). Anything that
isn't `textContent` — `placeholder`, `title`, dynamic label suffixes —
needs an explicit line inside `applyLanguage()`. When adding new UI
text: add the key to **both** `STR.zh` and `STR.en`, then either give
the element `data-i` or add the explicit assignment.

### Structure notes

- Everything runs inside one `"use strict"` IIFE at the bottom of
  `<body>`. Functions are plain `function` declarations (hoisted), so
  call-before-definition in event listeners is fine as long as it only
  executes at runtime (click, etc.) — the whole script has already run
  once by then.
- Section comments (`/* ---------------- xxx ---------------- */`)
  mark logical areas: i18n, state, date/shift helpers, scheduler,
  rendering, drag, modals, CSV import/export, top-level control wiring,
  init. Put new code in the matching section rather than at the end of
  the file.
- No test suite exists. Validate JS changes with
  `node --check <extracted script>` for syntax, then drive the page
  with Playwright per above for behavior — there is no other way to
  catch a runtime bug in this project.
