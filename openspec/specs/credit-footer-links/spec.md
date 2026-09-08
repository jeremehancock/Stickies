# Credit Footer Links

## Purpose

The app carries a small credit line naming its author and pointing at its
source. It appears in two places depending on the device: pinned to the bottom
of the board on desktop, and tucked at the foot of the "How it works" sheet on
touch, where the page pager already owns the bottom of the screen.

It is deliberately quiet. On desktop it sits behind the notes and lets pointer
events fall through, so dragging a note across it works as if it were not there,
while the links themselves stay clickable.

## Requirements

### Requirement: The credit line carries both an author link and a source link

The app SHALL show a credit line naming the author, and that credit SHALL carry
two outbound links: the author's personal website and the project's GitHub
repository. The GitHub link MUST sit immediately after the author link in the
same credit line.

#### Scenario: Desktop footer shows both links
- **WHEN** the app is open at a desktop width
- **THEN** the footer credit reads "Built and maintained by Jereme Hancock"
  with "Jereme Hancock" linking to the personal website, followed by a GitHub
  link pointing at the project repository

#### Scenario: Mobile help sheet shows both links
- **WHEN** the app is open on a touch device and the "How it works" sheet is
  opened
- **THEN** the credit at the foot of the sheet shows the same author link
  followed by the same GitHub link

#### Scenario: The GitHub link points at this project's repository
- **WHEN** the GitHub link is followed
- **THEN** it opens the Stickies repository at
  `https://github.com/jeremehancock/Stickies`

### Requirement: Credit links open in a new tab safely

Every link in the credit SHALL open in a new browsing context and SHALL carry
`rel="noopener noreferrer"`, so following one never navigates away from a board
in progress and never hands the opened page a reference back to this one.

#### Scenario: Following the GitHub link keeps the board open
- **WHEN** the user clicks or taps the GitHub link
- **THEN** the repository opens in a new tab and the Stickies tab is still on
  the board, unchanged

#### Scenario: Both credit links carry the same safety attributes
- **WHEN** the credit markup is inspected on desktop or in the help sheet
- **THEN** the author link and the GitHub link both specify `target="_blank"`
  and `rel="noopener noreferrer"`

### Requirement: The GitHub link is an accessible icon

The GitHub link SHALL be presented as the GitHub mark drawn as inline SVG, so it
needs no network request and stays available offline. It MUST expose an
accessible name describing where it goes, and the icon graphic itself MUST be
hidden from assistive technology so the name is not read twice.

#### Scenario: Screen reader announces the link's destination
- **WHEN** a screen reader reaches the GitHub link
- **THEN** it announces a link whose name identifies it as the project's GitHub
  repository, and does not announce the SVG separately

#### Scenario: The icon renders with no network access
- **WHEN** the app is opened offline
- **THEN** the GitHub mark still draws, because it is inline SVG rather than a
  fetched image or icon font

#### Scenario: Touch devices get a large enough tap target
- **WHEN** the app is running on a touch device
- **THEN** the GitHub link's hit area is at least 30 by 30 CSS pixels, larger
  than the drawn icon, so it can be tapped reliably with a finger without the
  credit line's spacing or height changing

#### Scenario: The icon matches the credit text
- **WHEN** the credit is shown in either theme
- **THEN** the GitHub mark is drawn in the same colour as the author link and
  picks up the same hover colour, so the two links read as one pair

### Requirement: The credit never intercepts board interaction

On desktop the credit sits behind the notes and SHALL let pointer events pass
through to whatever is underneath, except on the links themselves, which remain
clickable. Adding the GitHub link MUST NOT change that.

#### Scenario: Dragging a note over the credit still works
- **WHEN** the user drags a note across the area of the footer credit,
  including over the GitHub icon
- **THEN** the note follows the pointer as normal and no link is activated

#### Scenario: The GitHub link is still clickable
- **WHEN** the user clicks directly on the GitHub icon with no note above it
- **THEN** the link activates

### Requirement: The credit follows the layout it already uses per device

The credit SHALL keep its existing placement: pinned to the bottom of the board
on desktop, and moved into the "How it works" sheet on touch devices, where the
page pager occupies the bottom of the screen. The credit MUST stay on a single
line at a phone width.

#### Scenario: Touch devices hide the pinned footer
- **WHEN** the app is running on a touch device
- **THEN** no credit is pinned to the bottom of the board; it appears only
  inside the "How it works" sheet

#### Scenario: The credit does not wrap on a narrow phone
- **WHEN** the "How it works" sheet is open on a 360px-wide screen
- **THEN** the credit text and both links fit on one line without wrapping or
  overflowing the sheet

