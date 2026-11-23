# Baseline quality

These rules apply to every page, whatever features it has. Modern APIs don't help if the basics are
broken. Run through this file before you call any UI work done.

## Contrast: text **and** components, in **both** themes

- Body text needs ≥ 4.5:1, and large text (≥ 24px, or ≥ 18.66px bold) needs ≥ 3:1.
- **UI component boundaries need ≥ 3:1** against what's next to them (WCAG 1.4.11). That covers
  input borders, checkbox and radio outlines, toggle tracks and focus rings. A pale grey
  1px border such as `#ddd` on white is about 1.4:1 and **fails**.
- Check each colour pair in light **and** dark. A token that passes in one theme often fails in
  the other.

```css
:root {
  color-scheme: light dark;
  --surface:     light-dark(#ffffff, #1b1b1f);
  --text:        light-dark(#1a1a1a, #ececf0);   /* ~17:1 / ~15:1 */
  --muted:       light-dark(#5b5e66, #a5a8b0);   /* ≥ 4.5:1 on --surface */
  --control-edge: light-dark(#767a83, #8b8f98);  /* ≥ 3:1 on --surface: input borders */
  --danger:      light-dark(#b42318, #ff8b80);   /* ≥ 4.5:1 for error text */
}
input, select, textarea { border: 1px solid var(--control-edge); }
```

## Never rely on colour alone

Errors, selected states and required fields need a second cue: an icon, text, a border-weight
change or a shape. One way is to give error messages an icon with a text alternative:

```html
<p id="email-error" class="error" hidden>
  <svg aria-hidden="true" width="16" height="16" viewBox="0 0 16 16"><circle cx="8" cy="8" r="7" fill="currentColor"/><path d="M8 4v5M8 11.5v.5" stroke="white" stroke-width="1.6" stroke-linecap="round"/></svg>
  <span class="visually-hidden">Error:</span> Enter your email address.
</p>
```

Add a thicker border on invalid fields too (`box-shadow: inset 0 0 0 1px var(--danger)` adds weight
without shifting the layout).

## Forms that help people finish

- Every control has a real `<label for>`, and placeholders are never the label.
- Use the right `type`, `inputmode` and `autocomplete` (`email`, `tel`, `cc-number`, `cc-exp`,
  `one-time-code`, `street-address`…). Set `spellcheck="false"` and `autocapitalize="off"` on
  emails, usernames and codes.
- Give **one specific message per failure**: empty, wrong format, out of range, expired. Use the
  Constraint Validation API (`validity.valueMissing`, `patternMismatch`, `setCustomValidity()`)
  rather than a single generic "invalid" message.
- On a failed submit, **move focus to the first invalid field**. Native `form.requestSubmit()` /
  `reportValidity()` does this.
- Make sure errors are **announced**. The error element is referenced by the field's
  `aria-describedby`, and either focus moves to the field or an `aria-live="polite"` region gets
  the message. Inside a modal `<dialog>`, the live region must also be inside the dialog, because
  the rest of the page is inert.
- **Reserve space for error text** (`min-height` on the message slot), or put it in a place where
  it can't push content around, so a validation pass doesn't cause a layout shift.
- Mark required fields visibly (for example a `*` with an explanation), not only with `required`.

## Keyboard, focus and targets

- Every interactive element can be reached and used with the keyboard. There are no
  `div`/`span` click handlers, so use `<button>` and `<a href>`.
- Keep a visible `:focus-visible` indicator. Never remove it with `outline: none` unless you add a
  replacement with ≥ 3:1 contrast.
- Pointer targets are at least **24×24 CSS px** (WCAG 2.2 AA). Aim for **44×44** on primary and
  touch controls.
- Add `touch-action: manipulation` on buttons to remove the double-tap-zoom delay on older mobile browsers.

## High-contrast / forced-colours mode

Custom styling often disappears in Windows High Contrast mode. Check that selected and invalid
states still show:

```css
@media (forced-colors: active) {
  .segmented input:checked + label { outline: 2px solid CanvasText; }
  input:user-invalid { outline: 2px solid Mark; }
  .icon { forced-color-adjust: auto; }
}
```

## Page basics

- Set `<html lang>`, put `<meta charset>` first, and include a viewport meta, a `<title>` and a `<main>` landmark.
- Use one `<h1>` and keep heading levels in order.
- Add a skip link when there's navigation before the main content.
- Respect `prefers-reduced-motion` (see accessibility.md) **and** `prefers-contrast: more` where it's cheap to do.

## Security by default

Even for a static demo page, **propose a CSP**. For single-file pages, a meta tag with hashes is enough:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none'; script-src 'sha256-…'; style-src 'sha256-…'; img-src 'self' data:; base-uri 'none'; form-action 'self'; object-src 'none'">
```

Compute each hash from the **exact** bytes inside the `<script>`/`<style>` tag. If you can't
compute hashes, include the CSP as a commented recommendation and say so. Never ship hashes you
haven't verified, because a wrong hash blocks the script entirely.

## Final self-check before you finish

1. Tab through the whole page. Can you reach everything, and see where focus is?
2. Submit the form empty, then with each kind of bad value. Is every message specific, visible,
   announced and not shown by colour alone?
3. Switch to dark mode. Is every text and border pair still at or above its contrast minimum?
4. Turn on reduced motion. Has the movement stopped, while the state feedback still works?
5. Click everywhere inside overlays, including their padding. Does anything close by accident?
   For backdrop-click handlers on a `<dialog>`, check that the click was **outside the dialog's
   bounding box**, not only `e.target === dialog`.
