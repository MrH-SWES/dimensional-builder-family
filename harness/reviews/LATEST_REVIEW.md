# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button; users must drag the bead all the way back to 0 to clear. A reset affordance (button or double-tap bead) would improve UX.
- The zoom slider has no label or tooltip; its purpose is not obvious to new users.
- No ARIA roles, keyboard navigation, or screen-reader labels on the track or bead (bead has aria-label only).
- Ten-bar `::before` and cube `::after` pseudo-elements use `display: flex` on absolutely-positioned children — works in all modern browsers but consider wrapping in a `<span>` if compat becomes an issue.

Notes:
- Files changed: apps/number-line/index.html (full rebuild as Line Builder foundation), harness/reviews/LATEST_REVIEW.md (updated)

- What was removed:
  - Supply tray (action-tray, dispensers, preview-cube, preview-ten, bank-item) — gone from HTML and CSS.
  - State variables `totalTens` / `totalOnes` replaced by single `quantity` integer.
  - All bank/dispenser click handlers and HTML5 drag-and-drop (dragstart / dragover / drop) listeners.
  - Tap-to-fuse (10 ones → ten-bar) and tap-to-break (ten-bar → 10 ones) click interactions.
  - `dashboard` div and its associated styles.
  - Body vertical scroll (`min-height: 100dvh; overflow-y: auto`) replaced by `height: 100dvh; overflow: hidden` — track is now flex:1 and fills viewport instead.

- Pull bead behavior:
  - A 40 px chrome sphere (`.pull-bead`) lives inside `.tray` as an absolutely-positioned child.
  - Position: `left = quantity × slotW + trayPad` (derived from `slot-1.offsetWidth` and `slot-1.offsetLeft` so it is zoom-aware).
  - On mousedown / touchstart the bead enters dragging state.
  - Document-level mousemove / touchmove compute `delta = Math.round(dx / slotW)` from the drag-start clientX, clamp to [0, 150], set `quantity`, call `renderBlocks()` and `ensureBeadVisible()`.
  - `ensureBeadVisible()` uses `getBoundingClientRect()` to auto-scroll the container within 72 px of the scroller's visible edge.
  - Mouse pan of the scroller (click-drag on empty track) is preserved; explicitly skips the bead target.
  - Workspace axis-locked touch-pan is preserved and also skips the bead target.

- Place Value toggle behavior:
  - Apple-style 48 × 28 px pill toggle in the header, labelled "Place Value".
  - OFF (default): all `quantity` units rendered as red `.cube` elements in their respective `.slot` — raw unit display.
  - ON: `Math.floor(quantity / 10)` blue `.ten-bar` elements plus `quantity % 10` red `.cube` elements — auto-fused display.
  - Toggle re-renders synchronously on `change`; replaces tap-to-fuse / tap-to-break entirely.

- Mobile behavior:
  - `body { height: 100dvh; overflow: hidden }` + `flex: 1` on `.workspace` and `.tray-scroller` — track fills remaining viewport; never below the fold.
  - `@media (max-height: 500px)` compact mode: header 48 px, total 3 rem, workspace padding 6 px, "Place Value" text hidden (pill visible).
  - Horizontal track pan via `touch-action: pan-x` on scroller + workspace axis-locked handler.
  - Bead touchstart is `{ passive: false }` + `preventDefault()` + `stopPropagation()`.
  - `env(safe-area-inset-bottom)` on `.tray-scroller` padding-bottom for iOS home-indicator clearance.

- What remains placeholder / not yet implemented:
  - Grid mode (pull quantity into a flat array) — next Dimensional Builder mode, not yet built.
  - Keyboard stepping of the bead (ArrowLeft / ArrowRight).
  - Undo / step-back affordance.

- Risks / tradeoffs:
  - Document-level `touchmove` is `{ passive: false }` but calls `preventDefault()` only when `beadDragging` is true; other touches are unblocked immediately.
  - `renderBlocks()` is O(quantity) DOM work on every drag tick; 150 elements per frame is fine at current scale.
  - Single-file, zero-build, GitHub Pages deployable — no regressions on this constraint.
