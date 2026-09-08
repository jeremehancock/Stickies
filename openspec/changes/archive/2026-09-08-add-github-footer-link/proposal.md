## Why

The footer credit already points people at the author's personal website, but
Stickies is an open-source single-file app and there is currently no way to get
from the running app to its source. Anyone who wants to read the code, file an
issue, or self-host has to go looking for it. The credit line is the natural
place to put that link, since it is the one spot in the UI that already says
who made this.

## What Changes

- The desktop footer credit gains a GitHub link sitting next to the existing
  "Jereme Hancock" personal-site link, pointing at the project repository.
- The mobile version of the same credit — which lives at the bottom of the
  "How it works" help sheet, because the page pager occupies the bottom of the
  screen on touch devices — gains the same GitHub link in the same position.
- Both links open in a new tab and carry the same `rel="noopener noreferrer"`
  safety attributes the existing personal-site link already uses.
- The GitHub link is presented as an icon (the GitHub mark) with an accessible
  label, so the credit line stays short enough not to wrap on a narrow phone.
- The link is inert with respect to the board itself: on desktop the credit sits
  behind the notes and passes pointer events through, and the new link keeps
  that behaviour so it never swallows a drag meant for a note underneath.

## Capabilities

### New Capabilities
- `credit-footer-links`: the author/source credit shown in the app — where it
  appears on desktop versus touch, which outbound links it carries, how they
  open, and how it stays out of the way of interacting with the board.

### Modified Capabilities
<!-- None. The one existing capability spec, markdown-task-lists, is unrelated
     to the credit line and its requirements are unchanged. -->

## Impact

- **Code**: `index.html` only — the `.credit` and `.modal-credit` CSS blocks and
  the two markup sites that render the credit (the `<footer class="credit">`
  inside `#board`, and the `<p class="modal-credit">` at the foot of the help
  modal).
- **Data**: none. No storage, no export format, no settings touched.
- **Dependencies**: none added. The GitHub mark is inline SVG, matching how
  every other icon in the app is drawn, so it works offline.
- **Docs**: none required. The link is self-explanatory in the UI.
