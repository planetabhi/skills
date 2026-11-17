---
name: hand-drawn-svg
description: >
  Load this skill when turning SVG icons, illustrations, or path data into a hand-drawn, sketchy look with the kalakar package. Triggers on "kalakar", "hand-drawn", "sketchy", "rough SVG", "make this icon look drawn", "hachure", and "crosshatch". Covers the `render` function, every `Style` option, painting the returned `Artwork`, deterministic seeds, and roughening whole SVG documents without a DOM.
---

# Hand-drawn SVG

kalakar converts one SVG path `d` string into hand-drawn path `d` strings. The `render` function is pure, has zero runtime dependencies, and runs in Node.js and the browser without a DOM. The detailed rules are bundled under `./reference/`. Read the relevant file before you apply or cite a rule.

## Requirements

- The project lists `kalakar` in the `dependencies` field of its `package.json` file. When the package is missing, confirm with the user and install it with the project's package manager, such as `npm install kalakar`. Refer to [reference/integration.md](./reference/integration.md).
- The code that loads kalakar uses `import`, because kalakar publishes no CommonJS build.

## Core mandate

Pass `render` a path `d` string, never markup or an element. `render` returns geometry only, so you apply every color and width when you paint the result. Paint each `Artwork` field with the correct operation. Set a non-zero integer `seed` for any output that renders more than once. Keep the source element's own styling.

## Confirmation

When you are only reviewing, report findings without pausing. Before applying edits to existing files, confirm the intended change set first, and ask for any context you need to judge correctly instead of guessing.

## Decision sequence

Work every conversion through these steps in order:

1. Check the package. Confirm that the project depends on `kalakar` before you import it. Refer to [Requirements](#requirements).
1. Get a path `d` string. Convert `rect`, `circle`, `ellipse`, `line`, `polyline`, and `polygon` elements to path data first. Refer to [reference/whole-svg.md](./reference/whole-svg.md).
1. Set the intensity. Start from the defaults and change `looseness` first. Jitter and gaps use path units, so judge them against the `viewBox` size. Refer to [reference/style-options.md](./reference/style-options.md).
1. Choose the fill. Set a `fill` color and a `pigmentStyle` together, because kalakar skips the fill when either one is missing. Refer to [reference/style-options.md](./reference/style-options.md).
1. Pin the seed. A `seed` of `0` produces different output on every call. Refer to [reference/determinism.md](./reference/determinism.md).
1. Paint the result. Refer to [Paint the artwork](#paint-the-artwork) and [reference/painting.md](./reference/painting.md).
1. Integrate. Refer to [reference/integration.md](./reference/integration.md).
1. Secure untrusted input. Sanitize SVG from users or third-party files, escape values you write into markup, and limit the work `render` does. Refer to [reference/security.md](./reference/security.md).
1. Verify. Run the checklist. Code inspection cannot confirm how the output looks, so mark visual checks as pending verification when no rendered output is available. Refer to [reference/checklist.md](./reference/checklist.md).

## Paint the artwork

`render` returns an `Artwork` object. Each of the following arrays holds zero or one path string, and the string can be empty. Skip empty strings when you paint.

| Field | Operation | Color | Width |
| --- | --- | --- | --- |
| `strokes` | Stroke with `fill="none"` | The `stroke` color | `strokeWidth` |
| `pigments` | Fill | The `fill` color, with the source `fill-rule` | None |
| `pigmentStrokes` | Stroke with `fill="none"` | The `fill` color | `pigmentWidth`, or `Math.max(strokeWidth / 2, 0.5)` when `pigmentWidth` is `-1` |

Paint `pigments` and `pigmentStrokes` first and `strokes` last so the outline sits on top of the fill. The `layers` field holds the same data in this order.

## Quick start

The following example roughens a 24 by 24 square with a red hachure fill and builds SVG markup from the result:

```js
import { render } from 'kalakar';

const style = { seed: 42, stroke: '#000', strokeWidth: 1, fill: '#e11', pigmentStyle: 'hachure', pigmentWidth: 0.5 };
const art = render('M3 3 H21 V21 H3 Z', style);

const hatch = art.pigmentStrokes.filter(Boolean)
  .map((d) => `<path d="${d}" fill="none" stroke="#e11" stroke-width="0.5"/>`);
const outline = art.strokes.filter(Boolean)
  .map((d) => `<path d="${d}" fill="none" stroke="#000" stroke-width="1"/>`);

const svg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">${[...hatch, ...outline].join('')}</svg>`;
```

The example writes fixed colors and widths into the markup. Escape any value that comes from users or source files. Refer to [reference/painting.md](./reference/painting.md) and [reference/security.md](./reference/security.md).

## Reference map

Read the relevant file before you apply or cite a rule.

| File | Covers |
| --- | --- |
| [reference/api.md](./reference/api.md) | Entry points, exports, `render` behavior, supported path commands, and errors |
| [reference/style-options.md](./reference/style-options.md) | Every `Style` field with its default and its effect on the output |
| [reference/painting.md](./reference/painting.md) | Turning an `Artwork` into SVG markup |
| [reference/whole-svg.md](./reference/whole-svg.md) | Converting shape elements to path data and roughening a full document |
| [reference/determinism.md](./reference/determinism.md) | Seeds, repeatable output, and shared defaults |
| [reference/integration.md](./reference/integration.md) | Installing the package, module format, Node.js scripts, browsers, and Web Workers |
| [reference/security.md](./reference/security.md) | Untrusted SVG, escaping, resource limits, and package sources |
| [reference/troubleshooting.md](./reference/troubleshooting.md) | Symptoms, causes, and fixes |
| [reference/checklist.md](./reference/checklist.md) | Checks to run before you ship |

## Anti-patterns

- Passing SVG markup or an element to `render`.
- Filling `pigmentStrokes` or stroking `pigments`.
- Setting `pigmentStyle` without a `fill` color, or a `fill` color without a `pigmentStyle`.
- Leaving `seed` at `0` for output that renders more than once.
- Mutating `DEFAULT_STYLE` instead of passing a style object.
- Passing roughjs option names such as `roughness`, `bowing`, `fillStyle`, `hachureGap`, or `disableMultiStroke`. kalakar ignores them.
- Expecting `render` to apply colors, widths, or `transform` attributes.
- Inserting converted SVG from users or third-party files into a page without sanitizing it.
- Writing colors, widths, or other attribute values into markup without escaping them.

---

Authored by @planetabhi
