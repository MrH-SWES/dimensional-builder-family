# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button; users must drag the bead all the way back to 0 to clear. A reset affordance (button or double-tap bead) would improve UX.
- No ARIA roles or keyboard navigation on the track itself (bead has `aria-label`; input has `aria-label`; label has `title`).
- Ten-bar `::before` and cube `::after` pseudo-elements use `display: flex` on absolutely-positioned children — works in all modern browsers but consider wrapping in a `<span>` if compat becomes an issue.

Notes:
- Files changed: apps/number-line/index.html (removed zoom slider, added app-owned pinch/wheel/keyboard zoom), harness/reviews/LATEST_REVIEW.md (updated)

- Removed zoom UI:
  - `.zoom-controls` div (magnifying-glass SVG + `<input type="range" id="zoomSlider">`) removed from the header entirely.
  - `.zoom-controls` and `.zoom-slider` CSS rules replaced with a single comment.
  - The `zoomSlider` const and its `input` event listener removed from JS.
  - Header now contains only the logo and the Place Value pill toggle.

- App-owned pinch implementation (mobile / tablet):
  - `workspace.addEventListener('touchstart', …, { passive: false })` — when exactly two touches are active and `beadDragging` is false, pinch mode begins: stores `pinchStartDist` (Euclidean distance between the two pointers) and `pinchStartZoom`.
  - `workspace.addEventListener('touchmove', …, { passive: false })` — while `pinchActive && e.touches.length === 2`, computes new distance, derives `newZoom = pinchStartZoom * (dist / pinchStartDist)`, passes the midpoint clientX as the focal point to `setZoom()`, then returns early so pan logic is skipped.
  - `workspace.addEventListener('touchend', …)` — clears `pinchActive` when fewer than two touches remain; clears pan state when no touches remain.
  - If `beadDragging` is true when a second finger lands, pinch is not activated (requirement: no accidental zoom during bead drag).
  - Bead's existing `touchstart` calls `stopPropagation()`, so a single-finger bead drag cannot accidentally start pinch.

- Desktop wheel/keyboard zoom behavior:
  - `scroller.addEventListener('wheel', …, { passive: false })`: intercepts only when `e.ctrlKey` is true (Ctrl+wheel on Windows/Linux, trackpad-pinch on macOS — which browsers report as ctrlKey+wheel). Steps zoom by ±0.1 per wheel notch, with `focalClientX = e.clientX` for stable focal point. Non-Ctrl wheel events fall through normally for native scroll.
  - `document.addEventListener('keydown', …)`: `+` or `=` zooms in; `-` or `_` zooms out. Focal point is the horizontal center of the scroller viewport. Skipped when an `<input>` is focused.

- Focal-point preservation (`setZoom(newZoom, focalClientX)`):
  - Before applying the new zoom: captures `savedRelX = focalClientX − scroller.getBoundingClientRect().left` and `savedContentX = scroller.scrollLeft + savedRelX`.
  - After `renderBlocks()` (which updates all pixel positions at the new `--zoom`): sets `scroller.scrollLeft = savedContentX * ratio − savedRelX`, where `ratio = newZoom / oldZoom`. This keeps the pinch/cursor focal point visually stable.

- Mobile behavior:
  - `<meta name="viewport" content="…user-scalable=no">` already prevents browser-native pinch zoom.
  - Two-finger pinch on workspace zooms the line/workspace via JS.
  - Single-finger swipe still pans horizontally (axis-locked handler preserved).
  - Portrait and landscape layouts remain unchanged.

- What was preserved:
  - Drag bead right to create units; drag left to remove units.
  - Place Value toggle (wordless pill + SVG icon) unchanged.
  - Place Value off: raw red unit cubes. Place Value on: blue ten-sticks with leftover red cubes.
  - Live quantity numeral with pop animation.
  - Horizontal panning/scrolling on mobile.
  - Mobile landscape compact layout.
  - `ensureBeadVisible()` auto-scroll behaviour.
  - Mouse-pan on empty track.
  - All zoom scaling via CSS custom property `--zoom`; internal `zoom` variable replaces slider value.
  - Single-file, zero-build, GitHub Pages deployable.

- What remains for later passes:
  - Grid/Array mode — not yet built.
  - Solid mode — not yet built.
  - Enameled cast-iron / brushed-metal material redesign — deferred.
  - Keyboard stepping of the bead (ArrowLeft / ArrowRight).
  - Reset affordance (button or double-tap bead).

- Known limitations before Grid mode:
  - `renderBlocks()` is O(quantity) DOM work on every pinch tick; smooth at typical quantities (≤ 50) but may stutter at 100–150 elements during fast pinch. A rAF-gated debounce can be added if needed.
  - `navigator.vibrate()` is wrapped in a try/catch and gated on reduced-motion; silently ignored on iOS.
  - Ctrl+wheel step size is fixed at 0.1 per notch; some high-resolution trackpads may emit many small deltaY values in rapid succession, giving a fast zoom rate. A rAF debounce or deltaY accumulator can smooth this if needed.
