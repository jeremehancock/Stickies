## Why

Notes already render GitHub-style Markdown task lists (`- [ ] buy milk`), so a
checkbox appears — but it is inert. Tapping it does nothing except drop the note
into edit mode, where the checkbox disappears and you are left hand-editing
`[ ]` into `[x]`. A checkbox that looks clickable but isn't is worse than no
checkbox at all, and a sticky-note board is exactly where people keep short
to-do lists.

## What Changes

- Clicking or tapping a rendered checkbox toggles that item between unchecked
  and checked, and the note stays in display mode — it does not enter edit mode.
- The toggle is written back into the note's stored Markdown source (`- [ ]`
  becomes `- [x]` and back), so it survives reload, export/import, and shows up
  as the correct raw text when you do edit the note.
- Checkboxes become genuinely interactive controls: they are no longer rendered
  `disabled`, they get a pointer cursor, and on touch they get a tap target
  large enough to hit reliably.
- Everything else about a press on a note is unchanged. Dragging a note by
  starting the drag on a checkbox still moves the note and does not toggle it;
  clicking anywhere else in the note still opens edit mode; rendered links still
  open in a new tab.
- Offline behaviour is unchanged. With no Markdown libraries there are no
  rendered checkboxes, and notes stay plain text.

## Capabilities

### New Capabilities
- `markdown-task-lists`: rendering Markdown task lists inside a note and
  toggling their checkboxes directly from the rendered view, with the note's
  stored Markdown source as the single source of truth.

### Modified Capabilities
<!-- None. openspec/specs/ contains no existing capability specs yet, so the
     behaviour this change touches is captured entirely by the new capability
     above. -->

## Impact

- **Code**: `index.html` only — the Markdown rendering helpers
  (`renderMarkdownHtml` / `showRendered`), the note pointer-down/pointer-up
  drag handling that already special-cases rendered links, and the
  `.note-body.markdown` checkbox styles.
- **Data**: no storage format change. A note's `text` field is still raw
  Markdown; toggling a box just rewrites one character in it. Existing boards
  and exported JSON files stay compatible in both directions.
- **Dependencies**: none added. Still `marked` + `DOMPurify` from CDN, pinned,
  optional, with the plain-text fallback intact.
- **Docs**: the README's Markdown section and the in-app help entry for
  Markdown should mention that checkboxes are tappable.
