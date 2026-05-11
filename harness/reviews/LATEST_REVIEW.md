# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button; users must drag the bead all the way back to 0 to clear. A reset affordance (button or double-tap bead) would improve UX.
- The zoom slider has no label or tooltip; its purpose is not obvious to new users.
- No ARIA roles or keyboard navigation on the track itself (bead has `aria-label`; input has `aria-label`; label has `title`).
- Ten-bar `::before` and cube `::after` pseudo-elements use `display: flex` on absolutely-positioned children — works in all modern browsers but consider wrapping in a `<span>` if compat becomes an issue.

Notes:
- Files changed: apps/number-line/index.html (bead scaling, PV icon, tactile feedback), harness/reviews/LATEST_REVIEW.md (updated)

- Bead scaling fix:
  - Added `--bead-size: calc(20px * var(--zoom))` and `--bead-bottom: calc(7px * var(--zoom))` as CSS custom properties in `:root`, alongside the other `--zoom`-derived sizing variables.
  - `.pull-bead` now uses `width: var(--bead-size); height: var(--bead-size); bottom: var(--bead-bottom)` so the bead shrinks and grows in exact proportion with the blocks at every zoom step.
  - Grip lines (`.pull-bead-grip` and its `::before`/`::after`) are also rescaled: widths, heights, and horizontal offsets all use `calc(… * var(--zoom))`.
  - At maximum zoom (1.2) the bead is 24 px — identical visual weight to the previous 20 px fixed size. At minimum zoom (0.3) the bead is 6 px — a small dot proportional to the 19 px blocks. The ratio is constant across the full range.
  - No changes to JS drag logic or `positionBead()`.

- Toggle icon/wordless label:
  - Added a small inline SVG (38 × 18 px) inside `.pv-toggle-wrap`, directly before the toggle `<label>`.
  - The icon shows two red unit-cube squares on the left, a right-pointing arrow in the middle, and a blue ten-stick rectangle on the right. No text glyphs.
  - `aria-hidden="true" focusable="false"` keeps it invisible to screen readers; accessibility is carried by the existing `title` and `aria-label` on the toggle.
  - `.pv-icon` CSS: `display: flex; align-items: center; margin-right: 6px; opacity: 0.75; flex-shrink: 0` — compact and non-competing.
  - No "Place Value" text present in the rendered UI.

- Unit tactile feedback:
  - Added `@keyframes unitSnap` — a quick spring-scale pop (0.65 → 1.14 → 1.0) with an opacity fade-in, running at 0.18 s.
  - `updateBeadDrag()` now detects `adding` (newQ > quantity) and `newTen` (tens count increased in PV mode). When adding without a ten-merge, it passes `{ newUnit: true }` to `renderBlocks()`.
  - `renderBlocks(animFlags)` queries the last `.cube` in the DOM after rendering and adds `.unit-snap` (with a forced reflow) when `animFlags.newUnit` is true.
  - Optional: `navigator.vibrate([8])` fires a gentle 8 ms pulse per unit step on supported browsers.

- Ten-stick merge feedback:
  - Added `@keyframes tenMerge` — a heavier 3-stage animation: scale down from 0.88 → overshoot 1.05 → minor rebound 0.97 → settle 1.0, with opacity 0.45 → 1.0, running at 0.35 s (noticeably longer/heavier than unit snap).
  - When `adding && placeValueMode && newTens > prevTens`, `renderBlocks()` receives `{ newTen: true }`, which applies `.ten-merge` to the last ten-bar added.
  - Optional: `navigator.vibrate([18])` fires a slightly stronger 18 ms pulse on supported browsers.
  - `@media (prefers-reduced-motion: reduce)` suppresses both `.unit-snap` and `.ten-merge` animations.

- What was preserved:
  - Drag bead right to create units; drag left to remove units.
  - Place Value off: raw red unit cubes. Place Value on: groups of ten as blue ten-sticks with leftover red cubes.
  - Live quantity numeral with pop animation.
  - Horizontal panning/scrolling on mobile (touch-action: pan-x + axis-locked handler).
  - Mobile landscape compact layout.
  - `ensureBeadVisible()` auto-scroll behaviour.
  - Mouse-pan on empty track.
  - All zoom scaling via CSS custom property `--zoom`.
  - Single-file, zero-build, GitHub Pages deployable.

- What remains for later passes:
  - Grid/Array mode — not yet built.
  - Solid mode — not yet built.
  - Enameled cast-iron / brushed-metal material redesign — deferred.
  - Keyboard stepping of the bead (ArrowLeft / ArrowRight).
  - Reset affordance (button or double-tap bead).
  - Zoom slider label/tooltip.

- Risks / tradeoffs:
  - Document-level `touchmove` is `{ passive: false }` but calls `preventDefault()` only when `beadDragging` is true; other touches are unblocked immediately.
  - `renderBlocks()` is O(quantity) DOM work on every drag tick; 150 elements per frame is fine at current scale. Animation classes are applied only to the single newest element per tick.
  - `navigator.vibrate()` is wrapped in a try/catch and gated on reduced-motion media query; it is silently ignored on iOS and unsupported browsers.
