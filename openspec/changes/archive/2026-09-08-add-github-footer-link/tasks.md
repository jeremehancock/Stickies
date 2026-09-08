## 1. Styles

- [x] 1.1 In the "footer credit" CSS block in `index.html`, add a `.credit-gh`
      rule shared by both credit locations (`.credit .credit-gh` and
      `.modal-credit .credit-gh`): `display:inline-flex`, `align-items:center`,
      `vertical-align:middle`, `margin-left:8px`, `border-bottom:none` to
      cancel the underline `.credit a` / `.modal-credit a` applies, and
      `pointer-events:auto` so it stays clickable inside the pass-through
      desktop footer.
- [x] 1.2 Confirm the icon inherits colour from the link: no explicit `fill` or
      `color` in the CSS, so `currentColor` picks up the credit colour and the
      existing `:hover` accent colour in both themes.
- [x] 1.3 Add a `.touch .modal-credit .credit-gh` rule giving the icon 8px of
      padding with matching negative margins, so the tap target is 30px square
      while the credit's spacing and line height are unchanged.

## 2. Desktop footer markup

- [x] 2.1 In `<footer class="credit">` inside `#board`, add the GitHub anchor
      immediately after the "Jereme Hancock" link: `class="credit-gh"`,
      `href="https://github.com/jeremehancock/Stickies"`, `target="_blank"`,
      `rel="noopener noreferrer"`, `title="Stickies on GitHub"`,
      `aria-label="Stickies on GitHub"`.
- [x] 2.2 Inside that anchor, inline the official GitHub mark as a 16×16 SVG
      with `viewBox="0 0 16 16"`, `fill="currentColor"` and
      `aria-hidden="true"`, so it renders offline and is not announced twice.

## 3. Mobile help-sheet markup

- [x] 3.1 Add the identical anchor and SVG to `<p class="modal-credit">` at the
      foot of the "How it works" modal, immediately after the same author link.
- [x] 3.2 Diff the two credit blocks against each other and confirm their inner
      markup is character-for-character the same apart from the wrapper element,
      so the duplicated copies do not drift.

## 4. Verify in a browser

- [x] 4.1 Open `index.html` at a desktop width: the GitHub icon sits beside the
      author name, has no underline, aligns with the text baseline, and turns
      the accent colour on hover in both light and dark themes.
- [x] 4.2 Click the icon and confirm it opens the repository in a new tab with
      the Stickies board still intact in the original tab.
- [x] 4.3 Drag a note across the footer credit, passing over the GitHub icon,
      and confirm the note follows the pointer and no link is activated.
- [x] 4.4 Switch to a 360px-wide phone viewport with touch emulation: the pinned
      footer is hidden, and the credit inside the "How it works" sheet shows both
      links on a single unwrapped line.
- [x] 4.5 Tap the icon on the phone viewport and confirm it opens the repository
      in a new tab.
- [x] 4.6 Reload with the network blocked (offline) and confirm the GitHub mark
      still draws, since it is inline SVG.
