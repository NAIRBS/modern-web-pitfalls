# Performance

Load-time advice (Lighthouse-style) is usually fine. The gaps are **runtime responsiveness (INP)**
and **layout stability (CLS)**.

## 7. Runtime responsiveness and smoothness

### Don't
- Build custom `setTimeout(fn, 0)` queues to chunk background work.
- Render big data grids with no rendering containment. Every update after an interaction reflows the whole page.
- Hide inactive SPA views with `display: none`, or unmount them. That throws away cached layout and rendering state.
- Pause offscreen `<canvas>` animations with a hand-wired Intersection Observer when
  `content-visibility` can tell you directly.
- Show "more content →" scroll hints from `scroll` listeners that read `scrollLeft`/`scrollWidth`
  on every event, which forces layout.

### Do

```js
// Yield to the browser between chunks, with a fallback
const yieldToMain = () =>
  globalThis.scheduler?.yield ? scheduler.yield() : new Promise((r) => setTimeout(r, 0));

async function processAll(items) {
  for (const item of items) {
    process(item);
    await yieldToMain();
  }
}

// Prioritised background work
if (globalThis.scheduler?.postTask) {
  scheduler.postTask(sendAnalytics, { priority: 'background' });
} else {
  setTimeout(sendAnalytics, 0);
}
```

```css
/* Big lists / grids */
.grid-row {
  content-visibility: auto;
  contain-intrinsic-size: auto 48px; /* estimated row height; 'auto' remembers the real size */
}

/* Inactive SPA views: keep rendering state, skip rendering work */
.view[data-inactive] { content-visibility: hidden; }
```

```js
// Pair content-visibility: hidden with inert so the hidden view can't be focused or found
inactiveView.setAttribute('data-inactive', '');
inactiveView.inert = true;

// Pause offscreen canvas work (canvas wrapper has content-visibility: auto)
canvasWrapper.addEventListener('contentvisibilityautostatechange', (e) => {
  e.skipped ? animation.pause() : animation.play();
});
```

```css
/* Scroll affordance with scroll-state queries, where available */
.scroller { container-type: scroll-state; }
@container scroll-state(scrollable: right) {
  .scroll-hint { opacity: 1; }
}
```

Fall back to an `IntersectionObserver` on a sentinel element at the end of the scroller.
Never fall back to layout-forcing scroll listeners.

## 8. Layout shifts and visual flicker

### Don't
- Leave `<video>` without dimensions. It starts at 300×150 and jumps when metadata loads.
- Put `aspect-ratio` on a wrapper `<div>` and stretch the media inside it.
- Pair a web font with a system fallback that has different metrics. Text reflows on swap.
- Use `height: 100vh` for heroes and full-screen panels. Content jumps as mobile toolbars show and hide.
- Paste the classic A/B-test anti-flicker snippet (`body { opacity: 0 }` + multi-second timeout).
  It delays LCP and leaves invisible elements clickable.
- Start a cross-document view transition before the above-the-fold markup has parsed. Half-built elements pop in.

### Do

```html
<video src="intro.mp4" width="1280" height="720" controls preload="metadata"></video>
```

```css
video, img { max-width: 100%; height: auto; }
.thumb { aspect-ratio: 16 / 9; object-fit: cover; width: 100%; } /* on the media element itself */

body {
  font-family: "Inter", Arial, sans-serif;
  font-size-adjust: 0.52; /* match the web font's x-height ratio */
}
/* Or tune the fallback precisely */
@font-face {
  font-family: "Inter Fallback";
  src: local("Arial");
  size-adjust: 107%;
  ascent-override: 90%;
}

.hero { min-height: 100svh; }      /* stable: smallest viewport */
.fullscreen { height: 100dvh; }    /* tracks the toolbar when that's wanted */
```

```html
<!-- Client-side experiment: block render only on the script, only where supported -->
<script src="/experiment.js" blocking="render" async></script>

<!-- Cross-document view transitions: hold first render until key content is parsed -->
<link rel="expect" href="#main-content" blocking="render">
```

Keep render-blocking scripts small and fast. Prefer running experiments server-side or at the edge.
