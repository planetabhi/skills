# Integration

kalakar has no runtime dependencies and does not use the DOM. The same build runs in Node.js, in the browser main thread, and in Web Workers.

## Install the package

The skill describes the kalakar package, so the project must list `kalakar` as a dependency. Check the `dependencies` field of the project's `package.json` file before you write code that imports it.

When the package is missing, confirm with the user before you add it. Install it with the package manager that the project's lock file indicates:

| Lock file | Command |
| --- | --- |
| `package-lock.json` or none | `npm install kalakar` |
| `pnpm-lock.yaml` | `pnpm add kalakar` |
| `yarn.lock` | `yarn add kalakar` |
| `bun.lock` or `bun.lockb` | `bun add kalakar` |

The following example installs the package with npm:

```shell-session
$ npm install kalakar
```

A page without a bundler or a package manager can load the package from a content delivery network instead. Refer to [Content delivery network](#content-delivery-network).

## Module format

The package publishes ES modules only. Load it with `import`. The package marks itself as free of side effects, so bundlers can drop unused exports.

## Node.js script

The following example reads an icon path from a file, roughens it with a fixed seed, and writes the outline to a new SVG file:

```js
import { readFile, writeFile } from 'node:fs/promises';
import { render } from 'kalakar';

const d = (await readFile('icon.txt', 'utf8')).trim();
const art = render(d, { seed: 1 });
const paths = art.strokes.filter(Boolean)
  .map((s) => `<path d="${s}" fill="none" stroke="#000" stroke-width="1"/>`).join('');

await writeFile('icon.sketchy.svg', `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">${paths}</svg>`);
```

## Web Worker

`render` is synchronous. Large documents block the thread that calls it. Run `render` in a module Web Worker to keep a page responsive while settings change. For untrusted input, stop the worker with `worker.terminate()` when it exceeds a time limit. Refer to [security.md](./security.md).

The following example shows a module worker that roughens one path per message and returns errors as messages:

```js
import { render } from 'kalakar';

self.addEventListener('message', (event) => {
  const { id, d, style } = event.data;
  try {
    self.postMessage({ id, art: render(d, style) });
  } catch (err) {
    self.postMessage({ id, error: String(err?.message ?? err) });
  }
});
```

The following example starts the worker from the main thread:

```js
const worker = new Worker(new URL('./worker.js', import.meta.url), { type: 'module' });
```

## Content delivery network

The package includes its built files under `dist/`. A page without a bundler can import the build from jsDelivr. Replace `<version>` with an exact version so that the output does not change when a new version publishes:

```js
import { render } from 'https://cdn.jsdelivr.net/npm/kalakar@<version>/dist/index.js';
```

This import runs code from a third-party host in your page. For production, install the package and bundle it, or host the files yourself. Refer to [security.md](./security.md).
