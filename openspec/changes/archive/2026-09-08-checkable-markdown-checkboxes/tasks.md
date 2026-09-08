## 1. Safety guard first

- [x] 1.1 In `renderNote`, change the note body's `input` listener to ignore
      events that did not originate on the body itself (`if (e.target !== body) return;`)
      so a bubbling form-control event can never overwrite `note.text` with
      `body.innerText`. Do this before anything else — without it, later steps
      corrupt note text.
- [x] 1.2 Verify the guard changed nothing for normal typing: edit a note, type,
      click away, reload, and confirm the text saved.

## 2. Source-line helpers

- [x] 2.1 Add a helper near the Markdown section that scans a raw Markdown string
      and returns the positions of its task markers, in source order, skipping
      lines inside fenced code blocks (``` and ~~~) and allowing an optional
      blockquote prefix, indentation, and either bullet (`-`, `*`, `+`) or
      ordered (`1.`, `1)`) list markers, requiring whitespace or end of line
      after the `]`.
- [x] 2.2 Add a helper that, given a note's text, a task index, and a desired
      checked state, returns the text with only that one marker rewritten
      (`[ ]` ↔ `[x]`), and returns the text unchanged when the index has no
      matching marker.

## 3. Render checkboxes as real controls

- [x] 3.1 In `showRendered`, after setting `innerHTML`, walk
      `input[type="checkbox"]` in the body and remove the `disabled` attribute
      that `marked` emits, so the boxes are enabled and focusable.
- [x] 3.2 Update the `.note-body.markdown input[type="checkbox"]` rule: add
      `cursor: pointer` and an `accent-color` that suits the note ink in both
      themes; check the alignment against `li.task-list-item`'s negative margin.
      (Found a latent bug: `marked` emits a bare `<li>`, so that rule never
      matched and every task item rendered with a stray bullet. The DOM pass now
      tags the `<li>`, and the rule's negative margin — which flattened nested
      lists once it started applying — was replaced with padding on the list.)
- [x] 3.3 Add a touch-only rule sizing the checkbox to at least 18×18 CSS px and
      keeping it aligned with the item text.

## 4. Toggle on click / tap

- [x] 4.1 In `onNotePointerDown`, detect a press landing on
      `.note-body input[type="checkbox"]` (mirroring the existing `a[href]`
      detection) and record its index among the body's checkboxes on the `drag`
      object.
- [x] 4.2 In the body's `click` listener, cancel the default action for a
      checkbox target as it already does for links, so the browser's own toggle
      never fires and never fights the source rewrite.
- [x] 4.3 In `onPointerUp`, add a branch beside the `linkHref` branch — after the
      drag / pointercancel / scrolled checks, before the `enterEdit` fallback —
      that rewrites the note's text via the 2.2 helper, sets `input.checked` on
      the clicked element directly (no re-render, so scroll position is kept),
      and saves.
- [x] 4.4 Confirm the note does not enter edit mode on a checkbox tap: no caret
      on desktop, and no keyboard or note control bar on touch.

## 5. Keyboard path

- [x] 5.1 Add a `change` listener on the note body that handles checkbox toggles
      made with the keyboard: read the new `checked` value and rewrite the source
      marker to match, then save.
- [x] 5.2 Verify mouse and keyboard don't double-toggle — cancelling the click in
      4.2 should also suppress its `change` event.

## 6. Verify in the browser

- [x] 6.1 Desktop width: check and uncheck items in a mixed note (headings,
      links, nested task items, a plain list); confirm only the clicked marker
      changes by opening the note for editing afterwards.
- [x] 6.2 Confirm a toggle survives a reload, and survives an export → import
      round trip.
- [x] 6.3 Confirm a press starting on a checkbox and dragging moves the note and
      leaves the box unchanged.
- [x] 6.4 Phone width / touch: tap targets are hittable, a vertical swipe on a
      long note scrolls without toggling, and tapping a checkbox never opens the
      keyboard.
- [x] 6.5 Confirm the scroll position of a long note is preserved across a toggle.
- [x] 6.6 Confirm a note with `- [ ]` lines inside a fenced code block toggles the
      right item and leaves the fenced lines alone.
- [x] 6.7 Offline check: load the page with the CDN blocked and confirm notes fall
      back to plain text, with clicking still opening edit mode as before.

## 7. Documentation

- [x] 7.1 Update the README's Markdown section to say task-list checkboxes can be
      ticked directly on the note.
- [x] 7.2 Update the in-app help entry for Markdown (`#hMarkdown`) with the same.

## 8. Starter board

- [x] 8.1 Add a checklist section to the seeded "Try Markdown" note so a first-run
      board shows off tickable boxes, using the same `${tap}` wording variable the
      other starter notes use ("Click" on desktop, "Tap" on a phone).
- [x] 8.2 Place it directly under the title rather than appending it: that note
      already overflows its 360px max height, so anything added at the end would
      start out below the fold and never be seen.
- [x] 8.3 Verify on a fresh board (cleared storage) that the starter note renders
      two boxes, both visible without scrolling, and that tapping one ticks it.
