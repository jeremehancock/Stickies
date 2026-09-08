## Context

A note stores raw Markdown in `note.text`. In display mode the body element is
`contenteditable="false"` and holds HTML produced by `marked` and sanitized by
`DOMPurify` (`renderMarkdownHtml` → `showRendered`). Clicking a note runs
`enterEdit`, which swaps that HTML back to the raw source and turns editing on.

`marked` with `gfm: true` already renders `- [ ] x` as
`<li class="task-list-item"><input disabled type="checkbox"> x</li>`, and
`index.html` already styles both. The checkbox is inert for two reasons: `marked`
emits it `disabled`, and any press on the note body is claimed by the drag /
edit handling in `onNotePointerDown` / `onPointerUp`.

That handling already has the pattern we need. Rendered links are special-cased:
`onNotePointerDown` records `drag.linkHref` when the press lands on an `a[href]`,
and `onPointerUp` opens the link *instead of* calling `enterEdit` — but only if
the press did not become a drag or a scroll. Checkboxes should ride the same
rails.

Two constraints shape everything below:

- **The Markdown source is the source of truth.** Rendered HTML is disposable;
  it is regenerated from `note.text` on every render, and export/import moves
  `note.text` only. So a toggle must edit the source string, not the DOM state.
- **Markdown stays optional.** With no CDN libraries there is no rendered HTML
  and therefore no checkboxes, so all of this must sit behind the existing
  `markdownAvailable()` fallback and add nothing when it is false.

## Goals / Non-Goals

**Goals:**
- Clicking or tapping a rendered checkbox toggles that one item, in place, with
  the note staying in display mode.
- The toggle rewrites exactly one `[ ]` / `[x]` marker in `note.text` and leaves
  every other character alone.
- Dragging, scrolling, link-opening and click-to-edit keep working exactly as
  they do today.
- No new dependency, no build step, no storage format change.

**Non-Goals:**
- Undo/redo for a toggle. The existing undo stack is only for deleted notes, and
  a mis-tap is undone by tapping again.
- Toggling from inside edit mode. While editing you see raw text, which is the
  right place to hand-edit a marker.
- Any Markdown authoring affordance (a "insert checklist" button, auto-continuing
  a list on Enter). Out of scope.
- Syncing checked state anywhere. Everything stays in localStorage as before.

## Decisions

### 1. Map a clicked checkbox to a source line by ordinal position

To rewrite the right line we must know which task item was clicked. The chosen
approach: count the clicked checkbox's index among all `input[type="checkbox"]`
elements in the note body in document order, then walk the raw text and rewrite
the *n*-th task marker.

This works because `marked` emits checkboxes in the same order as the source
lines that produced them, including for nested lists — a parent item's checkbox
is emitted before the nested list rendered inside it, exactly as the parent line
precedes the nested lines in the source.

A source line counts as a task marker when it matches, after any blockquote
markers and indentation, a list bullet followed by `[ ]`, `[x]` or `[X]` and
then whitespace or end of line — roughly:

```
/^(\s*(?:>\s*)*(?:[-*+]|\d+[.)])\s+\[)([ xX])(\](?=\s|$))/
```

The blockquote prefix is included because `> - [ ] x` renders a real checkbox.
Lines inside fenced code blocks (``` or ~~~) are skipped while scanning, because
`- [ ] x` inside a fence renders as literal text, not a checkbox, and counting it
would shift every index after it.

Implementation confirmed this scan agrees with `marked` box-for-box across
ordinary notes (plain, nested, ordered, blockquoted, fenced, mixed with non-task
lists), with two rules found by checking against the real library rather than
assuming:

- `marked` requires a **literal space** after the `]` (`/^\[[ xX]\] /`), so a
  lone `- [ ]` with nothing after it renders as plain text and must not be
  counted. The regex requires that space too.
- One genuine divergence remains: a task line indented far enough to become an
  **indented code block** inside a list item (`- [ ] a`, blank line, six spaces,
  `- [ ] b`) renders as literal text but still reads as a marker to the scan.
  That is what decision 2a below exists for.

*Alternatives considered.* Re-serializing the rendered DOM back to Markdown:
rejected, it would rewrite the user's formatting wholesale. Walking `marked`'s
token stream to recover exact source offsets: more precise in theory, but it
means re-lexing on every click and reimplementing offset tracking, for an
accuracy gain that only shows up in Markdown a sticky note is unlikely to hold.
The ordinal scan is a dozen lines and fails safe (see risks).

### 2a. Verify the mapping before rewriting anything

Because that last divergence is real rather than theoretical, the toggle does not
trust the ordinal blindly. Before rewriting, it compares the number of markers
found in the source with the number of checkboxes actually rendered. If they
disagree, the note is repainted from its source and nothing is written, with a
short toast explaining that the box can be changed by editing the note.

This turns the worst case from *silently rewriting the wrong line* into *this one
unusual note does not support tap-to-tick*, which is a fair trade.

### 2. Toggle in place; do not re-render the note

The handler flips the source marker, writes `note.text`, calls `save()`, and sets
`input.checked` directly on the clicked element. It deliberately does **not** call
`showRendered`, which would rebuild `innerHTML`.

Rebuilding would reset `body.scrollTop` (a checked-off item halfway down a long
list would jump the note back to the top) and would throw away keyboard focus.
Flipping the one property keeps the DOM and the source in step at a fraction of
the work.

### 3. Suppress the native toggle and drive it from `onPointerUp`

The checkbox is rendered enabled so it looks and behaves like a control, but its
native click behaviour is cancelled in the existing body `click` listener,
alongside the `a[href]` case. The toggle is instead performed in `onPointerUp`,
in a new branch placed with the existing `linkHref` branch — after the "was it a
drag?" and "was it a scroll?" checks, before the `enterEdit` fallback.

The reason to route it through the drag machinery rather than just letting the
browser handle the click: **the note follows the pointer during a drag**, so a
checkbox pressed at the start of a drag is usually still under the pointer at the
end, and the browser would fire a legitimate click and toggle it. Deciding in
`onPointerUp` — where we already know whether the press became a drag, a native
scroll, or a plain tap — is the only place with enough information.

### 4. Guard the body's `input` listener against bubbling form events

This is the sharp edge in the change. The note body has:

```js
body.addEventListener("input", () => { note.text = getText(body); ... });
```

`getText` is `body.innerText` — the *rendered* text. A checkbox toggle fires an
`input` event that **bubbles**, so if it reaches this listener the note's
Markdown source is silently replaced by the plain text of its rendered output:
headings lose their `#`, links lose their URLs, and the note stops being
Markdown. That happens on the keyboard path (decision 5) and would happen on any
future path where a form control lives in the body.

The listener therefore gains an early `if (e.target !== body) return;` — it
should only ever respond to the user typing into the body itself. This guard is
correct on its own merits and should go in regardless of the rest.

### 5. Keyboard support via the `change` event

Because the checkbox is enabled it is tabbable, and Space toggles it natively.
Cancelling the click (decision 3) does not affect the keyboard, so a `change`
listener on the body handles that path: read the checkbox's new `checked` value,
rewrite the source line to match, save. Mouse and keyboard never both fire —
cancelling the click also cancels its `change`.

### 6. Styling

Remove `disabled` from checkboxes after sanitizing (walk
`body.querySelectorAll('input[type="checkbox"]')` in `showRendered`), rather than
trying to configure `marked`'s renderer — the DOM pass is independent of how the
library formats its output, so a pinned-version bump can't silently break it.

Add `cursor: pointer`, and an `accent-color` tied to the note's ink color so a
checked box reads correctly on every paper color instead of defaulting to browser
blue. On touch, size the box to at least 18×18 CSS px so it is a real tap target.

The alignment work turned out to be a latent bug rather than a tweak. The
stylesheet already had a `li.task-list-item` rule to drop the list bullet beside a
checkbox — but `marked` emits a bare `<li>` with no class at all, so that rule has
never matched and every task item has been rendering with a stray bullet next to
its box. The same DOM pass that enables the checkboxes now tags its `<li>`, which
makes the existing rule work.

That in turn exposed the rule's own `margin-left: -1.2em`, which cancels a nested
list's indent and flattens sub-items to their parent's level. Replaced with
padding on the list itself: an all-task list sits flush with the note's text so
the boxes line up down the left edge, while a nested one keeps a normal indent. A
list is only treated as a task list when *every* item is a task, so a list mixing
ticked items with plain bullets keeps its usual layout.

## Risks / Trade-offs

- **Ordinal mismatch rewrites the wrong line.** If the count of rendered
  checkboxes ever diverges from the count of scanned source markers (some exotic
  Markdown, or a future `marked` change), a click could toggle a neighbouring
  item. → Mitigation: the scan bails and changes nothing when the *n*-th marker
  does not exist, and the two constructs known to diverge (code fences,
  blockquotes) are handled explicitly. A mis-toggle is also visible and instantly
  reversible, not silent data loss.

- **The bubbling `input` event.** Described in decision 4; unguarded it corrupts
  note text. → Mitigation: the target guard, plus an explicit verification step
  that toggles a checkbox on a note with headings and links and then opens it for
  editing to confirm the source is intact.

- **An enabled checkbox is focusable, changing Tab order.** Tabbing through the
  board now stops on checkboxes. → Trade-off accepted: that is the correct
  behaviour for an interactive control, and it is what makes decision 5 work.

- **A tap that lands 2px off the box still opens edit mode.** → Mitigation: the
  larger touch target. Beyond that, opening the editor is a mild, obvious failure
  — the user taps away and tries again.

- **`accent-color` is unsupported on older browsers.** → It degrades to the
  default checkbox rendering, which is functional; nothing depends on it.

## Migration Plan

None needed. No storage format change: a board saved before this change renders
and toggles correctly after it, and a board with checked items exports to JSON
that older builds still read as plain Markdown text. Rollback is reverting the
commit.

## Open Questions

- Should a checked item's text be visually de-emphasised (dimmed or struck
  through)? GitHub does not do this; it is a small styling addition that can be
  decided while looking at it in the browser. Not required by the spec.
