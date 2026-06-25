# Stickies

A sleek, single-file sticky-note whiteboard for quickly capturing ideas. No build
step, no dependencies — just open `index.html` in a browser.

## Screenshot

![stickies screenshot](screenshot-updated.png)

## Use it

Open `index.html` (double-click the file, or serve the folder). That's it.

- **Add** — double-click anywhere on the board, or hit **+ Note**.
- **Type** — click a note and start writing.
- **Move** — drag it. The note lifts, tilts toward the direction you fling it, and
  springs back to rest when you let go.
- **Resize** — drag the corner grip. Notes stay in tidy sticky-note proportions
  (within a min and max size), and any text past the note's height scrolls inside it.
- **Change Color** — hover a note and tap a color dot (or pick the color for new
  notes from the top bar).
- **Delete** — hover and hit **×**. There's an **Undo** (also <kbd>Ctrl</kbd>/<kbd>⌘</kbd>+<kbd>Z</kbd>).
- **Rename the board** — click the board name in the top bar and type. The browser
  tab updates to match.
- **Switch theme** — Stickies opens in **dark mode**; toggle light/dark with the
  sun/moon button in the top bar. Your choice is remembered on this device.
- **Export / Import** — use the ↓ button to **export** the board to a
  `stickies-<board-name>.json` file, and the ↑ button to **import** one back.
  Each opens a short explainer first. ⚠️ **Importing completely overwrites** the
  board stored in this browser — there's no undo — so export first if you want a
  backup. The file is created locally; nothing is uploaded.

Works with mouse and touch.

## Storage

Your notes — along with the board name and your theme choice — are saved with your
browser's **`localStorage`** — on **this** device, in **this** browser only. They
are **not** synced across devices, shared, or backed up, and clearing your browser
data (or using a private window) wipes them. This is meant as a scratchpad for
ideas, not long-term storage.

Need a backup, or want to carry a board to another browser or device? **Export**
it to a JSON file and **Import** it wherever you like (see above).

## Built with

Vanilla HTML, CSS and JavaScript in one file. No frameworks.

## License

[MIT License](LICENSE)

## AI Disclosure

This project was created with the help of AI.
