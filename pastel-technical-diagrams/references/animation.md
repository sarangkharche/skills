# Animated diagrams

## Approved motion treatment

Sarang accepted this treatment on 16 November 2026 after reviewing animated samples.

1. Animate only the connector dashes. Keep the triangular arrowheads fixed and clearly visible.
2. Keep all blocks, icons and labels static. Preserve their pastel fills, outline colours, outline widths, position and size throughout playback. Do not pulse blocks or emphasise destinations on arrival.
3. Keep every intended flow connector moving continuously in its source to target direction. Use separate directional connectors for request and response when needed. Do not alternate connections, stage handoffs or stop after one pass.
4. Draw only one dashed stroke per connector. Do not add a solid grey track, background line, glow or shadow underneath it.
5. Use calm, constant speed and consistent dash spacing. Avoid bounce, zoom, flashing and speed changes. Motion is illustrative unless actual throughput or latency data is supplied. Do not imply that animation speed is a measurement.

This treatment applies when the user requests animation. Static diagrams remain static. Preserve an explicitly requested alternative.

## Native Draw.io source

Enable Flow Animation on each connector that represents flow. A starting edge style is:

```text
html=1;strokeColor=#111111;strokeWidth=1.5;endArrow=block;endFill=1;dashed=1;dashPattern=7 9;flowAnimation=1;flowDuration=1000;flowTiming=linear;flowDirection=normal;
```

Use the existing layout and connect real source and target shapes. Do not animate vertices or add a second edge underneath the animated edge. The duration above is a starting value for native flow, not a prescribed travel time or system latency. Adjust it after checking playback. Native Flow Duration controls pattern motion and should not be treated as a whole diagram timeline.

Native playback depends on the editor or viewer. Check the rendered result in the available environment before claiming it matches the preview. If the native renderer adds an unwanted track or cannot reproduce the treatment, keep the editable source and provide a separate preview that matches the approved appearance. State the limitation instead of claiming native parity.

## Preview delivery

Keep the native `.drawio` source editable. Deliver a playable preview when the user needs to see the motion. A browser preview can animate SVG connector dash offset while leaving nodes and arrowheads untouched. Use constant stroke width. Avoid adding a second path as a track.

Make the loop seamless: the dash offset at the end must match the start modulo one complete dash pattern. For example, `7 9` has a 16 unit period. Moving the offset from 0 to negative 16 over a chosen duration produces one seamless cycle. Animate all intended flow edges throughout that cycle. Follow the path from source to target, including connectors that point left or up.

Browser previews loop automatically unless reduced motion is requested. Provide working Pause, Play and Static view controls. Pause holds the current position. Static view stops motion and retains a readable diagram. Honour the reduced motion preference on load and when it changes. An explicit Play action may start a requested demonstration.

GIF exports loop indefinitely. Provide a static alternative and a controllable browser preview when practical, since a GIF cannot provide its own pause control. Use a shared colour palette across GIF frames so quantisation does not make static blocks flicker. An exported SVG, PNG or PDF is not automatically animated merely because the source contains flow settings.

## Verification

1. Check XML structure, unique IDs, parent references and edge endpoints.
2. Inspect the rendered diagram for text clipping, clear arrowheads, aligned paths and absence of a grey track.
3. Check playback beyond a complete cycle. Every intended flow connector must keep moving in the correct direction without a visible reset or pause.
4. Compare blocks across frames. Their fill, outline, labels and geometry must remain unchanged. When checking a raster preview, compare block pixels as well as the source settings.
5. Check browser controls and reduced motion behaviour when providing a browser preview. Check that exported GIF metadata requests an infinite loop.
6. Report native playback, preview playback and structural validation separately. Mark any unavailable check unverified.

## Supporting documentation

1. [Draw.io connector animation settings](https://www.drawio.com/docs/manual/styles/connector-animation-styles/) describes native direction, timing and duration controls.
2. [W3C pause, stop and hide guidance](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html) explains controls for qualifying automatic motion lasting more than five seconds alongside other content.

The continuously flowing arrows, static blocks and absence of a grey track are Sarang's accepted style preference. They are not a universal requirement from these sources.
