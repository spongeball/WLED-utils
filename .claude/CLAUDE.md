# CLAUDE.md — WLED Preset Editor Design System

Complete reference for style, components, layout, breakpoints, and conventions used in `wled-preset-editor.html` and `index.html`. Any new page or feature must follow this document exactly.

---

## Stack

- Pure HTML + CSS + vanilla JS. No frameworks, no build tools, no external dependencies.
- Single-file output (`wled-preset-editor.html`). All CSS and JS is inline.
- `index.html` is a separate entry page with its own self-contained styles.

---

## Typography

| Role | Font | Size | Color |
|---|---|---|---|
| Base body | Verdana, Geneva, sans-serif | 13px | `#eee` |
| Page title (`h1`) | inherit, bold | 15px | `#fff` |
| Preset heading (`h2`) | inherit, bold | 15px | `#fff` |
| Section head | inherit, bold | 12px | `#ccc` |
| Labels | inherit | 12px | `#aaa` |
| Hints / secondary text | inherit | 10–11px | `#666` |
| PID badge (preset ID) | inherit | 10px | `#fff` on `#3a5f8a` |
| Preset name in sidebar | inherit | 13px | `#eee` |
| Monospace (raw JSON) | monospace | 11px | `#8bc34a` (green) |
| Bulk section titles | inherit, uppercase | 10px | `#5a8a5a` |

---

## Color Palette

### Base surfaces
| Token | Value | Usage |
|---|---|---|
| `--bg` | `#1a1a1a` | Page background, main area |
| `--surface-1` | `#222` | Sidebar background |
| `--surface-2` | `#252525` | Section cards, tab-pane background |
| `--surface-3` | `#2d2d2d` | Topbar, sidebar-head, section heads, modal head |
| `--surface-4` | `#2a2a2a` | Segment card heads |
| `--surface-5` | `#1e1e1e` | Segment cards |

### Borders
| Token | Value | Usage |
|---|---|---|
| `--border-strong` | `#444` | Topbar bottom, sidebar right, modal |
| `--border-mid` | `#3a3a3a` | Section cards, segment cards |
| `--border-soft` | `#333` | Raw JSON, modal diff, `hr.sep` |
| `--border-subtle` | `#252525` | Preset list item separator |

### Interactive / accent
| Token | Value | Usage |
|---|---|---|
| `--blue` | `#3a5f8a` | Primary button bg, focused preset, PID badge |
| `--blue-light` | `#5a8fc0` | Primary button border, accent, range slider |
| `--blue-hover` | `#4a6f9a` | Primary button hover |
| `--blue-dark` | `#2a5080` | Checked checkbox in sidebar |
| `--blue-muted` | `#1e3050` | Checked-but-not-focused preset row |
| `--red` | `#6a2020` | Danger button bg |
| `--red-border` | `#993333` | Danger button border |
| `--red-hover` | `#7a2a2a` | Danger button hover |
| `--red-text` | `#fcc` | Danger button text |
| `--green-bg` | `#1e2a1e` | Bulk bar background |
| `--green-border` | `#3a5a3a` | Bulk bar border |
| `--green-head` | `#253025` | Bulk bar head background |
| `--green-text` | `#8bc88a` | Bulk bar title, toast text |
| `--green-toast-bg` | `#2d4a2d` | Toast background |
| `--green-toast-border` | `#5a8a5a` | Toast border |

### Status
| State | Color |
|---|---|
| LED on (dot) | `#5a5` (green) |
| LED off (dot) | `#444` (grey) |
| Diff added | `#7bc87a` |
| Diff removed | `#c87a7a` |

---

## Layout

### Page structure (desktop)

```
html/body (flex column, height 100%, overflow hidden)
├── #topbar          (flex row, flex-shrink: 0)
├── #sidebar-toggle  (mobile only, hidden on desktop)
└── #layout          (flex row, flex: 1, min-height: 0)
    ├── #sidebar     (240px fixed, flex column)
    │   ├── #sidebar-head
    │   └── #preset-list
    └── #main        (flex: 1, overflow-y: auto)
        ├── #no-preset       (shown when no file loaded)
        ├── #bulk-bar        (shown when ≥1 preset checked)
        └── #editor          (shown when a preset is focused)
            ├── #preset-title-bar
            ├── #actions
            ├── .tabs
            ├── #editor-columns (wide screen: flex row)
            │   ├── #editor-col-general → #tab-general
            │   └── #editor-col-segments → #tab-segments
            └── #tab-raw (outside columns)
```

### Breakpoints

| Name | Range | Behaviour |
|---|---|---|
| Mobile | `≤ 700px` | Sidebar becomes collapsible drawer; topbar wraps to two rows (title full-width, Load+Export below); form rows stack vertically; seg-card grid collapses to single column |
| Desktop (default) | `701px – 1399px` | Sidebar fixed left; tabbed editor (General / Segments / Raw JSON) |
| Wide | `≥ 1400px` | General and Segments tabs hidden; panels shown side by side (General 320px fixed, Segments flex); Raw JSON tab remains clickable; Back button appears inside raw pane |

---

## Components

### Topbar `#topbar`

```html
<div id="topbar">
  <h1>⚡ WLED Preset Editor</h1>
  <span class="ver">presets.json</span>
  <div class="topbar-spacer" style="flex:1"></div>
  <button id="btn-load">…</button>
  <button id="btn-export" class="primary">…</button>
</div>
```

- `#btn-load` and `#btn-export`: height 27px, font 12px, padding `3px 12px`. Slightly larger than default buttons but not primary-sized.
- `.topbar-spacer`: `display:none` on mobile (spacer disappears so buttons fill the row).
- On mobile: `h1` gets `width: 100%` forcing Load/Export onto a second row where they each take `flex: 1`.

### Sidebar `#sidebar`

- Width 240px, `min-width: 200px`.
- `#sidebar-head`: single row (`flex-wrap: nowrap`, `overflow: hidden`), gap 3px. Contains: New (primary small), Del (danger small), All (small), None (small), `.sel-info` counter.
- `#preset-list`: `overflow-y: auto`, `flex: 1`.

### Preset item `.preset-item`

```html
<div class="preset-item [focused] [checked]">
  <div class="pchk"></div>      <!-- checkbox: click toggles bulk selection -->
  <span class="pid">1</span>
  <span class="pname">Name</span>
  <span class="pon [on]"></span> <!-- green dot if on -->
</div>
```

- Row click → focus (opens editor).
- `.pchk` click → toggle bulk check (independent of focus).
- Shift + row click → range-check from current focused to clicked.
- `.focused`: `#3a5f8a` background.
- `.checked`: `#1e3050` background.
- `.focused.checked`: `#3a5f8a` (focused wins).

### Bulk bar `#bulk-bar`

Shown when `checkedIds.size > 0`. Hidden by default (`display: none`), shown with class `.visible`.

Structure: `#bulk-bar-head` (title + count) + `#bulk-bar-body` (sections of `.brow` rows).

Each field row:
```html
<div class="brow">
  <label class="benbl"><input type="checkbox" id="be-FIELD-en"> Field label</label>
  <!-- input/range/select/checkbox for value -->
</div>
```

Fields are split into sections (`.bsec`) with `.bsec-title` headings: Preset, Seg 0 – LED range, Seg 0 – Flags, Seg 0 – Effect, Seg 0 – Colors, Seg 0 – Advanced.

### Section `.section`

Collapsible panel used in the General tab.

```html
<div class="section">
  <div class="section-head" onclick="toggleSection(this)">
    <span>Title</span><span class="arrow">▾</span>
  </div>
  <div class="section-body [collapsed]">
    <!-- .row elements -->
  </div>
</div>
```

### Form row `.row`

```html
<div class="row">
  <label>Field name</label>
  <input type="…">
  <span class="hint">optional hint</span>
</div>
```

- Label: `min-width: 120px`, color `#aaa`, 12px.
- On mobile: `flex-direction: column`, label has no min-width.
- `.row.row-inline`: forces horizontal layout on all screen sizes (used for checkbox groups like On/Rev/Mirror/Freeze and o1/o2/o3).

### Segment card `.seg-card`

The body uses a **CSS grid** with two columns: `130px 1fr`.

```html
<div class="seg-card">
  <div class="seg-card-head">…</div>
  <div class="seg-card-body">
    <label class="seg-label">Field</label>
    <div class="inputs">…</div>
    <!-- full-width row: -->
    <div class="full-row"><hr class="sep"></div>
  </div>
</div>
```

- All `seg-label` elements line up at 130px from the left edge.
- `.inputs`: flex row, gap 6px, flex-wrap.
- `.full-row`: `grid-column: 1 / -1`.
- On mobile: grid collapses to single column.

### Tabs `.tabs`

```html
<div class="tabs">
  <div class="tab active" onclick="switchTab('general', this)">General</div>
  <div class="tab" onclick="switchTab('segments', this)">Segments</div>
  <div class="tab" onclick="switchTab('raw', this)">Raw JSON</div>
</div>
<div class="tab-pane active" id="tab-general">…</div>
<div class="tab-pane" id="tab-segments">…</div>
<div class="tab-pane" id="tab-raw">…</div>
```

On wide screens (≥1400px): tabs 1 and 2 are hidden via `nth-child`; both panes are shown simultaneously in `#editor-columns`. Raw JSON tab is always visible. When raw is active, JS adds `.raw-active` to `#editor`, which hides `#editor-columns` and shows `#tab-raw`.

### Modal

```html
<div id="modal-overlay" class="[visible]">
  <div id="modal">
    <div id="modal-head"><span id="modal-title"></span><button id="modal-x">…</button></div>
    <div id="modal-body">
      <div id="modal-desc"></div>
      <div id="modal-diff">…</div>
    </div>
    <div id="modal-foot">
      <button onclick="closeModal()">Cancel</button>
      <button class="primary" id="modal-ok">Confirm</button>
    </div>
  </div>
</div>
```

- Width 500px, `max-width: 96vw`.
- `#modal-diff`: monospace, green text, scrollable, max-height 220px (140px on mobile).
- Diff lines: `.diff-add` green `#7bc87a`, `.diff-remove` red `#c87a7a`.
- Clicking overlay closes modal.

### Toast `#toast`

Fixed bottom-center, fades in/out with `.show` class. Duration 2 seconds. Colors match green success theme. `z-index: 2000`.

### Drop zone `#drop-zone`

- Dashed border `#444`, padding `80px 40px`, max-width 700px, full width.
- Hover: border `#666`, text `#999`.
- Dragover: border `#5a8fc0`, background `#1a2a3a`, `pulse-border` animation (border oscillates between `#5a8fc0` and `#8ab8e0`).
- On mobile: padding `32px 16px`.

---

## Buttons

### Sizes

| Class | Height | Font | Padding | Usage |
|---|---|---|---|---|
| default | 26px | 12px | `4px 12px` | Standard actions |
| `.small` | 20px | 11px | `2px 7px` | Sidebar controls (New, Del, All, None) |
| `#btn-load`, `#btn-export` | 27px | 12px | `3px 12px` | Topbar file actions |

### Variants

| Class | Background | Border | Text |
|---|---|---|---|
| default | `#3a3a3a` | `#555` | `#ddd` |
| `.primary` | `#3a5f8a` | `#5a8fc0` | `#fff` |
| `.danger` | `#6a2020` | `#993333` | `#fcc` |

All buttons use `display: inline-flex; align-items: center; gap: 5px` to accommodate SVG icons.

### Icons

All icons are inline SVG, `stroke="currentColor"`, `fill="none"`, `stroke-width` 1.5 (or 1.8 for close/×). Size 13×13px for standard buttons, 14×14px for topbar buttons, 11×11px for small close icons. `vertical-align: -2px` to align with text baseline.

| Button | Icon description |
|---|---|
| Load JSON | Arrow pointing down into a tray |
| Export JSON | Arrow pointing up from a tray |
| Save changes / Apply | Checkmark polyline |
| Delete / Del | Trash can |
| New | Plus cross |
| Close (modal ×, remove segment) | Diagonal × cross |
| Copy | Two overlapping rectangles |
| Refresh | Near-complete circle with arrowhead |
| Back | Left-pointing chevron |
| All | Checkbox with checkmark |
| None | Empty checkbox outline |

---

## Inputs

All inputs share: `background: #1a1a1a`, `border: 1px solid #444`, `color: #eee`, `border-radius: 2px`, `height: 24px`, font-size 12px, font-family inherit.

| Type | Default width |
|---|---|
| `text` | 150px |
| `number` | 70px |
| `select` | min-width 150px |
| `range` | 120px (100% max 260px on mobile) |
| `color` | 32×24px |
| `checkbox` | 14×14px, `accent-color: #5a8fc0` |

Sliders use `accent-color: #5a8fc0`. Checkboxes use the same accent color.

`.range-val` span: `min-width: 28px`, centered, 11px, color `#aaa`.

---

## Scrollbar

Custom webkit scrollbar: 6×6px, track `#1a1a1a`, thumb `#444` with `border-radius: 3px`.

---

## JS Architecture

| Variable | Type | Purpose |
|---|---|---|
| `data` | Object | Full parsed `presets.json` in memory |
| `focusedId` | String\|null | ID of preset open in single editor |
| `checkedIds` | Set\<String\> | IDs checked for bulk edit |
| `modalCb` | Function\|null | Callback stored before modal confirm |

### Key functions

| Function | Description |
|---|---|
| `buildSidebar()` | Re-renders the full preset list |
| `focusPreset(id)` | Opens a preset in the editor, saves previous |
| `toggleChecked(id)` | Toggles a preset's bulk-check state |
| `rangeCheck(from, to)` | Checks all presets between two IDs |
| `refreshMain()` | Decides whether to show bulk bar or single editor |
| `collectEdits()` | Reads all form fields and writes to `data[focusedId]` |
| `renderEditor()` | Populates all form fields from `data[focusedId]` |
| `renderSegments(segs)` | Renders active segment cards |
| `buildSegCard(seg, si)` | Returns a segment card DOM element |
| `collectSegments(p)` | Reads segment card fields and writes to preset |
| `getBulkChanges()` | Returns enabled bulk fields as `{ key: value }` |
| `doBulkApply(ch)` | Applies bulk changes to all `checkedIds` |
| `showModal(title, desc, diff, cb)` | Opens confirm modal; `cb` called on confirm |
| `closeModal()` | Closes modal, nulls `modalCb` |
| `showToast(msg)` | Shows bottom toast for 2 seconds |
| `switchTab(name, el)` | Switches active tab; adds `.raw-active` on editor if raw |
| `syncRange(el, outId)` | Updates `.range-val` span from range input |
| `buildDiff(before, after)` | Returns HTML diff string for modal |
| `visibleKeys()` | Returns sorted array of non-empty preset keys |

### Modal pattern

**Always** save the callback before calling `closeModal()`:

```js
document.getElementById('modal-ok').onclick = () => {
  const fn = modalCb;   // save first
  closeModal();          // nulls modalCb
  if (fn) fn();          // then call
};
```

### Export pattern

Anchor must be appended to DOM before `.click()`:

```js
const a = document.createElement('a');
a.href = URL.createObjectURL(blob);
a.download = 'presets.json';
document.body.appendChild(a);
a.click();
document.body.removeChild(a);
```

---

## WLED Data

### Preset-level fields

| Field | Type | Notes |
|---|---|---|
| `n` | string | Preset name |
| `ql` | string | 2-char quick label |
| `on` | bool | Global on/off |
| `bri` | 0–255 | Global brightness |
| `transition` | int | ×100ms |
| `mainseg` | int | Main segment index |
| `seg` | array | Segment objects |

### Segment fields (seg[i])

| Field | Type | Notes |
|---|---|---|
| `id`, `start`, `stop` | int | LED range (stop exclusive) |
| `grp`, `spc`, `of` | int | Group, spacing, offset |
| `on`, `frz`, `rev`, `mi` | bool | On, freeze, reverse, mirror |
| `bri`, `cct` | 0–255 | Brightness, color temp |
| `fx` | 0–117 | Effect ID |
| `sx`, `ix` | 0–255 | Speed, intensity |
| `c1`, `c2`, `c3` | 0–255 | Custom effect params |
| `pal` | 0–70 | Palette ID |
| `col` | `[[r,g,b],[r,g,b],[r,g,b]]` | Primary, secondary, tertiary |
| `o1`, `o2`, `o3` | bool | Effect options |
| `si` | 0–3 | Sound input |
| `m12` | 0–5 | Map type |

Inactive segments contain only `{ stop: 0 }`. Active segments have `Object.keys(s).length > 1`.

### Effect GIF previews

Available at `https://raw.githubusercontent.com/scottrbailey/WLED-Utils/master/gifs/FX_{ID:03d}.gif`. Pre-rendered static GIFs, not live.

---

## File structure

```
index.html              Entry page with description and Open Editor button
wled-preset-editor.html Single-file editor (all CSS + JS inline)
README.md               User-facing documentation
CLAUDE.md               This file
```
