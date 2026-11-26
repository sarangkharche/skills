# Visual specification

## Palette

These are the recreation values used for the approved sample. They are not verified original source tokens. Use solid fills and no shadows or gradients.

| Role | Fill | Outline |
| --- | --- | --- |
| Canvas | `#FFFFFF` | none |
| Main text and connectors | `#111111` | `#111111` |
| Secondary text | `#666666` | none |
| Agent loop or stable prefix | `#DFE5FF` | `#7065EE` |
| Model or cached content | `#BFEFC0` | `#111111` |
| Tools or result content | `#DDF0E1` | `#6A9B77` |
| Context management | `#FADDBB` | `#D89A50` |
| Memory | `#E9DFF7` | `#8C69CE` |
| Retrieval | `#FFF0A8` | `#C6B54D` |
| Changed or problematic content | `#FBE0E0` | `#D55C68` |
| Neutral content | `#ECECF1` | `#666666` |
| Inactive outline | white | `#BBBBBB` |
| Strong memory index | `#BEAAEE` | `#8C69CE` |
| Very pale memory tier | `#F6F0FC` | `#8C69CE` |
| Strong context summary | `#F8C780` | `#D89A50` |

Additional tier colours are visual approximations from screenshots. Refine them against the nearest reference when necessary. Keep category colours consistent across blocks, markers, outlines and section numbers. Green and red can express cached versus changed status in comparisons. Do not colour every shape differently.

## Type and slide proportions

Use Arial regular as the implementation default, with Helvetica or a plain sans serif fallback. Original font identity is unknown. Use Courier New for code, function names, paths and filenames, not ordinary conceptual labels. Bold is limited to selected filenames or compact technical labels. Do not add artificial letter spacing.

Use a white landscape slide, normally 16:9. The approved sample uses a 1200 by 780 working frame; adapt the frame to the requested output. At a 1200 unit width, useful starting sizes are: title 32 to 36, section opener title 44 to 52, component label 22, strip label 17, secondary caption 15 to 18 and breadcrumb 16. These are proportional recreation values, not measured source point sizes. Maintain legibility at the actual display size.

Place normal titles at top left and small grey section breadcrumbs at top right. Optional numbered title circles have pastel fill, category outline and black numeral. For focus slides, place the diagram left and the section number, large title and short grey subtitle right. Keep roughly 48 units of outer margin and ample whitespace around the main figure.

## Shapes and icons

Stack strips use square corners, equal widths and narrow even gaps. Components use slight rounding, typically 5 to 10 units, not pill shapes. Thin outlines are about 1 to 1.5 units at 1200 width. Highlighted paths or selected content can use about 2 units. Keep background neutral and flat.

Use small black outline icons beside labels, with uniform stroke weight. Common motifs are wrench, layers, notebook, magnifier, folded page, folder, globe, database, cube, person, lock and crescent moon. Avoid filled clip art, coloured emoji and unrelated icon families. Highlighted memory pages may use purple strokes and pale lavender fill. Technical excerpts use a white dashed callout with restrained syntax accents.

## Connectors and grouping

Use thin black straight or orthogonal connectors with solid triangular arrowheads. Parallel links should stay parallel. Use bidirectional arrows only for two way relationships. Network background links may be thin grey lines without arrowheads; a selected path can use heavier black arrows. Curved arrows are reserved for loops or annotation leaders.

Dash meaning depends on the pattern: group boundary, appended placeholder, optional retrieval or omission marker. Keep each meaning clear within one diagram. Use black or grey dashed frames, dotted subdued controls and white unfilled pending boxes. Group stack sections with square brackets or curly braces and short external labels. White black outlined number circles may sit on sequence connectors.

Leave label clearance above horizontal lines. Neither label bounds nor glyphs may touch a wire, container edge or another label. Align model prompt and reply ports using their absolute coordinates so differently sized blocks do not create slanted wires.

## Emphasis and exclusions

Fade inactive components, labels, icons and links together to light grey on focus slides. Retain the active component's category fill, stronger outline and black label. Keep one consistent accent across section number and active step marker.

Never recreate Zoom controls, participant faces or names, reactions, PDF page counters, desktop borders, stray pen marks, cursor or blue text selection. Grey text selection in a screenshot is not an intentional default text highlight. Handwritten captions occur occasionally as secondary annotations; they do not replace the default typography.

### Compact process diagrams

Use the user's preferred compact process block of 120 × 60 Draw.io units as the starting size. Use short centred labels, regular Arial, flat pastel fills, thin outlines and generous white space. Give the main sequence one reading direction and keep exceptions on a separate aligned row. Use a dashed enclosing boundary only when it identifies a real system or group. Keep longer explanations in notes or on a detail page. Scale the composition proportionally for the display size. Reference: ByteByteGo, Building AI Agents slides, pages 9 and 11, https://assets.bytebytego.com/Building-AI-Agents-Slides.pdf.
