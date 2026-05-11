# Do Not Violate

These are hard constraints for every agent working in this repository. Violations will be rejected regardless of how well-reasoned the rationale appears.

---

## 1. Do Not Add Words to Student-Facing UI

No instructional text, labels, tooltips, help text, placeholder copy, or any English (or other language) string visible to the student during normal use. The only permitted text is the live numeric record.

**Why:** the doctrine is "no words, no directions." The teacher provides all verbal context.

---

## 2. Do Not Change App Behavior in a Docs Task

If the task description says "docs-only," "no behavior changes," or "do not edit index.html," obey literally. Do not refactor, fix, or improve application code during a documentation task, even if you notice something worth fixing.

---

## 3. Do Not Gamify

No stars, badges, confetti, scores, streaks, progress bars, or celebratory animations. The tool is silent about correctness. It does not reward or penalize.

---

## 4. Do Not Add External Dependencies

Every app in this family must be a single self-contained HTML file. No npm, no bundler, no CDN imports, no external fonts, no analytics, no tracking, no service workers unless explicitly approved. Zero-build, GitHub Pages deployable.

---

## 5. Do Not Break the Material System

New visual elements must use the enameled cast-iron palette for math objects and brushed/machined metal for infrastructure. Do not introduce:
- Gradients that look like plastic toys.
- Card-style white panels with box shadows.
- Colorful UI-kit buttons unrelated to the material vocabulary.

See `docs/01_MATERIAL_SYSTEM.md`.

---

## 6. Do Not Add Adaptive or Personalized Behavior

The tool does not track the student, adapt difficulty, log answers, or connect to any external service. Every session starts fresh.

---

## 7. Do Not Remove or Reorder the Existing Tray / Track Structure

The tray and track are load-bearing UI. Changing their DOM structure, CSS class names, or layout model without a full audit of all dependent JS will break beads, snap logic, and zoom behavior.

---

## 8. Do Not Skip the Harness Review

After any non-trivial change, update `harness/reviews/LATEST_REVIEW.md` with a clear pass/fail, must-fix list, should-fix-soon list, and notes on what changed. Future agents read this file first.

---

## Escalate, Do Not Guess

If a task conflicts with any rule above, stop and surface the conflict. Do not rationalize a workaround. Ask the human for clarification.
