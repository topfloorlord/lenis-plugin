---
name: lenis
description: Lenis smooth scroll (darkroomengineering/lenis) — setup, options, scrollTo, events, GSAP ScrollTrigger sync, React (lenis/react), snapping (lenis/snap), nested scroll, modals, anchors, reduced motion. Use when the user wants smooth scrolling, mentions Lenis, or combines smooth scroll with GSAP/ScrollTrigger, parallax or WebGL scroll.
license: MIT
---

# Lenis Smooth Scroll

Lenis is a lightweight, dependency-free smooth scroll library that runs on top of **native scroll**, so `position: sticky`, anchor links and accessibility keep working. Current version: 1.3.x. Package name is `lenis` (the old `@studio-freight/lenis` is deprecated — never use it).

**Related skills:** for scroll-driven animation use **gsap-scrolltrigger**; for tweens use **gsap-core**.

## Install

```bash
npm i lenis
```

```js
import Lenis from 'lenis'
import 'lenis/dist/lenis.css' // always include the recommended CSS
```

CDN / no build step:

```html
<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.27/dist/lenis.css">
<script src="https://unpkg.com/lenis@1.3.27/dist/lenis.min.js"></script>
<script>new Lenis({ autoRaf: true, autoToggle: true, anchors: true, allowNestedScroll: true, naiveDimensions: true, stopInertiaOnNavigate: true })</script>
```

That no-code config handles most common issues (other packages, modals, smooth anchors, scroll reset on page change).

## Setup

Simplest — Lenis runs its own loop:

```js
const lenis = new Lenis({ autoRaf: true })
lenis.on('scroll', (e) => console.log(e))
```

Custom rAF loop (only when not using `autoRaf`):

```js
const lenis = new Lenis()
function raf(time) {
  lenis.raf(time)
  requestAnimationFrame(raf)
}
requestAnimationFrame(raf)
```

**Rule:** exactly one thing drives `lenis.raf()` — `autoRaf: true`, your own rAF loop, OR the GSAP ticker. Never two.

## With GSAP ScrollTrigger (most common pairing)

```js
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
import Lenis from 'lenis'
import 'lenis/dist/lenis.css'

gsap.registerPlugin(ScrollTrigger)

const lenis = new Lenis() // autoRaf stays false: GSAP's ticker drives it
lenis.on('scroll', ScrollTrigger.update)
gsap.ticker.add((time) => {
  lenis.raf(time * 1000) // GSAP time is seconds, Lenis wants ms
})
gsap.ticker.lagSmoothing(0)
```

- Do **not** use `ScrollTrigger.scrollerProxy()` with Lenis on the window — Lenis uses native scroll, so ScrollTrigger reads it directly.
- Pinning, scrub and `start`/`end` work as normal.
- For scroll-to with GSAP, use `lenis.scrollTo()` rather than GSAP's ScrollToPlugin so they don't fight.

## React (lenis/react)

```jsx
import { ReactLenis, useLenis } from 'lenis/react'
import 'lenis/dist/lenis.css'

function App() {
  const lenis = useLenis((lenis) => {
    // called on every scroll
  })
  return (
    <>
      <ReactLenis root />
      {/* content */}
    </>
  )
}
```

- `root` (true) = use `<html>` as scroller and make the instance global via `useLenis`. `root="asChild"` renders wrapper elements for a custom container.
- `options` prop takes any Lenis option.
- `useLenis(callback, deps, priority)` returns the instance.

React + GSAP:

```jsx
import gsap from 'gsap'
import { ReactLenis } from 'lenis/react'
import { useEffect, useRef } from 'react'

function App() {
  const lenisRef = useRef()
  useEffect(() => {
    function update(time) {
      lenisRef.current?.lenis?.raf(time * 1000)
    }
    gsap.ticker.add(update)
    return () => gsap.ticker.remove(update)
  }, [])
  return <ReactLenis root options={{ autoRaf: false }} ref={lenisRef} />
}
```

Also add `lenis.on('scroll', ScrollTrigger.update)` (e.g. via `useLenis(ScrollTrigger.update)`) when using ScrollTrigger. In Next.js App Router, put `<ReactLenis>` in a `'use client'` component.

Vue: use `lenis/vue`. Framer: `lenis/framer`.

## Key options

| Option | Default | Notes |
|---|---|---|
| `autoRaf` | `false` | Run its own rAF loop. |
| `lerp` | `0.1` | Smoothing 0–1 (lower = smoother/slower). Overrides duration/easing. |
| `duration` / `easing` | `1.2` / expo-out | Only used when `lerp` isn't set. |
| `smoothWheel` | `true` | Smooth wheel input. |
| `syncTouch` | `false` | Smooth touch too (can be unstable on iOS < 16). Usually leave off. |
| `wheelMultiplier` / `touchMultiplier` | `1` | Speed. |
| `orientation` | `vertical` | or `horizontal`. `gestureOrientation`: `vertical`/`horizontal`/`both`. |
| `wrapper` / `content` | `window` / `<html>` | For a custom scroll container. |
| `anchors` | `false` | `true` or scrollTo options (e.g. `{ offset: -80 }`) to smooth anchor links. |
| `allowNestedScroll` | `false` | Let nested scrollables scroll natively (DOM check every scroll — costly). |
| `prevent` | — | `(node) => boolean` to skip smoothing for elements. |
| `autoToggle` | `false` | Start/stop based on wrapper overflow (needs lenis.css; modern browsers). |
| `autoResize` | `true` | ResizeObserver; otherwise call `resize()`. |
| `infinite` | `false` | Infinite scroll (needs `syncTouch` on touch). |
| `respectReducedMotion` | `true` | Honors prefers-reduced-motion. Leave on. |
| `stopInertiaOnNavigate` | `false` | Stop inertia on internal link click. |
| `virtualScroll` | — | Modify/veto events: return `false` to skip smoothing. |

## Methods, properties, events

- `lenis.scrollTo(target, { offset, duration, easing, lerp, immediate, lock, force, onComplete })` — target: number, selector, `'top'`/`'bottom'`, or element.
- `lenis.stop()` / `lenis.start()` — e.g. when opening/closing a modal.
- `lenis.resize()`, `lenis.destroy()` (always destroy on unmount/page change in SPAs).
- Properties: `scroll`, `progress` (0–1), `velocity`, `direction`, `isScrolling`, `isStopped`, `limit`.
- Events: `scroll` (instance), `virtual-scroll` (`{deltaX, deltaY, event}`).

## Nested scroll and modals

Prefer HTML attributes (cheapest):

```html
<div data-lenis-prevent>scrollable content</div>
```

Variants: `data-lenis-prevent-wheel`, `-touch`, `-vertical`, `-horizontal`. Or `prevent: (node) => node.id === 'modal'`, or `allowNestedScroll: true` (simplest, slower). For modals, also `lenis.stop()` on open and `lenis.start()` on close.

## Snapping (lenis/snap)

Lenis does **not** support CSS `scroll-snap`. Use the snap package:

```js
import Snap from 'lenis/snap'
const snap = new Snap(lenis, { type: 'mandatory' }) // 'proximity' (default) | 'mandatory' | 'lock'
snap.addElements(document.querySelectorAll('.section'), { align: 'start' })
```

Options: `distanceThreshold` ('50%'), `debounce` (500), `lerp`/`easing`/`duration`, `onSnapStart`, `onSnapComplete`. Methods: `next()`, `previous()`, `goTo(i)`, `resize()`.

## Do Not

- ❌ Skip `lenis/dist/lenis.css`.
- ❌ Drive `raf()` from two places (e.g. `autoRaf: true` plus the GSAP ticker) — causes double-speed or jitter.
- ❌ Forget `* 1000` when feeding GSAP ticker time into `lenis.raf()`.
- ❌ Use CSS `scroll-behavior: smooth` or CSS scroll-snap alongside Lenis.
- ❌ Install `@studio-freight/lenis` or `@studio-freight/react-lenis` (deprecated).
- ❌ Disable `respectReducedMotion` without a reason.

## Limitations / troubleshooting

- Smooth scroll stops over iframes (they don't forward wheel events).
- Capped at 60fps in Safari (30fps in low power mode).
- If nothing scrolls: test without Lenis, include the CSS, confirm something calls `raf()`, update to the latest `lenis`.

Docs: https://github.com/darkroomengineering/lenis
