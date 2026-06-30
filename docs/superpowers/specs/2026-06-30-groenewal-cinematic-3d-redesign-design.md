# Groenewal Incorporate — Cinematic 3D Redesign (Design Spec)

**Date:** 2026-06-30
**Site:** `index.html` (single-file static site, deployed via GitHub Pages at lukengroenewald.github.io)
**Status:** Approved direction, pending implementation plan

## Goal

Elevate the existing "Cyber-Architectural Blueprint" portfolio from a flat, SVG-faked tech aesthetic into a genuinely cinematic experience anchored by a real WebGL 3D centerpiece, with cinematic motion across the whole page. Shift the accent palette from acid lime to electric blue. Remove the personal phone number (done).

## Decisions (locked)

- **3D direction:** Mix of "Geometric AI Construct" (A) and "Full 3D Environment" (D), leaning A. The construct is the hero; the existing schematic/node diagram is absorbed into it as a real 3D scene.
- **Tech:** Three.js r128 loaded via CDN `<script>` tag. No build step — site stays a plain static `index.html`. Spline embed remains the documented fallback if the WebGL result is unsatisfactory (not chosen; user approved the Three.js prototype).
- **Placement:** Hero only — single WebGL canvas replacing the current SVG schematic. No persistent/multi-section canvases (performance).
- **Workflow nodes:** Kept. The 5 labeled nodes (BRIEFING, AI_COMPILATION, DB_STRUCTURE, SYSTEM_BUILD, PUSH) orbit the construct at varying depths with connector lines; labels surface on hover/proximity rather than always-on.
- **Cinematic scope:** Full page — hero 3D plus scroll-linked 3D depth/parallax on services, showcase, and section reveals (CSS 3D transforms, no extra WebGL).
- **Intro sequence:** Yes. First-visit-only boot animation (construct self-assembles, HUD text types in), ~1.5–2s, skippable, gated by `localStorage` so repeat visits skip straight to idle hero.
- **Palette:** Blue. Accent `#2E9BFF` (replaces lime `#BFFF00` everywhere). Construct core `#1E5BFF`. Glow orbs cobalt `#0066FF` and indigo `#3A4DFF`. Final shade subject to a one-value tweak on user confirmation.
- **Phone number:** Removed. Contact console shows business email (`groenewald.incoporate@gmail.com`) only; WhatsApp line, button, and copy reference deleted. (Already applied.)

## Architecture

Single `index.html`. Three.js added via one CDN `<script>`. New 3D logic lives in an inline `<script>` module (or clearly-delimited section) at the end of body, structured as small, independently-understandable units:

1. **`heroScene`** — sets up renderer/scene/camera, builds the construct (wireframe icosahedron shell + faint inner facets + glowing core sphere + additive glow sprite + point light), the 5 orbiting node groups, and their connector lines. Exposes `start()`, `pauseWhenHidden()`, and a `playIntro()` entry.
2. **`interaction`** — mouse-parallax and drag-rotate input mapped to the construct group rotation, with smoothed easing.
3. **`scrollDirector`** — scroll-linked camera dolly/tilt as the hero leaves the viewport; toggles render loop off when the hero canvas is fully scrolled past (visibility via `IntersectionObserver`).
4. **`bootIntro`** — first-visit assembly + HUD typing sequence; reads/writes `localStorage` flag; provides skip (click/tap/auto-complete).
5. **`cinematicReveals`** — extends existing `reveal-on-scroll` with 3D transform variants for services cards (perspective tilt-in) and showcase grid (focus/recede on hover). Pure CSS classes toggled by the existing `IntersectionObserver`.
6. **`guards`** — `prefers-reduced-motion` short-circuit (static construct, no heavy motion), mobile detection (capped `devicePixelRatio`, reduced geometry detail), and a WebGL-capability check with a static styled fallback so the hero never renders blank.

## Data Flow

- Page loads → `guards` evaluates reduced-motion / WebGL support / device tier → selects full, reduced, or static-fallback path.
- If full path: `heroScene.init()` builds objects → `bootIntro` runs (first visit) or skips → render loop starts → `interaction` and `scrollDirector` feed rotation/camera each frame → `IntersectionObserver` pauses the loop when the hero is offscreen.
- `cinematicReveals` runs independently of WebGL, driven by scroll, for the rest of the page.

## Error Handling / Resilience

- **No WebGL / Three.js fails to load:** detect (`window.THREE` absent or `WebGLRenderingContext` unsupported) → inject a static styled construct visual (CSS/SVG) in the hero slot. Page remains fully functional.
- **Reduced motion:** skip intro, skip orbit/parallax animation, render a single static framed pose (or static fallback). All content still present.
- **Low-end mobile:** cap pixel ratio at 2, lower icosahedron subdivision, fewer/simpler glow sprites; render loop still pauses offscreen to protect battery.

## Testing / Verification

- Visual: load in desktop browser, confirm construct renders, rotates on mouse/drag, nodes orbit with labels on hover, scroll dolly works, render pauses offscreen.
- First-visit intro plays once; reload confirms it is skipped (localStorage).
- Toggle OS reduced-motion → confirm static path.
- Simulate WebGL failure (block CDN) → confirm static fallback renders, no blank hero.
- Mobile viewport (DevTools + a real device if available) → confirm performance is smooth and layout intact.
- Confirm no remaining phone-number references anywhere in `index.html`.
- HTML structure validates (balanced tags), consistent with prior verification practice.

## Out of Scope

- The Apex Legal demo and other client sites (separate styling, per the No Style Recycling rule).
- Backend / forms / analytics.
- Spline implementation (fallback only, not being built).
