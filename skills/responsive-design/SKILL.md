---
name: responsive-design
description: >
  Load this skill for any project with a user interface that must work across
  screen sizes and input methods — websites, web apps, dashboards, marketing
  pages, emails, or design systems. Use when building or reviewing layouts,
  choosing breakpoints, setting up fluid type, sizing touch targets, or making
  components adapt to their container. One design must serve phones, tablets,
  laptops, desktops, touch, mouse, and keyboard. Prefers content-first,
  mobile-first, framework-agnostic CSS and Baseline-stable features.
---

# Responsive Design

Apply these rules to every layout and interactive surface. Examples are
framework-agnostic CSS; they translate directly to any framework or utility
system. A responsive interface adapts to the viewport, the container, the input
method, and the user's own settings — not to a fixed list of devices.

## Core mandate

Design for content and constraints, not for named devices. A layout is
responsive when it stays usable and readable at any width from ~320px to
ultra-wide, in both orientations, at up to 400% zoom, under touch or pointer,
and while respecting the user's font-size, motion, and colour-scheme
preferences. Start from the smallest reasonable width and enhance upward
(mobile-first): this yields less CSS and a working baseline everywhere.

Never key layout decisions to device brands, operating systems, or product
names — those change and multiply. Let the content decide when the layout needs
to change.

## Severity scale

| Level | Meaning |
| --- | --- |
| **Critical** | Content is unreachable, unreadable, or unusable at a common size or input |
| **Serious** | Major friction — horizontal scroll, clipped content, unusable targets |
| **Moderate** | Noticeable friction with a workaround |
| **Minor** | Polish gap; marginal impact |

Judge severity by the real effect on the task at a real size and input, not by
the rule alone.

---

## Critical: Set the foundation

Every responsive page needs a correct viewport meta tag. Without it, mobile
browsers render at a fake ~980px width and shrink everything.

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

Never add `user-scalable=no`, `maximum-scale=1`, or `minimum-scale=1` — blocking
zoom is a **Critical** accessibility failure (WCAG 1.4.4 / 1.4.10). Users with
low vision must be able to pinch-zoom.

Use a border-box model and fluid media so nothing forces horizontal scroll:

```css
*, *::before, *::after { box-sizing: border-box; }

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
  height: auto;
}
```

Always keep the intrinsic `width` and `height` attributes on `<img>` so the
browser reserves space and avoids layout shift, even with `height: auto` in CSS.

For full-height regions, avoid `100vh` on mobile — it can exceed the visible
area under dynamic browser UI and clip content. Use `100dvh` (follows the
shrinking/growing UI) or `100svh` (the smallest, safest). `dvh`/`svh`/`lvh` are
Baseline: widely available.

---

## Responsive strategies

Pick per component; most real designs mix them.

- **Fluid** — sizes flex continuously with available space (percentages, `fr`,
  `minmax()`, `clamp()`). Fewest breakpoints, smoothest scaling.
- **Adaptive** — distinct layouts snap in at chosen breakpoints.
- **Mobile-first** — author the smallest layout as the default, add complexity
  with `min-width` queries. Preferred: less override CSS, progressive.
- **Content-first** — let the content's own needs (line length, minimum card
  width) dictate where breakpoints fall.

---

## Breakpoints are conventions, not standards

There is no official set of breakpoints. Popular frameworks disagree — proof
that these are conventions, not rules:

| Framework | Breakpoints (min-width) |
| --- | --- |
| Tailwind | 640 / 768 / 1024 / 1280 / 1536 |
| Bootstrap 5 | 576 / 768 / 992 / 1200 / 1400 |
| MUI | 600 / 900 / 1200 / 1536 |

Do not copy a table and stop thinking. Start small, widen the viewport until the
layout looks strained (too-long lines, awkward gaps), and add a breakpoint
*there*. Verify with browser DevTools device mode. Prefer `min-width` (mobile-
first):

```css
/* Mobile-first: default is the small layout */
.layout { display: flex; flex-direction: column; gap: 1rem; }

@media (min-width: 48rem) {        /* ~768px */
  .layout { flex-direction: row; }
}
```

Use `rem`-based breakpoints (`48rem`) so they scale with the user's font size.

---

## Modern layout tools

Reach for intrinsic CSS layout before media queries. Flexbox, Grid, and logical
properties handle most adaptation with no breakpoints at all.

```css
/* Flexbox: items flow and wrap as space allows */
.toolbar { display: flex; flex-wrap: wrap; gap: 1rem; }

/* Grid with fr units */
.two-col { display: grid; grid-template-columns: 1fr 3fr; gap: 2rem; }

/* Logical properties adapt to writing direction (RTL/vertical) */
.card { margin-inline: auto; padding-block: 1rem; max-inline-size: 65ch; }
```

Use **subgrid** to align nested items (e.g. every card's title, body, and footer
line up across a grid) — Baseline: widely available.

```css
.cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr)); gap: 1.5rem; }
.card  { display: grid; grid-template-rows: subgrid; grid-row: span 3; }
```

### Intrinsic, zero-breakpoint grids

`auto-fit` + `minmax()` builds a grid that reflows by itself — the column count
follows the available width with no media query:

```css
/* auto-fit: cards stretch to fill; collapses to 1 column when narrow */
.card-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(18rem, 1fr)); gap: 1.5rem; }

/* auto-fill: keeps empty tracks — use for fixed-size items like icons */
.icon-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(6rem, 1fr)); gap: 1rem; }
```

Prefer `auto-fit` for content cards that should grow; `auto-fill` when you want
a stable column rhythm (icon grids, swatches).

---

## Media queries vs container queries

Media queries respond to the **viewport**. Container queries respond to a
**component's own container**, so one component adapts correctly wherever it is
placed (main column, narrow sidebar, modal). Container size queries are
Baseline: widely available.

| | Media queries | Container queries |
| --- | --- | --- |
| Responds to | Viewport / device | Nearest query container |
| Best for | Page-level layout | Reusable components |
| Portability | Context-dependent | Truly portable |

```css
/* Establish a containment context on the parent */
.card-host { container-type: inline-size; container-name: card; }

/* Card restyles based on ITS container, not the screen */
@container card (min-width: 25rem) {
  .card { display: grid; grid-template-columns: 8rem 1fr; }
}
```

A container query silently does nothing if no ancestor declares `container-type`
— this is a common **Serious** bug. When sizing inside a container, use the
logical units `cqi` (inline) and `cqb` (block), not `cqw`/`cqh`, so they follow
writing direction and stay i18n-safe.

Rule of thumb: **media queries for the page skeleton, container queries for the
components inside it.**

---

## Layout patterns

The classic responsive layout patterns catalogued in Luke Wroblewski's
*Multi-Device Layout Patterns* (2012). Each is a direction to choose *before*
designing — there is no single "best" one. (The term "responsive web design"
itself was coined earlier by Ethan Marcotte.)

- **Mostly Fluid** — a fluid multi-column grid that adds outer margins on large
  screens and stacks to one column at the narrowest width. Structure barely
  changes until the smallest size.
- **Column Drop** — multi-column that drops columns one at a time as width
  shrinks, until everything is stacked. Element sizes stay roughly constant.
- **Layout Shifter** — the most adaptive: content genuinely rearranges between
  breakpoints (not just reflowed), using different grids per size.
- **Tiny Tweaks** — a single column with small adjustments (font size, padding,
  image size). Good for text-heavy pages.
- **Off-Canvas** — secondary content (nav, filters, context bar) lives off the
  viewport on small screens behind a toggle, and becomes persistently visible as
  width grows. Reserve for global chrome like sidebars.

Two component-level moves that compose with the above:

- **Reflow** — stack horizontally-arranged elements vertically as width drops.
- **Priority+** — show the highest-priority items inline and push the rest into
  an overflow menu ("more"), instead of wrapping or hiding arbitrarily.

Never hide content purely because a screen is small — screen size does not
predict what a user wants. Prefer reflow, disclosure, or overflow over deletion.

---

## Input-method adaptation

You cannot assume the user's input method — touch, mouse, trackpad, stylus, or
keyboard — and many devices offer several. Adapt to capability, not assumption.

### Target size (three standards, use the strictest your context requires)

| Standard | Minimum | Notes |
| --- | --- | --- |
| WCAG 2.2 SC 2.5.8 (AA) | **24×24 CSS px** | Or smaller **if** ≥24px spacing to neighbours; plus inline/essential exceptions |
| Apple HIG | **44×44 pt** | iOS, iPadOS, watchOS |
| Material 3 | **48×48 dp** touch | ≥8dp spacing; pointer targets ≥44dp |

WCAG 24px is a floor for compliance; 44–48px is the ergonomic target for primary
touch actions. Expand a small visual control's tappable area with padding rather
than shrinking the hit target:

```css
/* 24px icon, 44px tap area via padding */
.icon-button { inline-size: 24px; block-size: 24px; padding: 10px; }

/* Primary touch controls */
.button { min-block-size: 44px; padding: 0.75rem 1rem; }
```

### Adapt to pointer capability, not screen size

Use pointer/hover media features instead of assuming "small = touch":

```css
/* Coarse pointer (finger): give roomier targets */
@media (pointer: coarse) {
  .button { min-block-size: 48px; }
}

/* Only decorate with hover where hover truly exists */
@media (hover: hover) {
  .card:hover { box-shadow: 0 0 0 2px currentColor; }
}
```

Never gate essential information or actions behind `:hover` alone — touch and
keyboard users cannot hover. Treat swipe, long-press, and other gestures as
enhancements and always provide a visible button alternative.

### Thumb zones

On phones held one-handed, the bottom third of the screen is the easy-reach
zone; the top is hardest. Put primary navigation and actions within reach —
prefer a bottom tab bar over a top-only nav, and place a floating action button
bottom-right for the right-handed majority.

---

## Safe areas

On notched and rounded displays, pad interactive edges into the safe area so
controls clear the notch, home indicator, and landscape cutouts. `env()` and the
`safe-area-inset-*` variables are Baseline: widely available.

The cutout insets resolve to `0` unless the viewport meta opts into drawing
under the cutouts with `viewport-fit=cover`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

```css
.app-bar {
  padding-block-end: max(1rem, env(safe-area-inset-bottom));
}
.content {
  padding-inline: max(1rem, env(safe-area-inset-left)) max(1rem, env(safe-area-inset-right));
}
```

`max()` guarantees a minimum padding on devices that report zero insets.

---

## Responsive typography

Scale type fluidly with `clamp()`, but **always anchor to `rem`** so the user's
font-size preference is respected. Viewport-only sizing (`font-size: 4vw`) is a
**Serious** accessibility failure — it ignores user settings and can defeat
zoom.

```css
:root {
  --step-0: clamp(1rem,   0.9rem + 0.5vw, 1.25rem);   /* body */
  --step-2: clamp(1.5rem, 1rem   + 2vw,   2.5rem);    /* heading */
}
h2 { font-size: var(--step-2); }
p  { font-size: var(--step-0); max-inline-size: 65ch; } /* ~66 char lines */
```

Constrain line length (the *measure*) to roughly 45–75 characters (`ch`) for
readability; add a breakpoint or `max-inline-size` when lines grow past ~10
words. For component-scoped text, combine `clamp()` with `cqi` inside a
container.

---

## Responsive images

Serve appropriately sized images and, where the composition needs it, different
crops per breakpoint.

```html
<!-- Resolution switching: browser picks a file for the rendered size -->
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1600.jpg 1600w"
  sizes="(min-width: 48rem) 50vw, 100vw"
  width="1600" height="900" alt="…" />

<!-- Art direction: different crop/aspect per breakpoint -->
<picture>
  <source media="(min-width: 48rem)" srcset="hero-wide.jpg" />
  <img src="hero-square.jpg" width="800" height="800" alt="…" />
</picture>
```

Keep `width`/`height` (or `aspect-ratio`) to reserve space and prevent layout
shift. Provide meaningful `alt` text.

---

## User preferences

Beyond viewport and input, honour the user-preference media features
`prefers-reduced-motion` and `prefers-color-scheme`. These are accessibility and
theming concerns rather than layout, but a responsive UI should respect them
alongside its adaptive styles.

---

## Testing checklist

- [ ] Correct `<meta name="viewport">`; zoom is **not** disabled.
- [ ] No horizontal scroll from ~320px up to ultra-wide.
- [ ] Layout works in both portrait and landscape.
- [ ] Readable and operable at 200% and 400% zoom (WCAG 1.4.4 / 1.4.10).
- [ ] Type scales with `clamp()` + `rem`; measure stays ~45–75ch.
- [ ] Touch targets meet the required standard; adequate spacing.
- [ ] Nothing essential is hover-only; gestures have button fallbacks.
- [ ] Container-scoped components tested in narrow *and* wide containers.
- [ ] Images use `srcset`/`sizes` (or `<picture>`) and reserve space.
- [ ] Safe-area insets applied on edge-anchored controls.
- [ ] `prefers-reduced-motion` and `prefers-color-scheme` honoured.
- [ ] `100dvh`/`100svh` used instead of `100vh` for full-height regions.
- [ ] Verified on real devices, not just a resized desktop browser.
