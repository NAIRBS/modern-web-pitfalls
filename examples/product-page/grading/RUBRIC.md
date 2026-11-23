# Grading rubric: product page (holdout task)

All pages were built from this prompt:

> Build a single-file product page (plain HTML/CSS/JS, no frameworks or external libraries) for a
> fictional pair of headphones, "Aria One", $249:
> - A hero that fills the phone screen, with product name, price and an "Add to cart" button.
> - A swipeable image carousel of 5 product images (use inline SVG placeholders) that works with touch swipe, prev/next buttons, and slide indicators.
> - A "See it in action" section with the product video at /media/demo.mp4 (1280×720).
> - Headings use the self-hosted web font "Brand Sans" from /fonts/brand-sans.woff2 with a system font fallback.
> - A reviews section that renders 300 generated reviews (name, 1–5 star rating, text) with a "Filter by rating" control that updates the list.
> - A "Review guidelines" button that opens a small non-modal panel, which closes on outside click or Esc.

Score each check PASS or FAIL with a short reason that quotes code. Grade only from the code. Ignore
comments that name a skill, guide or tool. Apply each check identically across all pages, and
re-check side by side before you finish.

## Part A: modern-platform checks (14), taken from Chrome's "Top 10 things AI gets wrong in web dev"

H1. The carousel scrolls with CSS Scroll Snap (`scroll-snap-type`) and native scrolling, with no JS pointer/touch drag physics (`pointermove`/`touchmove` + transform).
H2. The carousel sets `overscroll-behavior-x: contain` or `none`.
H3. The carousel works by keyboard: prev/next are `<button>`s, and the indicators are buttons with accessible labels and a current-state marker (`aria-current`/`aria-selected`/`aria-pressed`).
H4. Smooth scrolling (CSS `scroll-behavior: smooth` or JS `behavior: 'smooth'`) happens only when reduced motion is not requested.
H5. The active-slide/indicator state updates without a layout-reading `scroll` listener: IntersectionObserver, `scrollsnapchange`, or a scroll-driven/scroll-state approach. A `scroll` listener that reads `scrollLeft`/`getBoundingClientRect` FAILS, even when debounced or rAF-throttled.
H6. `<video>` has `width` and `height` attributes, or `aspect-ratio` set on the video element itself.
H7. The web-font fallback is metric-matched (`font-size-adjust`, or an `@font-face` fallback using `size-adjust`/`ascent-override`), and `font-display` is set.
H8. The hero uses `dvh`/`svh`/`lvh` (a `vh` fallback line is fine).
H9. The long review list uses `content-visibility: auto` with `contain-intrinsic-size`, or virtualization.
H10. Re-rendering the list on filter is batched (one `DocumentFragment`/`replaceChildren`/`innerHTML` write) or yields (`scheduler.yield`/`postTask`/chunking). A per-item append into the live DOM inside a loop that also reads layout FAILS.
H11. The guidelines panel uses the `popover` attribute (with `popovertarget` or `showPopover()`), not a custom positioned overlay with a document click handler.
H12. Carousel images have intrinsic size (`width`/`height` attributes, or `aspect-ratio` on the image or slide), so they cause no layout shift.
H13. Show/hide of the panel and any animations use no JS timers or class-toggle animation sequencing. Pure CSS (`@starting-style`, `allow-discrete`), or no animation at all, PASSES.
H14. A Content-Security-Policy is suggested or set.

## Part B: general quality basics (10)

G1. The filter control has a programmatically associated label.
G2. Star ratings have a text alternative (for example "4 out of 5 stars"), not just glyphs.
G3. Text has plausible WCAG AA contrast (judge from the colour values; flag clear failures only), and UI component borders are ≥ 3:1.
G4. Information isn't conveyed by colour alone (active indicator, ratings, selected filter).
G5. A visible focus indicator exists for all interactive elements.
G6. Sensible landmarks and headings: one `<h1>`, ordered headings, and a `<main>`.
G7. Images have appropriate `alt` (or are correctly decorative), and the carousel has an accessible name and slide labels ("1 of 5" or equivalent).
G8. The filter result is communicated: the count or result updates are announced (`aria-live`/`role="status"`) or otherwise clearly exposed.
G9. Buttons and controls are real `<button>`/`<select>`/`<input>` elements, and targets are at least 24px.
G10. No obvious bugs: nothing would throw or break a stated feature.

## Part C: free-form

Up to 5 bullets on other significant problems or strengths.

## Output format (JSON)

```json
{ "<page id>": { "A": {"H1": ["PASS","reason"], ...}, "B": {"G1": ["PASS","reason"], ...}, "C": ["..."], "scoreA": 0, "scoreB": 0 } }
```
