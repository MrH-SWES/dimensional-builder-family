# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- The equation font size is set relative to `--grid-size`. On very small screens (narrow mobile) the grid shrinks and the equation can become hard to read; a `min-font-size` clamp would help.
- Sandbox limits are hardcoded (MAX_COLS = 12, MAX_ROWS = 12) with no UI indicator of the boundary. Users hitting the limit get no feedback.
- No ARIA roles, keyboard navigation, or screen-reader labels. Accessibility is entirely absent.

Notes:
- Files changed: apps/array-builder-v2/index.html (full rewrite from real sources), harness/reviews/LATEST_REVIEW.md (new)

- What was reused from Number Line:
  - Exact CSS custom properties: --bg (#F5F5F7), --glass, --surface, --text-main, --text-gray, --text-light, --accent, --border
  - Shadow tokens: --shadow-soft, --shadow-heavy, --shadow-block (the inset-highlight + drop-shadow system)
  - Material colours: --ones (#DC143C red) and --tens (#4169E1 blue)
  - Spring easing: cubic-bezier(0.34, 1.56, 0.64, 1) on transitions and animations
  - Header: glass morphism (backdrop-filter blur(40px)), 80px height, logo "Math Things" with logo-bead radial-gradient chrome sphere
  - Logo-bead hover animation (scale + translateY)
  - Footer: "Math Things · Math, in your hands" with footer-bead inline decoration
  - Lexend font weight stack (300 / 400 / 600 / 800 / 900)
  - Body layout: height 100dvh, flex column, overflow hidden, -webkit-font-smoothing antialiased

- What was preserved from Array Builder:
  - Workspace background: dot-grid (radial-gradient 2px dots) + top spotlight (radial-gradient from white), both layered on --bg
  - Dashed #target-zone outline centred in workspace, landing-strip highlight during drag
  - .block (red unit cubes): absolute positioned, --block-size square, --ones colour, --shadow-block, numbered .val spans
  - Staggered merge animation: outer numerals nudge → all converge to centre → single numeral survives → blue .row-fused replaces blocks (3-phase, 1500 ms total)
  - .row-fused (blue composed bar): --tens colour, --shadow-block, repeating vertical grid lines via ::after pseudo-element, .fused-label centred numeral
  - .bead-handle (chrome sphere): --bead-surface radial gradient, same inset/drop shadow, cursor grab / grabbing, scale(0.85) active press
  - Horizontal handle (handleX): pointerdown → pointermove delta / GRID → renderHorizontal; pointerup → animateStaggeredMerge if cols ≥ 2
  - Vertical handle (handleY): pointerdown → pointermove delta / GRID → renderVertical; pointerup → finish() or animateUnfuse()
  - Touch coordinate fallback (e.clientX ?? e.touches?.[0]?.clientX ?? startX) applied to both handles, matching original Array Builder pattern; NaN-safe via defaulting to startX when both are unavailable
  - .skip-tag running counts on right edge of each row (light while dragging, .lit = --tens bold when complete)
  - updateMath() repeated-addition display while vertical dragging (e.g. 4 + 4 + 4)
  - animateUnfuse() snap-back to horizontal when Y handle is released at 1 row
  - resetBtn floating circle icon (SVG arrows), shows after first fuse

- How the row-to-flat interaction works:
  1. User drags the chrome bead handle rightward; red unit blocks (--ones) materialise one per grid cell. Numbers 1…N label each position.
  2. On release (≥ 2 cols): animateStaggeredMerge() plays — numerals converge and fade, single count survives, then the block row swaps to a single blue .row-fused bar with a vertical bead handle below it.
  3. User drags the vertical bead downward; additional .row-fused bars stack beneath with 1-px gaps. Right-edge .skip-tag shows cumulative totals. updateMath() shows repeated-addition sum (e.g. 4 + 4 + 4).
  4. On release (≥ 2 rows): finish() is called. After 320 ms pause, all .row-fused elements are replaced by a single .flat-layer div — same total width × height, --tens blue, carrying a visible 2D grid pattern (repeating-linear-gradient in both axes) showing columns and rows within the unified surface. Animation: flatResolve (scale 0.96→1, opacity 0→1, spring easing, 500 ms).
  5. 420 ms after the flat layer appears, the multiplication record renders: `rows × cols = total` in large (grid-relative) Lexend 900 weight, using --text-light .operator tokens between the numerals. Animation: pop (scale 0.5→1, spring). The equation is always last — it appears only after the layer exists.
  6. Reset button (top-right) is visible throughout steps 3–5 and returns to step 1 cleanly.

- What is placeholder:
  - Sandbox limits hardcoded at 12×12 with no boundary indicator for users.
  - Equation display font size is purely grid-relative; no minimum size clamp for narrow viewports.
  - No persistence (localStorage) or shareable state.
  - No sound / haptic feedback.
  - No accessibility (ARIA, keyboard nav, screen-reader labels).

- Risks/tradeoffs:
  - Removing the level / tutorial system (intentional per spec) means first-time users have no guided discovery. The bead handle affords drag but there is no caption pointing at it.
  - Sandbox-only mode removes the motivating "target shape". Appropriate for a freeplay manipulative but may feel open-ended without a surrounding task context.
  - The flat-layer resolve clears the individual row-fused elements from the DOM. If a teacher wants students to count individual rows after completion, the rows are gone (only the flat layer remains).
  - Grid sizing recalculates on resize and calls init(), which resets any in-progress build. Consistent with Array Builder v1 behaviour but loses work on viewport change.
  - --bg: #F5F5F7 (Number Line) is lighter than Array Builder's #E5E5EA; dot-grid opacity was reduced to 10% accordingly so the dots don't overpower the lighter surface.
