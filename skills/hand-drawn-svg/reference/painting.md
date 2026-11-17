# Painting

This file describes how to turn an `Artwork` into SVG markup. `render` returns geometry only. You choose the paint operation, color, and width for each field.

## Paint operations

| Field | Operation | Color | Width | Other attributes |
| --- | --- | --- | --- | --- |
| `pigments` | Fill, with `stroke="none"` or no stroke | The `fill` color | None | The source `fill-rule` and `fill-opacity` |
| `pigmentStrokes` | Stroke, with `fill="none"` | The `fill` color | The resolved hatch line width | The source `fill-opacity`, applied as `stroke-opacity` |
| `strokes` | Stroke, with `fill="none"` | The `stroke` color | `strokeWidth` | The source `stroke-opacity` |

The resolved hatch line width is `pigmentWidth` when it is greater than `0`. Otherwise it is `Math.max(strokeWidth / 2, 0.5)`. kalakar uses the same value to space the lines, so painting with it keeps the line width and gap in proportion.

## Paint order

Paint `pigments` and `pigmentStrokes` before `strokes`. SVG paints later elements on top, so this order places the outline over the fill. The `layers` array uses the same order.

## Build the markup

The following example converts an `Artwork` into an SVG fragment. It skips empty strings, escapes string attribute values, and converts widths to numbers, because values from user files can contain quotes or angle brackets:

```js
import { render } from 'kalakar';

const escape = (value) => String(value)
  .replace(/&/g, '&amp;').replace(/"/g, '&quot;').replace(/'/g, '&#39;')
  .replace(/</g, '&lt;').replace(/>/g, '&gt;');

function paint(art, { stroke = '#000', strokeWidth = 1, fill, fillRule, pigmentWidth = -1 }) {
  const outlineWidth = Number(strokeWidth);
  const hatchWidth = Number(pigmentWidth) > 0 ? Number(pigmentWidth) : Math.max(outlineWidth / 2, 0.5);
  const rule = fillRule ? ` fill-rule="${escape(fillRule)}"` : '';
  const out = [];
  for (const d of art.pigments.filter(Boolean)) {
    out.push(`<path d="${d}" fill="${escape(fill)}"${rule}/>`);
  }
  for (const d of art.pigmentStrokes.filter(Boolean)) {
    out.push(`<path d="${d}" fill="none" stroke="${escape(fill)}" stroke-width="${hatchWidth}"/>`);
  }
  for (const d of art.strokes.filter(Boolean)) {
    out.push(`<path d="${d}" fill="none" stroke="${escape(stroke)}" stroke-width="${outlineWidth}"/>`);
  }
  return out.join('');
}

const style = { seed: 7, fill: '#e11', pigmentStyle: 'solid' };
const markup = paint(render('M3 3 H21 V21 H3 Z', style), style);
```

The output path strings contain only JavaScript number strings, spaces, commas, and the letters `M` and `C`, so they need no escaping. Refer to [security.md](./security.md) before you insert markup built from user files into a page.

## Outline and fill together

`render` draws an outline whenever `stroke` is not `'none'`, and the default `stroke` is `'#000'`. When the source element has a fill and no stroke, pass `stroke: 'none'` so that kalakar returns no outline. Otherwise a black outline appears around a shape that had none.

## Fill rule

The solid fill keeps each subpath separate, so the `fill-rule` you paint `pigments` with decides which areas become holes. Copy the `fill-rule` from the source element.

Hatch lines ignore the painted `fill-rule`. kalakar computes them with the even-odd rule. Refer to [style-options.md](./style-options.md).

## Transforms

`render` does not read `transform` attributes. It returns coordinates in the same system as the input. Keep the `transform` attribute of the source element on the output, or wrap the output in a `g` element that carries it.
