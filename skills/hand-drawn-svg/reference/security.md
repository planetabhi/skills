# Security

This file describes the risks of roughening SVG from users or third-party files, and how to contain them.

## Treat untrusted SVG as active content

SVG documents can contain `script` elements, event handler attributes such as `onload`, `foreignObject` elements that hold HTML, and links with `javascript:` URLs. A whole-document conversion keeps every attribute that does not hold geometry, so these survive into the output. Refer to [whole-svg.md](./whole-svg.md).

Before you insert the output into a page with `innerHTML`, or serve it from your own origin, apply one of the following protections:

- Sanitize the source document with a maintained SVG sanitizer before you convert it.
- Display the result through an `img` element. Browsers do not run scripts in SVG loaded this way.

When you parse SVG in Node.js, use an XML parser that does not resolve external entities.

## Escape values you write into markup

The path strings that `render` returns contain only JavaScript number strings, spaces, commas, and the letters `M` and `C`. They need no escaping.

Colors, widths, IDs, class names, and other attribute values from source files or users can contain quotes or angle brackets. Escape the characters `&`, `<`, `>`, `"`, and `'` in every such value, and convert numeric options with `Number()` before you write them. Refer to [painting.md](./painting.md) for an example.

## Limit work for untrusted input

`render` runs synchronously and has no time limit. The following inputs control how much work it does:

- The length of the path data. Cap the length you accept, and cap the number of elements in a document.
- The hatch gap. The number of hatch lines grows with the shape size divided by the gap, and kalakar accepts gaps down to 0.1 units. A shape 10,000 units across with a `hatchGap` of `0` produces about 100,000 lines in each direction. Set a minimum for `hatchGap` and `pigmentWidth` when users choose them.
- The `fixedDecimals` value. A value above 100 makes `render` throw a `RangeError`. Accept integers from 0 to 100 only.

Run `render` for untrusted input in a Web Worker or a separate process. Stop the worker with `worker.terminate()` when it exceeds your time limit.

## Validate the input type

Pass a string as the path data. `null`, `undefined`, and the empty string return an empty `Artwork`. Other types, such as numbers, make `render` throw a `TypeError`.

## Keep error details on the server

Some error messages include part of the path data, such as `Param not a number: L,Z`. On a server, log the error and return a generic message to the client.

## Do not use seeds for secrets

`Hand` and `randomSeed` do not produce cryptographically secure values. A seed of `0` uses `Math.random`. Do not use them for tokens, IDs, or any value that must be hard to guess.

## Load the package from a trusted source

- Install the package by its exact name, `kalakar`, and check the spelling before you run the install command.
- A content delivery network import runs code from a third-party host in your page. Pin an exact version. For production, install the package and bundle it, or host the files yourself. Refer to [integration.md](./integration.md).
