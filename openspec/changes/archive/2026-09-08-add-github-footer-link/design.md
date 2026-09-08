## Context

The credit line exists twice in `index.html`, because desktop and touch get
deliberately different layouts from the same file:

- `<footer class="credit">` lives inside `#board`. It is `position:fixed` at the
  bottom centre with `z-index:1`, so notes stack on top of it, and it sets
  `pointer-events:none` on itself with `pointer-events:auto` back on the anchor.
  That is what lets you drag a note across the credit without the browser
  treating it as a click on the credit, while the link itself stays clickable.
- `<p class="modal-credit">` sits at the foot of the "How it works" modal. On
  touch, `.touch .credit{display:none}` hides the pinned footer (the page pager
  owns the bottom of the screen there) and `.touch .modal-credit{display:block}`
  reveals this one instead. On desktop it is the other way round.

Both currently render the same sentence with one link. The two blocks are not
shared markup — they are duplicated — so any change has to be made twice and
kept consistent by hand.

Constraints that shape the design: no build step, no new dependencies, works
offline, and the mobile credit has to survive a 360px-wide sheet without
wrapping.

## Goals / Non-Goals

**Goals:**
- Put a link to `https://github.com/jeremehancock/Stickies` next to the existing
  personal-site link, in both credit locations.
- Open it in a new tab, with the same `rel="noopener noreferrer"` hygiene the
  existing link already has.
- Keep the credit on one line at phone width.
- Keep the desktop credit's pass-through-to-notes behaviour intact.

**Non-Goals:**
- De-duplicating the two credit blocks into one shared template. That would
  mean generating the credit from JS, which is more machinery than a second
  link is worth in a file that deliberately hand-writes its markup.
- Adding any other social or project links.
- Changing where the credit appears on either device, or restyling it.

## Decisions

### Icon-only GitHub link, not a text link

The credit is one non-wrapping line inside a 360px sheet on mobile. Adding the
words "on GitHub" or a second sentence risks either wrapping or shrinking the
font. An icon costs about 16px of width.

The accessibility cost of an icon-only link is that it has no visible text to
announce, so it gets an explicit `aria-label` ("Stickies on GitHub"), a `title`
for the desktop hover tooltip — matching how every toolbar button in this file
is already labelled — and `aria-hidden="true"` on the `<svg>` so a screen reader
reads the label once rather than trying to describe the graphic too.

*Alternative considered:* text link reading "GitHub". Rejected on width; the
mobile line is already close to full.

### Use the official GitHub mark as inline SVG

Every icon in this file is inline SVG using `stroke="currentColor"` in a
Lucide-ish 24×24 stroke style. The GitHub mark is a solid glyph and does not
exist in that style — a stroked approximation of the Octocat reads as a generic
blob at 16px. So this one icon breaks the house style and uses the official
mark: a single filled `<path>` on a `0 0 16 16` viewBox with
`fill="currentColor"`.

`fill="currentColor"` is the important part: it means the icon inherits the
link's text colour, so it gets the credit's normal colour and the accent colour
on hover for free, in both light and dark themes, with no extra CSS.

*Alternative considered:* an `<img>` pointing at GitHub's hosted mark. Rejected —
it would be a network request, so it would break offline, which is a hard
constraint for this app.

### Style the icon link as a sibling with its own rule, not by reusing `.credit a`

`.credit a` sets `font-weight:600` and a `border-bottom` underline. An underline
under a square icon looks like a mistake. So the icon link gets its own class
(`.credit-gh`) that opts out of the border, sets `display:inline-flex` with
`vertical-align:middle` so the glyph sits on the text baseline rather than
hanging low, and keeps the `pointer-events:auto` that the text link needs on
desktop.

One class is written to cover both locations (`.credit .credit-gh` and
`.modal-credit .credit-gh`) so the two copies of the markup stay identical apart
from their wrapper, which makes the duplication easier to keep in sync.

### Grow the tap target on touch with padding, not a bigger icon

The mark is drawn at 14px so it sits proportionally beside 12px credit text. As
a hit area that is far too small for a finger. Rather than drawing a bigger
icon, the touch copy gets 8px of padding with matching negative margins, which
grows the hit area to about 30px square while leaving the drawn icon, the gap
after the author's name, and the line's height exactly as they were.

*Alternative considered:* a larger icon on touch. Rejected — it would make the
icon visually dominate a line that is meant to be quiet, and it risks wrapping
the credit at 360px.

### Separate the two links with a small gap, not punctuation

A middot or pipe between the name and the icon adds visual noise to a line whose
whole job is to be quiet. A `margin-left` of about 8px on the icon link reads as
"and also this", which is the intent.

## Risks / Trade-offs

- **The two credit blocks are duplicated, so they can drift** → Both are edited
  in the same change with identical inner markup, and the shared `.credit-gh`
  class means a future style change only has to be made once. The tasks list
  makes checking both sites an explicit step.
- **An icon-only link is less discoverable than a labelled one** → Mitigated by
  the GitHub mark being about as universally recognised as an icon gets, plus a
  `title` tooltip on desktop. On touch there is no hover tooltip, but the icon
  sits directly beside the author's name inside a help sheet, where its meaning
  is clear from context.
- **The icon widens the mobile credit line and could push it to wrap** → The
  added width is roughly one character plus the gap. Verified by opening the
  help sheet at a 360px viewport; if it ever did wrap, the fallback is dropping
  the word "and maintained" from the sentence rather than shrinking the icon.
- **Breaking the app's stroke-icon house style for one glyph** → Accepted
  deliberately. Recognisability of a brand mark beats internal consistency here,
  and the icon still inherits colour like every other icon does.

## Migration Plan

Not applicable — this is additive markup and CSS in a single static file. There
is no data, no storage format, and no persisted state involved. Rollback is
reverting the commit.

## Open Questions

None. The repository URL, the link target, and both insertion points are all
settled.
