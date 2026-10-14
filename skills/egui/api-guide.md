# egui API guide

Baseline: **egui/eframe/epaint 0.36.2**, checked against installed crate sources on
2026-10-14. This verifies API declarations and documented behavior, not compiled
examples or a rendered application. Recheck the target application's lockfile;
prefer its existing wrappers and helpers when they already solve the task.

## Choose an API

| Task | Starting point in 0.36.2 |
| --- | --- |
| Ordinary controls | `Button`, `Checkbox`, `TextEdit::singleline` / `multiline`, and `ComboBox::from_id_salt(...).show_ui(...)`; retain built-in interaction when custom painting is unnecessary. |
| Local layout | `ui.horizontal`, `ui.vertical`, and `ui.horizontal_wrapped`; use `Grid` for aligned form rows and `ScrollArea` for overflow. |
| Large uniform lists | `ScrollArea::show_rows` renders a visible row range; keep row height consistent and identities independent of visible indices. |
| Main regions | `Panel::left/right/top/bottom` followed by `CentralPanel`; each `.show(...)` takes `&mut Ui`. |
| Custom geometry | `ui.allocate_exact_size(size, sense)` for layout-managed allocation, or `ui.interact(rect, id, sense)` for an existing rectangle; paint through `ui.painter()`. |
| Bounded child UI | `ui.scope_builder(UiBuilder::new().max_rect(rect), ...)`; set clipping deliberately where needed. |
| Text | `Label` / `RichText` for normal widgets; `Painter` and font layout for custom rendering. Measure with the same family, size, weight, and wrapping used to paint. |
| Floating surfaces | `Window`, `Area`, and `Modal`; their `.show(...)` takes `&Context`. |

Discover the target project's button, typography, form, and layout helpers before
adding replacements. Shared color/font constants and custom painting can be
intentional; a framework abstraction is not automatically an improvement.

## Lifecycle and context

- `App::ui(&mut self, ui: &mut egui::Ui, frame: &mut eframe::Frame)` draws the
  root viewport when it needs repainting. Access the context through `ui.ctx()`.
  The root UI has no margin/background by itself; use a panel or frame as needed.
- `App::logic(&mut self, ctx: &egui::Context, frame: &mut eframe::Frame)` runs
  before `ui`, and also while hidden when repaint is requested. It must not show
  UI or paint. Hidden-window input events/time can reflect the last visible
  frame, so this is not a fresh-input stream or independent periodic worker.
- Use lifecycle callbacks to consume completed work and coordinate state, not
  to run blocking network, file, or compute jobs. Jobs requiring continuous
  progress belong in background workers with explicit completion/wakeup paths.
- `ui.max_rect()` describes the local UI region. `ctx.content_rect()` describes
  safe viewport content; `ctx.viewport_rect()` includes the full viewport area.
- eframe calls `App::save` only when its `persistence` feature is enabled. Reuse
  existing settings/database storage; adopt eframe persistence only as an
  intentional project choice.

Contract: [eframe App 0.36.2](https://docs.rs/eframe/0.36.2/eframe/trait.App.html).

## Stable identity, input, and focus

Use persistent entity keys for repeated widgets, not list position or changing
labels. `ui.push_id(...)` scopes auto IDs and `ui.make_persistent_id(...)`; it
does not scope an explicit `Id::new(...)` supplied to `ui.interact`. Include the
entity and control role in an explicit composite ID when needed.

`Sense::click()` is focusable. `response.clicked()` includes pointer activation,
accessibility activation, and Space/Enter while focused. `clicked_by(...)` omits
keyboard/accessibility activation. Do not add duplicate key handlers for an
ordinary button. For value-changing custom controls, call `mark_changed()` on
the response after mutation.

Use `has_focus()` to paint focus, and `request_focus()` on deliberate entry or
navigation events rather than every frame. Verify Tab/Shift+Tab order. Read
input using `ui.input(...)` / `ctx.input(...)`; consume handled shortcuts with
`input_mut` and `consume_key` / `consume_shortcut`, respecting text editing and
the top dialog. Keep context-lock closures short and avoid nested context calls.

## Accessible custom controls

Painting a label does not create an accessible name or relationship. Prefer
built-in widgets where practical; for painted controls:

- Register button semantics with
  `response.widget_info(|| egui::WidgetInfo::labeled(egui::WidgetType::Button, enabled, label))`.
  Report the effective enabled state. `WidgetInfo::selected` carries checked or
  selected state for controls such as checkboxes. Register metadata after
  handling changes and `mark_changed()`.
- Associate a real label response with an input using
  `response.labelled_by(label.id)`. A painter-only label has no widget response
  to associate; extend the shared helper or provide accessible metadata.
- Keep disabled visuals, hit testing, and semantics consistent.
  `ui.add_enabled(...)` / `ui.add_enabled_ui(...)` disable interaction;
  switching only to `Sense::hover()` does not by itself report disabled metadata.
- Supply visible keyboard focus and meaningful names for icon-only actions.
  Test activation by keyboard as well as pointer.

## Textures and repaint scheduling

Create pixel data with `ColorImage::from_rgb([width, height], bytes)` or
`from_rgba_unmultiplied(...)` for the matching byte format. Validate dimensions,
byte count, and channel ordering before constructing the image.

`ctx.load_texture(name, image, TextureOptions::LINEAR)` returns a
`TextureHandle`; retain it and update with `texture.set(image, options)` rather
than creating a new texture every frame. Keep upload/presentation work in the UI
path. For live images, a latest-frame handoff can prevent an obsolete-frame
backlog; preserve ordering for events that cannot be dropped. Fit images without
distorting their aspect ratio unless distortion is intentional.

`request_repaint()` can wake eframe from a worker.
`request_repaint_after(Duration)` schedules a later wake; the shortest outstanding
delay wins. A repaint request does not guarantee a visible frame. Schedule
updates for actual data, pending work, or active transitions; avoid an
unconditional repaint loop for an otherwise idle interface. Keep background
job cadence independent of rendering.

## Dialogs, viewports, and closing

- `Window::open(&mut open)` binds visibility and supplies a close button.
  Keep visibility state in the application; avoid reinitializing it every frame.
- `Area` with foreground order and a painted backdrop is not by itself a
  guarantee of modal keyboard behavior. `Modal::new(id).show(ctx, ...)` provides
  modal handling. Its `should_close()` covers backdrop clicks, `ui.close()`, and
  eligible top-modal Escape. Apply the application's dismissal policy explicitly;
  cancellation must not trigger the destructive action it was meant to cancel.
- `show_viewport_immediate` takes a stable viewport ID and a callback receiving
  `(&mut Ui, ViewportClass)`. Call it while that viewport should exist. In the
  `EmbeddedWindow` case, egui has already created a window: draw contents without
  nesting another. If an explicit embedded close button is needed, inspect the
  `embed_viewports()` fallback and use a `Window::open` binding.
- Read native close intent with
  `ctx.input(|i| i.viewport().close_requested())`. `ViewportCommand::Close` /
  `CancelClose` target the current viewport; `send_viewport_cmd_to` selects an
  explicit viewport. Preserve pending-save guards and shutdown behavior even
  when hidden. Hiding a window and terminating the application are distinct.

## Common version mistakes

These are checked replacements for the 0.36.2 baseline, not a migration history
or a reason to upgrade another application.

| Older example | 0.36.2 form |
| --- | --- |
| `App::update(ctx, frame)` | `App::ui(ui, frame)`; non-render lifecycle work in `logic(ctx, frame)` |
| `SidePanel` / `TopBottomPanel` | `Panel::left/right/top/bottom(...).show(ui, ...)` |
| `CentralPanel::show(ctx, ...)` | `.show(ui, ...)`; windows/areas still take a context |
| `Frame::none()` / `TextEdit::frame(false)` | `Frame::NONE` / `TextEdit::frame(Frame::NONE)` |
| `Rounding` / `.rounding(...)` | `CornerRadius` / `.corner_radius(...)` |
| `ctx.screen_rect()` | `content_rect()` for safe content; `viewport_rect()` for the full viewport |
| Viewport callback receiving a context | Callback receives a UI; obtain its context with `ui.ctx()` |

## Source anchors

Find the locked crates under `$CARGO_HOME/registry/src/*/`, normally
`~/.cargo/registry/src/*/`. Relevant sources:

- `eframe/src/epi.rs`: application callbacks and persistence.
- `egui/src/{ui.rs,context.rs,response.rs,sense.rs,data/output.rs}`: layout,
  identity, input, accessibility, repaints, and animation.
- `egui/src/containers/{panel.rs,window.rs,area.rs,modal.rs,scroll_area.rs,frame.rs}`
  and `egui/src/widgets/text_edit/builder.rs`: containers and versioned signatures.
- `epaint/src/{image.rs,texture_handle.rs}`: pixel images and texture updates.

Background reference: [egui-agent-skills](https://github.com/gregorycarnegie/egui-agent-skills)
and its [license notices](https://github.com/gregorycarnegie/egui-agent-skills/blob/main/LICENSE).
This guide is independently written from crate sources; no upstream skill text
or snippets were copied. Preserve applicable notices if adapting upstream text.
