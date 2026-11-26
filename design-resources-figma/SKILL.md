---
name: design-resources-figma
description: Use on every visual design task across projects when creating, implementing or reviewing interfaces, layouts or graphic design. Apply composition principles and platform specific guidance to the rendered result.
---

# Design Resources Figma

Start with the five composition checks in entry 22 of the resource index. Use them to identify what matters for the current change and verify the result at its intended viewing size.

Look up the relevant topic in [the design resource index](references/design-resources.md). Each entry links to a source and gives a Market Feelings example. Adapt the example to the current interface and preserve its design system.

Choose guidance by platform. For example, use Apple typography guidance for a native label and the CSS text metrics references for a web label. For other platforms, transfer the design principle and check implementation details against that platform’s documentation.

Use the source to resolve the specific decision. For optical centring, inspect the rendered lettering inside the control. For a slider, exercise its input methods and check the announced value. For motion, check the interaction with reduced motion enabled. Report which checks you actually performed.

For gradient lettering, use entry 18 to check the least legible part of the text in its actual state. Use entry 15 for control boundaries and state indicators. A single average colour can hide a local contrast failure.

For a custom slider, choose entries 19 and 20 on SwiftUI or 08, 14 and 21 on the web. Use entry 13 when the project already uses Motion for React. Check a middle value and both endpoints with the input methods the platform supports.

When a brief needs an asset or design tool, use entry 23 to pick a source and check the result in context.
