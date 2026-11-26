---
name: analyze-components
description: "Audit every component, style and variable used in a scope, build a structured audit document on the canvas, then fix the issues found."
metadata:
  source: https://www.figma.com/community/skill/61009/analyze-components
---

# Analyze Components

Audit every component, style and variable used in a scope, build a structured audit document on the canvas, then fix the issues found.

The hard part of this skill is not the analysis. It is producing tables that do not break. Section 2 is a layout contract; follow it exactly. Every rule in it exists because violating it produces a visibly broken table.

---

## 1. Scope and inputs

Resolve scope once, at the start, and record it in a variable used by every later step:

- If `figma.currentPage.selection` is non-empty, scope is the union of the selected nodes (a "screen" audit).
- Otherwise scope is the entire current page.

State the resolved scope in the document subtitle so the reader knows what was covered.

For traversal:

```js
figma.skipInvisibleInstanceChildren = true; // large speedup, safe for auditing
const scopeRoots = figma.currentPage.selection.length
  ? [...figma.currentPage.selection]
  : [figma.currentPage];
```

If any step needs to resolve nodes outside the current page (component home pages, for example), load all pages first — but feature-check, because some hosts do not implement it:

```js
if (typeof figma.loadAllPagesAsync === "function") {
  try { await figma.loadAllPagesAsync(); caps.allPages = true; }
  catch (e) { caps.allPages = false; }
} else { caps.allPages = false; }
```

When `caps.allPages` is false, resolve home pages only for components reachable from the current page, report the rest as "unknown", and note the limitation in the document subtitle. Do not let this abort the run.

---

## 2. Layout contract (non-negotiable)

### 2.1 Use axis-absolute sizing only

**Never use `primaryAxisSizingMode` or `counterAxisSizingMode`.** Those properties are relative to `layoutMode`, so the same line of code means "hug height" in a vertical frame and "hug width" in a horizontal one. Mixed-direction tables (vertical table, horizontal rows, vertical text cells, horizontal preview cells) make this impossible to reason about, and it is the single largest source of broken columns.

Use `layoutSizingHorizontal` and `layoutSizingVertical` instead. They are absolute: horizontal always means width, vertical always means height.

Of the three accepted values, this skill uses two. **`FILL` is banned outright, and `HUG` is allowed on height only.** See 2.3.

Alignment properties keep their axis-relative names and are fine to use: `primaryAxisAlignItems` and `counterAxisAlignItems`. Note that `primaryAlignItems` does not exist and throws.

### 2.2 Fixed operation order

Every node is built in this order. Deviating throws or silently produces the wrong size.

1. Create the node.
2. Set `layoutMode`, padding, `itemSpacing`, alignment, fills, stroke, `cornerRadius`, `clipsContent`.
3. `parent.appendChild(node)`.
4. Set numeric (FIXED) axes: set `layoutSizing*` to `"FIXED"`, then `resize(...)`.
5. Set `"HUG"` / `"FILL"` axes **last**.
6. Append children.

Reasons: `layoutSizing*` throws if the node has no auto-layout parent, so it must come after append. `resize()` on an auto-sized axis flips that axis to FIXED, so any HUG/FILL axis must be re-applied afterwards. The `size()` helper in section 3 encodes all of this; use it rather than hand-writing sizing.

### 2.3 Derive every width from one constant

Column widths must sum to the exact inner content width of their container. Hard-coded column widths are the reason tables overflow and get clipped.

`DOC_W`, `DOC_PAD` and `CONTENT_W` are defined inside `AUDIT_LIB` (section 3), not in a separate block. Everything in this skill that needs them gets them from the same paste. Do not define them anywhere else and do not assume they are in scope from an earlier `evaluate_script` call.

**Every width is a number, set with `resize()`.** No `FILL`, no `HUG` width, anywhere, including text and badges.

```js
const sec   = section(doc);                                   // FIXED at CONTENT_W
const table = makeTable(sec, { key: "inventory", spec: spec, header: PURPLE });
cellText(cells.name, col, rec.name, { weight: W_SEMI });      // FIXED at col.w - 20
```

`FILL` and `HUG` widths are both resolved by the layout engine. In at least one agent host that resolution does not happen in time, and a `FILL` child reports width `0` while genuinely rendering collapsed: an eleven-character string 0px wide and 160px tall, wrapped one character per line. A width you set yourself always reads back correctly, because it was never derived. So the library computes every width up front and asserts that each `resize()` took effect.

`HUG` survives on the vertical axis only. Height is measured by the text engine from a known width, which is a different mechanism and has been reliable.

Table rows have zero padding and zero `itemSpacing`, so the sum of cell widths must equal the table width exactly. `fitColumns()` forces the total to match and throws if it cannot.

Never write a literal pixel width into a table spec unless the column is marked `fixed: true` (narrow numeric columns). Everything else is a weight.

### 2.3a Why the collapse is silent

A width that fails to resolve does not throw. Figma falls back to minimum content width, which for text means hard-wrapping one character per line. The document builds, every call returns success, and the only evidence is the rendering. That is why the ban is absolute rather than a preference: there is no runtime signal to react to.

`size()` enforces it. It rejects `FILL`, rejects `undefined` and `NaN`, and asserts after each `resize()` that the node actually took the width. An out-of-scope constant throws at the root instead of leaving a frame at its 100px default.

### 2.4 Set defaults explicitly

Never rely on defaults for `itemSpacing` or padding. A frame that inherits a non-zero `itemSpacing` adds `gap × (cells − 1)` to the row and pushes the last column out of the table. On every row and table frame, set all four paddings and `itemSpacing` explicitly, even when the value is 0.

Use `strokeAlign = "INSIDE"` on tables; `"OUTSIDE"` adds 2px outside the layout box and overflows the parent. Do not assume an inside stroke is free: some hosts subtract it from the content box. `makeTable()` budgets `2 * BORDER` either way, which costs at most two invisible pixels when the host does not charge for it. Setting `BORDER = 0` removes both the stroke and the allowance.

Keep `clipsContent = false` on tables while building. Clipping hides overflow, so the screenshot looks fine while the table is wrong. Enable clipping only after the verification pass in section 8 returns clean.

### 2.5 Tag roles instead of inferring structure

The sizing and verification passes must never guess what a node is from its shape. "Horizontal frame inside a vertical frame" matches table rows, stat rows, and badges alike, so a pass written that way will force badges to fixed widths and collapse preview cells.

Tag every structural node at creation time and drive later passes off the tag:

| Role | Applied to |
|---|---|
| `doc` | Root audit frame |
| `section` | A titled section container |
| `table` | Table container (vertical) |
| `table-header` | Header row |
| `table-row` | Data row |
| `cell` | Text/badge cell (vertical) |
| `cell-preview` | Preview cell (horizontal) |
| `preview-stage` | Fixed-size non-auto-layout preview holder |
| `stat-row` / `stat-card` | Overview stat strip |
| `badge` | Hug-width pill |
| `divider` | 1px rule |

Nodes tagged `badge` and `preview-stage`, and anything of type `INSTANCE`, are **never** touched by the sizing or verification passes. Their sizing is correct at creation and any later "fix" breaks them.

Plugin data survives across separate `evaluate_script` calls. Node object references do not.

### 2.6 Do not measure in the call that builds

A dimension Figma computed (the resolved width of a `FILL` child, the height of a `HUG` frame) is not guaranteed to be readable in the same `evaluate_script` call that created the node. Depending on how the host flushes layout, `node.width` on a freshly appended `FILL` child can return `0`. A dimension you set yourself with `resize()` always reads back correctly, because it was never derived.

Rules:

- Build with numbers you already have. Derive them from `CONTENT_W`, never from a sibling's resolved size.
- The one safe read-back is a value you just set with `resize()`. `size()` relies on this and nothing else.
- Do not assert on computed geometry during a build call. An assertion that reads a `FILL` width will fail on a document that is actually fine.
- Put every measurement — `verify()`, overflow checks, the preflight read — in a **separate later call**. By then layout has settled and the numbers are real.
- `assert()` on values you set is fine and worth keeping. `makeDoc()` checks its own `resize()` took effect, which is a real check with no timing dependency.

### 2.7 Host quirks to code around

Agent hosts are not the full plugin runtime. Probe with `capabilities()` in the first call and branch on the result rather than assuming.

| Capability | Observed failure | What the library does |
|---|---|---|
| `setPluginData` / `getPluginData` | Silently unavailable. Reads return `""`, so `JSON.parse` throws and role lookups return nothing | Roles live in layer names; metadata uses `setSharedPluginData` with a namespace, wrapped in try/catch |
| `figma.loadAllPagesAsync` | Not implemented | Feature-check before calling; degrade to current page and say so in the document |
| `strokeAlign = "INSIDE"` | Consumes 1px per side of the content box, so a 1160px table gives rows 1158px | Columns are fitted to `width - 2 * BORDER`; rows are centred so any slack is symmetric |
| Node identity across calls | A node's id can change between calls, so a stored id stops resolving | Every table carries `@table:key` in its name; re-find with `findByRole(doc, "table", key)` when an id lookup fails |
| Malformed remote components | Property inspection throws on a component set with definition errors | Every per-component inspection is wrapped in try/catch and recorded as an issue row |

`JSON.parse` is never called directly on metadata. Use `parseJSON(value, fallback)`, which treats an empty string as a missing value rather than a crash.

### 2.8 Never migrate a previous report

Do not update, patch, or port content from an earlier audit frame. Node ids and structure will not match, and a failed migration leaves the canvas in a worse state than no report at all.

Build a new document every run. If a previous audit exists on the page, rename it `Component Audit (superseded YYYY-MM-DD)` and leave it untouched, or delete it only if the user asked for that. On any build failure, call `abandon(doc, err)`: it removes the frame when no data rows exist yet, and otherwise renames it so the partial state is obvious.

---

## 3. Shared helper library

Paste this block at the top of **every** `evaluate_script` call in this skill. Fonts, helpers and variables do not persist between calls.

```js
// ---------- AUDIT_LIB v4 ----------
// Paste this block whole into every evaluate_script call. Nothing here may
// depend on a variable defined outside it.
//
// This library assumes a hostile host. It encodes four rules learned the hard
// way, each explained in section 2:
//   1. No FILL and no HUG width anywhere. Every width is a number we computed.
//   2. Roles live in layer names, not plugin data. Names always work.
//   3. Column specs are stored as metadata AND recoverable from the header row.
//   4. A table's 1px inside border can consume 2px of row width, so budget it.

const DOC_W      = 1240;                  // change freely; all widths follow
const DOC_PAD    = 40;
const CONTENT_W  = DOC_W - DOC_PAD * 2;   // 1160
const BORDER     = 1;                     // table stroke; set to 0 for no border
const CELL_PL = 12, CELL_PR = 8, CELL_PT = 10, CELL_PB = 10;
const ROW_MIN_H = 52, HEAD_MIN_H = 40, STAGE_H = 44;
const MIN_TABLE_W = 400, MIN_COL_W = 40;
const BADGE_W = 88;
const NS = "audit";                       // shared plugin data namespace

const WHITE = 1, ROW_ALT = 0.965, CARD = 0.96, LINE = 0.88, MUTED = 0.6, INK = 0.13;
const PURPLE = [0.42, 0.28, 0.70];
const RED    = [0.85, 0.18, 0.18];
const ORANGE = [0.90, 0.55, 0.10];
const BLUE   = [0.20, 0.45, 0.85];
const GREEN  = [0.18, 0.70, 0.35];
const DARK   = [0.20, 0.20, 0.20];
const GRAY   = [MUTED, MUTED, MUTED];

function assert(cond, msg) { if (!cond) throw new Error("AUDIT: " + msg); }

const W_REG  = { family: "Inter", style: "Regular" };
const W_SEMI = { family: "Inter", style: "Semi Bold" };
const W_BOLD = { family: "Inter", style: "Bold" };
async function loadFonts() {
  await Promise.all([W_REG, W_SEMI, W_BOLD].map(f => figma.loadFontAsync(f)));
}

const solid = (v, a = 1) => {
  const c = Array.isArray(v) ? v : [v, v, v];
  return [{ type: "SOLID", color: { r: c[0], g: c[1], b: c[2] }, opacity: a }];
};

// ---- Host capability probe. Run once, in the first call, and branch on it.
function capabilities() {
  const caps = {
    loadAllPagesAsync: typeof figma.loadAllPagesAsync === "function",
    getNodeByIdAsync:  typeof figma.getNodeByIdAsync === "function",
    pluginData: false, sharedPluginData: false
  };
  let probe = null;
  try {
    probe = figma.createFrame();
    try { probe.setPluginData("k", "1"); caps.pluginData = probe.getPluginData("k") === "1"; }
    catch (e) { caps.pluginData = false; }
    try {
      probe.setSharedPluginData(NS, "k", "1");
      caps.sharedPluginData = probe.getSharedPluginData(NS, "k") === "1";
    } catch (e) { caps.sharedPluginData = false; }
  } finally { if (probe) { try { probe.remove(); } catch (e) {} } }
  return caps;
}

// ---- Roles live in the layer name: "@role:key Label".
// Private plugin data is unavailable in some hosts; names never are, and they
// make the audit readable in the layers panel as a bonus.
function tag(n, role, key, label) {
  n.name = "@" + role + (key ? ":" + key : "") + (label ? " " + label : "");
  return n;
}
function roleOf(n) {
  const m = /^@([a-z-]+)/.exec((n && n.name) || "");
  return m ? m[1] : "";
}
function keyOf(n) {
  const m = /^@[a-z-]+:([^ ]+)/.exec((n && n.name) || "");
  return m ? m[1] : "";
}
function findByRole(root, role, key) {
  if (!root || typeof root.findOne !== "function") return null;
  return root.findOne(n => roleOf(n) === role && (!key || keyOf(n) === key));
}

// ---- Metadata: shared plugin data, never private. Always failure tolerant.
function setMeta(n, k, v) {
  try { n.setSharedPluginData(NS, k, String(v)); return true; } catch (e) { return false; }
}
function getMeta(n, k) {
  try { return n.getSharedPluginData(NS, k) || ""; } catch (e) { return ""; }
}
function parseJSON(s, fallback) {
  if (typeof s !== "string" || !s.trim()) return fallback;
  try { return JSON.parse(s); } catch (e) { return fallback; }
}

// ---- Sizing. Width must be a number. Height is a number or "HUG".
// FILL and HUG widths are rejected outright: both are resolved by the layout
// engine, and a host that has not flushed layout reports them as 0.
function size(node, w, h) {
  assert(typeof w === "number" && isFinite(w) && w > 0,
    "width must be a positive number for " + node.name + ", got " + String(w));
  assert(h === "HUG" || (typeof h === "number" && isFinite(h) && h > 0),
    'height must be a number or "HUG" for ' + node.name + ", got " + String(h));
  node.layoutSizingHorizontal = "FIXED";
  if (typeof h === "number") node.layoutSizingVertical = "FIXED";
  node.resize(w, typeof h === "number" ? h : Math.max(node.height, 0.01));
  if (h === "HUG") node.layoutSizingVertical = "HUG";
  assert(Math.abs(node.width - w) <= 1,
    node.name + " width is " + Math.round(node.width) + ", expected " + w);
  return node;
}

function frame(parent, o) {
  const f = figma.createFrame();
  if (o.mode) f.layoutMode = o.mode;
  f.itemSpacing   = o.gap || 0;
  f.paddingTop    = o.pt || 0;
  f.paddingRight  = o.pr || 0;
  f.paddingBottom = o.pb || 0;
  f.paddingLeft   = o.pl || 0;
  if (o.mode) {
    f.primaryAxisAlignItems = o.main  || "MIN";
    f.counterAxisAlignItems = o.cross || "MIN";
  }
  f.fills = o.fill === undefined ? [] : solid(o.fill);
  f.cornerRadius = o.radius || 0;
  f.clipsContent = !!o.clip;
  if (o.stroke !== undefined) {
    f.strokes = solid(o.stroke);
    f.strokeWeight = 1;
    f.strokeAlign = "INSIDE";
  }
  parent.appendChild(f);
  size(f, o.w, o.h);
  if (o.role) tag(f, o.role, o.key, o.label); else if (o.name) f.name = o.name;
  return f;
}

// Text always takes an explicit width. Use cellText() inside table cells.
function text(parent, chars, o) {
  o = o || {};
  assert(typeof o.w === "number",
    "text needs an explicit width: " + String(chars).slice(0, 40));
  const t = figma.createText();
  t.fontName = o.weight || W_REG;
  t.fontSize = o.size || 11;
  t.lineHeight = { unit: "PERCENT", value: o.lh || 145 };
  t.textAutoResize = "HEIGHT";
  t.textAlignHorizontal = o.align || "LEFT";
  t.fills = solid(o.color === undefined ? INK : o.color);
  t.characters = chars == null ? "" : String(chars);
  parent.appendChild(t);
  size(t, o.w, "HUG");
  return t;
}

const cellTextW = col => Math.max(col.w - CELL_PL - CELL_PR, 8);

function cellText(cell, col, chars, o) {
  o = Object.assign({}, o || {});
  o.w = cellTextW(col);
  o.align = o.align || col.align || "LEFT";
  return text(cell, chars, o);
}

// Badges are fixed width. A hug-width badge depends on the layout engine
// measuring its child, which is the behaviour this library refuses to trust.
function badge(parent, label, rgb, w) {
  const bw = w || BADGE_W;
  const b = frame(parent, {
    mode: "HORIZONTAL", pl: 6, pr: 6, pt: 4, pb: 4, radius: 4, fill: rgb,
    main: "CENTER", cross: "CENTER", role: "badge", w: bw, h: "HUG"
  });
  text(b, label, { size: 10, weight: W_SEMI, color: 1, align: "CENTER", w: bw - 12 });
  return b;
}

function divider(parent, w, v) {
  const r = figma.createRectangle();
  r.fills = solid(v === undefined ? LINE : v);
  parent.appendChild(r);
  size(r, w === undefined ? CONTENT_W : w, 1);
  return tag(r, "divider");
}

// Distribute width across columns. `fixed: true` columns keep their pref px;
// the rest split what is left by pref weight. The last flex column absorbs
// rounding so the total is exact.
function fitColumns(spec, contentW) {
  const fixedSum = spec.filter(c => c.fixed).reduce((a, c) => a + c.pref, 0);
  const flex = spec.filter(c => !c.fixed);
  const flexSum = flex.reduce((a, c) => a + c.pref, 0) || 1;
  const avail = contentW - fixedSum;
  assert(!flex.length || avail >= flex.length * MIN_COL_W,
    "fitColumns: only " + avail + "px for " + flex.length + " flex columns");
  const last = flex[flex.length - 1];
  let used = 0;
  const out = spec.map(c => {
    let w;
    if (c.fixed) w = c.pref;
    else if (c === last) w = avail - used;
    else { w = Math.round(avail * (c.pref / flexSum)); used += w; }
    return Object.assign({}, c, { w: w });
  });
  const total = out.reduce((a, c) => a + c.w, 0);
  assert(total === contentW, "fitColumns: " + total + " != " + contentW);
  return out;
}

function makeDoc(name) {
  const d = figma.createFrame();
  d.layoutMode = "VERTICAL";
  d.itemSpacing = 40;
  d.paddingTop = d.paddingRight = d.paddingBottom = d.paddingLeft = DOC_PAD;
  d.fills = solid(WHITE);
  d.clipsContent = false;
  figma.currentPage.appendChild(d);
  size(d, DOC_W, "HUG");            // parent is a PAGE: FIXED only, never FILL
  tag(d, "doc", "component-audit", name || "Component Audit (building)");
  setMeta(d, "contentWidth", String(CONTENT_W));
  d.x = Math.round(figma.viewport.center.x - DOC_W / 2);
  d.y = Math.round(figma.viewport.center.y);
  return d;
}

// Leave no orphan frames behind on failure.
function abandon(doc, err) {
  try {
    if (!doc || !doc.parent) return;
    const hasRows = typeof doc.findOne === "function" &&
                    !!doc.findOne(n => roleOf(n) === "table-row");
    if (!hasRows) doc.remove();
    else doc.name = "@doc INCOMPLETE " + ((err && err.message) || "build failed");
  } catch (e) {}
}

function section(doc, key) {
  return frame(doc, {
    mode: "VERTICAL", gap: 12, role: "section", key: key, w: CONTENT_W, h: "HUG"
  });
}

// makeTable(parent, { key, spec, header, width })
function makeTable(parent, o) {
  assert(o && o.key && Array.isArray(o.spec), "makeTable needs { key, spec, header }");
  const outer = Math.round(o.width === undefined ? CONTENT_W : o.width);
  assert(isFinite(outer) && outer >= MIN_TABLE_W,
    "makeTable width " + outer + " must be a number >= " + MIN_TABLE_W);
  const inner = outer - BORDER * 2;   // an inside stroke can eat a pixel per side
  const cols = fitColumns(o.spec, inner);
  const t = frame(parent, {
    mode: "VERTICAL", gap: 0, radius: 8, clip: false,
    stroke: BORDER ? LINE : undefined, cross: "CENTER",
    role: "table", key: o.key, w: outer, h: "HUG"
  });
  setMeta(t, "cols", JSON.stringify(cols));
  setMeta(t, "rowWidth", String(inner));
  const head = frame(t, {
    mode: "HORIZONTAL", gap: 0, fill: o.header, cross: "CENTER",
    role: "table-header", w: inner, h: "HUG"
  });
  head.minHeight = HEAD_MIN_H;
  cols.forEach(c => {
    const cell = makeCell(head, c);
    cellText(cell, c, c.label, { size: 10, weight: W_SEMI, color: 1 });
  });
  return t;
}

// Column spec: metadata first, header row as the fallback. Header cell widths
// were set with resize(), so reading them back is safe.
function readCols(table) {
  const stored = parseJSON(getMeta(table, "cols"), null);
  if (stored && stored.length) return stored;
  const head = table.children.filter(c => roleOf(c) === "table-header")[0];
  assert(head, "table " + table.name + " has no header row; cannot recover columns");
  return head.children.map(c => ({
    key: keyOf(c) || c.name,
    w: Math.round(c.width),
    preview: keyOf(c) === "preview",
    align: "LEFT"
  }));
}

function rowWidth(table) {
  const stored = Number(getMeta(table, "rowWidth"));
  if (isFinite(stored) && stored >= MIN_TABLE_W) return stored;
  return readCols(table).reduce((a, c) => a + c.w, 0);
}

function makeCell(row, col) {
  return frame(row, {
    mode: col.preview ? "HORIZONTAL" : "VERTICAL",
    gap: 4, pl: CELL_PL, pr: CELL_PR, pt: CELL_PT, pb: CELL_PB,
    main: "CENTER", cross: "CENTER",
    clip: !!col.preview,
    role: col.preview ? "cell-preview" : "cell", key: col.key,
    w: col.w, h: "HUG"
  });
}

// Returns { row, cells, cols }. Every width comes from the spec, not geometry.
function makeRow(table, index) {
  const cols = readCols(table);
  assert(cols.length, "table has no columns");
  const row = frame(table, {
    mode: "HORIZONTAL", gap: 0, fill: index % 2 === 0 ? WHITE : ROW_ALT,
    cross: "CENTER", role: "table-row", key: String(index),
    w: rowWidth(table), h: "HUG"
  });
  row.minHeight = ROW_MIN_H;
  const cells = {};
  cols.forEach(c => { cells[c.key] = makeCell(row, c); });
  return { row: row, cells: cells, cols: cols };
}
// ---------- end AUDIT_LIB ----------
```

### 3.1 Preview instances

A live instance inside an auto-layout cell gets stretched, pushes siblings, and inflates row height. Put it in a fixed-size **non-auto-layout stage** instead. A plain frame positions children absolutely, so nothing can resize the instance and nothing the instance does can affect the row.

```js
// col is the column spec entry for the preview column, so the stage width is
// known without reading the cell's geometry.
async function previewCell(cell, col, componentId, sampleInstanceId) {
  const STAGE_W = cellTextW(col);        // from the spec, never from the cell

  const stage = figma.createFrame();     // no layoutMode -> absolute children
  stage.fills = [];
  stage.clipsContent = true;
  cell.appendChild(stage);
  size(stage, STAGE_W, STAGE_H);
  tag(stage, "preview-stage");

  let inst = null;
  try {
    const node = await figma.getNodeByIdAsync(componentId);
    if (node && node.type === "COMPONENT_SET") {
      const v = node.defaultVariant || node.children.filter(c => c.type === "COMPONENT")[0];
      if (v) inst = v.createInstance();
    } else if (node && node.type === "COMPONENT") {
      inst = node.createInstance();
    }
  } catch (e) { inst = null; }           // malformed library components throw here
  if (!inst && sampleInstanceId) {
    try {
      const sample = await figma.getNodeByIdAsync(sampleInstanceId);
      if (sample && sample.type === "INSTANCE") { inst = sample.clone(); inst.resetOverrides(); }
    } catch (e) { inst = null; }
  }
  if (!inst) {
    const ph = figma.createText();
    ph.fontName = W_REG; ph.fontSize = 10; ph.fills = solid(MUTED);
    ph.characters = "no preview";
    stage.appendChild(ph);
    ph.x = 8; ph.y = Math.round((STAGE_H - ph.height) / 2);
    return null;
  }

  stage.appendChild(inst);
  const k = Math.min(1,
    (STAGE_W - 8) / Math.max(inst.width, 1),
    (STAGE_H - 8) / Math.max(inst.height, 1));
  if (k < 1) { try { inst.rescale(Math.max(k, 0.05)); } catch (e) {} }
  inst.x = Math.round((STAGE_W - inst.width) / 2);
  inst.y = Math.round((STAGE_H - inst.height) / 2);
  return inst;
}
```

Always prefer a live instance over a rasterized screenshot. Image hashes from `figma.createImage()` do not reliably survive across `evaluate_script` calls.

---

## 4. Step 1 — Component inventory

Traverse the scope and collect every `INSTANCE`. Use the async accessor; `instance.mainComponent` throws in dynamic-page documents.

```js
const main = await instance.getMainComponentAsync();
if (!main) continue;                      // unresolvable, count separately
const set = main.parent && main.parent.type === "COMPONENT_SET" ? main.parent : null;
const key = set ? set.id : main.id;
```

Wrap every per-component inspection in try/catch. A remote component set with definition errors throws on property access and will otherwise kill the whole traversal:

```js
let variants = 0, props = null, broken = null;
try {
  variants = set ? set.children.length : 0;
  props = set ? set.componentPropertyDefinitions : null;
} catch (e) {
  broken = e.message || "component definition error";
}
```

Record per group: component name, component set name, main component and set node IDs, `main.remote`, home page name, variant count, instance count, one sample instance ID for preview fallback, and `broken` when inspection failed. Every component with `broken` set becomes a Critical row in Issues & Actions rather than an aborted run.

Home page resolution:

```js
function homePage(node) {
  let n = node;
  while (n && n.type !== "PAGE") n = n.parent;
  return n ? n.name : null;
}
```

Sort groups by instance count descending. Flag:

- **Remote components** — `main.remote === true`. These legitimately have no home page in this file. Do not report them as orphans.
- **Orphaned components** — local (`remote === false`) components whose parent chain does not reach a `PAGE`. These are read-only internal nodes and cannot be moved, only cloned.
- **Naming issues** — variant property names with typos, or generic names such as "Property 1".
- **Duplicate sets** — the same set name appearing under more than one node ID.
- **Misplaced components** — components whose home page does not match the expected convention.

---

## 5. Step 2 — External dependency scan

Run as a second traversal. Guard `figma.mixed` everywhere; an unguarded mixed style ID produces a symbol where a string is expected and corrupts the counts.

### Styles

```js
const STYLE_PROPS = ["fillStyleId", "strokeStyleId", "textStyleId", "effectStyleId"];
for (const prop of STYLE_PROPS) {
  const id = node[prop];
  if (typeof id !== "string" || !id) continue;   // skips figma.mixed and ""
  const style = await figma.getStyleByIdAsync(id);
  if (!style) continue;
  // record: style.name, style.type, style.remote, style.description, id
}
```

For text nodes with per-character styling, `textStyleId` is `figma.mixed`. Recover the real values:

```js
if (node.type === "TEXT" && node.textStyleId === figma.mixed) {
  for (const seg of node.getStyledTextSegments(["textStyleId", "fillStyleId"])) {
    // handle seg.textStyleId / seg.fillStyleId the same way
  }
}
```

### Variables

`boundVariables` nests differently per property (a single alias, an array for fills, an object for component properties). Flatten generically:

```js
function aliasIds(bv) {
  const ids = [];
  const visit = v => {
    if (!v || typeof v !== "object") return;
    if (Array.isArray(v)) return v.forEach(visit);
    if (v.type === "VARIABLE_ALIAS" && v.id) return ids.push(v.id);
    Object.values(v).forEach(visit);
  };
  visit(bv);
  return ids;
}

for (const id of aliasIds(node.boundVariables)) {
  const variable = await figma.variables.getVariableByIdAsync(id);
  if (!variable) continue;
  const collection = await figma.variables
    .getVariableCollectionByIdAsync(variable.variableCollectionId);
  // record: variable.name, collection && collection.name,
  //         variable.resolvedType, collection && collection.remote
}
```

Both variable getters are async. Await them.

### Primary vs ghost

Group styles by library source (`style.description`, then naming prefix such as `Brand/…`). Group variables by collection. The library with the highest total usage across styles and variables is the **primary** library. Everything remote that is not from the primary library is a **ghost** reference: ghost components, ghost styles, ghost variables. Record the inferred primary library name and state it in the document so the reader can tell whether the heuristic picked correctly.

Sort by usage count descending within each group.

---

## 6. Step 3 — Build the audit document

Grayscale throughout. Color only on severity badges, status badges, and section-warning titles. No emojis anywhere.

The palette lives in `AUDIT_LIB`. Do not redefine it.

### Root frame and preflight

Build the root and prove it is correctly sized **before** creating any content. This is one cheap call and it catches the failure that otherwise only shows up as a mangled screenshot at the end.

**Call 1 — probe the host, build the root.** One call, self-cleaning.

```js
await loadFonts();
const caps = capabilities();                   // see 2.7
const doc = makeDoc("Component Audit");        // asserts its own resize took

let probeReport = null;
try {
  const probe = section(doc, "preflight");
  const t = cellText(probe, { w: CONTENT_W, align: "LEFT" }, "width probe");
  probeReport = { sectionWidth: probe.width, textWidth: t.width, textHeight: t.height };
  probe.remove();
} catch (e) {
  abandon(doc, e);
  throw e;
}
return { docId: doc.id, docKey: "component-audit", caps: caps, probe: probeReport };
```

Every width in that probe was set with `resize()`, so the reads are trustworthy in the same call. `sectionWidth` and `textWidth` must both equal `CONTENT_W`, and `textHeight` must be a single line. `size()` already asserts each one, so reaching the `return` means the geometry is sound.

Report `caps` to the user when anything is false, and record the degraded capabilities in the "How to Read This Document" card so the report is self-describing.

If a later call cannot resolve `docId`, recover by role rather than failing:

```js
let doc = null;
try { doc = await figma.getNodeByIdAsync(docId); } catch (e) { doc = null; }
if (!doc) doc = findByRole(figma.currentPage, "doc", "component-audit");
assert(doc, "audit document not found; do not migrate an old one, start over");
```

Every direct child of `doc` is FIXED at `CONTENT_W`. Create sections with `section(doc)`: vertical, tagged `section`, FIXED width at `CONTENT_W`, `HUG` height, `itemSpacing` 12.

Each section is created as `const sec = section(doc);` and every table call below uses that local `sec`. Separate major sections with `divider(sec)`, which defaults to `CONTENT_W`.

### Section order

1. Header
2. How to Read This Document
3. Overview
4. Issues & Actions
5. Component Inventory
6. External Dependencies (summary + stat row)
7. Ghost Components (only if any)
8. Ghost Styles (only if any)
9. Primary Library Styles
10. External Variables (only if any)

**Header.** Title "Component Audit" (28px Bold). Subtitle (12px, `MUTED`) assembled from parts, skipping any part with no value, so an empty scope name cannot leave a dangling separator:

```js
const scopeLabel = scopeRoots[0].type === "PAGE"
  ? "Full page: " + figma.currentPage.name
  : "Selection: " + scopeRoots.map(n => n.name).filter(Boolean).join(", ");
const bits = [
  scopeLabel,
  uniqueCount + " unique components",
  instanceCount + " total instances",
  remoteCount + " remote components",
  pageCount + " component pages",
  styleCount + " external styles",
  variableCount + " external variables"
].filter(b => b && !/:\s*$/.test(b));
text(sec, bits.join("  |  "), { size: 12, color: MUTED, w: CONTENT_W });
```

**How to Read This Document.** A card (`fill: CARD`, `radius: 8`, padding 20, FIXED at `CONTENT_W`, `HUG` height) with one short paragraph per section plus the severity legend: Critical, Warning, Suggestion.

**Overview.** Title (24px Bold) then a stat row: horizontal, `gap: 16`, FIXED at `CONTENT_W`, `HUG` height, holding four cards. Compute the card width rather than using `layoutGrow`, which is layout-resolved like `FILL`:

```js
const GAP = 16, N = 4;
const cardW = Math.floor((CONTENT_W - GAP * (N - 1)) / N);
const row = frame(sec, { mode: "HORIZONTAL", gap: GAP, role: "stat-row",
                         w: CONTENT_W, h: "HUG" });
const card = frame(row, { mode: "VERTICAL", gap: 4, fill: CARD, radius: 8,
                          pl: 20, pr: 20, pt: 20, pb: 20,
                          role: "stat-card", w: cardW, h: "HUG" });
text(card, value, { size: 24, weight: W_BOLD, w: cardW - 40 });
text(card, label, { size: 11, color: MUTED, w: cardW - 40 });
```

Four cards at `CONTENT_W` 1160 come out 278px each with 16px gaps, totalling exactly 1160. Card contents: value (24px Bold) above label (11px, `MUTED`).

Cards: Scope / "Screens Audited"; component count / "Unique Components"; instance count / "Total Instances"; page count / "Component Pages".

### Issues & Actions

```js
const issueTable = makeTable(sec, { key: "issues", header: PURPLE, spec: [
  { key: "sev",    label: "Severity",        pref: 100, fixed: true },
  { key: "issue",  label: "Issue",           pref: 300 },
  { key: "action", label: "Action Required", pref: 260 },
  { key: "status", label: "Status",          pref: 110, fixed: true },
  { key: "result", label: "Result",          pref: 200 }
]});
```

Per row:

```js
const { cells, cols } = makeRow(issueTable, i);
const col = k => cols.filter(c => c.key === k)[0];
badge(cells.sev, rec.severity, SEV_COLOR[rec.severity]);
cellText(cells.issue,  col("issue"),  rec.issue);
cellText(cells.action, col("action"), rec.action);
badge(cells.status, "Not Started", GRAY);
cellText(cells.result, col("result"), "");
```

Severity colors: Critical `RED`, Warning `ORANGE`, Suggestion `BLUE`. Status: "Done" `GREEN`, "Not Started" gray `0.6`.

### Component Inventory

One single flat table. Do not split by category, do not create sub-tables.

```js
const invTable = makeTable(sec, { key: "inventory", header: PURPLE, spec: [
  { key: "preview",  label: "Preview",      pref: 170, preview: true },
  { key: "name",     label: "Component",    pref: 240 },
  { key: "category", label: "Category",     pref: 120 },
  { key: "variants", label: "Variants",     pref: 70,  fixed: true, align: "CENTER" },
  { key: "uses",     label: "Uses",         pref: 60,  fixed: true, align: "CENTER" },
  { key: "page",     label: "Page",         pref: 110 },
  { key: "id",       label: "Component ID", pref: 240 }
]});
```

Row body:

```js
const { cells, cols } = makeRow(invTable, i);
const col = k => cols.filter(c => c.key === k)[0];
await previewCell(cells.preview, col("preview"), rec.componentId, rec.sampleInstanceId);
cellText(cells.name,     col("name"),     rec.name, { weight: W_SEMI });
cellText(cells.category, col("category"), rec.category);
cellText(cells.variants, col("variants"), rec.variants, { align: "CENTER" });
cellText(cells.uses,     col("uses"),     rec.uses,     { align: "CENTER" });
cellText(cells.page,     col("page"),     rec.page || "unknown");
cellText(cells.id,       col("id"),       rec.componentId, { size: 9, color: MUTED });
```

Sort by usage count descending. Assign every component a short category label: Tags, Icons, Headers, Audit, Utility, Surfaces, Data Table, Color, Changelog, Annotations, Org, Remote.

Subtitle above the table: "All X components in scope. Each row includes an inline preview at default state."

### External Dependencies

Divider, then title "External Dependencies" (20px Bold), then a summary line: `X ghost components from N libraries | Y external styles from N libraries | Z external variables from N collections`. Then a stat row with four cards: ghost components, external styles, ghost styles, external variables.

### Ghost Components (only if any)

Title 16px Semi Bold in `RED`. Subtitle 12px `MUTED` explaining that these come from libraries other than the primary design system. Header color `RED`.

```js
makeTable(sec, { key: "ghost-components", header: RED, spec: [
  { key: "preview", label: "Preview",          pref: 170, preview: true },
  { key: "name",    label: "Component",        pref: 200 },
  { key: "lib",     label: "Library Source",   pref: 160 },
  { key: "count",   label: "Instances",        pref: 80, fixed: true, align: "CENTER" },
  { key: "action",  label: "Suggested Action", pref: 380 }
]})
```

Ghost previews usually cannot be instantiated from the main component, so pass `sampleInstanceId` and let `previewCell` clone an existing instance. Pass the preview column spec too: `previewCell(cells.preview, cols.filter(c => c.key === "preview")[0], mainId, sampleId)`.

### Ghost Styles (only if any)

Title 16px Semi Bold in `ORANGE`. Subtitle: "These styles come from a different library than the primary design system." Header color `ORANGE`.

```js
makeTable(sec, { key: "ghost-styles", header: ORANGE, spec: [
  { key: "name",    label: "Ghost Style Name",       pref: 260 },
  { key: "type",    label: "Type",                   pref: 80, fixed: true },
  { key: "uses",    label: "Uses",                   pref: 60, fixed: true, align: "CENTER" },
  { key: "replace", label: "Suggested Replacement",  pref: 280 },
  { key: "desc",    label: "Description",            pref: 280 }
]})
```

Type values: Fill, Stroke, Text, Effect. Render the replacement cell text in `BLUE`.

### Primary Library Styles

Title "Primary Library Styles ({library name})", 16px Semi Bold, `INK`. Subtitle with total count and total usage. Header color `DARK`.

```js
makeTable(sec, { key: "primary-styles", header: DARK, spec: [
  { key: "name", label: "Style Name", pref: 300 },
  { key: "type", label: "Type",       pref: 80, fixed: true },
  { key: "uses", label: "Uses",       pref: 80, fixed: true, align: "CENTER" },
  { key: "id",   label: "Style ID",   pref: 500 }
]})
```

Show the top 20 by usage, descending. If more exist, append one full-width summary row: "+ N more styles with fewer than X usages each". Build that summary row as a normal row whose first cell holds the text and whose remaining cells are empty, so the column geometry stays intact. Style ID text: 9px, `MUTED`.

### External Variables (only if any)

Title 16px Semi Bold in `ORANGE`, header color `ORANGE`.

```js
makeTable(sec, { key: "external-vars", header: ORANGE, spec: [
  { key: "name",  label: "Variable Name",  pref: 250 },
  { key: "coll",  label: "Collection",     pref: 200 },
  { key: "type",  label: "Type",           pref: 100, fixed: true },
  { key: "uses",  label: "Uses",           pref: 70, fixed: true, align: "CENTER" },
  { key: "lib",   label: "Library Source", pref: 340 }
]})
```

Group and sort by collection, then usage count descending.

---

## 7. Batching

Preview instances are expensive. Cap each `evaluate_script` call at **25 rows**.

Rules for every call after the first:

1. Re-paste `AUDIT_LIB` whole. It carries the constants as well as the helpers, so nothing can be out of scope.
2. `await loadFonts()` again. Font loads do not persist between calls, and setting `characters` without the font loaded throws, leaving a half-built table.
3. Re-fetch nodes by ID with `await figma.getNodeByIdAsync(id)`. Never `figma.getNodeById` (sync, throws in dynamic-page documents) and never a reference from a previous call.
4. Read the column spec from metadata rather than recomputing: `readCols(table)`. `makeRow` already does this. Recomputing risks a different distribution if any input changed.
5. Continue the alternating row index across batches so the striping stays consistent. Pass the running index in explicitly.
6. Re-assert geometry on entry, before appending anything:

```js
let table = null;
try { table = await figma.getNodeByIdAsync(tableId); } catch (e) { table = null; }
if (!table) table = findByRole(figma.currentPage, "table", tableKey);   // ids can change
assert(table && table.type === "FRAME", "table " + tableKey + " not found");
const cols = readCols(table);            // metadata, or recovered from the header row
assert(cols.length, "table " + tableKey + " has no column spec");
```

Pass both `tableId` and `tableKey` between calls. Ids are faster; the key is what survives when a host reassigns identity. `readCols()` falls back to measuring the header row's cells, whose widths were set with `resize()` and therefore read back reliably, so a table can always rebuild its own spec.

Return the table node ID from each call so the next call can pick it up. Do not run the sizing or verification passes until every batch has completed.

---

## 8. Sizing and verification pass

Run once, after all content exists, in a dedicated call. Both functions are role-driven, bottom-up, and skip badges, preview stages and instances.

```js
function normalize(node) {
  if (!("children" in node)) return;
  const r = roleOf(node);
  if (r === "badge" || r === "preview-stage" || node.type === "INSTANCE") return;

  for (const child of [...node.children]) normalize(child);   // children first

  if (r === "table-row" || r === "table-header") {
    const t = node.parent;
    const cols = readCols(t);
    node.itemSpacing = 0;
    node.paddingLeft = node.paddingRight = 0;
    node.children.forEach((cell, i) => {
      if (!cols[i]) return;
      size(cell, cols[i].w, "HUG");
    });
    size(node, rowWidth(t), "HUG");
    return;
  }
  if (r === "cell" || r === "cell-preview" || r === "stat-card" || r === "doc") {
    node.layoutSizingVertical = "HUG";
    return;
  }
  if (r === "table") {
    size(node, Number(getMeta(node, "rowWidth")) + BORDER * 2 || CONTENT_W, "HUG");
    return;
  }
  if (r === "section" || r === "stat-row") size(node, CONTENT_W, "HUG");
}
```

Two properties of this pass matter. It never widens a cell-preview cell, because width comes only from the stored column spec and the cell branch touches vertical alone — that was the failure of the old shape-inference version. And it never reads a computed width, so running it cannot be affected by whether layout has flushed.

```js
function verify(root) {
  const problems = [];
  const walk = n => {
    if (!("children" in n)) return;
    const r = roleOf(n);
    if (r === "badge" || r === "preview-stage" || n.type === "INSTANCE") return;

    if (r === "table-row" || r === "table-header") {
      const want = rowWidth(n.parent);
      const sum = n.children.reduce((a, c) => a + c.width, 0)
                - n.itemSpacing * Math.max(n.children.length - 1, 0)
                - n.paddingLeft + n.paddingRight;
      if (Math.abs(sum - want) > 1)
        problems.push(n.name + ": cells total " + Math.round(sum) + " vs row " + want);
      const cols = readCols(n.parent);
      if (cols.length !== n.children.length)
        problems.push(n.name + ": " + n.children.length + " cells vs " + cols.length + " columns");
    }

    // Nothing in this skill may use FILL. If it appears, someone hand-rolled a node.
    if (n.layoutSizingHorizontal === "FILL")
      problems.push(n.name + ": uses FILL width");

    for (const c of n.children) {
      if (c.type !== "TEXT") continue;
      if (c.layoutSizingHorizontal === "FILL")
        problems.push("text uses FILL: " + c.characters.slice(0, 30));
      const chars = c.characters.replace(/\s/g, "").length;
      if (chars > 8 && c.width < 60)
        problems.push("collapsed text (" + Math.round(c.width) + "px): " + c.characters.slice(0, 30));
      if (chars > 8 && c.height > c.fontSize * 3 && c.width < c.fontSize * 6)
        problems.push("per-character wrapping: " + c.characters.slice(0, 30));
    }

    if (n.layoutMode) {
      const innerW = n.width  - n.paddingLeft - n.paddingRight;
      const innerH = n.height - n.paddingTop  - n.paddingBottom;
      const horiz  = n.layoutMode === "HORIZONTAL";
      const along  = n.children.reduce((a, c) => a + (horiz ? c.width : c.height), 0)
                - n.itemSpacing * Math.max(n.children.length - 1, 0);
      const across = n.children.reduce((a, c) => Math.max(a, horiz ? c.height : c.width), 0);
      if (along - (horiz ? innerW : innerH) > 1 || across - (horiz ? innerH : innerW) > 1)
        problems.push(n.name + ": children overflow (" + Math.round(along) + "/" + Math.round(across) + ")");
    }

    n.children.forEach(walk);
  };
  walk(root);
  return problems;
}
```

Run this as its own `evaluate_script` call, after the last batch. Per 2.6, `verify()` reads computed geometry, so it must not share a call with node creation.

Procedure:

1. `const doc = await figma.getNodeByIdAsync(docId)`
2. `normalize(doc)`
3. `const problems = verify(doc)`
4. If `problems.length`, return them from the script and fix the cause. Do not paper over it by enabling clipping.
5. When clean, set `clipsContent = true` on each node tagged `table` so the rounded corners mask correctly.
6. Zoom to the document and take a screenshot to confirm visually.

Report the problem list to the user if anything remains rather than claiming success.

---

## 9. Step 4 — Fix issues

Proceed immediately after the document is built. Do not ask for confirmation.

**Orphaned components.** Read-only internal nodes cannot be moved.

```js
const clone = orphan.clone();
targetPage.appendChild(clone);
for (const inst of instancesOf(orphan)) inst.swapComponent(clone);
```

Place the clone on the closest-matching component page, or Utility if none matches. Log the swap count.

**Naming issues.** Rename directly when the node is writable: `node.name = corrected`. For variant properties use `componentSet.editComponentProperty(key, { name: corrected })` and rename the variant children to match. Count affected instances.

**Duplicate component sets.** Choose one canonical set. If it is orphaned or read-only, clone it, fix the clone, and place it on a component page. Swap instances by matching variant property values. Log the swap count per replaced set.

**After fixing.** Update the Issues & Actions table in place: locate the status badge, swap its fill to `GREEN`, set its text to "Done", and fill the Result cell with the counts. Changing badge text can change badge width, which is fine because badges hug. Re-run `normalize(doc)` and `verify(doc)` afterwards.

If component sets need to be created: use `figma.combineAsVariants(components, parent)`, then set `layoutPositioning = "AUTO"` on each child.

---

## 10. Failure modes this design prevents

| Symptom | Cause | Prevented by |
|---|---|---|
| `JSON.parse` fails before any change is applied | Private plugin data unsupported, so the stored column spec read back as `""` | Metadata uses shared plugin data via `setMeta`/`getMeta`, and `parseJSON()` treats empty as missing (2.7) |
| Roles never match, so the sizing pass does nothing | `getPluginData` returned `""` for every role tag | Roles live in layer names (2.7, 3) |
| Last column overhangs by 2px | An inside stroke consumed 1px per side of the row's content box | Columns fit to `width - 2 * BORDER`, rows centred (2.4) |
| A stored node id stops resolving between calls | Host reassigned node identity | `findByRole(root, "table", key)` recovers by name (2.7, 7) |
| One malformed library component kills the run | Property access throws on a component set with definition errors | Per-component try/catch; the failure becomes a Critical row (4) |
| An empty frame is left on the canvas after a failure | No cleanup path | `abandon(doc, err)` removes or clearly renames the partial frame (2.8) |
| Report "migration" fails and nothing is produced | Tried to update a previous audit whose structure no longer matched | Never migrate; build fresh and supersede the old one (2.8) |
| Preflight reports a FILL width of 0 on a document that is fine | Layout-derived geometry read back in the same `evaluate_script` call that created the node | Nothing in the build path reads geometry; all measurement moved to a later call (2.6, 6) |
| Build aborts at the first table | `makeTable` asserted on its own resolved width | `makeTable` fits columns to an explicit number, defaulting to `CONTENT_W` (3) |
| Every heading wraps one character per line | Root frame never received its width, so it hugged at the 100px default and every FILL descendant collapsed to min-content | `makeDoc()` asserts the root width; `size()` rejects `undefined`/`NaN`; preflight probe runs before any content (2.3a, 6) |
| Document builds with no error but is unreadable | `FILL` child inside a `HUG` parent degrades silently | `size()` rejects FILL; collapse detectors in `verify()` (8) |
| Constants out of scope in a later script call | `DOC_W` / `CONTENT_W` were defined outside the pasted helper block | All constants live inside `AUDIT_LIB` (3) |
| Tables built at the wrong scale | Column widths computed from a constant that did not match the real frame | `makeTable()` measures its own resolved width and asserts a floor (2.3) |
| Subtitle shows a stray separator | String template concatenated an empty scope name | Subtitle assembled from a filtered parts array (6) |
| Last columns cut off | Column widths summed to more than the container's inner width | `fitColumns()` derives from `CONTENT_W` and throws on mismatch (2.3) |
| Preview column collapses to hug width | `primaryAxisSizingMode = "AUTO"` means hug *width* on a horizontal cell | Axis-absolute sizing only (2.1) |
| Badges stretched to fixed widths | Sizing pass matched badges as "table rows" by shape | Role tags; badges excluded (2.5, 8) |
| Stat cards uneven | Same shape-inference problem | `stat-row` / `stat-card` roles with computed card widths |
| Columns misaligned row to row | Recomputed or drifting column widths between batches | Column spec stored on the table via shared plugin data (7) |
| Table half-built, rest missing | Fonts not reloaded in a later `evaluate_script` call | Reload fonts every call (7) |
| Script throws on `resize` | `resize()` before `appendChild`, or on a HUG axis | Fixed operation order in `size()` (2.2) |
| Last column drifts a few px | Non-zero inherited `itemSpacing` on rows | Explicit `itemSpacing: 0` and zero padding (2.4) |
| Text clipped vertically | Cells had no vertical padding and fixed heights | Cells hug height with 10px vertical padding |
| Previews overlap neighbors | Instance sized by auto-layout | Fixed-size non-auto-layout stage with clipping (3.1) |
| Wrong or zero style counts | `figma.mixed` treated as a string ID | Type guard on every style ID (5) |
| Traversal throws | `mainComponent` / `getNodeById` sync access | `getMainComponentAsync`, `getNodeByIdAsync`, `loadAllPagesAsync` |
| Table looks fine, is actually broken | `clipsContent: true` during build | Clip only after `verify()` is clean (2.4, 8) |

---

## 11. API checklist

- `primaryAxisAlignItems`, not `primaryAlignItems`. The latter throws.
- `layoutSizingHorizontal` / `layoutSizingVertical` require an auto-layout parent, so set them after `appendChild`.
- `"HUG"` requires the node itself to have auto-layout, or be a text node.
- `resize()` flips an auto-sized axis to FIXED; re-apply HUG/FILL afterwards.
- `strokeAlign = "INSIDE"` keeps strokes out of the layout box.
- `await figma.loadFontAsync()` for every weight, in every script call, before any `characters` assignment.
- Set `fontName` before `characters`.
- Set `textAutoResize = "HEIGHT"` before `layoutSizingHorizontal = "FILL"`.
- `await instance.getMainComponentAsync()`, never `instance.mainComponent`.
- `await figma.getNodeByIdAsync(id)`, never `figma.getNodeById(id)`.
- `await figma.getStyleByIdAsync(id)` after guarding `typeof id === "string"`.
- `figma.variables.getVariableByIdAsync` and `getVariableCollectionByIdAsync` are both async.
- `figma.skipInvisibleInstanceChildren = true` before large traversals.
- Internal read-only nodes cannot be moved; `clone()` first.
- Plugin data persists across `evaluate_script` calls; node references do not.
- Image hashes from `figma.createImage()` may not persist across calls; place live instances instead.
- `figma.createFrame()` defaults to 100×100. Setting `layoutMode` without sizing leaves an auto-layout frame that hugs; a `FILL` child inside it collapses to min-content rather than throwing.
- The root frame's parent is a `PAGE`, so `FILL` is invalid on it. Resize it directly and verify `width` afterwards.
- Read back what you set, not what Figma computed. `node.width` after your own `resize()` is reliable; a `FILL` or `HUG` width may not be.
- Do not use `FILL`, `HUG` width, or `layoutGrow` anywhere. All three are layout-resolved.
- Do not use `setPluginData` / `getPluginData`. Use `setSharedPluginData` / `getSharedPluginData` with a namespace, wrapped in try/catch.
- Never `JSON.parse` metadata directly; empty strings are the normal result of an unsupported call.
- Feature-check `figma.loadAllPagesAsync` before calling it.
- An inside stroke may or may not consume layout width. Budget for it.
- Wrap per-component property access in try/catch; malformed library components throw.
- Clean up after a failed build. An orphan frame on the canvas is worse than no output.
