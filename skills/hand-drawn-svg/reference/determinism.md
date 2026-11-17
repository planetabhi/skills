# Determinism

kalakar draws every offset from a random source. The `seed` option decides whether that source repeats.

## Seed rules

| `seed` value | Result |
| --- | --- |
| `0` or not set | `render` uses `Math.random`, and each call returns different output. |
| A non-zero integer from -2,147,483,648 to 2,147,483,647 | `render` returns the same output for the same path data and style on every call. |
| A number outside that range or with a fraction | kalakar converts it to a 32-bit integer. A value that converts to `0`, such as `0.5` or `2 ** 32`, produces different output on each call. |

Use a non-zero integer from the table. `randomSeed()` returns an integer from `0` up to, but not including, `2 ** 31`, so it can return `0`.

## Pin the seed

Set a fixed seed for any output that renders more than once. The following cases need a fixed seed:

- Server rendering, where the server and the browser must produce the same markup.
- Snapshot tests, where the output must match a stored copy.
- Interfaces that call `render` again after unrelated changes, where the drawing must not change.

## Vary the seed per element

The same path data, style, and seed always produce the same output. Repeated shapes with one shared seed look identical. Derive a different seed for each element, such as a base seed plus the element index.

## Fill and outline independence

kalakar draws the fill from a separate random source with the same seed. Adding, removing, or changing the fill does not change the outline for a given seed.

## Do not modify DEFAULT_STYLE

`resolveStyle` copies `DEFAULT_STYLE` on every call. A change to `DEFAULT_STYLE` changes the output of every later `render` call in the same module instance. Pass a style object to `render` instead.
