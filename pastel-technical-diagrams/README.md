# Pastel Technical Diagrams

Create editable technical diagrams in Sarang's saved pastel presentation style. The preset covers white slide backgrounds, soft category colours, Arial typography, thin outlines, small outline icons and generous spacing.

## Install

Copy this entire folder into your assistant's skills directory. For Codex, run from the repository root:

```sh
cp -R ./pastel-technical-diagrams ~/.codex/skills/
```

Start a new chat so the skill can be discovered. Automatic selection remains enabled by default. The Draw.io connector is preferred for interactive diagrams. The native files can also be opened in Draw.io Desktop or imported into [diagrams.net](https://app.diagrams.net/).

## Use

```text
$pastel-technical-diagrams Create a system architecture diagram for a support agent.
```

```text
$pastel-technical-diagrams Compare two context stacks using my pastel reference style.
```

## Included documents and examples

1. [Skill instructions](SKILL.md)
2. [Style guide](references/style-guide.md)
3. [Layout patterns](references/patterns.md)
4. [Reference image index](references/reference-index.md)
5. [Machine readable palette and font settings](assets/tokens.json)
6. [Editable architecture example](assets/architecture.drawio)
7. [Editable layer stack starter](assets/layer-stack.drawio)

The layout guide covers architecture, focus slides, stacks, comparisons, registries, tables, charts, trimming, memory documents, folders, wiki networks, tiered layers and recaps.

## Fidelity and limits

The original font and exact original colour tokens are unknown. Arial and Courier New are explicit recreation choices. The palette records the values used for the sample, which Sarang positively reviewed. Four supplied reference screenshots are included. Later screenshots are documented as visual patterns rather than claimed as retained images.

Native shapes, labels and connectors are editable. Four architecture icons use remote SVG image resources returned by the Draw.io shape service. Their internal paths are not editable and loading them needs network access. Use native stencils for a fully self contained rendition. No font files or original slide sources are bundled. Reference screenshots remain visual examples; no original rights are asserted.
