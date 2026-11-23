# User experience

## 4. Swipe and touch gestures

Hand-written touch physics runs on the main thread and never feels quite native. Let the browser's
scroller do the physics.

### Don't
- Build swipe-to-dismiss or swipe-to-reveal from `pointermove`/`touchmove` listeners and `transform` maths.
- Build swipeable tabs without `overscroll-behavior-x`. An over-swipe then triggers browser back/forward.
- Drive "swipe progress" effects (fade, scale, reveal) from JS scroll callbacks.
- Ship swipe-action panels that keyboard users can't reach.

### Do

```html
<ul class="swipe-row">
  <li class="content">Message from Sam</li>
  <li class="actions"><button>Archive</button><button>Delete</button></li>
</ul>
```

```css
.swipe-row {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  overscroll-behavior-x: contain; /* stops back/forward navigation on over-swipe */
  scrollbar-width: none;
}
.swipe-row > * { scroll-snap-align: start; flex: 0 0 auto; }
.swipe-row > .content { width: 100%; }

/* Progress effect without JS */
@supports (animation-timeline: scroll()) {
  .swipe-row { scroll-timeline: --swipe x; }
  .actions { animation: reveal linear both; animation-timeline: --swipe; }
  @keyframes reveal { from { opacity: 0; } to { opacity: 1; } }
}
```

Make the actions reachable without a swipe: the buttons are real focusable `<button>`s. When one
receives focus, the scroller brings it into view. You can also add a visible "More actions" button.

## 5. Dialogs, popovers and top-layer transitions

### Don't
- Use a `position: fixed` overlay with a document click handler to "light dismiss".
- Animate open/close by toggling classes with `requestAnimationFrame()`, waiting on `transitionend`,
  and adding a `setTimeout()` fallback.
- Lock scrolling with `html { overflow: hidden }`. It loses the scroll position and causes layout jumps.
- Listen for `popstate` to close a modal on Android's back gesture.

### Do
- Non-modal, light-dismiss UI (menus, tooltips, pickers): `popover` attribute + `popovertarget`.
- Modal UI: `<dialog>` + `showModal()`. Native top layer, Esc/back-gesture close, focus handling.
- Animate both with pure CSS: `@starting-style` + `transition-behavior: allow-discrete`.
- Contain scroll with `overscroll-behavior: contain` on the dialog instead of locking the page.

```html
<button popovertarget="menu">Options</button>
<div id="menu" popover>…</div>
```

```css
dialog, [popover] {
  opacity: 0;
  translate: 0 8px;
  transition: opacity .2s, translate .2s, overlay .2s allow-discrete, display .2s allow-discrete;
}
dialog[open], [popover]:popover-open {
  opacity: 1;
  translate: 0 0;
  @starting-style { opacity: 0; translate: 0 8px; }
}
dialog::backdrop { background: rgb(0 0 0 / .4); }
dialog { overscroll-behavior: contain; max-height: 90dvh; overflow: auto; }

@media (prefers-reduced-motion: reduce) {
  dialog, [popover] { translate: none; }
}
```

Add `closedby="any"` to a `<dialog>` for light dismiss where supported. Feature-detect it and
fall back to a backdrop click check if needed.

## 6. Dark mode and theme-adaptive controls

### Don't
- Skip `<meta name="color-scheme">`. Dark-mode users get a white flash while CSS loads.
- Write custom dark colours without setting `color-scheme`. Scrollbars and native form controls stay light.
- Duplicate every token in `@media (prefers-color-scheme: dark)` blocks when `light-dark()` would do.
- Offer a two-state light/dark toggle that writes to `localStorage` with no way back to "System".
- Forget other open tabs. They keep the old theme until reloaded.

### Do

```html
<head>
  <meta name="color-scheme" content="light dark">
  <script>
    // Run before first paint
    const t = localStorage.getItem('theme'); // 'light' | 'dark' | null (= system)
    if (t) document.documentElement.style.colorScheme = t;
  </script>
</head>
```

```css
:root {
  color-scheme: light dark;
  --bg: light-dark(#fff, #121212);
  --fg: light-dark(#1a1a1a, #eaeaea);
}
body { background: var(--bg); color: var(--fg); }
```

```js
function applyTheme(value) { // 'light' | 'dark' | 'system'
  if (value === 'system') {
    localStorage.removeItem('theme');
    document.documentElement.style.colorScheme = '';
  } else {
    localStorage.setItem('theme', value);
    document.documentElement.style.colorScheme = value;
  }
}
// Keep other tabs in sync
addEventListener('storage', (e) => {
  if (e.key === 'theme') document.documentElement.style.colorScheme = e.newValue ?? '';
});
```

Use a three-option control (Light / Dark / System), not a binary switch.
