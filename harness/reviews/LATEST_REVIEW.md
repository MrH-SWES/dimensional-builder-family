# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button in the simplified Number Line; first-time users have no way to clear the tray without refreshing the page.
- The zoom slider has no label or tooltip; its purpose is not immediately obvious.
- No ARIA roles, keyboard navigation, or screen-reader labels in either app. Accessibility is entirely absent.

Notes:
- Files changed: apps/number-line/index.html (mobile landscape scroll + portrait pan area fix), harness/reviews/LATEST_REVIEW.md (updated)

- Root cause of landscape vertical scroll lock:
  - `body` had `height: 100dvh; overflow: hidden` which hard-locked the viewport to exactly one screen height.
  - In landscape, the header (80 px) + workspace content + footer exceeded the available short viewport height, but the overflow was hidden rather than scrollable.
  - The user had no way to scroll the page to reach the number line.

- Root cause of limited portrait pan area:
  - `touch-action: pan-x` was set only on `.tray-scroller` and its children (`.tray-wrapper`, `.tray`, `.number-line-top`, `.number-line-bottom`, `.slot`).
  - Touches starting on the `.workspace` background (above or around the scroller) were not intercepted; the browser fell back to default (no horizontal pan, no vertical page scroll either because overflow was hidden on body).
  - Pan only worked when the initial touch landed directly on the track strip.

- How fixed:
  - CSS: Changed `body` from `height: 100dvh; overflow: hidden` to `min-height: 100dvh; overflow-y: auto; touch-action: pan-y`.
    - `min-height` lets the body grow beyond the viewport in landscape so the page becomes vertically scrollable.
    - `overflow-y: auto` enables native vertical page scroll when content exceeds viewport height.
    - `touch-action: pan-y` explicitly declares that vertical page scrolling is allowed (overrides any implicit restriction).
    - Desktop is unaffected: content fits within the viewport height, so body stays at exactly 100dvh and the workspace still vertically centers its children via `flex: 1; justify-content: center`.
  - CSS: Added `padding-bottom: max(24px, env(safe-area-inset-bottom))` to `footer` so the track is not obscured by the iOS home indicator / browser chrome in landscape.
  - JS: Added axis-locked workspace touch-pan handler (`touchstart`/`touchmove`/`touchend` on `.workspace`).
    - On `touchstart` (excluding `.cube`, `.ten-bar`, `.action-tray`, `input` targets): records start position and `scroller.scrollLeft`.
    - On `touchmove`: determines dominant axis after 5 px of movement.
      - Horizontal axis locked → calls `e.preventDefault()` and sets `scroller.scrollLeft` directly (pans the number line).
      - Vertical axis locked → does nothing; native page vertical scroll proceeds.
    - Registered with `{ passive: false }` on `touchmove` so `preventDefault()` is legal.
    - This extends horizontal panning to the full workspace area, not just the exact track strip.

- What was preserved:
  - Red ones (#DC143C), blue tens (#4169E1), inventory tray, large running total display.
  - Auto-bundle (10 ones → ten-bar) and break-apart (click ten-bar) interactions.
  - Mouse-based click-drag pan (unchanged; still works on desktop).
  - Zoom slider (0.3-1.2x) and all dynamic scaling.
  - Full brand design language: glass header, Math Things logo, Lexend font, footer bead.
  - Desktop layout remains polished and centered (min-height + flex: 1 preserve centering).
  - No algorithm/procedure tabs reintroduced.

- Remaining mobile limitations:
  - HTML Drag-and-Drop API (dragstart/drop) does not fire on touch; dragging a piece from the bank tray to the track via touch does not work. Clicking the dispenser buttons to add pieces works on mobile. Implementing native touch-drag-to-drop is a future enhancement.
  - Touching a block and swiping will pan horizontally (intended), but the block click (break-apart/bundle) will not fire if the touch moves significantly — this is correct native browser behaviour for distinguishing tap vs. scroll.
  - On some Android browsers, `env(safe-area-inset-bottom)` may not be respected unless the page declares `viewport-fit=cover` in the meta viewport tag (not added here to avoid other layout side effects).

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
