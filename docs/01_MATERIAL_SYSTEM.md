# Material System

All visual elements in every Math Things app fall into one of two material categories. This system must be applied consistently across all tools in the family.

## Math Objects — Enameled Cast Iron

Think Le Creuset cookware applied to base-ten blocks.

**What qualifies:** unit cubes, ten-bars (ten-sticks), hundred-flats, thousand-cubes, array cells, beads, any object that represents a counted quantity.

**Visual properties:**
- Saturated, high-chroma color (not pastel, not neon).
- Slightly matte surface — not glossy plastic, not pure flat.
- Subtle inner shadow or rim highlight to suggest thickness and weight.
- Color encodes role:
  - **Red / warm** → ones / units
  - **Blue / cool** → tens
  - **Green** → hundreds (when implemented)
  - **Gold / amber** → thousands (when implemented)
- Motion feels weighty: snap is immediate, but a brief settle (bounce ≤ 80 ms) is acceptable to reinforce mass.

**What to avoid:** gradients that look like plastic toys, drop shadows that float the object off the surface, bright outlines that make objects look like stickers.

## Infrastructure — Brushed / Machined Metal

Think the tray, track, frame, and header as milled aluminum or steel.

**What qualifies:** trays, slots, track channels, app header bar, mode controls, toggle pills, zoom controls, any chrome that is not itself a counted object.

**Visual properties:**
- Desaturated neutral: light silver-gray (#D0D4D8 range) or dark gunmetal (#2A2D30 range).
- Surface texture implied through subtle linear gradients aligned to the axis of the element (horizontal for trays, vertical for columns).
- Slot dividers and seams are visible as machined grooves — thin, dark lines (1 px, ~30% opacity).
- Controls (toggles, pills) are recessed or flush — they do not float above the surface.

**What to avoid:** card-style white panels with box shadows, rounded pill buttons with colorful backgrounds, anything that looks like a mobile app UI kit.

## Interaction States

| State | Math Object | Infrastructure |
|---|---|---|
| Default | Full material color | Neutral metal |
| Hover (desktop) | Slight brightening of enamel rim | Subtle inset shadow |
| Active / dragging | Object lifts slightly (scale 1.03), shadow appears | Track groove darkens |
| Merging / splitting | Brief color pulse on merged unit | Seam briefly glows |

## Dark / Light Modes

Infrastructure inverts (light → dark metal); math object colors remain the same saturated enamel hues. The objects must remain visually dominant in both modes.

## Slot Backgrounds

Alternating slot backgrounds (light/dark) are a machined-surface affordance, not decoration. They help the eye count columns. Use a 2–3% luminance step between alternating slots.
