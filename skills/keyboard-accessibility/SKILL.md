---
name: keyboard-accessibility
description: >
  Load this skill for any project with interactive UI — buttons, links, forms,
  modals, dropdowns, menus, tabs, sliders, toolbars, carousels, or custom
  widgets — in React or vanilla JavaScript. Never ship an interactive element
  that cannot be fully operated by keyboard alone. Always ensure a visible
  focus indicator, logical tab order, correct widget key behaviour, and no
  keyboard traps. Targets WCAG 2.2 Level AA.
---

# Keyboard Accessibility

Apply these rules to every interactive element and feature. Examples are
React-first; vanilla JavaScript is shown only where it differs meaningfully.

## Core mandate

All interactive functionality must be fully usable with a keyboard alone — no
mouse or touch required, except where the function genuinely depends on
path-based movement (e.g. freehand drawing). Keyboard access is not the same as
"can Tab to it": native and composite widgets use different key conventions,
and users must be able to reach, operate, and leave every component.

## Severity scale

| Level | Meaning |
| --- | --- |
| **Critical** | Blocks task completion entirely for keyboard/AT users |
| **Serious** | Significantly impairs keyboard access; workaround unreasonable |
| **Moderate** | Creates friction; a workaround exists |
| **Minor** | Best-practice gap; marginal impact |

Judge severity by the actual effect on the task, not the rule alone: can the
user reach, operate, and leave the component; is an essential task blocked; is
focus lost or placed misleadingly.

---

## Critical: Use native, keyboard-reachable elements

Every element activatable by mouse must be reachable and operable by keyboard.
Prefer native elements — they provide focusability, keyboard behaviour,
semantics, states, and browser integration for free.

```jsx
// Good — built-in keyboard support, focus, Enter/Space activation
<button type="button" onClick={save}>Save</button>
<a href="/about">About</a>

// Avoid — a div needs full ARIA + JS to merely match native behaviour
<div role="button" tabIndex={0} onClick={save}>Save</div>
```

Adding `role="button"` and `tabindex="0"` does **not** add button behaviour.
A custom control must also implement `Enter`/`Space` activation, disabled
state, focus styling, and the expected accessible name. Do not make mouse or
touch handlers the only way to operate a control. Test the actual result, not
merely that a handler exists.

---

## Critical: No keyboard trap (WCAG 2.1.2)

If focus can enter a component, the user must be able to move it away with
`Tab`, `Shift+Tab`, arrow keys, or `Escape` (per the component). A trap with no
exit is **Critical**. Restricting focus temporarily is *not* a trap when it is
a modal dialog with a standard, documented exit (see below).

Test embedded editors, third-party widgets, iframes, media players, and canvas
apps carefully — they commonly consume `Tab` and need a documented exit.

---

## Critical: Expected key behaviours

Deviating from expected widget keys breaks the mental model AT users depend on.
Do not override native control behaviour without a documented need. For custom
widgets follow the current [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/)
(versioned independently of WCAG — APG patterns are design guidance, not WCAG
requirements by themselves).

| Control | Required keys |
| --- | --- |
| Button | `Enter`, `Space` |
| Link | `Enter` |
| Checkbox | `Space` to toggle |
| Radio group | Arrow keys to move; `Space` to select |
| `<select>` | Platform conventions |
| Menu / menubar | Arrow keys; `Enter` to activate; `Escape` to close |
| Tabs | Arrow keys between tabs; `Tab` moves into the panel; `Home`/`End` first/last |
| Dialog | `Escape` closes; focus stays inside while open |
| Combobox | Depends on editable/select-only; `Escape` closes the popup |
| Tree | Up/Down navigate; Right expands/enters; Left collapses/parent |
| Slider | Arrows change value; `Home`/`End` min/max; `PageUp`/`PageDown` larger steps |
| Accordion | `Enter`/`Space` toggles; Up/Down between headers |
| `<details>` summary | `Enter`/`Space` toggles |

Do not implement a widget from this summary alone — use the complete current
APG pattern and test with real browsers and assistive technologies.

---

## Critical: Dialog focus management

Incorrect dialog focus is **Critical** — users lose their place or cannot reach
dialog controls.

### Preferred: native `<dialog>` with `showModal()`

A modal dialog opened with `showModal()` automatically makes the rest of the
document `inert`, traps focus, closes on `Escape`, and exposes
`aria-modal="true"` — all browser-provided (Baseline: widely available). This
is preferred over hand-rolled focus-trap classes.

```jsx
import { useRef, useEffect } from "react";

function DeleteDialog({ open, onClose }) {
  const dialogRef = useRef(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (open) dialog.showModal();
    else if (dialog.open) dialog.close();
  }, [open]);

  return (
    <dialog ref={dialogRef} aria-labelledby="del-title" onClose={onClose}>
      <h2 id="del-title">Delete project?</h2>
      <p>This action cannot be undone.</p>
      <form method="dialog">
        <button autoFocus>Cancel</button>
        <button value="confirm">Delete project</button>
      </form>
    </dialog>
  );
}
```

Choose initial focus by content: a short confirmation may focus Cancel; a long
informational dialog may focus a heading with `tabindex="-1"` so it reads from
the top. `showModal()` focuses the first focusable element (or the element with
`autofocus`) by default. On close, return focus to the trigger — or, if the
trigger was removed, to the nearest logical workflow location.

Do not assume `aria-modal="true"` alone creates modality — ARIA does not make
content inert, contain focus, add keyboard handling, or restore focus.

Even with native behaviour, test: initial focus, `Tab`/`Shift+Tab`
containment, `Escape` and a visible close control, focus restoration, stacked
overlays, screen reader announcement, and your browser support matrix.

### Fallback: `inert` (when native `<dialog>` is unavailable)

The `inert` attribute removes a subtree from the tab order **and** the
accessibility tree, and blocks clicks (Baseline: widely available). It is
simpler and more reliable than manual focusable-element cycling.

```js
function openDialog(dialog, siblingsSelector) {
  document.querySelectorAll(siblingsSelector).forEach(el => el.inert = true);
  dialog.hidden = false;
  dialog.querySelector("button, [href], input, select, textarea, [tabindex]")?.focus();
}
function closeDialog(dialog, siblingsSelector, trigger) {
  document.querySelectorAll(siblingsSelector).forEach(el => el.inert = false);
  dialog.hidden = true;
  trigger.focus();
}
```

Where `inert` is unavailable, use the production-tested
[`focus-trap`](https://github.com/focus-trap/focus-trap) /
[`focus-trap-react`](https://github.com/focus-trap/focus-trap-react) library or
the [`wicg-inert` polyfill](https://github.com/WICG/inert) rather than
hand-rolling focus cycling.

---

## Serious: Focus visibility (WCAG 2.4.7, 1.4.11)

Every focusable element needs a clear, persistent focus indicator. Removing
outlines without an equally visible replacement is **Serious**. Use
`:focus-visible` so the indicator shows for keyboard (not mouse) focus, and
give it at least 3:1 contrast against adjacent colours (1.4.11). A hairline
`1px` outline is too weak — use at least `2px`.

```css
:focus-visible {
  outline: 3px solid #005fcc; /* ensure >=3:1 against the background */
  outline-offset: 2px;
}
@media (forced-colors: active) {
  :focus-visible { outline-color: Highlight; }
}
```

Distinguish the criteria — do not describe the AAA formula as AA:
- **2.4.7 Focus Visible (AA):** a visible mode of focus exists
- **1.4.11 Non-text Contrast (AA):** indicator needs 3:1 contrast
- **2.4.13 Focus Appearance (AAA):** enhanced minimum area and change

Test focus in light, dark, increased-contrast, and forced-colours modes, and on
every background a control appears against.

---

## Serious: Focus not obscured (WCAG 2.4.11)

Sticky headers/footers, cookie banners, and chat panels can hide a focused
element even though focus is technically visible. A focused element **entirely**
hidden behind a sticky overlay fails 2.4.11 (AA) and is **Serious**; partial
obscuring is Moderate.

```css
html {
  scroll-padding-block-start: var(--sticky-header-height, 0);
  scroll-padding-block-end: var(--sticky-footer-height, 0);
}
```

CSS scroll spacing helps but does not prove conformance — test every
breakpoint, zoom level, and persistent overlay. Keeping the whole focused
component visible also satisfies 2.4.12 (AAA).

---

## Serious: Focus order (WCAG 2.4.3)

Tab order must follow the logical reading and interaction sequence. Illogical
order is **Serious** — screen reader users build a spatial model from it.

- Use semantic DOM order as the primary mechanism; keep visual order aligned
  with DOM order. Do not use CSS `order`, grid placement, or absolute
  positioning to create a misleading visual sequence.
- **Never use positive `tabindex`** — it creates a separate, fragile sequence.
- `tabindex="0"` — only when an element legitimately belongs in sequential
  focus and no native element provides the semantics.
- `tabindex="-1"` — only for programmatic focus targets and inactive items in
  roving-tabindex patterns.
- Do not add non-interactive containers/headings to the Tab order.
- Avoid `autofocus` unless initial focus there is expected and tested.
- When content is inserted, removed, reordered, or filtered, preserve focus or
  move it somewhere logical — never leave it on a removed element.

In React, do not use the array index as a `key` for reorderable lists; a stable
key keeps DOM identity so focus is not silently lost on re-render.

---

## Serious: Composite widgets (roving tabindex)

Composite widgets (toolbars, radio groups, tabs, menus, trees) expose **one**
Tab stop; arrow keys move among internal items. Two recognised strategies:
roving `tabindex` (one item `0`, others `-1`, updated on move) or
`aria-activedescendant` (DOM focus stays on the container). Choose the one the
widget's APG pattern specifies. Do not add roving tabindex to a native radio
group — the browser already provides it.

```jsx
import { useRef } from "react";

function Toolbar({ items }) {
  const ref = useRef(null);

  function onKeyDown(e) {
    const buttons = Array.from(ref.current.querySelectorAll("button:not([disabled])"));
    const current = buttons.indexOf(document.activeElement);
    let next = current;
    if (e.key === "ArrowRight" || e.key === "ArrowDown") next = (current + 1) % buttons.length;
    else if (e.key === "ArrowLeft" || e.key === "ArrowUp") next = (current - 1 + buttons.length) % buttons.length;
    else if (e.key === "Home") next = 0;
    else if (e.key === "End") next = buttons.length - 1;
    else return;
    e.preventDefault();
    buttons.forEach((b, i) => (b.tabIndex = i === next ? 0 : -1));
    buttons[next].focus();
  }

  return (
    <div role="toolbar" aria-label="Text formatting" ref={ref} onKeyDown={onKeyDown}>
      {items.map((label, i) => (
        <button key={label} type="button" tabIndex={i === 0 ? 0 : -1} aria-pressed="false">
          {label}
        </button>
      ))}
    </div>
  );
}
```

A production widget must also handle orientation, text direction, disabled
items, and dynamic additions. See the
[APG keyboard interface practice](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
for the arrow-key directions per widget type.

---

## Serious: `aria-hidden` vs `hidden` vs `inert`

| Mechanism | Visual | Sequential focus | Accessibility tree |
| --- | --- | --- | --- |
| `hidden` / `display: none` | Hidden | Removed | Removed |
| `visibility: hidden` | Hidden | Removed | Removed |
| `inert` | Visible unless styled | Descendants removed | Descendants removed |
| `aria-hidden="true"` | Unchanged | **Unchanged** | Content hidden from AT |

**Never place a focusable element inside an `aria-hidden="true"` subtree** —
`aria-hidden` does not remove keyboard focus, so it can create a control that
receives focus but is invisible to screen readers. Use `hidden` for closed
disclosures; use `inert` when visible content must become non-interactive.

---

## Moderate: Skip link & landmarks (WCAG 2.4.1)

A missing skip link forces sighted keyboard users to Tab through all navigation
on every page. Provide one as the first focusable element, visible on focus.

```html
<a class="skip-link" href="#main">Skip to main content</a>
<header>…</header>
<nav aria-label="Primary">…</nav>
<main id="main" tabindex="-1">…</main>
```

```css
.skip-link {
  position: fixed;
  inset-block-start: 0.5rem;
  inset-inline-start: 0.5rem;
  z-index: 1000;
  padding: 0.75rem 1rem;
  color: #fff;
  background: #000;
  transform: translateY(-200%);
}
.skip-link:focus { transform: translateY(0); }
```

The target `<main>` needs `tabindex="-1"` to receive programmatic focus. The
link must be **visible on focus** — a permanently `display: none` skip link
fails 2.4.1. Use native landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`)
without redundant roles, and give multiple `<nav>` landmarks distinct names.

---

## Moderate: Client-side navigation focus (React Router)

On a client-side route change, move focus so keyboard and screen reader users
are not stranded: update the title, focus the new view's main heading or
container, and announce the change once.

```jsx
import { useEffect, useRef } from "react";
import { useLocation } from "react-router";

function RouteFocus({ title, children }) {
  const location = useLocation();
  const headingRef = useRef(null);

  useEffect(() => {
    document.title = title;
    headingRef.current?.focus();
  }, [location, title]);

  return (
    <main>
      <h1 ref={headingRef} tabIndex={-1}>{title}</h1>
      {children}
    </main>
  );
}
```

Pair this with a visually hidden `aria-live="polite"` region to announce the
new view. Do not move focus merely because content updated — only when the user
needs a new context or would otherwise lose their place. Never use positive
`tabindex` or synthetic `Tab` events to manage focus.

---

## Moderate: Character-key shortcuts (WCAG 2.1.4)

If a shortcut uses only a printable character, provide at least one of: a way
to turn it off, a way to remap it to include a non-printing modifier, or
activation only while the relevant component has focus. Do not intercept
browser, OS, or AT shortcuts.

---

## Moderate: Touch & pointer equivalents (WCAG 2.5.x)

| Requirement | Description | Severity |
| --- | --- | --- |
| 2.5.1 Pointer Gestures | Multipoint/path gestures need a single-pointer or keyboard alternative | Serious |
| 2.5.3 Label in Name | Visible label text must be in the accessible name | Serious |
| 2.5.8 Target Size Minimum | Targets at least 24×24 CSS px (44×44 recommended) | Moderate |
| `user-scalable=no` | Never block pinch-zoom | Serious |
| Drag-to-reorder | Must have a keyboard alternative (e.g. move up/down buttons) | Serious |

```html
<!-- Good -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

---

## Event-handling notes

- Use the `click` event (or React `onClick`) for activation — it fires for
  mouse, `Enter`, and `Space` on native buttons/links, so you rarely need a
  manual `keydown` handler on a `<button>`.
- Use `keydown` for widget navigation that must feel immediate (arrow keys in a
  menu, toolbar, or listbox), and call `preventDefault()` on keys you handle.
- Do not attach interaction logic only to `mousedown`, `mouseover`, or
  `touchstart` — those exclude keyboard users.

---

## Testing checklist

- [ ] Every interactive element is reachable with `Tab`.
- [ ] Focus indicator is always visible and ≥3:1 contrast (light and dark).
- [ ] Tab order matches the visual/reading order.
- [ ] Elements activate with the correct keys per the widget table.
- [ ] No keyboard trap (except an intentional modal with a working `Escape`).
- [ ] `Escape` closes modals, popups, and menus.
- [ ] Skip link is first in the DOM and visible on focus.
- [ ] No focusable element sits inside an `aria-hidden="true"` subtree.
- [ ] Sticky elements never fully obscure the focused element.
- [ ] Composite widgets use roving tabindex or `aria-activedescendant`.
- [ ] Route changes move focus and announce the new view.
- [ ] Drag interactions have a keyboard alternative; targets ≥24×24 px.
- [ ] Tested with a screen reader running, at 200%/400% zoom, and 320px width.

Automated tools catch positive `tabindex`, handlers on non-interactive
elements, focusable descendants of `aria-hidden`/inert content, and missing
accessible names — but cannot confirm the focus order is meaningful or that
custom interactions follow their expected key model. Keep manual keyboard
walkthroughs for every new or changed pattern.

---

## Key WCAG 2.2 criteria

- 2.1.1 Keyboard (A) — Critical if violated
- 2.1.2 No Keyboard Trap (A) — Critical if violated
- 2.1.4 Character Key Shortcuts (A)
- 2.4.1 Bypass Blocks (A)
- 2.4.3 Focus Order (A)
- 2.4.7 Focus Visible (AA)
- 2.4.11 Focus Not Obscured Minimum (AA)
- 2.4.13 Focus Appearance (AAA)
- 2.5.1 Pointer Gestures (A)
- 2.5.3 Label in Name (A)
- 2.5.8 Target Size Minimum (AA)

## References

- [WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/)
- [APG: Developing a Keyboard Interface](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
- [APG: Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- [APG: Combobox Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)
- [MDN: The `<dialog>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog)
- [MDN: The `inert` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert)
- [MDN: `:focus-visible`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible)
- [`focus-trap-react`](https://github.com/focus-trap/focus-trap-react)

> **Standards horizon:** These rules target WCAG 2.2 AA. WCAG 3.0 is in
> development; keyboard and focus requirements are expected to remain broadly
> compatible. Monitor <https://www.w3.org/TR/wcag-3.0/>.
