# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- The equation font size is set relative to `--grid-size`. On very small screens (narrow mobile) the grid shrinks and the equation can become hard to read; a `min-font-size` clamp would help.
- Sandbox limits are hardcoded (MAX_COLS = 12, MAX_ROWS = 12) with no UI indicator of the boundary. Users hitting the limit get no feedback.
- No ARIA roles, keyboard navigation, or screen-reader labels. Accessibility is entirely absent.

Notes:
- Files changed: apps/array-builder-v2/index.html (flat-layer handoff + mobile header), harness/reviews/LATEST_REVIEW.md (updated)

- What changed in this revision:
  - Flat-layer handoff redesigned: `finish()` no longer replaces all `.row-fused` elements with a single undifferentiated rectangle. Instead each row is kept in the DOM, stripped of its handle/tag/label, and given the `.flat-done` class. A transparent `.flat-glow` overlay is appended on top, framing the whole layer with a 2px blue border + soft blue drop-shadow without filling the interior. The 4px natural gaps between rows (GRID minus block-size) remain visible as row seams. After a 10-by-8 build the final state reads as eight distinct blue sticks sharing one unified surface, not a blue rectangle.
  - `.row-fused::after` vertical column lines are still present in the settled state via inheritance — column structure inside each stick is also preserved.
  - Row settle animation (rowSettle): each row fades from opacity 0.5 and scaleY 0.93 to full size, staggered by 28 ms per row so rows appear to "lock" one at a time from top to bottom.
  - Flat glow animation (flatResolve): the unified frame scales from 0.97→1 and opacity 0→1 at 0.6 s spring timing.
  - Mobile header: `@media (max-height: 600px), (max-width: 430px)` reduces header from 80 px to 54 px and footer padding to 10 px. JS `updateGridSize()` reads the same breakpoint and reduces the availH subtraction accordingly. `init()` adjusts the target-zone centering offset (headerHalf: 27 px on mobile vs 40 px desktop).
  - Removed: `.flat-layer` CSS class (replaced by `.flat-done` + `.flat-glow`).

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
  3. User drags the vertical bead downward; additional .row-fused bars stack beneath with 4-px gaps. Right-edge .skip-tag shows cumulative totals. updateMath() shows repeated-addition sum (e.g. 4 + 4 + 4).
  4. On release (≥ 2 rows): finish() is called. After a 320 ms pause each .row-fused bar has its handle, skip-tag, and label removed and gains .flat-done (softer shadow, rowSettle animation staggered 28 ms per row). A transparent .flat-glow div (same width × height as the full stack, background: transparent) is appended and animates in with flatResolve, giving a blue-tinted outer border and glow without hiding the row seams.
  5. 420 ms after the settle begins, the multiplication record renders: `rows × cols = total` in large (grid-relative) Lexend 900 weight, using --text-light .operator tokens between the numerals. Animation: pop (scale 0.5→1, spring). The equation is always last — it appears only after the layer exists.
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
  - Individual row elements remain in the DOM after finish(); this is intentional (the row structure must stay readable) and enables teachers or a future layer to count or re-inspect rows.
  - Grid sizing recalculates on resize and calls init(), which resets any in-progress build. Consistent with Array Builder v1 behaviour but loses work on viewport change.
  - --bg: #F5F5F7 (Number Line) is lighter than Array Builder's #E5E5EA; dot-grid opacity was reduced to 10% accordingly so the dots don't overpower the lighter surface.
