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
- Files changed: apps/number-line/index.html (control refinements), harness/reviews/LATEST_REVIEW.md (updated)

- Toggle label change:
  - Removed the visible `<span class="pv-toggle-label">Place Value</span>` from the HTML.
  - Removed the `.pv-toggle-label` CSS rule and the `gap: 8px` from `.pv-toggle-wrap` (now unnecessary).
  - The pill toggle itself remains in the header at the same position.
  - Accessibility preserved via `title="Toggle Place Value grouping"` on the `<label>` and `aria-label="Place Value grouping"` added to the `<input>`.
  - The compact-landscape `@media (max-height: 500px)` rule that hid `.pv-toggle-label` was removed (no longer needed).

- Bead position/scale:
  - Bead diameter reduced from 40 px to 20 px — now a small metal knob rather than a large sphere.
  - Vertical positioning changed from `top: 50%; transform: translate(-50%, -50%)` to `bottom: 7px; transform: translateX(-50%)` — bead now sits on the bottom rail of the tray, clear of the cube/stick blocks above it.
  - Z-index raised from 60 to 1000 so the bead renders above the `tray::after` decorative strip (z-index 999).
  - Grip lines scaled down proportionally (8 px tall, 1.5 px wide, 4 px gaps).
  - Hover/dragging transforms updated to use `translateX(-50%) scale(…)` to match the new base transform.
  - JS drag logic and `positionBead()` formula unchanged — bead still leads from the first empty slot boundary.

- Numeral alignment fix:
  - Added `padding-left: var(--tray-pad); padding-right: var(--tray-pad)` to both `.number-line-top` and `.number-line-bottom`.
  - This matches the tray's internal horizontal padding so each `width: var(--slot-w)` label cell lines up with its corresponding slot cell. Previously the label rows had no padding offset, causing an off-by-trayPad shift between labels and slots.

- What was preserved:
  - Drag bead right to create units; drag left to remove units.
  - Place Value off: raw red unit cubes. Place Value on: groups of ten as blue ten-sticks with leftover red cubes.
  - Live quantity numeral with pop animation.
  - Horizontal panning/scrolling on mobile (touch-action: pan-x + axis-locked handler).
  - Mobile landscape compact layout.
  - `ensureBeadVisible()` auto-scroll behaviour.
  - Mouse-pan on empty track.
  - All zoom scaling via CSS custom property `--zoom`.

- What remains for later passes:
  - Grid/Array mode — not yet built.
  - Enameled cast-iron / brushed-metal material redesign — deferred.
  - Keyboard stepping of the bead (ArrowLeft / ArrowRight).
  - Reset affordance (button or double-tap bead).
  - Zoom slider label/tooltip.

- Risks / tradeoffs:
  - Document-level `touchmove` is `{ passive: false }` but calls `preventDefault()` only when `beadDragging` is true; other touches are unblocked immediately.
  - `renderBlocks()` is O(quantity) DOM work on every drag tick; 150 elements per frame is fine at current scale.
  - Single-file, zero-build, GitHub Pages deployable — no regressions on this constraint.
