# Latest Review

Status: PASS

Must Fix:
- None

Should Fix Soon:
- No reset button in the simplified Number Line; first-time users have no way to clear the tray without refreshing the page.
- The zoom slider has no label or tooltip; its purpose is not immediately obvious.
- No ARIA roles, keyboard navigation, or screen-reader labels in either app. Accessibility is entirely absent.

Notes:
- Files changed: apps/number-line/index.html (simplified), apps/addition-subtraction-machine/index.html (new — full algorithm version), harness/reviews/LATEST_REVIEW.md (updated)

- What moved into Addition/Subtraction Machine (apps/addition-subtraction-machine/index.html):
  - The complete current two-tab app was copied verbatim (title updated to "Addition/Subtraction Machine | Math Things.")
  - Tab 1 (free-play icon): same free-play bank + number line
  - Tab 2 (+/− icon): equation bar (opInputA, opInputB, op toggle, refresh), guided bank (opGuidedBank with ones/tens dispensers, empty/staged/removed preview blocks), algorithm sidebar (carry row, two addend rows, operator, rule line, result row), all op-mode JS (buildOperation, updateGuidedBank, checkOpAddBundling, flyBlocks, triggerAutoZoom, animateZoom)
  - All borrowing/carrying visuals: borrow-cross, borrow-mini, anim-borrow-cue, anim-pulse-link, text-pulse-link, emptyTargetPulseOnes/Tens animations
  - Mode-switching event listeners on both nav tab buttons

- What remains in Number Line (apps/number-line/index.html):
  - Red unit cubes (--ones #DC143C) dispensed by click or drag from the 1s bank tray
  - Blue ten-bars (--tens #4169E1) dispensed by click or drag from the 10s bank tray
  - Auto-bundle: dropping/clicking to 10 ones automatically composes them into a ten-bar (magicSnap animation)
  - Break-apart: clicking a ten-bar on the track breaks it back into 10 individual cubes
  - Scrollable number line with alternating light/dark decade bands, top number labels (1–150), bottom unit labels (1–10 per decade), bold decade anchors
  - Large running total display (7rem, 900 weight) above the tray, animates on every change (totalPop)
  - Zoom slider (0.3–1.2×) that rescales all blocks, slots, labels dynamically via CSS custom properties
  - Click-drag pan on the tray scroller
  - Full brand design language: glass header, Math Things logo with chrome bead, Lexend font stack, --shadow-block token, footer bead

- What was intentionally removed from Number Line:
  - Nav tabs (two-mode switcher removed; app is now single-mode free-play)
  - Equation bar (#opEqBar): number inputs, +/− operator toggle, refresh button
  - Guided bank (#opGuidedBank): ones/tens dispensers with empty/staged/removed slot previews
  - Algorithm sidebar (#algSidebar): carry row, addend rows with tens/ones digit boxes, operator, rule line, result row, borrow-cross/borrow-mini overlays
  - All op-mode state: mode, currentOp, queueOnes, queueTens, stagedOnes, stagedTens, algStateA_Tens, algStateA_Ones
  - Functions: buildOperation(), updateGuidedBank(), checkOpAddBundling(), flyBlocks(), triggerAutoZoom(), animateZoom()
  - CSS: .tab, nav, .persistent-eq-bar, .eq-input, .op-toggle, .op-btn, .btn-refresh, .sidebar, .algorithm-problem, .alg-row, .digit-box, .t-dig, .o-dig, .alg-operator, .alg-line, .alg-result, .carry-row, .carry-box, .borrow-cross, .borrow-mini, .preview-cube-empty, .preview-ten-empty, .preview-cube-staged, .preview-ten-staged, .guided-dispense-area, .ones-grid, .tens-flex, .locked-strict, .anim-pulse-link, .text-pulse-link, .anim-borrow-cue
  - Animations: blockPulseLink, textPulseLink, borrowCuePulse, emptyTargetPulseOnes, emptyTargetPulseTens
  - hasAutoZoomed flag and auto-zoom-on-completion logic (was only triggered at end of op mode)

- Risks/tradeoffs:
  - Safe split strategy used: full app copied first into addition-subtraction-machine/index.html, then number-line/index.html was rewritten from scratch. No functionality deleted; only moved/separated.
  - The simplified Number Line has no reset button (the original had none in free-play mode either; reset was only shown after fuse in Array Builder v2). Adding a reset button is recommended but left as a future should-fix.
  - Drag-from-line behaviour (draggable=true is set on cubes/ten-bars) is preserved in both apps but drop targets on the bank tray side were not implemented in either version; blocks dragged off the line currently do nothing. This is pre-existing behaviour.
  - Both files are single-file, self-contained, and GitHub Pages deployable with no build step.
