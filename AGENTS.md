# Agent Guidance

## Repository role

This repository owns `Doria\Tui`, the terminal user interface library written
in Doria. The `doria` compiler repository owns the language, the standard
library and `Doria\Std\Term`; this library consumes them and never restates or
extends them. The master plan (`doria/docs/doria-end-to-end-plan.md`, sections
2.1 and 13) owns this library's place in the roadmap and its acceptance
checkpoint.

Read `/Users/andrewmasiye/.claude/AGENTS.md` before working here. It governs
publishing, branches, cleanup and resource use for every repository.

## Design authority

- Accepted package decisions live in `docs/decisions/`. An API illustration
  that no accepted decision specifies is marked as a proposal.
- The application model is model, update, view (Andrew, 2026-11-15): the
  application owns one state value, `update` consumes a key or resize event and
  changes it, `view` builds a fresh element tree, and the library lays it out
  and writes only the cells that changed. Widgets hold no callbacks and no
  references to their parents.
- Text width comes from `$text->displayWidth`. Do not keep width tables here.
- Raw mode, the alternate screen and their restoring guards belong to
  `Doria\Std\Term`. Frame buffers, layout, focus and diffing belong here.
- A missing language or standard-library facility is reported to the compiler
  owner as a dependency. Do not work around it in this library.

## Engineering

- Follow Doria's naming charter and ownership model as `doria/AGENTS.md`
  states them.
- No source file contains an escape sequence or a platform conditional.
- Every behavior lands with a headless test that compares cell snapshots, and
  every terminal-facing behavior with a pty or ConPTY check on Linux, macOS
  and Windows.
