# API

This file describes the public exports of kalakar and how `render` treats its input.

## Entry points

kalakar publishes ES modules only, with no CommonJS build. The `kalakar` and `kalakar/core` entry points export the same members:

| Export | Kind | Description |
| --- | --- | --- |
| `render` | Function | Roughens one path `d` string and returns an `Artwork`. |
| `resolveStyle` | Function | Merges a `Style` over `DEFAULT_STYLE` and returns a new `ResolvedStyle`. |
| `DEFAULT_STYLE` | Object | Holds the default value of every option. |
| `marksToPath` | Function | Serializes a `Layer` into a path `d` string. |
| `Hand` | Class | Provides the seeded random source that `render` uses. |
| `randomSeed` | Function | Returns a random integer from `0` up to, but not including, `2 ** 31`. |
| `Style` | Type | Describes the options you pass to `render`. Every field is optional. |
| `ResolvedStyle` | Type | Describes a style with every default filled in. |
| `PigmentStyle` | Type | Lists the fill styles `'none'`, `'solid'`, `'hachure'`, and `'crosshatch'`. |
| `Artwork` | Type | Describes the return value of `render`. |
| `Layer` | Type | Describes one group of marks with a role. |
| `Mark` | Type | Describes one pen instruction. |
| `Point` | Type | Describes an `[x, y]` pair. |

## Render a path

The following example passes path data and a style to `render`:

```js
import { render } from 'kalakar';

const art = render('M3 3 H21 V21 H3 Z', { seed: 42 });
```

`render` has the following behavior:

- It accepts an optional style. Omitted fields take their values from `DEFAULT_STYLE`.
- It copies the style before use and does not modify the object you pass.
- It returns an `Artwork` with empty arrays when the path string is empty.
- It returns coordinates in the same coordinate system as the input. It does not read or apply `transform` attributes.
- It returns geometry only. The output strings contain no colors or widths.

## Artwork

The `Artwork` object has the following fields:

| Field | Type | Contents |
| --- | --- | --- |
| `strokes` | `string[]` | The outline. Empty when `stroke` is `'none'`. |
| `pigments` | `string[]` | The solid fill. Empty unless `pigmentStyle` is `'solid'` and a `fill` color is set. |
| `pigmentStrokes` | `string[]` | The hachure or crosshatch lines. Empty unless `pigmentStyle` is `'hachure'` or `'crosshatch'` and a `fill` color is set. |
| `layers` | `Layer[]` | The same data as structured marks, ordered fill first and outline last. |

Each string array holds zero or one string. `pigmentStrokes` holds an empty string when the shape is too small for any hatch line. Refer to [style-options.md](./style-options.md) for the size rule.

Each output string uses only the `M` and `C` commands, in the following format:

```text
M0.12 -0.4 C6.9 0.3, 13.1 -0.2, 20.3 0.1
```

## Layers and marks

A `Layer` has a `role` and a `marks` array. The following table maps each role to its `Artwork` field:

| `role` | `Artwork` field |
| --- | --- |
| `'stroke'` | `strokes` |
| `'pigment'` | `pigments` |
| `'pigment-sketch'` | `pigmentStrokes` |

A `Mark` has a `verb` and a `coords` array. The `Mark` type allows the verbs `'move'`, `'curve'`, and `'line'`, but `render` only produces `'move'` and `'curve'`. A `'move'` mark holds two numbers and a `'curve'` mark holds six numbers.

The `fixedDecimals` option rounds the strings only. Marks in `layers` keep full precision.

## Serialize a layer

`marksToPath(layer, fixedDecimals)` returns the path `d` string for one layer. When `fixedDecimals` is a number of `0` or more, it rounds each coordinate to that many decimal places and drops trailing zeros. It ignores a negative value.

## Supported path data

`render` accepts the commands `M`, `L`, `H`, `V`, `C`, `S`, `Q`, `T`, `A`, and `Z` in absolute and relative form. Commas, spaces, tabs, and line breaks separate values.

Before roughening, kalakar converts the path in the following ways:

- It converts relative commands to absolute commands.
- It converts `H` and `V` to straight segments.
- It converts `S`, `Q`, and `T` to cubic curves.
- It converts each `A` arc to one or more cubic curves.
- It draws an arc with a zero radius as a straight segment.
- It drops an arc with non-zero radii that starts and ends at the same point.
- It inserts `M0,0` when the path does not start with `M` or `m`.

## Errors

`render` throws on malformed path data. The following table lists the cases:

| Input | Example | Result |
| --- | --- | --- |
| A command has too few numbers | `M0 0 L10` | Throws an `Error` with the message `Path data ended short`. |
| A command letter appears where a number belongs | `M0 0 L10 Z` | Throws an `Error` whose message starts with `Param not a number`. |
| The path contains an unsupported character | `M0 0 X5` | Throws a `TypeError`. |
| The path data is not a string, `null`, or `undefined` | `42` | Throws a `TypeError`. |
| `fixedDecimals` is above 100 | `{ fixedDecimals: 101 }` | Throws a `RangeError`. |

Wrap `render` in `try` and `catch` when the path data or style comes from users or third-party files. Refer to [security.md](./security.md).

## Resolve a style

`resolveStyle(style)` returns a new object that merges `style` over `DEFAULT_STYLE`. Omitting `style` returns a copy of the defaults. Refer to [determinism.md](./determinism.md) for why you must not modify `DEFAULT_STYLE`.

## Hand and randomSeed

`new Hand(seed).next()` returns a number from `0` up to, but not including, `1`. A valid seed produces the same sequence every time. A seed of `0` uses `Math.random`. `randomSeed()` can return `0`. Refer to [determinism.md](./determinism.md) for the seed rules.
