# Style options

This file lists every field of the `Style` object, its default, and its effect on the output. Every field is optional.

## Units

Jitter, gaps, and widths use the units of the path data, not screen pixels. kalakar adds offsets directly to the path coordinates, so the effect scales with the `viewBox`. On a 24 by 24 icon, the default `maxJitter` of `2` is about 8% of the icon width.

## Intensity

| Field | Type | Default | Effect |
| --- | --- | --- | --- |
| `looseness` | `number` | `1` | Multiplies every random offset. A value of `0` removes all offsets. |
| `waver` | `number` | `1` | Scales how far straight segments bow. It has no effect on curves. |
| `maxJitter` | `number` | `2` | Sets the largest random offset for points. |
| `seed` | `number` | `0` | Selects the random sequence. A value of `0` produces different output on every call. Refer to [determinism.md](./determinism.md). |

The following rules apply to these fields:

- With `looseness` set to `0`, the output follows the input geometry exactly, but straight segments become cubic curves. Unless `singlePass` is `true`, kalakar still draws two identical passes.
- The bow of a straight segment grows with `waver`, `maxJitter`, and the segment length. Setting `maxJitter` to `0` also removes the bow.
- A straight segment shorter than 10 times `maxJitter` uses one tenth of its length as the offset limit instead.
- Straight segments of 200 units or longer receive smaller offsets. The scale drops linearly from 0.9 at 200 units to 0.4 at 500 units and stays at 0.4 for longer segments.
- Curves use `maxJitter` as the offset limit, or `1` when `maxJitter` is `0`. Setting `maxJitter` to `0` straightens the bow of straight segments but still moves curve points. Set `looseness` to `0` to stop all movement.

## Outline

| Field | Type | Default | Effect |
| --- | --- | --- | --- |
| `stroke` | `string` | `'#000'` | A value of `'none'` skips the outline, and `strokes` is empty. kalakar does not use any other value. |
| `strokeWidth` | `number` | `1` | Does not change the outline geometry. kalakar uses it only to compute the automatic `pigmentWidth` and `hatchGap`. |
| `singlePass` | `boolean` | `false` | Draws each segment once instead of twice. |
| `pinVertices` | `boolean` | `false` | Keeps the end points of each segment exact. Curve control points still move. |

The default double pass draws the second line of each straight segment with half the offset of the first.

## Fill

| Field | Type | Default | Effect |
| --- | --- | --- | --- |
| `fill` | `string` | Not set | Turns on the fill when `pigmentStyle` is not `'none'`. The values `'none'`, `'transparent'`, and the empty string leave the fill off. kalakar does not use the color itself. |
| `pigmentStyle` | `'none'`, `'solid'`, `'hachure'`, or `'crosshatch'` | `'none'` | Selects the fill style. |
| `pigmentRoughnessGain` | `number` | `0.8` | Adds to `looseness` for the solid fill only. kalakar ignores it when `looseness` is `0`. |
| `hatchAngle` | `number` | `-41` | Sets the angle of the hatch lines in degrees. |
| `hatchGap` | `number` | `-1` | Sets the distance between hatch lines. A negative value uses the resolved hatch line width times 4. |
| `pigmentWidth` | `number` | `-1` | Sets the hatch line width used to compute the automatic `hatchGap`. A value of `0` or less uses `Math.max(strokeWidth / 2, 0.5)`. It does not change the hatch line geometry. |
| `pigmentSinglePass` | `boolean` | `false` | Draws each hatch line once instead of twice. |

### Solid fill

The `'solid'` style returns one continuous contour per subpath in `pigments`. kalakar always draws the solid fill with a single pass. It uses its own random offsets, so the fill edge does not line up exactly with the outline. kalakar keeps each subpath separate, so the `fill-rule` you paint with decides which areas become holes.

### Hachure and crosshatch

The `'hachure'` style returns parallel lines at `hatchAngle` in `pigmentStrokes`. The `'crosshatch'` style adds a second set at `hatchAngle` plus 90 degrees in the same string. Hatch lines use the same `looseness`, `waver`, `maxJitter`, and `pinVertices` values as the outline.

The following rules decide where hatch lines appear:

- kalakar samples each curve into 16 straight segments to find the shape edges.
- kalakar always uses the even-odd rule for hatch lines. Any area that even-odd treats as a hole gets no lines, whatever `fill-rule` you paint with.
- The first line sits half a gap in from the edge, and each following line sits one gap further. A shape that measures half a gap or less across the hatch direction gets no lines, and `pigmentStrokes` holds an empty string.
- kalakar raises any gap below 0.1 to 0.1. A `hatchGap` of `0` produces lines 0.1 units apart.

The following example computes the automatic gap for a stroke width of `1`:

```js
const strokeWidth = 1;
const pigmentWidth = Math.max(strokeWidth / 2, 0.5);
const hatchGap = pigmentWidth * 4;
```

The result is a gap of `2`, so a shape needs to measure more than 1 unit across the hatch direction to get a line.

## Output

| Field | Type | Default | Effect |
| --- | --- | --- | --- |
| `fixedDecimals` | `number` | Not set | Rounds output coordinates to this many decimal places and drops trailing zeros. A negative value has no effect. It rounds the strings only, not the marks in `layers`. |
