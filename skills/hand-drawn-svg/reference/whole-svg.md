# Whole SVG documents

`render` roughens one path `d` string. To roughen a full SVG document, convert each shape element to path data, call `render` once per element, and replace the element with the painted result.

## Convert shape elements

The following table converts each SVG shape element to path data. Read missing numeric attributes as `0`.

| Element | Path data | Skip when |
| --- | --- | --- |
| `path` | The `d` attribute | `d` is missing or empty |
| `line` | `M<x1> <y1> L<x2> <y2>` | Never |
| `polyline` | `M` to the first point, then `L` to each following point | `points` holds fewer than two points |
| `polygon` | The `polyline` path followed by `Z` | `points` holds fewer than two points |
| `rect` | `M<x> <y> L<x+width> <y> L<x+width> <y+height> L<x> <y+height> Z` | `width` or `height` is `0` or less |
| `circle` | `M<cx-r> <cy> A<r> <r> 0 1 0 <cx+r> <cy> A<r> <r> 0 1 0 <cx-r> <cy> Z` | `r` is `0` or less |
| `ellipse` | `M<cx-rx> <cy> A<rx> <ry> 0 1 0 <cx+rx> <cy> A<rx> <ry> 0 1 0 <cx-rx> <cy> Z` | `rx` or `ry` is `0` or less |

The `rect` conversion in the table ignores `rx` and `ry`, so rounded corners become square. Add arc commands at each corner to keep them.

The `circle` and `ellipse` conversions use two half arcs, because kalakar drops an arc that starts and ends at the same point. Refer to [api.md](./api.md).

kalakar does not roughen other elements, such as `text`, `image`, and `use`. Leave them unchanged.

## Keep the source styling

Replace geometry attributes and keep every other attribute. The following attributes hold geometry: `d`, `points`, `x`, `y`, `x1`, `y1`, `x2`, `y2`, `width`, `height`, `cx`, `cy`, `r`, `rx`, and `ry`.

Keep attributes such as `class`, `id`, `style`, and `transform` on the output, so that CSS rules and transforms still apply. `render` does not read `transform`. Refer to [painting.md](./painting.md).

Keeping attributes also keeps event handlers, scripts, and links in an untrusted document. Escape every attribute value you write back, and sanitize documents from users or third-party files. Refer to [security.md](./security.md).

## Map element attributes to the style

Build the style for each element from its own attributes:

- Set `strokeWidth` from the element's `stroke-width`. kalakar uses it to compute the automatic hatch spacing.
- Set `fill` from the element's `fill` when it is not `none`.
- Set `stroke` to `'none'` when the element has a fill and no stroke. Otherwise kalakar returns an outline in the default black.
- Copy `fill-rule` and `fill-opacity` to the painted fill. Refer to [painting.md](./painting.md).

## Handle errors per element

`render` throws on malformed path data. Wrap each call in `try` and `catch`, and keep the original element when the call throws. One bad element then leaves the rest of the document intact. Refer to [api.md](./api.md) for the error cases.

## Seed each element

Shapes with the same path data, style, and seed produce the same output. Give each element its own seed so that repeated shapes do not look identical. Refer to [determinism.md](./determinism.md).
