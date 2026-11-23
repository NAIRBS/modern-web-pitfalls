# Accessibility

## 1. Keyboard focus and `inert`

### Don't
- Open a panel, drawer or route view and leave focus on the button that opened it.
- Close a view and drop focus on `<body>`.
- Build a custom modal from a `<div>` with `aria-modal="true"` and a hand-written Tab/Shift+Tab focus trap.
- Make every button in a toolbar (or every cell in a grid) its own tab stop.
- Restyle `:focus-visible` with a subtle colour that disappears against the background.

### Do
- Move focus into anything you open, and give it back to the trigger when it closes.
- Use a modal `<dialog>`: `showModal()` makes the rest of the page inert, traps focus, closes on Esc
  and restores focus for you. For non-dialog overlays, set `inert` on the background content.
- Composite widgets (toolbar, tablist, menu, listbox, grid) get **one** tab stop. Arrow keys move
  focus inside it (roving tabindex).
- Focus indicators need at least 3:1 contrast against the colours next to them.

```html
<button id="open">Settings</button>
<dialog id="settings" aria-labelledby="settings-title">
  <h2 id="settings-title">Settings</h2>
  <form method="dialog"><button autofocus>Close</button></form>
</dialog>
<script>
  open.addEventListener('click', () => settings.showModal()); // focus is restored on close
</script>
```

```js
// Roving tabindex for a toolbar
const items = [...toolbar.querySelectorAll('button')];
items.forEach((b, i) => (b.tabIndex = i === 0 ? 0 : -1));
toolbar.addEventListener('keydown', (e) => {
  const i = items.indexOf(document.activeElement);
  const next = { ArrowRight: i + 1, ArrowLeft: i - 1, Home: 0, End: items.length - 1 }[e.key];
  if (next === undefined) return;
  e.preventDefault();
  const target = items[(next + items.length) % items.length];
  items[i].tabIndex = -1;
  target.tabIndex = 0;
  target.focus();
});
```

```css
:focus-visible {
  outline: 3px solid currentColor; /* or a token that is checked for >= 3:1 on every surface */
  outline-offset: 2px;
}
```

## 2. Reduced motion

Simple `transition`/`@keyframes` checks are usually fine. Newer features are where it goes wrong.

### Don't
- `html { scroll-behavior: smooth; }` with no condition.
- Scroll-driven animations (`animation-timeline: scroll()/view()`) that ignore the preference.
- Reduced-motion carousels that set `transition-duration: 0` but keep the `transform` scale or slide,
  so items still jump while scrolling.
- A global `*, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }`
  reset. It also kills non-motion state transitions such as colour and opacity feedback.
- Turning on cross-document view transitions for everyone, then overriding `::view-transition-*`
  animations for reduced-motion users.

### Do
Opt **in** to motion. Don't opt out after the fact.

```css
@media (prefers-reduced-motion: no-preference) {
  html { scroll-behavior: smooth; }

  @view-transition { navigation: auto; }

  .reveal {
    animation: fade-in linear both;
    animation-timeline: view();
  }

  .carousel-item { transition: transform 300ms ease; }
  .carousel-item:not(.active) { transform: scale(0.9); }
}
```

Leave non-motion feedback such as colour, opacity fades and focus rings in place for everyone.

## 3. Forms: validation, hints, autofill

### Don't
- Re-implement "show errors after the user interacts" with JS `blur` listeners. That flags required
  fields the user merely tabbed past.
- Set `aria-invalid="true"` on blur before the visible error appears, or style `:user-invalid` in CSS
  without ever updating `aria-invalid`.
- Put a format hint (for example `MM/YY`) only in `placeholder`. It disappears on input and has poor contrast.
- Put hints or password rules **below** the field, where the mobile keyboard or autofill popover covers them.
- Mark `:autofill` fields with a border colour change alone.

### Do

```html
<label for="exp">Expiry date</label>
<p id="exp-hint" class="hint">Format: MM/YY</p>
<input id="exp" name="cc-exp" autocomplete="cc-exp" inputmode="numeric"
       pattern="(0[1-9]|1[0-2])\/\d{2}" required aria-describedby="exp-hint exp-error">
<p id="exp-error" class="error" hidden>Enter a date like 04/28.</p>
```

```css
input:user-invalid { border-color: var(--danger); }
input:user-invalid ~ .error { display: block; }

/* Colour + a non-colour cue that does not shift layout */
input:autofill {
  box-shadow: inset 4px 0 0 var(--accent);
}
```

```js
// Keep aria-invalid in step with the visible error: update it when :user-invalid would apply.
const inputs = [...form.querySelectorAll('input')];
const sync = (input) => {
  const shown = input.matches(':user-invalid');
  input.setAttribute('aria-invalid', String(shown));
  document.getElementById(`${input.id}-error`)?.toggleAttribute('hidden', !shown);
};
for (const input of inputs) {
  input.addEventListener('input', () => sync(input));
  input.addEventListener('change', () => sync(input));
}
// A submit attempt makes :user-invalid apply to untouched fields too
form.addEventListener('invalid', () => requestAnimationFrame(() => inputs.forEach(sync)), true);
```
