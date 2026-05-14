# Latest Review

## Grid Mode Geometry + Hundred-Flat Fusion — 2026-05-14

Status: PENDING — awaiting human acceptance-test run

### Must Fix
- None identified by automated analysis.

### Should Fix Soon
- No reset affordance. Users must drag both beads back to zero manually.
- Keyboard stepping of the x-bead (ArrowLeft / ArrowRight) not yet implemented.
- No zoom-to-fit for large grids on mobile.

### Summary of Changes

**Architecture: unified `#gridCanvas` replaces `nlTray + arraySurface` in Grid mode**

Previous Grid mode reused `#nlTray` as row 1 (with 8 px vertical padding + 1 px border)
and `#arraySurface` as rows 2..N (no padding, 1 px seam borders).  
Row 1 was taller than rows 2..N by 17 px, making cells non-square and breaking
hundred-flat geometry.

The new approach:

- `#nlTray` is a **zero-height metric anchor** in Grid mode (`height: 0; overflow: visible;
  padding-top/bottom: 0; border: none; background: transparent`). Its `slot-1` DOM node
  remains for `getSlotMetrics()` (offsetWidth / offsetLeft). Its slots are transparent in
  Grid mode via CSS.
- `#gridCanvas` (`<div class="grid-canvas" id="gridCanvas">`) is added after `#nlTray`
  in x-content-area and is shown only in Grid mode. It contains ALL rows (1..rows) as
  `.grid-row` divs. Each `.grid-row` has `height: var(--slot-w)` (no border) so every row
  is exactly one cell-height. Ten rows = 10 × `--slot-w` = exactly the same as ten columns
  → **square cells and square hundred-flats**.
- `#arraySurface` is hidden via `display: none !important` in Grid mode (superseded by
  `#gridCanvas`). `renderArraySurface()` is no longer called from `renderBlocks()`.

**X-bead centerline**

`body.grid-mode .pull-bead` sets `bottom: auto; top: calc(var(--slot-w) / 2);
transform: translate(-50%, -50%)`. Since `#nlTray` has `height: 0` and `overflow: visible`,
the bead's center is at `nlTray.offsetTop + slotW/2 = nlTop.offsetHeight + slotW/2`, which
equals row 1's center in `#gridCanvas`. ✓

**Y-axis track alignment**

`body.grid-mode .y-axis-track` sets `padding-top: 0; padding-bottom: 0; border: none;
box-shadow: 0 0 0 1px rgba(0,0,0,0.1), inset 0 4px 15px rgba(0,0,0,0.1)`. Removing the
1 px border prevents a 2 px height mismatch; removing vertical padding ensures each y-slot
is exactly `--slot-w` tall. `alignYAxis()` sets `margin-top = nlTop.offsetHeight` so the
track's top coincides with `#gridCanvas`'s top (which also starts at `nlTop.offsetHeight`
because nlTray has height 0). ✓

**Y-bead centerline**

`positionYBead()` uses `yAxisTrack.offsetTop + yAxisTrack.offsetHeight`. With no vertical
padding or border on the track, `offsetHeight = rows × slotW` exactly, placing the bead
center at the bottom edge of the last grid row. ✓

**Bottom rail**

`.grid-canvas::after` provides the heavy white 6 px bottom rail at the bottom of the full
array (`position: absolute; bottom: 0`). The old rail was on `#nlTray::after` which was
hidden in Grid mode. ✓

**Hundred-flat fusion (`renderGridCanvas`)**

New function `renderGridCanvas()` replaces `renderArraySurface()` for Grid mode.

- PV off: red unit cubes in every slot of every row (identical per row).
- PV on:
  - **Hundred-flats** (`hvFlatsX = ⌊quantity/10⌋, hvFlatsY = ⌊rows/10⌋`): one `.hundred-flat`
    div per (tx, ty) pair, `position: absolute` in `#gridCanvas` at
    `left = trayPad + tx×10×slotW`, `top = ty×10×slotW`. Width and height both
    `10×slotW` → **square** at every zoom level.
  - **Ten-sticks** in non-hundred-flat rows (rows `hvFlatsY×10+1..rows`): one `.ten-bar`
    per complete decade, `position: absolute` within the `.grid-row` at
    `left = tx×10×slotW` (no extra trayPad offset — the row is already shifted by
    `#gridCanvas`'s horizontal padding).
  - **Unit cubes** in partial-column slots of non-hundred-flat rows.
  - **Unit cubes** in the partial-column slots of hundred-flat rows (columns to the right
    of all complete ten-groups).

**Hundred-flat style**

`.hundred-flat` is `10×slotW` square, `background: repeating-linear-gradient` overlays
1 px grid-lines on a warm golden yellow `#C8960C`, `transform: rotateX(10deg)` for depth.
`::before` pseudo shows `'100'` stamp (same pattern as cube `'1'` and ten-bar `'10'`).

**`renderBlocks()` routing**

```
if (mode === 'line')  → render in nlTray (unchanged)
else                  → renderGridCanvas()
```

`renderArraySurface()` is defined but no longer called; `arraySurface` is always hidden in
Grid mode via CSS.

**What was preserved from Line mode**

- Drag x-bead right/left creates/removes unit cubes; bead snaps per slot.
- `unitSnap` and `tenMerge` animations (Line mode only).
- Place Value toggle, live record, `ensureBeadVisible()`, pinch/Ctrl+wheel/keyboard zoom,
  horizontal pan — all unchanged.

**Acceptance tests (self-review)**

| Test | Status |
|---|---|
| Line mode still works — x-bead, blocks, PV, zoom, animations all untouched | Expected PASS |
| Grid 7×5 shows one unified 7-by-5 square-cell array | Expected PASS |
| Bottom rail under row 5, not row 1 | Expected PASS (grid-canvas::after) |
| X/Y axes share one origin; x/y ticks align to square-cell grid | Expected PASS |
| Beads on axis centerlines | Expected PASS |
| PV on, 10×10 → one square yellow hundred-flat | Expected PASS |
| PV on, 40×17 → four square flats + aligned 40×7 structure | Expected PASS |
| Record shows correct products | Expected PASS |
| No Solid mode added | PASS |
| No visible instruction text added | PASS |

---



### Must Fix
- None

### Should Fix Soon
- Hundred-flat fusion not yet implemented. When x = 10 and y = 10 with Place Value on, each row independently shows a single blue ten-stick. No yellow hundred-flat forms. **Next specific milestone before Solid mode.**
- No reset affordance. Users must drag both beads back to zero manually. A reset affordance (button or double-tap bead) would improve UX.
- Keyboard stepping of the x-bead (ArrowLeft / ArrowRight) not yet implemented.
- No zoom-to-fit for large grids on mobile. At x > ~8 or y > ~10 at default zoom, the grid requires panning/scrolling.

### Notes

**Files changed:**
- `apps/number-line/index.html` — Grid mode refactored (x/y axes + unified array surface)
- `harness/reviews/LATEST_REVIEW.md` — this file

---

**x-axis / y-axis model:**
- Grid mode now grows from the same upper-left origin as Line mode.
- The existing horizontal number line is the x-axis across the top (labels + tray with x-bead and blocks).
- A new y-axis track runs top-to-bottom on the left side. It renders as a vertical counterpart to the number line: square slots labeled 1..rows, same background and border-radius styling as the x-axis tray.
- In grid mode the tray-wrapper switches to `flex-direction: row`: y-axis area on the left, x-content area (nlTop + nlTray + arraySurface + nlBot) on the right.
- `alignYAxis()` measures `nlTop.offsetHeight` and sets the y-axis-track's `margin-top` so that y-slot 1 aligns pixel-accurately with the x-axis tray row.

**y-axis bead behavior:**
- The y-bead is now attached to and positioned within `.y-axis-area` (left side), not the far-right column edge.
- `positionYBead()` centers the bead horizontally on the y-axis area (`left = yAxisArea.offsetWidth / 2`) and places it at the bottom edge of the y-axis track (`top = yAxisTrack.offsetTop + yAxisTrack.offsetHeight`).
- `updateYBeadDrag()` uses `getSlotMetrics().w` (the square slot side) as the snap unit instead of the old `nlTray.offsetHeight` fallback. This is more accurate and zoom-aware.
- Dragging the y-bead down increases `rows`; dragging up decreases `rows` (minimum 1).
- When `quantity = 0` or in Line mode, the y-bead is hidden.

**Unified array rendering:**
- Removed `createExtraRow()` and `renderExtraRowBlocks()` (old stacked-rails approach).
- Added `renderArraySurface()`: renders rows 2..rows as `.array-row` divs inside a single `#arraySurface` element.
- `.array-row` has no individual border-radius, background, or box-shadow — it is a flat row inside the unified surface.
- `.array-surface` provides the unified outer shape: same background as the track, `border-radius: 0 0 16px 16px`, `border-top: none` so it joins flush with `#nlTray`.
- `#nlTray` in grid mode has `border-radius: 16px 16px 0 0` and no bottom border (seamless joint with array surface).
- Row seams are visible via `border-top: 1px solid rgba(0,0,0,0.08)` on each `.array-row`.
- Column seams come from alternating `bg-light`/`bg-dark` slots as before.
- When `rows = 1`, `arraySurface` is hidden (`display: none`) and `#nlTray` stands alone as a regular rounded pill.

**Place Value behavior:**
- PV off: raw red unit cubes fill each row of the array (same as x-axis row).
- PV on: each row of the array shows blue ten-bars plus red leftover cubes, identical to the x-axis row.
- Place Value toggle still works in both Line and Grid mode.
- Hundred-flat fusion (x = 10, y = 10 with PV on → yellow flat) is **deferred** as the next milestone.

**Record behavior:**
- Line mode: shows `n` (unchanged).
- Grid mode: shows `x × y = total` (e.g. `7 × 5 = 35`). When `quantity = 0`, shows `0`.
- Updates live while dragging either bead.
- Font size reduced to `2.8rem` in grid mode (unchanged from previous).

**Mobile / desktop behavior:**
- Pinch-to-zoom, Ctrl+wheel, and keyboard +/- zoom all still work.
- Horizontal panning (mouse drag / single-finger swipe, axis-locked) still works.
- `body.grid-mode .tray-scroller` has `overflow-y: auto` for vertical scroll when rows exceed the viewport.
- Portrait and landscape remain usable; no automatic zoom-to-fit (deferred).

**What was preserved from Line mode:**
- Drag x-bead right/left creates/removes unit cubes; bead snaps per slot.
- Place Value toggle (wordless pill + SVG icon) unchanged.
- Live quantity numeral with pop animation.
- `ensureBeadVisible()` auto-scroll.
- `unitSnap` and `tenMerge` animations.
- Single-file, zero-build, GitHub Pages deployable.

**What remains before Solid mode:**
1. Hundred-flat fusion across grid rows (10 × 10 → yellow flat) — **next milestone**.
2. Reset affordance (button or double-tap bead).
3. Keyboard stepping of x-bead (ArrowLeft / ArrowRight).
4. Zoom-to-fit affordance for large grids on mobile.
5. Full enameled cast-iron / brushed-metal material redesign.

---

## Previous Review — Docs Update — 2026-05-11

Status: PASS (docs-only, no behavior changes)

### Docs Created

| File | Purpose |
|---|---|
| `docs/00_MATH_THINGS_DESIGN_DOCTRINE.md` | Master design doctrine: no words, teacher teaches, equation is the shadow, material identity overview, interaction rules. |
| `docs/01_MATERIAL_SYSTEM.md` | Full material specification: enameled cast iron for math objects, brushed/machined metal for infrastructure. Color palette, states, slot backgrounds. |
| `docs/02_DIMENSIONAL_BUILDER_DOCTRINE.md` | App-specific rules for Number-Line (Line/Grid), Array Builder, Array Builder v2, and the planned Solid mode. Dimensional progression table and remaining milestones. |
| `docs/12_DO_NOT_VIOLATE.md` | Hard constraints: no words in UI, no behavior changes in docs tasks, no gamification, no external dependencies, no adaptive behavior, material system compliance, harness review requirement. |

### How Future Agents Should Use These Docs

1. **Start with `docs/12_DO_NOT_VIOLATE.md`** before making any change. Confirm the task does not conflict with any hard constraint.
2. **Read `docs/00_MATH_THINGS_DESIGN_DOCTRINE.md`** to understand the philosophical intent of every design decision.
3. **Consult `docs/01_MATERIAL_SYSTEM.md`** whenever adding or modifying any visual element to ensure it belongs to the correct material category.
4. **Consult `docs/02_DIMENSIONAL_BUILDER_DOCTRINE.md`** for app-specific rules (bead behavior, record format, mode names, deferred milestones).
5. **Update `harness/reviews/LATEST_REVIEW.md`** after any non-trivial change with a pass/fail, must-fix list, should-fix-soon list, and a summary of what changed.

---

## Previous Review

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
