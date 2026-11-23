---
name: modern-web-pitfalls
description: Corrects the ten web-dev mistakes coding agents make most often (focus/inert handling, reduced motion, form validation, swipe gestures, dialogs/popovers, dark mode, INP responsiveness, layout shift, DOM XSS/CSP, passkeys). Use whenever writing or reviewing HTML, CSS or front-end JavaScript - especially modals, popovers, carousels, swipe UIs, forms, theming, data grids, view transitions, A/B test snippets, HTML sanitization, CSP headers or WebAuthn/passkey flows.
---

# Modern Web Pitfalls

Frontier models default to older, hand-rolled patterns where the web platform now has a native
primitive. Unguided, they pass only about half of the best-practice checks in Chrome's eval suite.
This skill lists the patterns to avoid and what to write instead.

## How to use this skill

1. Before writing UI code, scan the **checklist** below for the features you are about to build.
2. For each feature, open the matching reference file and follow its rules and snippets:
   - Accessibility (focus, reduced motion, forms): [references/accessibility.md](references/accessibility.md)
   - UX (gestures, dialogs/popovers, dark mode): [references/ux.md](references/ux.md)
   - Performance (INP, CLS, flicker): [references/performance.md](references/performance.md)
   - Security (XSS, sanitization, CSP, passkeys): [references/security.md](references/security.md)
3. **Progressive enhancement is the rule.** Newer APIs (`setHTML()`, `scheduler.yield()`,
   scroll-state queries, `blocking="render"`, `contentvisibilityautostatechange`, WebAuthn JSON
   helpers) must be feature-detected, with a fallback that still works.
4. When reviewing existing code, flag every "Don't" pattern you find and propose the "Do" replacement.

## Checklist

### Accessibility
- [ ] Opening a view moves focus into it; closing it returns focus to the trigger.
- [ ] Modals use `<dialog>` + `showModal()` (or `inert` on the background), not JS focus traps + `aria-modal`.
- [ ] Toolbars, tab lists, grids and menus use **roving tabindex** (one tab stop per widget).
- [ ] Custom `:focus-visible` outlines have ≥ 3:1 contrast against what's next to them.
- [ ] Input borders and other UI component boundaries have ≥ 3:1 contrast against the background in **both** themes (WCAG 1.4.11). Faint 1px grey borders usually fail.
- [ ] `scroll-behavior: smooth`, scroll-driven animations and `@view-transition` are gated behind `prefers-reduced-motion: no-preference`.
- [ ] Reduced-motion carousels remove the movement (`transform`), not only the duration.
- [ ] No blanket `* { transition-duration: 0.01ms }` reset.
- [ ] Error styling uses `:user-invalid`; `aria-invalid` is kept in sync with what's visible.
- [ ] Format hints are visible text **above** the input, linked via `aria-describedby`, not `placeholder`.
- [ ] `:autofill` styling pairs colour with a non-colour cue that doesn't shift layout (e.g. inset `box-shadow`).

### User experience
- [ ] Swipe-to-dismiss/reveal and carousels use CSS Scroll Snap, not `pointermove`/`touchmove` physics.
- [ ] Horizontal swipe areas set `overscroll-behavior-x: contain` (or `none`) so over-swipe doesn't navigate back/forward.
- [ ] Swipe-progress effects use scroll-driven animations, not JS scroll callbacks.
- [ ] Light-dismiss overlays use the `popover` attribute; modals use `<dialog>`.
- [ ] Enter/exit animations use `@starting-style` + `transition-behavior: allow-discrete`, not rAF/`transitionend`/`setTimeout` class juggling.
- [ ] No `overflow: hidden` on `<html>` for scroll lock, no `popstate` hacks for Android back.
- [ ] `<meta name="color-scheme" content="light dark">` is in `<head>`; CSS sets `color-scheme`.
- [ ] Theme colours use `light-dark()`; the theme toggle has a **System** option and syncs tabs via the `storage` event.

### Performance
- [ ] Background work yields with `scheduler.yield()` / `scheduler.postTask()` (feature-detected), not `setTimeout` queues.
- [ ] Large lists/grids use `content-visibility: auto` + `contain-intrinsic-size`.
- [ ] Inactive SPA views use `content-visibility: hidden` + `inert`, not `display: none` or teardown.
- [ ] Offscreen canvas animations pause via `contentvisibilityautostatechange` where available.
- [ ] Scroll hints use scroll-state container queries where available, Intersection Observer otherwise; never layout-forcing scroll listeners.
- [ ] `<video>` has `width`/`height`; `aspect-ratio`/`object-fit` go on the media element itself.
- [ ] Web-font fallbacks use `font-size-adjust` (or `@font-face` metric overrides).
- [ ] Full-height sections use `dvh`/`svh`/`lvh`, not bare `vh`.
- [ ] No anti-flicker `opacity:0` snippets; use `<script blocking="render">` where available.
- [ ] Cross-document view transitions wait for critical markup via `<link rel="expect" blocking="render">`.

### Security
- [ ] Untrusted HTML goes through `setHTML()` when supported, else **lazy-loaded DOMPurify**. Never a hand-rolled `DOMParser` sanitizer.
- [ ] Trusted Types policies actually sanitize; never `(input) => input`.
- [ ] Static pages: hash-based CSP (`'sha256-…'`), not `script-src 'self'` allowlists.
- [ ] Nonce-based CSP includes `'strict-dynamic'` and `'report-sample'`.
- [ ] Suggest a CSP and cross-origin isolation headers even when not asked.
- [ ] Passkeys use `PublicKeyCredential.parseCreationOptionsFromJSON()` / `parseRequestOptionsFromJSON()` / `credential.toJSON()`, not `atob`/`btoa` helpers.
- [ ] Call `PublicKeyCredential.signalUnknownCredential()` when the server rejects a credential ID.
- [ ] Check `PublicKeyCredential.getClientCapabilities()` before rendering passkey UI.
- [ ] Reauth `navigator.credentials.get()` calls take an `AbortSignal`.
- [ ] Use `uiMode: "immediate"`, not the outdated `mediation: "immediate"`.

## Source

Based on "Top 10 things AI gets wrong in web dev" by Philip Walton (Chrome for Developers, Nov 2026):
https://developer.chrome.com/blog/top-10-things-ai-gets-wrong-in-web-dev.
For the official, fuller guidance see Google Chrome's Modern Web Guidance
(`npx modern-web-guidance@latest install`).
