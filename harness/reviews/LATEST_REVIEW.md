# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button in the simplified Number Line; first-time users have no way to clear the tray without refreshing the page.
- The zoom slider has no label or tooltip; its purpose is not immediately obvious.
- No ARIA roles, keyboard navigation, or screen-reader labels in either app. Accessibility is entirely absent.

Notes:
- Files changed: apps/number-line/index.html (mobile scroll fix), harness/reviews/LATEST_REVIEW.md (updated)

- Cause of mobile scrolling bug:
  - `.tray-scroller` had `overflow-x: auto` but no explicit `touch-action` property.
  - Without `touch-action: pan-x`, mobile browsers (especially iOS Safari) do not know to treat horizontal swipes as scroll gestures on that element.
  - Additionally, the `draggable="true"` HTML attribute on `.cube` and `.ten-bar` implicitly sets `touch-action: none` on those elements, which blocked pan gestures from propagating up to the scroller even when the user started a swipe on a block.

- How scrolling/panning was fixed:
  - Added `touch-action: pan-x` to `.tray-scroller` — the primary scroll container.
  - Added `-webkit-overflow-scrolling: touch` to `.tray-scroller` for legacy iOS Safari momentum scrolling.
  - Added `touch-action: pan-x` to `.tray-wrapper`, `.tray`, `.number-line-top`, `.number-line-bottom`, and `.slot` so that any touch originating on the track background pans the scroller.
  - Added `touch-action: pan-x manipulation` to `.cube` and `.ten-bar` so that a horizontal swipe starting on a block still pans the scroller (pan-x), while a tap still fires a fast click event (manipulation).
  - Increased `.tray-wrapper` end-padding from 60px to 80px so the last numbered mark scrolls fully into view on mobile.

- What was preserved:
  - Red ones (#DC143C), blue tens (#4169E1), inventory tray, large running total display.
  - Auto-bundle (10 ones -> ten-bar) and break-apart (click ten-bar) interactions.
  - Mouse-based click-drag pan (unchanged; still works on desktop).
  - Zoom slider (0.3-1.2x) and all dynamic scaling.
  - Full brand design language: glass header, Math Things logo, Lexend font, footer bead.
  - Desktop layout remains polished and centered.
  - No algorithm/procedure tabs reintroduced.

- Remaining mobile limitations:
  - HTML Drag-and-Drop API (dragstart/drop) does not fire on touch; dragging a piece from the bank tray to the track via touch does not work. Clicking the dispenser buttons to add pieces works on mobile. Implementing native touch-drag-to-drop is a future enhancement.
  - Touching a block and swiping will pan horizontally (intended), but the block click (break-apart/bundle) will not fire if the touch moves significantly -- this is the correct native browser behaviour for distinguishing tap vs. scroll.

- What moved into Addition/Subtraction Machine (apps/addition-subtraction-machine/index.html):
  - The complete prior two-tab app was copied verbatim (title updated to "Addition/Subtraction Machine | Math Things.")
  - Tab 1 (free-play icon): same free-play bank + number line
  - Tab 2 (+/- icon): equation bar (opInputA, opInputB, op toggle, refresh), guided bank (opGuidedBank with ones/tens dispensers, empty/staged/removed preview blocks), algorithm sidebar (carry row, two addend rows, tens/ones digit boxes, operator, rule line, result row), all op-mode JS (buildOperation, updateGuidedBank, checkOpAddBundling, flyBlocks, triggerAutoZoom, animateZoom)
  - All borrowing/carrying visuals: borrow-cross, borrow-mini, anim-borrow-cue, anim-pulse-link, text-pulse-link, emptyTargetPulseOnes/Tens animations
  - Mode-switching event listeners on both nav tab buttons

- What remains in Number Line (apps/number-line/index.html):
  - Red unit cubes (--ones #DC143C) dispensed by click or drag from the 1s bank tray
  - Blue ten-bars (--tens #4169E1) dispensed by click or drag from the 10s bank tray
  - Auto-bundle: dropping/clicking to 10 ones automatically composes them into a ten-bar (magicSnap animation)
  - Break-apart: clicking a ten-bar on the track breaks it back into 10 individual cubes
  - Scrollable number line with alternating light/dark decade bands, top number labels (1-150), bottom unit labels (1-10 per decade), bold decade anchors
  - Large running total display (7rem, 900 weight) above the tray, animates on every change (totalPop)
  - Zoom slider (0.3-1.2x) that rescales all blocks, slots, labels dynamically via CSS custom properties
  - Click-drag pan on the tray scroller (desktop); native touch pan (mobile)
  - Full brand design language: glass header, Math Things logo with chrome bead, Lexend font stack, --shadow-block token, footer bead

- What was intentionally removed from Number Line (prior session):
  - Nav tabs, equation bar, guided bank, algorithm sidebar, all op-mode state and functions

- Risks/tradeoffs:
  - Both files are single-file, self-contained, and GitHub Pages deployable with no build step.
  - The simplified Number Line has no reset button; adding one is a recommended future should-fix.
  - Drag-from-line behaviour (draggable=true) is preserved on desktop; touch drag-from-bank is not supported (pre-existing limitation).
