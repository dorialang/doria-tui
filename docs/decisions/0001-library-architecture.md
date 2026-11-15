# Decision 0001: Library Architecture

- **Status:** Accepted
- **Date:** 2026-11-15
- **Owners:** Doria TUI library design
- **Scope:** How `Doria\Tui` is layered, where state lives, how events reach an application, what it draws on, and what it does without a terminal

## Context

The Doria master plan (`doria/docs/doria-end-to-end-plan.md`, sections 2.1
and 13) schedules a Doria-authored terminal user interface library after
Stage 46, which delivered `Doria\Std\Term`. Its checkpoint is a searchable
list with a detail panel, keyboard focus, resize handling and terminal
restoration on structured exits, from one source on Linux, macOS and Windows.
The plan's package design notes (`doria/docs/application-package-design.md`)
list the questions to settle before a public API: component identity, state
and callback ownership, Unicode width, event priority and backpressure, text
editing, fallback behavior, and terminal-session exclusivity.

Andrew answered them on cards on 2026-11-15. His words are quoted below.

## Decision

### Public layers

"Layered, state as owned values (Recommended)". The library is four layers.
Each is public and usable without the layers above it.

1. **Terminal session and cell surface.** Owns raw mode and, when used, the
   alternate screen through `Doria\Std\Term` for as long as a run lasts, and
   restores both when the run returns or throws. Holds a frame of cells and
   writes only the runs of cells that changed, through `Console`.
2. **Layout and drawing.** The `Element` interface, constraints and regions.
   An element measures itself and draws into a region of cells.
3. **Input.** An event loop over `RawMode::getNextEvent` with deadlines, and
   focus routing. A widget state that takes keys implements one public
   key-handling interface.
4. **Application models.** Model, update, view is the first application model
   the library ships. Another model, such as a retained widget tree, an
   immediate-mode loop or a game frame loop, is written against layers 1 to 3
   with no change to the library.

Built-in widgets (text, container, list, input field, status bar) use only
these public interfaces. A widget written outside the package can do anything
a built-in one can. Doria's `internal` restricts a member to its declaring
class, so one built-in type cannot reach another's internals; the language
enforces this rule.

### Model, update, view

"Model, update, view (Recommended)". The application owns one state value.
Each event goes to `update()`, which changes that state. `view()` builds a
fresh element tree from it, and the library lays the tree out and writes the
cells that changed.

"update() receives the event (Recommended)". `update()` receives each
`TerminalEvent` as `Doria\Std\Term` delivers it. The application passes a key
to the state of the focused widget through its key-handling method and reads
the result. Elements carry no closures and send no messages.

### Widget state is an owned value

A widget's own state, such as a list's selected row and scroll offset or an
input's text and cursor, is an ordinary typed value (`ListState`,
`InputState`, or a third party's own state class). Whatever uses the widget
owns it: the model in model, update, view; the widget object in a retained
tree; the loop's own store in immediate mode. `view()` passes each state to
its element.

- Dropping a state value releases what it holds, through ordinary Doria
  destruction. The library has no reconciler that decides when a removed
  widget's state ends.
- Every state is type-checked. The library never stores state of a type it
  does not know.
- An application model that wants state kept by key can be built in layer 4
  on these values.

### Layout

"Size rules per child (Recommended)". A row or column (`Stack`) gives each
child one sizing rule along its axis: an exact number of cells, its measured
size, or a fill with a weight, each optionally held between a minimum and a
maximum. Exact sizes are given first, then measured sizes, in order, each no
larger than what is left; fills then share the rest by weight, and the cells
an uneven share leaves go to the first fills. Across the axis each child gets
the stack's whole extent. A child that gets no space is not drawn.

### Text width

Every measurement of text uses `$text->displayWidth` (compiler Decision
0146). The library keeps no width tables.

### Text input

"One line, essential keys (Recommended)". The first input field holds one
line. Typing inserts at the cursor; Left, Right, Home, End, Backspace and
Delete move and erase by grapheme cluster; text longer than the field scrolls
sideways. Word movement, selection and undo can be added later without
changing this API.

### Screen

"Alternate screen, inline on request (Recommended)". A run takes the
alternate screen by default, so the shell's text and scrollback return when
it ends. An application can ask to draw inline instead, in a fixed number of
rows below the prompt that stay on screen after it ends.

### Without a terminal

"Typed error, plain text on request (Recommended)". Starting a run when
standard input or standard output is not a terminal throws a `TuiError` that
names the stream. Any view can be rendered once as plain text lines at a
given width, so a program can print a static report for automation or a
screen reader when it chooses to.

## Decided Without A Card

- **Events are handled in batches.** The loop handles every event already
  waiting, then draws once. A burst of keys, such as a paste, does not draw a
  frame per key. `Doria\Std\Term` already merges resizes between two waits
  into one event, so the queue the library reads stays bounded.
- **One session at a time.** `Doria\Std\Term` refuses a second raw mode while
  one is alive. The library starts no second session; a dialog is part of the
  same view.

## Acceptance

- The plan's checkpoint program: a searchable list with a detail panel, an
  input field, a status bar, keyboard focus, scrolling, resize handling and
  terminal restoration, from one source with no platform conditionals.
- Headless cell-snapshot tests for layout, Unicode width, clipping, focus,
  scrolling and resize.
- Pty runs on Linux and macOS and ConPTY runs on Windows that hold every
  screen of the checkpoint program to the headless snapshots.
- A second application model and a third-party widget, both built from
  outside the package, that use only the public layers.
- Performance checks for bounded writes, and no allocation for an unchanged
  frame.

## Explicit Non-Goals

- A widget layer in the standard library. `Doria\Std\Term` stays primitive.
- Mouse input, which `Doria\Std\Term` does not deliver yet.
- A multi-line editor and text wrapping, which need a line-breaking decision.
- Async background work, which arrives with the compiler's structured tasks.

## Consequences

- API names in examples are proposals until the code that defines them lands
  with its tests.
- A facility the library needs from the language or the standard library is
  reported to the compiler as a dependency and is never worked around here.
  Display width was the first (compiler Decision 0146).
