# Dimensional Builder Doctrine

Applies to: `apps/number-line/`, `apps/array-builder/`, `apps/array-builder-v2/`

The Dimensional Builder family models quantity across one, two, and (eventually) three spatial dimensions. Each app in the family shares the same material system and interaction philosophy; they differ only in the dimension of the workspace.

## App-Specific Rules

### Number-Line (1D — Line / Grid Mode)

- **Line mode:** a single horizontal tray. Dragging the x-bead right adds unit cubes; left removes them.
- **Grid mode:** the same tray repeated in rows. A y-bead controls the row count. The record displays `x × y = total`.
- **Place Value toggle:** regroups existing quantity into tens (blue) + ones (red). Toggle is wordless — icon only.
- **Record:** live numeral (or expression) reflects current physical state. Font shrinks to fit longer expressions.
- **No reset button** (current milestone): users drag the bead back to 0.
- **MAX_ROWS = 20**, **MAX_X = 150** (as implemented).
- **Place Value / Grid fusion rules:** hundred-flats (10×10) take priority; after all complete 10×10 flats fuse, leftover complete runs of 10 fuse into blue ten-sticks in the direction that preserves the rectangular array structure. Partial columns inside the hundred-flat row-band become vertical ten-sticks; partial rows below the hundred-flat column-band get horizontal ten-sticks.

### Array Builder (2D)

- A rectangular array of unit cells.
- Width and height beads (or equivalent) control dimensions.
- Record shows `width × height = area`.
- Place Value on the array is a future consideration; not required at first milestone.

### Array Builder v2

- Successor to Array Builder; may include improved gesture handling or material redesign.
- Inherits all Array Builder rules until a separate doctrine is written.

## Dimensional Progression

| Mode / App | Dimensions | Object | Record format |
|---|---|---|---|
| Line mode | 1D | Unit cubes on a track | `n` |
| Grid mode | 2D | Rows of cubes / ten-bars | `x × y = total` |
| Solid mode (future) | 3D | Stacked layers | `x × y × z = total` |
| Array Builder | 2D | Rectangular array | `w × h = area` |

## Solid Mode (Future)

- A z-bead (depth) will extend Grid mode into 3D.
- Record will show `x × y × z = total`.
- Hundred-flat and thousand-cube fusion become relevant.
- Not to be implemented until Grid mode is considered complete.

## Interaction Constraints

- Beads snap per slot (integer quantities only).
- No fractional values anywhere in the Dimensional Builder family.
- Pinch-to-zoom and pan are always available on the workspace.
- Axis-locked touch drag: a drag that begins more horizontal than vertical is treated as an x-bead drag; more vertical than horizontal triggers y-bead or pan behavior.

## What Remains Before Solid Mode

1. Reset affordance (button or double-tap bead).
2. Keyboard stepping of x-bead (ArrowLeft / ArrowRight).
3. ~~Hundred-flat fusion across Grid rows.~~ (Implemented: vertical ten-sticks fill leftover partial columns inside the hundred-flat zone.)
4. Zoom-to-fit affordance for large grids on mobile.
5. Full enameled cast-iron / brushed-metal material redesign.
