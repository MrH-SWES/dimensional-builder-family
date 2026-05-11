# Math Things Design Doctrine

## Core Principles

### No Words, No Directions
Student-facing UI carries zero instructional text. Labels, tooltips, instructions, and explanatory copy do not exist on screen. The teacher teaches; the tool provides structure. Controls must be self-evident from their physical form and behavior.

### Teacher Teaches — Tool Provides Structure
The app is not a tutor. It does not scaffold, hint, correct, or narrate. It is a physical object the teacher picks up and uses during instruction. The teacher provides all verbal and conceptual context; the tool responds faithfully to manipulation.

### Equation Is the Shadow of the Object
Numerical output (the record) is a consequence of the physical state — not the goal. The student builds; the number appears as a side effect. Equations reflect what was constructed; they are never the primary focus.

### Student-Facing UI: Quiet and Physical
Every interactive element should feel like a physical object — something you push, pull, slide, or stack. Animations reinforce physics (snap, settle, merge). No decorative graphics, no celebratory feedback, no gamification.

## Material Identity

All Math Things share a single material language. See `docs/01_MATERIAL_SYSTEM.md` for full specification.

**Math objects** (blocks, bars, flats, cubes, beads): enameled cast iron, like Le Creuset base-ten blocks. Saturated, slightly matte color. Dense, tactile, weighty in motion.

**Infrastructure** (trays, tracks, frames, headers, controls): brushed or machined metal. Neutral, receding, structural.

## Interaction Rules

- Drag to build; release to commit.
- Snap is crisp and immediate — no easing on final position.
- Merge/split animations are brief (≤ 120 ms) and communicate physical regrouping, not magic.
- Pinch-to-zoom and pan are always available on the workspace.
- Place Value is a toggle that regroups existing quantity — it never changes count.

## What Math Things Are Not

- Not a game.
- Not self-guided.
- Not adaptive.
- Not connected to accounts, progress, or external services.
- Not responsible for teaching anything on its own.
