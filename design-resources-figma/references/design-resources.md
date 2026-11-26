# Design Resources Figma

Use this index to find a source for a specific design question. Each entry names the platform, explains the source and gives a project application. Applications describe design or verification work. They do not establish that the app implements or passes a check.

## Typography and optical alignment

### 01. MDN | text-box-trim

**Platform** Web

**Source** [text-box-trim](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/text-box-trim)

**Topic** Explains the invisible space above and below letters.

**Application** Start here for the mood label and the Next button. Check browser support before depending on this property.

### 02. W3C | Text edges

**Platform** Web standard with principles applicable to native interfaces

**Source** [Text edges](https://www.w3.org/TR/css-inline-3/#text-edges)

**Topic** Defines cap height and baseline.

**Application** Use these two edges to centre an uppercase label inside the black artwork.

### 03. Apple | Typography

**Platform** iOS

**Source** [Typography](https://developer.apple.com/design/human-interface-guidelines/typography)

**Topic** Covers hierarchy, legibility and Dynamic Type.

**Application** Give every mood the same size at the same user text setting and check the longest label first.

### 04. Apple | Applying custom fonts

**Platform** SwiftUI

**Source** [Applying custom fonts](https://developer.apple.com/documentation/swiftui/applying-custom-fonts-to-text)

**Topic** Shows how to use a brand font in SwiftUI and scale it relative to a system text style.

**Application** Useful for bringing Nohemi into the iPhone build.

## Layout and controls

### 05. Apple | Layout

**Platform** Apple platforms with design principles applicable elsewhere

**Source** [Layout](https://developer.apple.com/design/human-interface-guidelines/layout)

**Topic** Covers safe areas, alignment and adaptable layouts.

**Application** Use it to check the relationship between the artwork, prompt, slider and button on smaller phones.

### 06. Apple | Design Resources

**Platform** iOS

**Source** [Design Resources](https://developer.apple.com/design/resources/)

**Topic** Provides official UI kits and templates.

**Application** Compare button proportions, margins and navigation against the iPhone templates.

### 07. Apple | Sliders

**Platform** Apple platforms with design principles applicable elsewhere

**Source** [Sliders](https://developer.apple.com/design/human-interface-guidelines/sliders)

**Topic** Explains the track, thumb and endpoints.

**Application** Use it as the reference for how the silver mood slider should read and respond.

### 08. W3C | Slider pattern

**Platform** Web

**Source** [Slider pattern](https://www.w3.org/WAI/ARIA/apg/patterns/slider/)

**Topic** Covers keyboard controls, accessible names and spoken values.

**Application** In the web component, let people hear “Extreme fear” when they select that stop.

## Motion, glow and feedback

### 09. Apple WWDC | Animate with springs

**Platform** Apple platforms with design principles applicable elsewhere

**Source** [Animate with springs](https://developer.apple.com/videos/play/wwdc2023/10158/)

**Topic** Demonstrates how duration, bounce and gesture velocity change the feel of a spring.

**Application** Watch this for the transition from dragging the thumb to settling on a stop.

### 10. Apple | Motion

**Platform** Apple platforms with design principles applicable elsewhere

**Source** [Motion](https://developer.apple.com/design/human-interface-guidelines/motion)

**Topic** Explains when movement helps people follow an action.

**Application** Use the glow to acknowledge slider movement while the artwork silhouette stays fixed.

### 11. Apple | Playing haptics

**Platform** iOS

**Source** [Playing haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)

**Topic** Explains tactile feedback.

**Application** Use it to decide whether a light selection tick helps people feel each mood change.

### 12. W3C | Colour interpolation

**Platform** Web standard with principles applicable to native interfaces

**Source** [Colour interpolation](https://www.w3.org/TR/css-color-4/#interpolation)

**Topic** Explains how colours mix between gradient stops.

**Application** Compare the purple to orange midpoint in different colour spaces before adding blur to soften it.

### 13. Motion | useReducedMotion

**Platform** React

**Source** [useReducedMotion](https://motion.dev/docs/react-use-reduced-motion)

**Topic** Shows how a React component can respond to the device motion preference.

**Application** Keep the selected mood clear when the animated glow is reduced.

## Accessibility and visual QA

### 14. W3C | Dragging movements

**Platform** Web

**Source** [Dragging movements](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements)

**Topic** Explains how to provide an alternative to dragging.

**Application** For the web slider, test whether tapping a stop selects that mood.

### 15. W3C | Non-text contrast

**Platform** Web standard with principles applicable to native interfaces

**Source** [Non-text contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast)

**Topic** Explains contrast for the visual cues people need to identify controls and states.

**Application** Check the silver thumb against both portions of the track.

### 16. Apple | Accessibility audits

**Platform** iOS

**Source** [Accessibility audits](https://developer.apple.com/documentation/accessibility/performing-accessibility-audits-for-your-app)

**Topic** Shows how to run accessibility checks on the native app.

**Application** Pair the audit with a real VoiceOver pass through the slider and Next button.

### 17. Playwright | Visual comparisons

**Platform** Web

**Source** [Visual comparisons](https://playwright.dev/docs/test-snapshots)

**Topic** Shows how to compare a rendered screen with an approved screenshot.

**Application** Capture each mood at one repeatable animation frame so changes in label position or size are easy to spot.

### 18. W3C | Text contrast

**Platform** Web accessibility standard

**Source** [Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)

**Topic** For ordinary text use 4.5:1. Qualifying large text uses 3:1.

**Application** Check local foreground and background pairs across gradient lettering. Record the lowest ratio and the state where it occurs. Use intended composited colours and avoid sampling antialiased glyph edges.

### 19. Apple | Reduce Motion

**Platform** SwiftUI

**Source** [accessibilityReduceMotion](https://developer.apple.com/documentation/swiftui/environmentvalues/accessibilityreducemotion)

**Topic** Reads the system motion preference.

**Application** With Reduce Motion enabled, drag and release the slider. Verify that the selected value remains clear with decorative movement reduced.

### 20. Apple | Adjustable controls

**Platform** SwiftUI

**Source** [Accessibility modifiers](https://developer.apple.com/documentation/SwiftUI/View-Accessibility)

**Topic** Exposes adjustable actions for custom controls.

**Application** For a custom stepped slider, use VoiceOver to increment and decrement through the same values as touch. Check the spoken label, selected value and both endpoints.

### 21. MDN | Slider semantics

**Platform** Web

**Source** [Slider role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Roles/slider_role)

**Topic** Describes native range inputs, accessible names and human readable values.

**Application** Start with a labelled range input where appropriate. For named stops, update aria-valuetext with the selection. Verify keyboard changes and spoken values.

### 22. Shillington | Five principles of graphic design

**Source** [The ABC of Design](https://www.shillingtoneducation.com/blog/the-abc-of-design-five-principles-of-graphic-design)

1. **Alignment.** Pick shared edges or a centre axis for related elements. Inspect the rendered labels and controls together.
2. **Balance.** Look at the whole composition at its intended size. A bright accent can outweigh a much larger quiet area. Adjust placement or emphasis if it draws attention away from the task.
3. **Contrast.** Check text and control boundaries against the actual surface beneath them, including the brightest part of a gradient.
4. **Hierarchy.** Name what someone should notice first and what they need next. Check whether position, size and weight produce that reading order at a glance.
5. **Repetition.** Give controls with related purposes consistent spacing, type and shape. Reserve distinctive styling for meaningful differences.

On a rest screen, the time remaining should be readable at a glance. On the Workout Director rest screen, +15 s and Skip rest share one button style. Check their rendered dimensions, fill, border, type and interaction states together.

### 23. Shillington | Finding assets and design tools

**Source** [48 Essential Free Resources for Graphic Designers](https://www.shillingtoneducation.com/blog/free-resources-for-graphic-designers)

Choose a resource for a specific gap in the brief. Start with the project's own assets and typefaces.

| When the design needs | Where to look | What to check |
| --- | --- | --- |
| Photography of people | Nappy or Pexels | Check whether the crop still conveys the activity at the final size. |
| An illustration or interface icon | unDraw for recolourable illustrations or Ionicons for controls | Match the palette, stroke weight and optical size of nearby artwork. |
| A new type pairing | Fontjoy for exploration or Google Fonts for specimens | Try the actual heading, labels and numerals at their intended sizes. |
| A supporting colour | Coolors | Start from the brand colour and test the candidate on the actual surface. |
| Video or presentation artwork | Mixkit for footage or Shotsnapp for device mockups | Check the final crop and keep device framing in presentation artwork. |

Check the original provider's current licence, attribution requirements and export limits before using a selected asset.
