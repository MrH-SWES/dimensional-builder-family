# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button; users must drag the bead all the way back to 0 to clear. A reset affordance (button or double-tap bead) would improve UX.
- No ARIA roles or keyboard navigation on the track itself (bead has `aria-label`; y-bead has `aria-label`; input has `aria-label`; label has `title`).
- Ten-bar `::before` and cube `::after` pseudo-elements use `display: flex` on absolutely-positioned children — works in all modern browsers but consider wrapping in a `<span>` if compat becomes an issue.
- Hundred-flat fusion across rows is not yet implemented. In Grid mode with PV on, each row independently shows tens + ones; no blue hundred-flat forms when 10 full rows of 10 exist. Document as a later milestone.
- The y-bead row-height calculation uses `nlTray.offsetHeight` measured live, which is accurate but means the snap-to-row threshold depends on the current zoom level. If the user changes zoom during a y-drag (unlikely), the row count could jump. Acceptable for now; a rAF-gated or pre-capture approach can smooth this if needed.
- Mobile portrait with many rows (> ~10 at default zoom): the tray-scroller gains a vertical scrollbar and the grid can be scrolled; this is functional but not polished. A future pass should add inertia scrolling feedback or a zoom-to-fit affordance.

Notes:
- Files changed: apps/number-line/index.html (added Grid mode), harness/reviews/LATEST_REVIEW.md (updated)

- Title changed from "Line Builder" to "Dimensional Builder".

- Mode control:
  - A two-position segmented icon toggle was added to the left of the PV pill in the header.
  - Line button: SVG of a horizontal line with end ticks and a small bead circle; `aria-label="Line mode"`.
  - Grid button: SVG 3×2 grid of outlined rectangles; `aria-label="Grid mode"`.
  - Clicking a button calls `setMode('line'|'grid')`, which toggles `body.grid-mode` and re-renders.

- Grid/y-bead behavior:
  - Grid mode renders the same blocks into rows 2..N as `.tray.extra-row` elements inserted before `nlBot`.
  - Each extra row is a full flex tray (same CSS class as the main tray) with slot divs for cubes and absolute-positioned ten-bars, so column alignment is identical to row 1.
  - The y-bead is an absolutely positioned `.y-bead` element inside `.tray-wrapper`; it appears at the right edge of the active column stack, centered on the bottom edge of the last row.
  - Dragging the y-bead down increases `rows`; dragging up decreases `rows` (minimum 1 when x > 0).
  - When x = 0, the y-bead is hidden and the grid is empty.
  - `MAX_ROWS = 20`.
  - Row seam: extra rows have `margin-top: 3px` which creates a visible horizontal gap, making each row countable.
  - Column seam: alternating `bg-light`/`bg-dark` slot backgrounds carry into extra rows naturally.

- Record behavior:
  - Line mode: record shows the quantity (e.g. `14`), same as before.
  - Grid mode: record shows `x × y = total` (e.g. `7 × 5 = 35`) using the Unicode × character.
  - If x = 0 in Grid mode, record shows `0`.
  - Record updates live while dragging x or y bead.
  - Font size reduced to `2.8rem` in Grid mode (`2rem` in compact landscape) to accommodate the longer string.

- Place Value behavior in Grid mode:
  - PV off: each row renders raw red unit cubes (same as Line mode row rendering).
  - PV on: each row renders complete ten-sticks (blue) plus leftover ones (red), identical to the main tray.
  - Hundred-flat fusion across rows is NOT implemented. When PV is on and x = 10 with y ≥ 10, individual rows show a single blue ten-stick; they do not fuse into a hundred-flat. **Should Fix Soon / later milestone.**

- Mobile behavior:
  - Mobile portrait: `.tray-scroller` gains `overflow-y: auto` and `touch-action: pan-x pan-y` via `body.grid-mode` CSS; vertical scrolling works natively when rows exceed the viewport.
  - Mobile landscape: same, with the compact-landscape header/font rules still active.
  - Horizontal panning continues to work (axis-locked touch handler unchanged).
  - Pinch-to-zoom continues to work; the two-finger pinch path ignores y-bead drag state.
  - Limitation: if y > ~10 at default zoom on a phone portrait, the grid requires vertical scrolling to see all rows. No automatic zoom-to-fit; user can Ctrl+wheel or pinch to reduce zoom.

- What was preserved from Line mode:
  - Drag x-bead right/left creates/removes unit cubes; bead snaps per slot.
  - Place Value toggle (wordless pill + SVG icon) unchanged.
  - PV off: raw red unit cubes. PV on: blue ten-sticks with leftover red cubes.
  - Live quantity numeral with pop animation.
  - Gesture zoom: pinch on workspace, Ctrl+wheel on desktop, +/- keyboard.
  - Horizontal panning: mouse drag on empty track; single-finger swipe (axis-locked).
  - `ensureBeadVisible()` auto-scroll behaviour.
  - `renderBlocks()` animation flags (newUnit snap, ten-bar merge) still fire for the main tray row in both modes.
  - Single-file, zero-build, GitHub Pages deployable.

- What remains before Solid mode:
  - Solid/3D mode — not yet built.
  - Hundred-flat fusion across rows — deferred.
  - Enameled cast-iron / brushed-metal full material redesign — deferred.
  - Keyboard stepping of the x-bead (ArrowLeft / ArrowRight).
  - Reset affordance (button or double-tap bead).
  - Zoom-to-fit affordance for large grids on mobile.

- Known limitations:
  - `renderBlocks()` is O(quantity × rows) DOM work on every pinch tick or drag tick; smooth for typical values (x ≤ 50, y ≤ 10) but may stutter at maximums (x = 150, y = 20 = 3000 elements). A virtualised row approach can be added if needed.
  - `navigator.vibrate()` is wrapped in a try/catch and gated on reduced-motion; silently ignored on iOS.
  - Ctrl+wheel step size is fixed at 0.1 per notch; fast trackpad wheels can zoom quickly (same as before).
