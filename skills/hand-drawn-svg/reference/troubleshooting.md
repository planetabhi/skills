# Troubleshooting

The following table lists common symptoms, their causes, and the fix for each:

| Symptom | Cause | Fix |
| --- | --- | --- |
| `pigments` and `pigmentStrokes` are empty. | `fill` is missing, `'none'`, `'transparent'`, or empty, or `pigmentStyle` is `'none'`. | Set a `fill` color and a `pigmentStyle`. |
| `pigmentStrokes` holds an empty string. | The shape measures half a hatch gap or less across the hatch direction. | Lower `hatchGap`, or lower `pigmentWidth` or `strokeWidth` when `hatchGap` is `-1`. |
| Hatch lines are missing inside overlapping subpaths. | kalakar computes hatch lines with the even-odd rule. | Use `'solid'` and paint with `fill-rule="nonzero"`, or merge the subpaths in the source. |
| A solid fill covers a hole. | The painted `fill-rule` does not match the source. | Copy `fill-rule` from the source element to the painted fill. |
| A black outline appears around a shape that had none. | `stroke` defaults to `'#000'`. | Pass `stroke: 'none'`. |
| The output changes on every call. | `seed` is `0`, not set, or converts to `0`. | Set a non-zero integer seed. Refer to [determinism.md](./determinism.md). |
| Repeated shapes look identical. | The shapes share path data, style, and seed. | Give each element its own seed. |
| The effect looks far stronger on a small icon. | Offsets use path units. | Lower `maxJitter` or `looseness` for small `viewBox` sizes. |
| `maxJitter: 0` still moves curves. | Curves use an offset limit of `1` when `maxJitter` is `0`. | Set `looseness` to `0`. |
| Lines look heavy or doubled. | kalakar draws two passes by default. | Set `singlePass` for the outline or `pigmentSinglePass` for hatch lines. |
| Corners drift away from their positions. | Segment end points move by default. | Set `pinVertices` to `true`. |
| The output strings are large. | Coordinates keep full precision. | Set `fixedDecimals`, such as `2`. |
| `render` throws `Path data ended short` or `Param not a number`. | The path data is malformed. | Validate the source path, and wrap `render` in `try` and `catch`. |
| `render` throws a `TypeError`. | The path data contains an unsupported character. | Remove the character, or keep the original element when `render` throws. |
| The output sits in the wrong place. | `render` ignores `transform` attributes. | Keep the source `transform` on the output. |
| Every output changes after an unrelated change. | Code modified `DEFAULT_STYLE`. | Pass a style object instead. |
