# egui skill

A compact agent skill for building and reviewing Rust interfaces with egui and
eframe. It covers API selection, layout, custom widgets, accessibility, textures,
repainting, window lifecycle, and visual design.

- [SKILL.md](skills/egui/SKILL.md): entry point and workflow.
- [API guide](skills/egui/api-guide.md): task-oriented API reference and version pitfalls.
- [Design guide](skills/egui/design-guide.md): layout, interaction, motion, and visual review.

The skill lives in `skills/egui/` and is named `egui`. Its API reference was
checked against version **0.36.2**; always verify the target application's locked
version before using an API. Design defaults are recommendations, not a required
theme or architecture.
