# Markdown Task Lists

## Purpose

Notes render GitHub-style Markdown task lists, and their checkboxes are live: you
tick them directly on the note rather than editing the text. A note's raw
Markdown stays the single source of truth for what is checked, so ticking a box
edits one character of that text and nothing else.

Like all Markdown rendering in this app, this depends on the optional CDN
libraries. Offline, notes fall back to plain text and there are no checkboxes.

## Requirements

### Requirement: Rendered task lists show interactive checkboxes

When Markdown rendering is available, a note whose text contains GitHub-style
task list items (a list marker followed by `[ ]` or `[x]`) SHALL render each
item with a checkbox that is an enabled, interactive control: it MUST NOT be
rendered `disabled`, it MUST show a pointer cursor on hover, and its checked
state MUST reflect the `[x]` / `[ ]` marker in the note's stored text.

#### Scenario: Unchecked and checked items render with matching state
- **WHEN** a note's text contains `- [ ] buy milk` and `- [x] buy bread`
- **THEN** the rendered note shows two checkboxes, the first unchecked and the
  second checked, each followed by its item text

#### Scenario: Checkbox is presented as clickable
- **WHEN** the pointer hovers a rendered checkbox
- **THEN** the cursor is a pointer and the checkbox is not greyed out as a
  disabled control

#### Scenario: Task items render without a list bullet
- **WHEN** a note contains a list whose every item is a task
- **THEN** no list bullet is drawn beside any checkbox, and the boxes line up
  down the left edge where the bullets would have been

#### Scenario: Nested task lists keep their indent
- **WHEN** a task item has task items nested beneath it
- **THEN** the nested items are indented relative to their parent so the nesting
  is still visible

#### Scenario: A list mixing tasks and plain items keeps its normal layout
- **WHEN** a list contains both task items and plain bullet items
- **THEN** the plain items keep their bullets and the list keeps its usual indent

#### Scenario: Touch devices get a large enough tap target
- **WHEN** the app is running on a touch device
- **THEN** each rendered checkbox is at least 18 by 18 CSS pixels so it can be
  tapped reliably with a finger

### Requirement: Tapping a checkbox toggles it without entering edit mode

A click or tap that begins and ends on a rendered checkbox, without the note
being dragged, SHALL toggle that item's checked state and SHALL NOT put the
note into edit mode. On a touch device it SHALL NOT open the on-screen
keyboard or the per-note control bar.

#### Scenario: Clicking an unchecked box checks it
- **WHEN** the user clicks the checkbox of `- [ ] buy milk` on a note in
  display mode
- **THEN** the checkbox becomes checked, the note stays in display mode, and no
  caret appears in the note

#### Scenario: Clicking a checked box unchecks it
- **WHEN** the user clicks the checkbox of `- [x] buy milk`
- **THEN** the checkbox becomes unchecked and the note stays in display mode

#### Scenario: Tapping a checkbox on a phone does not open the keyboard
- **WHEN** the user taps a checkbox on a touch device
- **THEN** the item toggles, the note does not gain focus, and the on-screen
  keyboard and note control bar stay closed

#### Scenario: Clicking elsewhere in the note still edits
- **WHEN** the user clicks the note's text, away from any checkbox
- **THEN** the note enters edit mode as before, showing its raw Markdown with
  the caret where the user clicked

### Requirement: A toggle rewrites the note's stored Markdown source

Toggling a checkbox SHALL rewrite exactly the one task marker it corresponds to
in the note's stored Markdown text — `[ ]` becomes `[x]` and `[x]` or `[X]`
becomes `[ ]` — and SHALL persist the note. Every other character of the note's
text, including indentation, list markers, surrounding blank lines and the text
of other items, MUST be left byte-for-byte unchanged. The stored Markdown
remains the single source of truth for checked state.

#### Scenario: Only the clicked item changes
- **WHEN** a note contains `- [ ] one`, `- [ ] two`, `- [ ] three` and the user
  clicks the second checkbox
- **THEN** the note's text becomes `- [ ] one`, `- [x] two`, `- [ ] three`

#### Scenario: The toggle survives a reload
- **WHEN** the user checks an item and then reloads the page
- **THEN** the note renders with that item still checked

#### Scenario: Editing the note shows the rewritten source
- **WHEN** the user checks an item and then clicks into the note to edit it
- **THEN** the raw Markdown shown for editing contains `[x]` for that item

#### Scenario: Toggling does not replace the note text with rendered text
- **WHEN** a checkbox is toggled on a note containing headings, links or other
  Markdown
- **THEN** the note's stored text is still the original Markdown source, not the
  plain text of the rendered output, and the note still renders as Markdown

#### Scenario: The note's scroll position is preserved
- **WHEN** the user scrolls down a long note and toggles a checkbox
- **THEN** the note stays scrolled where it was rather than jumping to the top

### Requirement: The clicked checkbox maps to the correct source line

The system SHALL identify the source line to rewrite by the clicked checkbox's
position in document order among the note's rendered checkboxes, matched against
task list markers found in the same order in the note's raw text. Task markers
that appear inside fenced code blocks MUST be skipped, because they render as
literal text rather than checkboxes.

Before rewriting, the system SHALL verify that the number of markers found in
the source matches the number of checkboxes actually rendered. If the two
disagree, or if no source marker can be matched to the clicked checkbox, the
system SHALL leave the note's text unchanged, repaint the note from its source,
and tell the user the box can be changed by editing the note — rather than risk
rewriting the wrong line.

#### Scenario: Nested task items map correctly
- **WHEN** a note contains a task item with an indented task item beneath it and
  the user clicks the nested item's checkbox
- **THEN** only the nested item's marker is rewritten

#### Scenario: Task syntax inside a code fence is ignored
- **WHEN** a note contains a fenced code block whose lines include `- [ ] demo`,
  followed by a real task item, and the user clicks the one rendered checkbox
- **THEN** the real task item is rewritten and the line inside the code fence is
  left unchanged

#### Scenario: Unmatched checkbox makes no change
- **WHEN** the clicked checkbox has no corresponding marker in the raw text
- **THEN** the note's text and rendered output are left unchanged

#### Scenario: A note whose markers and boxes disagree is left alone
- **WHEN** a note contains task syntax that renders as literal text rather than a
  checkbox — a line indented far enough to become a code block inside a list
  item, say — so that the source has more markers than the note has boxes
- **THEN** clicking any checkbox on that note changes nothing, the note is
  repainted from its source, and the user is told to edit the note instead

### Requirement: Dragging from a checkbox still moves the note

A press that starts on a checkbox and then moves past the drag threshold SHALL
move the note as any other press does, and SHALL NOT toggle the checkbox when
the press is released. A vertical swipe that scrolls a long note's text on touch
SHALL likewise not toggle the checkbox.

#### Scenario: Drag started on a checkbox moves the note
- **WHEN** the user presses a checkbox and drags the note across the board
- **THEN** the note moves and tilts as usual and the checkbox's state is unchanged

#### Scenario: Swiping a long note to scroll does not toggle
- **WHEN** on touch the user starts a vertical swipe on a checkbox in an
  overflowing note
- **THEN** the note's text scrolls and the checkbox's state is unchanged

### Requirement: Checkboxes are keyboard operable

A rendered checkbox SHALL be reachable with the keyboard and toggling it with
the keyboard SHALL rewrite the note's Markdown source the same way a click does.

#### Scenario: Space toggles a focused checkbox
- **WHEN** the user tabs to a rendered checkbox and presses Space
- **THEN** the item toggles and the note's stored Markdown is rewritten to match

### Requirement: Offline behaviour is unchanged

When the Markdown libraries are unavailable, notes SHALL continue to display raw
plain text with no checkboxes, and no checkbox behaviour SHALL be introduced.

#### Scenario: Offline note shows plain text
- **WHEN** the page runs with no network and a note contains `- [ ] buy milk`
- **THEN** the note displays that line as literal text with no checkbox, and
  clicking it enters edit mode as before
