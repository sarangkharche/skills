---
name: component-handoff
description: "Created by **Mauro Berteri**. Generate one dev-handoff frame beside the source in the current Figma design file. The source component is read-only. The handoff is made of editable Figma text, frames, vectors, and **live instances** of actual source variants. This file contains the three visual systems; no external template file is needed."
metadata:
  source: https://www.figma.com/community/skill/67998/component-handoff
---

# Component Handoff

Created by **Mauro Berteri**. Generate one dev-handoff frame beside the source in the current Figma design file. The source component is read-only. The handoff is made of editable Figma text, frames, vectors, and **live instances** of actual source variants. This file contains the three visual systems; no external template file is needed.

## The result that must exist

The chosen style is a binding input. Do not replace it with a generic layout. Every run has these **nine** visible sections in this order:

1. Header / Component Info
2. Preview
3. Variant Grid
4. Do's and Don'ts
5. Interaction States
6. Anatomy
7. Agentic Documentation
8. Code & MCP
9. Copyright

For Card (modo-3), `Component Info` and `Body` are the **two direct children** of the root; Body contains sections 2–9. For Minimal, the nine sections form one flat padded stack. For Brutalist, `Container` (header) and sections 2–9 are direct root children; each content section is inset 64px horizontally. A missing section is a failed run, even if earlier sections look polished. Do not finish, summarize, or select the frame until the read-back in Final QA passes.

The component supplies content and live previews; these instructions supply structure and style. Do not paste the old Button examples into another component. Do not invent a variant, default, code import, Storybook URL, accessibility guarantee, or design-system rule. Use `—` for unknown external references and label inferred usage guidance as a suggestion. Never modify or detach a source component/variant/library node. Instances created **inside the new handoff** may be configured.

## Execution contract

- Load `figma-use` before calling `use_figma`; include it in `skillNames`. Use its current Plugin API rules and reference typings for exact setters. In particular: use top-level `await` and `return`, load current fonts before text mutations, return every created/mutated node ID, switch to the target page with `await figma.setCurrentPageAsync(page)` at most once per call, and make related containers with `figma.createAutoLayout()`.
- The input URL's `node-id` takes precedence over current selection. Resolve `256-16531` to `256:16531`. Accept a `COMPONENT_SET`, standalone `COMPONENT`, instance (resolve its main component and parent set), or a gallery frame whose **direct** child components/instances represent one coherent family. A frame is the placement anchor, not itself a variant. If several unrelated families remain, report them and request a specific source.
- Read the selected source and settle the **complete content plan before writing**. Create the final copy and final instances as each section is populated; do not build nine Button-placeholder sections and hope to replace them later.
- Each write call is retry-safe, returns root/section/created node IDs and counts, and retains IDs outside the file for the next call. A section or a few related sections can be one call when safe. If a call fails, follow `figma-use`'s `safeToRetryWithoutCanvasRead` rule. Do not quietly continue from a partial section.
- One composition screenshot after the nine sections exist, plus one after any visual repair, is sufficient. Inspect structure using returned IDs or metadata. Section counts and style-specific hierarchy are hard gates; screenshots check typography, clipping, colors, spacing, and legibility.

## 0. Lock choices, then inspect

First use the existing two-widget interaction if the invoking prompt has not already supplied its answers. Load the `interactive` module, use `show_widget`, and have each confirmation call `sendPrompt()` with a terse machine-readable result. Widget 1 has three selectable style cards with their hero-font/container description and sends `Style: modo-1` (Minimal), `Style: modo-2` (Brutalist), or `Style: modo-3` (Card). Widget 2 captures Language (Español or English; widget default Español), Version, Owner (Me or typed name), Library name, and Description (Keep / Trim / Replace), then sends e.g. `Language: es · Version: 1.0 · Owner: me · Library: Acme DS · Description: keep`. Save one immutable `RUN` object from the replies. Do **not** infer style from the component, start drawing while a choice is pending, or silently fall back to Minimal. If widgets are unavailable, ask for the missing style and metadata once, then continue. A later user choice overrides any fixture below.

For Owner=Me, read `figma.currentUser.name`; put the real name in the handoff, never “Owner” or “Me.” Translate visible labels/copy for the chosen language, but preserve component, owner, library, code, and URL spelling. Confirm the chosen style's title-font family/styles with `figma.listAvailableFontsAsync()` before rendering; load the actual font before writing text. If the title font is unavailable, report the substitution in the result instead of pretending the chosen style was reproduced exactly.

Description choices are concrete: **Keep** copies the source description exactly; **Trim** edits it shorter without adding claims; **Replace** uses replacement text entered in Widget 2 or explicitly supplied by the user. If Replace has no text, collect it before building. If the source has no description, use a short factual summary of the observed component and call it generated copy, or `—` if the function cannot be established. Never use the old Button placeholder description.

Build a read-only `SOURCE` manifest with: file key; page, anchor, component/set IDs; display name; description; default/representative variant; each **observed** variant `{id,name,axes,width,height}`; root property definitions; nested component names; and any known code reference. Promote a variant `COMPONENT` to its parent `COMPONENT_SET` before accessing `componentPropertyDefinitions`. Inspect the union of variants, not only the first/default. For a gallery of standalone symbols, parse their repeated `Axis=Value` name pairs and retain exact variant IDs. Never manufacture the missing Cartesian combinations.

Inspect actual prototype reactions where present, along with design variants, before naming Interaction States. A static variant can prove its depicted condition; it cannot prove keyboard behavior or the runtime transition into that condition.

Clean the display name only when clearly mechanical: `sidebar.right/sidebar.right` → `sidebar.right`; preserve its dot and case. Keep the original source name/ID for instances and links. The source `256:16531` is a gallery frame with nine direct symbols; a selection-only check that rejects that frame is wrong for this input.

Use this read-only resolver as the start of inspection. Substitute `__INPUT_URL_JSON__` with a JSON string literal (or `""`); the execution tool must target that URL's file. The result is a source identity, **not** permission to mutate it.

```js
const INPUT_URL = __INPUT_URL_JSON__;
let anchor;
if (INPUT_URL) {
  const match = INPUT_URL.match(/[?&]node-id=([^&#]+)/);
  if (!match) return { ok: false, error: 'URL_HAS_NO_NODE_ID' };
  anchor = await figma.getNodeByIdAsync(decodeURIComponent(match[1]).replace(/-/g, ':'));
  if (!anchor) return { ok: false, error: 'NODE_NOT_IN_ACTIVE_FILE' };
} else {
  const selection = figma.currentPage.selection;
  if (selection.length !== 1) return { ok: false, error: 'SELECT_ONE_COMPONENT' };
  anchor = selection[0];
}
let page = anchor;
while (page.parent && page.parent.type !== 'DOCUMENT') page = page.parent;
if (page.type !== 'PAGE') return { ok: false, error: 'NO_SOURCE_PAGE' };
if (figma.currentPage.id !== page.id) await figma.setCurrentPageAsync(page);
anchor = await figma.getNodeByIdAsync(anchor.id);
const family = async n => {
  if (n.type === 'COMPONENT_SET') return n;
  if (n.type === 'INSTANCE') n = await n.getMainComponentAsync();
  if (n && n.type === 'COMPONENT') return n.parent && n.parent.type === 'COMPONENT_SET' ? n.parent : n;
  return null;
};
let target = await family(anchor), gallery = [];
if (!target && 'children' in anchor) {
  const direct = anchor.children.filter(n => ['COMPONENT','COMPONENT_SET','INSTANCE'].includes(n.type));
  const roots = new Map();
  for (const n of direct) { const f = await family(n); if (f) roots.set(f.id, f); }
  if (roots.size === 1) target = [...roots.values()][0];
  else {
    const components = [];
    for (const n of direct) {
      const c = n.type === 'COMPONENT' ? n : n.type === 'INSTANCE' ? await n.getMainComponentAsync() : null;
      if (c && c.type === 'COMPONENT' && !components.some(x => x.id === c.id)) components.push(c);
    }
    const axes = name => name.split(',').map(part => part.split('=')[0].trim()).sort().join('|');
    if (components.length >= 2 && components.every(n => n.name.includes('=') && axes(n.name) === axes(components[0].name))) {
      gallery = components; target = components[0];
    }
  }
  if (!target) return { ok: false, error: 'AMBIGUOUS_GALLERY', candidates: [...roots.values()].map(n => n.name) };
}
if (!target) return { ok: false, error: 'NO_COMPONENT_FAMILY', type: anchor.type };
const variants = target.type === 'COMPONENT_SET' ? target.children.filter(n => n.type === 'COMPONENT') : gallery.length ? gallery : [target];
const parts = (gallery.length ? anchor.name : target.name).split('/').map(x => x.trim()).filter(Boolean);
const displayName = parts.length > 1 && parts.at(-1) === parts.at(-2) ? parts.at(-1) : (gallery.length ? anchor.name : target.name);
return { ok: true, pageId: page.id, anchorId: anchor.id, targetId: target.id, displayName, variants: variants.map(v => ({ id: v.id, name: v.name, width: v.width, height: v.height })) };
```

Then inspect the returned variants and property owner in the target page. For a `COMPONENT_SET`, use its definitions and each child's variant properties; for standalone gallery symbols, parse the axes from the names and retain exact variant IDs for previews. A gallery name is evidence of an observed combination, not a `setProperties()` key.

For display labels, strip ornamental prefixes such as `⚙️` or `🧪` from axis names, so `⚙️ Tab` is shown as `Tab`; retain the unmodified key and value in `SOURCE` for any actual instance property setting. If `figma.fileKey` or a source field is unavailable, record that gap; never synthesize an ID or a default value.

Save a content plan `PLAN` before creating nodes:

```text
style, language, owner, library, version, description,
rootName, anchorId, pageId, sourceComponentId,
previewVariantId,
variantGroups: [{label, variants:[{sourceVariantId, caption}]}],
states: [{label, sourceVariantId, description}],
dos: [three {title, description}], donts: [three {title, description}],
anatomyRows: [{label, value}],
agentMarkdown, mcpLink, importReference, sourceFileReference, storybookReference
```

Every `sourceVariantId` must resolve to a real component variant from `SOURCE`; every caption must accurately describe that variant. If a section's content is unknown, keep the section and render an honest `—` or short “not specified” note inside it.

### How to choose grid rows and state examples

Pick **one meaningful grouping axis** for Variant Grid and make one labeled row per observed value of that axis. Here the `Tab` axis gives Design (3), Prototype (3), Comments (2), Inspect Properties (1). Within each row, show only the real combinations and caption them with the remaining distinguishing axes, in stable source order. Do not flatten all nine into an arbitrary 3×3 sequence or print emoji-prefixed raw component names as captions.

Interaction States documents representative **observable conditions of this component**, not a generic Button checklist. Prefer six distinct, supported examples when the source has them. For this sidebar: Default, Frame Selected, Comments Active, Local Styles, Prototype Device, Viewer Mode. Each has a full-size live instance and a short factual description. Do not insert a “missing hover/pressed/focus/loading” warning merely because those Button states have no variants here. If a different source truly has fewer meaningful states, show the supported ones and state the limit in the section description.

## 1. Style lock — use one column from start to finish

Do not average styles or borrow one style's padding/font/fill for another. These are the fixed visual tokens. Geometry is in pixels; colors are hex here and must be converted to 0–1 RGB for Plugin API paints.

| Role | Minimal · modo-1 | Brutalist · modo-2 | Card · modo-3 |
|---|---|---|---|
| Root | width 1200; white; radius 0; padding 64 all sides | width 1200; **entire root** `#f5f0e9`; radius 0; top/sides 0, bottom 64 | width 1200; white; radius 48; top/sides 24, bottom 64 |
| Root tree | flat vertical stack; all nine sections direct siblings; gap 64 | `Container` header, then eight content sections as direct root siblings; gap 64; each content section inset 64 horizontally | `Component Info`, `Body` direct siblings; gap **48**; Body horizontal padding **48**, section gap **64** |
| Content width | 1072 | 1072 | 1056 |
| Header shell | `Component Info`, gap 24, no fill/border | see the three-piece bar below | `Component Info`, `#d0b6ff`, radius 32, pad 48, internal gap 81 |
| Metadata structure | `Header Top` (title + version), then `Component Details`, gap 24 | separate `Component Details` bar: FILL `Owner Container` spacer then HUG Owner/Library/Version boxes with touching borders | `Component Details`: grouped white Owner/Version/Library pills left; purple Documentation pill right |
| Hero title | Inter Bold 64, `#1c1c1f` | IBM Plex Sans Medium 81, `#1a1e1b` | Geist Medium 81, `#54299b` |
| Hero description | Inter Regular 16, line height 150%, `#8c8c8c` | Inter Regular 24, line height 140%, `#1a1e1b` | Inter Regular 24, line height 140%, `#1a1e1b` |
| Section title | Inter Semi Bold 40, `#1c1c1f` | IBM Plex Sans SemiBold 48, `#1a1e1b` | Geist Medium 48, `#54299b` |
| Section description | Inter Regular 20, line height 140%, `#1a1e1b` | same | same |
| Owner/Library value | Inter Regular 20 | Inter Medium 20 | Inter Medium 20 |
| Version | green `#00b894` pill, radius 100, pad 6/14, Inter Semi Bold 16 white, `v1.0` shape | box height 70, HUG width, pad 0/24, title-font SemiBold 20, `V.1.0.0` shape | white pill, radius 80, pad 16/24, Geist SemiBold 20, `V.1.0.0` shape |
| Documentation control | none | in top bar, pad 0/24, dark text + dark 32×32 arrow | purple `#54299b` pill, radius 80, pad 16/24, gap 10, white text + white 32×32 arrow |
| Preview container | `#f2f2f5` | white + 0.5 `#1a1e1b` stroke | `#f4f4f4` |
| Preview padding | 80 | 81 | 81 |
| Anatomy, Agentic, Code containers | `#f4f4f4` | white + 0.5 dark stroke | `#f4f4f4` |
| Variant/state cell light | `#f2f2f5` | white + 0.5 dark stroke | `#f4f4f4` |
| Variant/state cell dark | `#2e2e33` | `#1a1e1b` | `#1d1a1e` |
| Row-group / guidance-card title | Inter Semi Bold 24 / 18 | IBM Plex Sans SemiBold 24 / 18 | Geist SemiBold 24 / 18 |

The **title role** is the hero, section titles, grid row labels, guidance-card titles, and version value; use that style's family/weight above. The **body role** is everything else, always Inter, including Agentic Documentation (never Roboto Mono). Two deliberate exceptions: Anatomy’s component name uses Inter Semi Bold 24 in every style; version uses the title family even beside Inter owner/library values. Check exact available style spellings (`Semi Bold` versus `SemiBold`) rather than guessing.

Shared small type: variant/state caption Inter Medium 11; guidance and state description Inter Regular 13 with 150% line height; anatomy eyebrow Inter Semi Bold 12, label Inter Medium 12, value Inter Regular 18 with 150% line height; Agentic tag Inter Regular 12/16 and Markdown Inter Regular 13/22; Code labels Inter Medium 12, link/value Inter Regular 18 with 150% line height, Dev Mode rows Inter Regular 14 with 150% line height; Copyright Inter Regular 12. Use secondary `#646464` for explanatory copy and `#8c8c8c` for muted labels where indicated. Each text node gets a fixed wrapping width and auto height.

Brutalist's 70px header bar is a full-width horizontal row with `#f5f0e9` fill and 0.5px `#1a1e1b` stroke: `System Name Box` (HUG width, pad 0/24, use the real Library name) → FILL spacer → `Documentation Box` (HUG width, pad 0/24, flush right). Its Owner/Library/Version `Component Details` bar is separate. Do not drop the System Name Box or put Documentation on the left. Card's `Component Details` has grouped white Owner/Version/Library pills on the left and the purple Documentation pill on the right. Minimal has `Header Top` title/version and then metadata, with no Documentation button.

In Card, **all three** Owner/Version/Library pills share a white fill, radius80, 16px vertical/24px horizontal padding; labels use small muted Inter and values use Inter Medium 20 (Version value Geist SemiBold 20). Size each pill to its actual content. The Documentation pill uses the same shell metrics with purple fill and white label/icon. Keep the three white pills grouped left and Documentation aligned right; do not reduce the metadata to initials or a single plain text line.

All Preview/Anatomy/Agentic/Code containers, variant cells, state cells, and guidance cards use radius **16** unless a specific header/pill/root rule above overrides it. Any rounded variant/state cell with a bottom overlay sets `clipsContent=true`. Check/X circles are 24×24 and contain a real white vector glyph; a colored ellipse alone is incomplete. Section/card gaps and padding should come from the tables and section recipes below, never the generic layout's 41px inset.

## 2. Build geometry from the selected source

Measure every planned variant by creating a throwaway instance or reading its component dimensions; remove throwaway instances. Use actual instance size when laying out preview and cells. For a grid/state cell:

```text
cellWidth  = max(200, max rendered variant width in that row + 48)
cellHeight = rendered instance height + 24 top + 24 gap + 32 bottom caption
rowGap = 16 horizontal; wrapped-line gap = 24
```

Each source instance sits at (24,24) inside its cell at original scale. Set the cell's width explicitly and use HUG height where variants differ; the row wrapper hugs its tallest cell. An auto-layout row uses `layoutWrap='WRAP'`, so the content width determines columns. Do not set all cells to FILL, fix the count at 3/4, shrink live instances to thumbnails, or let a wide instance bleed into its neighbor. If a single full-size cell exceeds the style's content width, widen the root/content together while retaining the style's horizontal padding. In the reference Card, source instances are 240×704, so cells are **288×784** and three fit in a 1056-wide row at 16px gaps.

The caption/label is a 32px-high frame overlaid at the bottom of each cell. Build **all** siblings in a row, let layout settle, then in a second pass set each overlay to `layoutPositioning='ABSOLUTE'`, `resize(cell.width,32)`, `x=0`; set ABSOLUTE before x/y. Default `y=cell.height-32`. **Minimal variant captions alone** use `y=cell.height-32-20`; leave their 20px inset clear. Minimal variant captions are centered; Brutalist captions are centered on tan fill with 0.5 dark stroke; Card variant captions are **right aligned**, pad right 16, no fill/border. State labels use the default flush-bottom y and are centered in all three styles; Minimal uses a neutral fill, Brutalist tan/stroked, Card no fill/border. Do not copy Card's variant-caption alignment to its state labels.

For a state card, put the live instance in the rounded cell, its label overlay at the bottom, then a separate description text **below** the cell. Description text has a real FIXED width equal to the cell width and `textAutoResize='HEIGHT'`; the outer `{State} Wrapper` is vertical HUG with 16px between cell and description. Never fix the description wrapper or the row wrapper to 100px. In the reference Card, each state wrapper is 288×860; three fit across, then three wrap to a second line with 24px between lines.

Sizing rule: frames that contain related children use auto layout; append children **before** setting HUG/FILL on a child; call `resize()` **before** layoutSizing setters because it resets those modes. Only an auto-layout frame or eligible child can use HUG/FILL. A wrapping TEXT node uses fixed width plus `textAutoResize='HEIGHT'`, not horizontal HUG or a guessed fixed height. Set API line height with `{unit:'PERCENT',value:150}` or `{unit:'PIXELS',value:22}`, never a bare number. Unspecified *container* dimensions hug content where valid; this is not a blanket instruction to put HUG on every node. Ensure text height is at least its actual rendered line height. The failed reference has a 10px-high description box containing 20px text; reject that as clipping.

Each Do/Don't guidance row has three cards for this reference. Use a white card, 0.5px green/red outline, radius16, pad24, gap24, HUG height. At Card width 1056, three cards with 16px gaps are about 341.33px each. A long title can wrap to two lines without hiding its description. An optional illustrative source instance may appear inside a card only if it fits at readable size; the desired sidebar example uses icon/title/description cards without adding irrelevant instance thumbnails.

Use the actual vector paths below inside 24×24 green/red circles. An ellipse without the inner path is not an icon. The arrow is for Brutalist/Card Documentation controls only.

```js
const ICONS = {
  check: fill => `<svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M4.61 8.13 6.68 10.20 11.39 5.49" stroke="${fill}" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round"/></svg>`,
  close: fill => `<svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M5.05 5.05 10.95 10.95M10.95 5.05 5.05 10.95" stroke="${fill}" stroke-width="1.4" stroke-linecap="round"/></svg>`,
  arrow: stroke => `<svg width="32" height="32" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M9.33 22.67 22.67 9.33M22.67 22.67V9.33H9.33" stroke="${stroke}" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`
};
```

Make each icon an editable **24×24 circular auto-layout frame** (green or red fill, radius12, both axes centered), then append the 16×16 `figma.createNodeFromSvg(ICONS.check('#ffffff'))` or close vector inside it. Do not try to append a vector to an ellipse, which cannot have children. Colors: green `#21b873`, red `#e54040`, green card stroke `#00b894`, red card stroke `#e54040`; body text `#1a1e1b`, secondary `#646464`, muted `#8c8c8c`.

## 3. Render the complete section tree

Create a root named `[displayName] · Handoff` on the **anchor's page**. Append it to that page, compute placement from the anchor's `absoluteBoundingBox` in page coordinates, and put it to the right of the source and any existing handoff frame on the same row, with at least 200px clear canvas gap. Do not cover the source or previous result. Create the style-specific root/header/section hierarchy from Section 1, then create all nine named section shells in order. Return the IDs of the root, header, optional body, and every section before filling them. Keep source IDs separate from created IDs.

Fill each shell from `PLAN`. A section is complete only when all its required nodes/text/instances exist. It is fine to batch multiple sections per call when safe to retry, but never report a successful handoff after only six. For each new TEXT: load its intended font before assigning characters, then use the style's role, fixed wrapping width, and auto-height. For non-Inter title fonts, first check their actual available style spellings. Return IDs, row/card counts, sourceVariantIds, and bounds from each write. After a partial failure, inspect or remove the partial created section and retry that section; leave the source untouched.

### 3.1 Header

Use the **display name** as hero title, real description according to the metadata choice, resolved Owner, Library, and Version. Do not put a tiny badge containing owner initials where the full Owner/Library/Version metadata belongs. Card has the lavender `Component Info` card with purple hero, white metadata pills, and the purple Documentation pill with arrow. Its `Component Info` and `Body` remain direct root siblings. Minimal and Brutalist follow their own header structures in Section 1. Set the description text to the final content before sizing; it must wrap and show all lines.

### 3.2 Preview

Show **one** live instance of `PLAN.previewVariantId`, centered at original scale inside a radius16 Preview container. Write **one** caption directly under the instance inside that container, describing its actual axes. Do not repeat the caption elsewhere. For Card, the section is title (62px in this fixture) → 24px gap → description (28px in this fixture) → 24px gap → Preview container with 81px padding. With a 240×704 instance, the desired container is 1056×903: instance at (408,81), caption beneath. Other source sizes grow the container.

### 3.3 Variant Grid

Render one group per `PLAN.variantGroups`, each as `Row Label` then `Cells · {group label}`. Each cell has exactly one live source instance and a caption for the other axes. Preserve planned group order and observed variants; never insert hypothetical combinations. Use the Section 2 sizing and overlay rules. For this sidebar, four groups with **3/3/2/1** cells are mandatory; all nine instances stay 240×704 within 288×784 cells. The failed flat 3×3 enumeration is not a valid grouping even though it has nine instances.

### 3.4 Do's and Don'ts

Render a Do heading and **three** Do cards, then a Don't heading and **three** Don't cards. For sources without published usage guidance, author specific, plausible suggestions from observed props/variants and label the section copy as suggestions. Do not state inferred advice as a verified system rule. Each card includes a real check/X vector icon, a title, and a description; cards HUG their content. Do not cut down to 2+2 to save time or substitute two giant generic columns.

### 3.5 Interaction States

Render the `PLAN.states` labels and descriptions, each with exactly one full-size live instance in its own cell. These are meaningful supported conditions, not every raw variant and not generic hover/focus/loading slots. Use the state-label and description rules from Section 2. For this sidebar the six labels in Section 0 must appear in two rows of three. Do not shrink them to ~72×211 thumbnails or add an unrelated “missing expected interaction states” banner.

### 3.6 Anatomy

Use a radius16 container with an eyebrow, the display component name in **Inter Semi Bold 24**, and vertically stacked `Label`/`Value` property rows. Source the values from variant counts, axes, root component properties, and nested instances; avoid a bag of dependency badges and a duplicate source preview. The reference sidebar has seven rows: `Variants`, `Selection`, `Tab`, `State`, `Editor`, `Properties`, `Nested components`. Each row has a 12px Inter Medium label and 18px Inter Regular value. Unknown values are `—`, not invented defaults.

### 3.7 Agentic Documentation

Always include this section, even when source evidence is sparse. Inside a radius16 container, render a small `markdown` tag and a **readable Markdown block** containing the real component name, overview, usage, actual observed axes/properties with values, the Do/Don't guidance with its suggestion status, and accessibility notes. Accessibility notes distinguish observed design evidence from implementation requirements; never assert WCAG conformance, keyboard support, or ARIA behavior based only on a static frame. Use Inter Regular 13px with 22px line height and fixed text width/auto-height. Do not use Roboto Mono. Populate the block before sizing it; do not leave the Button placeholder copy.

### 3.8 Code & MCP

Always include this section. Derive the Figma link from the actual current `fileKey`, file name, and `SOURCE.anchorId`: `{input URL origin, or https://www.figma.com}/design/{fileKey}/{encoded fileName}?node-id={anchorId with : replaced by -}`. Preserve `https://staging.figma.com` for a staging input; do not link a staging-only file on the production host. Include the link under `MCP Link`, plus `Dev Mode` rows for import, source file, and Storybook. Use real supplied code references if known; otherwise `—`. If the file key is unavailable, show `—` rather than fabricate a link. The reference has a Storybook `—`; do not fabricate a URL. Body uses Inter, not the title family.

### 3.9 Copyright

Always append a separate small Inter text node named `Copyright`: `Generated with /component-handoff · Mauro Berteri`. It is section nine, not an optional afterthought.

## 4. Final QA — read back, render, repair

After all writes, read the newly created root and its descendants. Use the IDs returned by the write calls; do not find an earlier handoff by name and mistake it for this run. Reject completion and repair the relevant section if any row below fails.

| Gate | Required read-back |
|---|---|
| Hierarchy | Minimal: nine ordered root children from `Component Info` through `Copyright`. Brutalist: ordered root children `Container`, `Preview Section`, `Variant Grid Section`, `Do's and Don'ts Section`, `Interaction States Section`, `Anatomy Section`, `Agentic Documentation Section`, `Code & MCP Section`, `Copyright`. Card: root has exactly `Component Info`, `Body`; Body has the latter eight nodes in that order. |
| Header/style | Chosen style's root fill/radius/padding, hero font/color, section-title font/color, and metadata grouping match Section 1. Card has lavender header, white metadata pills, purple Documentation pill, 48px root sibling gap, and 48px Body inset. Brutalist has a tan root and the three-piece header bar. |
| Preview | One live instance of the planned real source component at its native dimensions; one caption inside Preview; no caption duplicate. |
| Variant Grid | Every observed `SOURCE` variant appears exactly once as a live instance, grouped by one meaningful axis, with real-source captions. Count each `Cells · {group}` and confirm its cell count equals `PLAN`. |
| Do's and Don'ts | Exactly three complete Do cards and three complete Don't cards for the reference; each has its vector check/X, title, and visible description. For a different run, PLAN still has three of each unless the user explicitly changes the requirement. |
| Interaction States | Every planned named state has one unscaled live instance, bottom label, and visible description below its cell; no invented state or unrelated missing-state banner. |
| Anatomy | Eyebrow, component name, and a `Label`/`Value` row for every `PLAN.anatomyRows` entry; no duplicate source preview substituted for the rows. |
| Agentic Documentation | A nonempty Markdown block with the real component name, observed axes/properties, usage, guidance, and qualified accessibility notes; `markdown` tag; no Button placeholder or Roboto Mono. |
| Code & MCP | Source link uses real key/name/node and correct host; import/source/Storybook rows exist, with `—` where unknown. |
| Copyright | Separate `Copyright` text with the exact line in §3.9. |

For the supplied regression fixture (`256:16531`), a Card/English run must yield `sidebar.right · Handoff`, the Card hierarchy above, **9** grid variants in Tab rows **3/3/2/1**, **6** full-size state examples named in §0, **3+3** guidance cards, **7** Anatomy rows, and populated Agentic Documentation, Code & MCP, and Copyright. That implies **16** live instances across Preview/Grid/States when the guidance cards contain none. The reference happens to be 1200×10481 with 240×704 source instances and 288×784 cells; do not force that height or those cell dimensions on a different source.

Take a screenshot of the complete root after structural checks. Inspect at readable zoom, including lower sections: all text is visible inside its frame, nothing overlaps or escapes rounded containers, source instances remain full size, captions sit on their own cell, three cards fit or wrap cleanly, and Card styling remains lavender/purple/white throughout. Repair only the failing section and take one more screenshot. Re-run the read-back after repairs. A shorter generic sheet that stops after Anatomy is **not** a completed handoff.

Return the new frame's link, style, observed variant/state counts, and any honest gaps (missing font, absent code reference, unavailable source description). Do not claim parity from a screenshot alone; structural read-back and visual inspection must both pass.
