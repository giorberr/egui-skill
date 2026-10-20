---
name: egui
description: Develop, refactor, and review egui/eframe interfaces in Rust applications, including layout, custom widgets, accessibility, textures, repainting, and window lifecycle. Use for GUI work, not unrelated backend development.
---

# egui

Build interfaces that fit the application's existing architecture and visual
language. Keep framework facts, observed project conventions, and design
recommendations distinct.

## Start with the application

- Inspect `Cargo.toml`, `Cargo.lock`, the UI entry point, and relevant shared
  widgets, theme tokens, state, and background workers. Locate existing helpers
  before creating alternatives; confirm their signatures and behavior.
- Match APIs to the locked version. The reference baseline is **0.36.2**, not a
  requirement to upgrade; **0.37 has API differences**. The project's lockfile
  and actual dependency source are authoritative, not the skill's baseline.
  Preserve the configured renderer and enabled features unless the requested
  change needs something different.
- Keep blocking I/O and sustained computation off rendering/lifecycle callbacks.
  Background operations that must continue while hidden should not depend on
  visible frames. Preserve the project's storage and shutdown conventions.

Load only the reference relevant to the task:

- [API guide](api-guide.md): widgets, layout, input, accessibility, textures,
  repaint scheduling, dialogs, and version-sensitive APIs.
- [Design guide](design-guide.md): hierarchy, spacing, control states, motion,
  and screenshot review.

## Validate the change

Run relevant existing checks from the application checkout; use
`cargo check --locked` for compilation when applicable. After an egui/eframe
upgrade, also run existing interaction tests and Clippy with warnings denied
(`cargo clippy --locked -- -D warnings`), covering relevant feature configurations.
Exercise affected input and lifecycle behavior, including text/IME and
programmatic focus when changing widget IDs. For visual changes, inspect actual
screenshots at relevant sizes/scales and include long text, errors, and
disabled/focused states. Distinguish compilation, headless interaction tests,
platform stubs, visual checks, and real target execution in the report;
compilation and headless tests do not establish visual correctness. Never claim
a check that was not performed. Add UI testing or profiling dependencies only
when the task justifies them.
