# egui design guide

## Observe before prescribing

Inspect the application's existing screens, theme, typography, spacing, widget
helpers, and layout constraints. Record which conventions are actually present;
keep those observations separate from recommendations. Source inspection alone
does not establish that a screen looks correct.

Reuse established helpers and shared tokens when adding comparable UI. Custom
painting, color constants, and explicit font sizes can be legitimate design
choices. Avoid introducing a second styling system, theme, navigation structure,
or layout rewrite without a reason grounded in the requested change.

## Recommended visual defaults

These are starting points when the project has no established equivalent.
Adapt them to the product, input method, and display.

- Use a clear title/section/body/supporting-text hierarchy. Choose a small type
  scale and a consistent spacing rhythm; avoid isolated size adjustments that
  make nearby controls drift out of alignment.
- Keep accent color restrained, with one obvious primary action per action
  group. Distinguish secondary and destructive actions through wording and
  treatment, not hue alone.
- Use consistent surface, border, radius, and disabled-state treatments. Keep
  contrast readable in every supported theme and control state.
- Prefer readable labels and stable geometry. Reserve space for changing values
  when layout jumps would distract. Align related fields and table columns;
  use scrolling or intentional text fitting rather than accidental clipping.
- Measure custom text in the same font family, size, and weight used to draw it.
  Check fallback fonts, non-ASCII text, and platform differences when relevant.
- Treat requested initial window sizes and explicit minima as configuration,
  not proof that content fits. Check actual available space and scaling; avoid
  assuming logical points equal physical pixels.

## Interaction and feedback

Use clear action labels and immediate feedback. Show loading, empty, success,
and error states deliberately. Keep controls stable while work is pending;
prevent duplicate submissions when repeated activation would be incorrect.
Explain why an action is unavailable and make recovery steps easy to find.

Combine color with text or icons for meaningful status. Important explanations
must be reachable without hover alone. Truncated content needs a usable way to
inspect the full value when it matters.

Provide visible keyboard focus, predictable Tab/Shift+Tab order, accessible
names, and correct selected/disabled states. Include labels in small controls'
hit areas where appropriate. A painted control still needs interaction and
accessibility semantics; see the [API guide](api-guide.md).

Dialogs should make the main action, cancellation, and consequences clear.
Choose initial focus deliberately, preserve pending-work guards, and verify
Escape/backdrop behavior. Closing a child window should have the intended scope.

## Motion recommendations

- Start optional hover/focus/surface transitions around **100–180 ms**, using
  subtle color or opacity changes. Immediate changes are often sufficient.
- Dispatch actions and update errors, status, and enabled state immediately;
  animation must not delay operation or hide essential feedback.
- Avoid decorative loops, unnecessary pulses, and animated movement of updating
  list rows. Keep functional live-image refresh separate from decorative motion.
- Request animation repaints only while a transition is active. In the 0.36.2
  baseline, `ctx.animate_bool_with_time(id, target, seconds)` requests repaint
  while its value is between endpoints. Do not add a permanent repaint loop.
- With animation disabled, snap to the final state and retain all text, status,
  and interaction cues. Treat reduced motion as an explicit behavior to design
  and verify, not a setting that every application already provides.

## Visual and interaction review

Inspect actual rendered screenshots at representative window sizes and display
scales, including explicit minima and restored/maximized states when relevant.
Check long text, empty/full lists, errors, pending work, disabled controls,
keyboard focus, dialogs, and each supported theme. Look for clipping, overlap,
poor contrast, misalignment, unreachable actions, and distorted images.

Exercise keyboard and pointer behavior as well as appearance. If lifecycle or
background work changed, test hiding/minimizing, closing child windows, and
shutdown separately: a screenshot cannot establish those behaviors. Platform
fonts, window managers, and rendering backends may differ.

Report what was compiled, exercised, and visually inspected, together with any
material untested cases. Never claim a screenshot review without viewing the
screen, or add snapshot/profiling infrastructure solely because this guide
mentions validation.
