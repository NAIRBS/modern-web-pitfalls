# Grading rubric

Every page was built from the same prompt: a full-height hero with a "Settings" button that opens
an animated modal dialog containing a persistent light/dark theme switcher and a form with email +
card expiry (MM/YY) validation and a Save button.

Score every check as PASS or FAIL, with a short reason that quotes the relevant code. Grade only
from the code. Ignore any comments that name a skill, guide or tool. Be strict and consistent
across pages: the same code pattern must get the same verdict on every page.

## Part A: modern-platform checks (17)

A1. Modal is a native `<dialog>` opened with `showModal()`.
A2. Focus returns to the Settings button when the dialog closes (native restore counts).
A3. `color-scheme` is set in CSS, so native controls follow the theme.
A4. `aria-invalid` matches the visible error state: never true while no error is shown, always true while one is.
A5. The expiry format hint is visible text (not only a placeholder), placed above the input, and linked with `aria-describedby`.
A6. The full-height hero uses `dvh`/`svh`/`lvh` (a `vh` line used only as a fallback is fine).
A7. `<meta name="color-scheme" content="light dark">` (or equivalent) is in `<head>`.
A8. Theme colour tokens use `light-dark()`.
A9. The theme control offers a "System"/"Auto" option that returns to following the OS.
A10. A `storage` event listener keeps the theme in sync across tabs.
A11. Open/close animation is pure CSS (`@starting-style` and/or `transition-behavior: allow-discrete`, or equivalent), with no JS class toggling, `animationend`/`transitionend` waits or `setTimeout` for the animation.
A12. Motion (translate/scale) is opted into with `prefers-reduced-motion: no-preference`, or removed under `reduce`. A blanket `0.01ms` duration override FAILS.
A13. Native light dismiss (`closedby="any"`) is used, with or without a fallback.
A14. Error visibility is driven by `:user-invalid` (directly or by checking `matches(':user-invalid')`), not custom blur/touched tracking.
A15. `:autofill` (or `:-webkit-autofill`) is styled with a non-colour cue.
A16. The dialog contains scroll (`overscroll-behavior: contain`) or otherwise avoids page scroll chaining.
A17. A Content-Security-Policy is suggested or set (meta tag, header comment or similar).

## Part B: general quality basics (10)

B1. Every input has a programmatically associated `<label>`.
B2. Error messages are announced (`aria-live`/`role="alert"`, or linked via `aria-describedby` and focus is moved to the invalid field on submit).
B3. Errors are conveyed by more than colour (icon, text prefix, border weight change or similar).
B4. Everything works by keyboard: the dialog opens and closes (Esc), the theme control is operable and the form submits.
B5. A visible focus indicator exists for all interactive elements (not removed with `outline: none` without a replacement).
B6. Text and controls have plausible WCAG AA contrast in both themes (judge from the colour values; flag clear failures only).
B7. Inputs use correct `type`/`inputmode`/`autocomplete` (`email`, `cc-exp`).
B8. Expiry validation rejects month 00 or 13 and expired dates, with a specific message for each case.
B9. No flash of the wrong theme on load: the saved theme is applied before first paint.
B10. No obvious bugs: nothing would throw, get stuck half-open or break the stated features.

## Part C: free-form

List any other significant problems or notable strengths (max 5 bullets).

## Output format (JSON)

```json
{
  "<page id>": {
    "A": {"A1": ["PASS", "reason"], ...},
    "B": {"B1": ["PASS", "reason"], ...},
    "C": ["...", "..."],
    "scoreA": 0, "scoreB": 0
  }
}
```
