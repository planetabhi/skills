# Checklist

Run the following checks before you ship a change that uses kalakar. Mark a check as pending verification when it needs rendered output that you cannot inspect.

## Input

- [ ] The project lists `kalakar` in its `package.json` dependencies, or the page imports it from a pinned content delivery network URL.
- [ ] Each `render` call receives a path `d` string, not markup or an element.
- [ ] Shape elements are converted to path data. Refer to [whole-svg.md](./whole-svg.md).
- [ ] Each `render` call that reads user or third-party data sits inside `try` and `catch`.

## Style

- [ ] Every option name exists in the `Style` type. Refer to [style-options.md](./style-options.md).
- [ ] `fill` and `pigmentStyle` are set together, or neither is set.
- [ ] `stroke` is `'none'` for elements that have no outline in the source.
- [ ] `maxJitter`, `hatchGap`, and `pigmentWidth` suit the `viewBox` size.
- [ ] No code modifies `DEFAULT_STYLE`.

## Determinism

- [ ] Output that renders more than once uses a non-zero integer seed.
- [ ] Repeated elements use different seeds.

## Painting

- [ ] `strokes` are stroked with the `stroke` color and `strokeWidth`.
- [ ] `pigments` are filled with the `fill` color and the source `fill-rule`.
- [ ] `pigmentStrokes` are stroked with the `fill` color and the resolved hatch line width.
- [ ] Fills are painted before outlines.
- [ ] Empty strings are skipped.
- [ ] Color values from user files are escaped before they go into markup.
- [ ] Width values are converted with `Number()` before they go into markup.
- [ ] Source attributes such as `class`, `id`, `style`, and `transform` are kept.

## Security

- [ ] SVG from users or third-party files is sanitized before it reaches a page, or the result displays through an `img` element.
- [ ] Path data length, element count, `hatchGap`, `pigmentWidth`, and `fixedDecimals` are limited for untrusted input. Refer to [security.md](./security.md).
- [ ] `render` for untrusted input runs in a Web Worker or a separate process with a time limit.
- [ ] Server responses do not include raw `render` error messages.
- [ ] Production pages do not load kalakar from an unpinned content delivery network URL.

## Output

- [ ] The rendered result matches the intended look at the target size. Pending verification without rendered output.
- [ ] Output size is acceptable, with `fixedDecimals` set when needed.
