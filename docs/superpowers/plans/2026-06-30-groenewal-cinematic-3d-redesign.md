# Groenewal Incorporate — Cinematic 3D Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the Groenewal Incorporate portfolio (`index.html`) from a flat SVG-faked tech page into a cinematic experience anchored by a real Three.js/WebGL hero construct (cool cyan theme), with a first-visit boot intro, full-page cinematic motion, and graceful fallbacks.

**Architecture:** Single static `index.html` deployed via GitHub Pages. Three.js loads from a CDN `<script>` (no build step). All new behavior lives in clearly-delimited inline `<script>` blocks at the end of `<body>`, decomposed into small units: `guards`, `heroScene`, `interaction`, `scrollDirector`, `bootIntro`, `cinematicReveals`. The existing CSS-variable-driven palette means the color migration is mostly changing `--accent` and the two glow vars.

**Tech Stack:** HTML5, CSS3 (custom properties, 3D transforms), vanilla JS, Three.js r128 (CDN, global `THREE`), IntersectionObserver, localStorage.

**Verification note:** This is a static site with no unit-test runner. "Verify" steps are concrete browser observations using the live preview (the brainstorm companion or any `file://` / local server) plus console-error checks and an HTML tag-balance sanity check. Each task ends with a focused commit.

**Spec:** `docs/superpowers/specs/2026-06-30-groenewal-cinematic-3d-redesign-design.md`

**Working file (all tasks):** `C:\Users\luken\Desktop\lukengroenewald.github.io\index.html`

---

## Task 1: Land the phone-number removal (already in working tree)

The personal phone number / WhatsApp has already been stripped from the contact console in the working tree (per user request: no personal number, business email only). This task verifies and commits it as a discrete change.

**Files:**
- Modify: `index.html` (contact console section + `.console-details-single` CSS — already edited)

- [ ] **Step 1: Verify no phone references remain**

Run (Grep tool or):
```bash
grep -niE "wa\.me|\+27|whatsapp|tel:|82 322" index.html
```
Expected: only the two unrelated *Stack Burger* showcase mentions of "WhatsApp API"/"WhatsApp routing" (lines in the showcase card). NO `+27`, NO `wa.me`, NO `WhatsApp Interface`, NO `WhatsApp channels` in the contact section.

- [ ] **Step 2: Verify the contact console shows email only**

Confirm the contact section contains a single `.console-detail-item` (Secure Mail Endpoint → `groenewald.incoporate@gmail.com`) inside `.console-details-single`, and a single `.btn-tech` mail button — no WhatsApp button.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Remove personal phone number from contact console (email only)"
```

---

## Task 2: Migrate palette from acid lime to cool cyan

The palette is driven almost entirely by CSS custom properties. Change the variable values and the two hardcoded `#BFFF00` SVG label fills.

**Files:**
- Modify: `index.html` `:root` block (the `--accent*` and `--glow-*` declarations) and the two `<text fill="#BFFF00">` in the hero SVG.

- [ ] **Step 1: Replace the accent + glow variables**

Find:
```css
      --accent: #BFFF00;          /* Acid Neon Lime */
      --accent-dim: rgba(191, 255, 0, 0.15);
      --accent-muted: rgba(191, 255, 0, 0.4);

      --glow-violet: #7A00FF;     /* Cyber Violet */
      --glow-blue: #0066FF;       /* Deep Cobalt */
```
Replace with:
```css
      --accent: #22D3EE;          /* Electric Cyan */
      --accent-dim: rgba(34, 211, 238, 0.15);
      --accent-muted: rgba(34, 211, 238, 0.4);

      --glow-violet: #0E7490;     /* Deep Teal (glow A) */
      --glow-blue: #00D3FF;       /* Cyan (glow B) */
```

- [ ] **Step 2: Fix the two hardcoded glow-orb gradient stop colors**

The orb backgrounds hardcode the old rgb values in their transparent stops. Find:
```css
      background: radial-gradient(circle, var(--glow-violet) 0%, rgba(122,0,255,0) 70%);
```
Replace with:
```css
      background: radial-gradient(circle, var(--glow-violet) 0%, rgba(14,116,144,0) 70%);
```
Find:
```css
      background: radial-gradient(circle, var(--glow-blue) 0%, rgba(0,102,255,0) 70%);
```
Replace with:
```css
      background: radial-gradient(circle, var(--glow-blue) 0%, rgba(0,211,255,0) 70%);
```

- [ ] **Step 3: Fix the two hardcoded SVG label fills**

These two get removed entirely in Task 3 (the SVG is replaced), but update them now so the page is consistent if Task 3 is deferred. Find both `fill="#BFFF00"` occurrences (BRIEFING and PUSH labels) and change to `fill="#22D3EE"`.

- [ ] **Step 4: Verify no lime remains**

Run:
```bash
grep -niE "BFFF00|191, ?255, ?0|122,0,255|0,102,255|7A00FF" index.html
```
Expected: no matches.

- [ ] **Step 5: Verify in browser**

Open `index.html` in the preview. Expected: every accent (nav links, buttons, section codes, telemetry numbers, card-draw borders, cursor) is now cyan; glow orbs are teal/cyan; layout unchanged; no console errors.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Migrate accent palette from acid lime to cool cyan"
```

---

## Task 3: Replace hero SVG with WebGL canvas scaffold + static fallback

Swap the flat SVG schematic for a canvas mount point with a CSS/SVG static fallback (shown until/unless WebGL initializes), and load Three.js from CDN.

**Files:**
- Modify: `index.html` — hero `.schematic-box` inner content; add Three.js `<script>` before the existing inline script; add CSS for the canvas + fallback.

- [ ] **Step 1: Add CSS for the hero canvas and fallback**

Add to the stylesheet (near the `.schematic-box` rules):
```css
    .hero-canvas-wrap { position: relative; width: 100%; min-height: 360px; }
    #heroCanvas { display: block; width: 100%; height: 420px; }
    .hero-fallback {
      position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
      opacity: 1; transition: opacity 0.6s ease;
    }
    .hero-fallback.hidden { opacity: 0; pointer-events: none; }
    .hero-node-label {
      position: absolute; transform: translate(-50%, -50%);
      font-family: var(--font-mono); font-size: 0.6rem; letter-spacing: 0.08em;
      color: var(--accent); background: rgba(6,7,9,0.75); padding: 0.15rem 0.4rem;
      border: 1px solid var(--accent-dim); pointer-events: none; white-space: nowrap;
      opacity: 0; transition: opacity 0.2s ease; z-index: 6;
    }
    .hero-node-label.show { opacity: 1; }
```

- [ ] **Step 2: Replace the SVG schematic with the canvas + fallback**

Find the entire `<svg class="schematic-svg" ...>...</svg>` block inside `.schematic-box` and replace it with:
```html
          <div class="hero-canvas-wrap" id="heroCanvasWrap">
            <canvas id="heroCanvas" aria-hidden="true"></canvas>
            <div class="hero-fallback" id="heroFallback">
              <!-- Static fallback: simplified construct, shown if WebGL/reduced-motion -->
              <svg width="220" height="220" viewBox="0 0 220 220" role="img" aria-label="Groenewal system construct">
                <polygon points="110,20 190,65 190,155 110,200 30,155 30,65" fill="none" stroke="#22D3EE" stroke-width="1.5" opacity="0.85"/>
                <polygon points="110,55 160,82 160,138 110,165 60,138 60,82" fill="none" stroke="#22D3EE" stroke-width="1" opacity="0.4"/>
                <circle cx="110" cy="110" r="16" fill="#0AB4D6" opacity="0.7"/>
                <circle cx="110" cy="110" r="30" fill="none" stroke="#00D3FF" stroke-width="0.5" opacity="0.5"/>
              </svg>
            </div>
          </div>
```

- [ ] **Step 3: Add the Three.js CDN script**

Immediately before the existing `<script>` at the bottom of `<body>`, add:
```html
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
```

- [ ] **Step 4: Verify in browser**

Open the page. Expected: the hero right-column now shows the static SVG construct fallback (cyan hexagon + core); page layout intact; no console errors; Network panel shows `three.min.js` loaded (200).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Replace hero SVG schematic with WebGL canvas scaffold and static fallback"
```

---

## Task 4: Build the static construct scene

Initialize the Three.js renderer/scene/camera and build the construct (wireframe icosahedron shell, faint inner facets, glowing core sphere + additive halo, point light). Render a single frame (no animation yet). Hide the fallback once it renders.

**Files:**
- Modify: `index.html` — add a new inline `<script>` block (the 3D module) after the Three.js CDN script and after the existing UI script. Wrap in an IIFE.

- [ ] **Step 1: Add the heroScene init module**

Add a new `<script>` block at the end of `<body>` (after the existing script):
```html
  <script>
  (function(){
    var THREE = window.THREE;
    var canvas = document.getElementById('heroCanvas');
    var wrap = document.getElementById('heroCanvasWrap');
    var fallback = document.getElementById('heroFallback');
    if (!THREE || !canvas || !wrap) return;

    // expose for later tasks
    var HERO = window.__hero = {};

    var W = wrap.clientWidth, H = canvas.clientHeight || 420;
    var scene = new THREE.Scene();
    var camera = new THREE.PerspectiveCamera(45, W/H, 0.1, 100);
    camera.position.set(0, 0, 6);

    var renderer;
    try {
      renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true, alpha: true });
    } catch (e) { return; } // leaves static fallback visible
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.setSize(W, H, false);

    var group = new THREE.Group(); scene.add(group);

    var icoGeo = new THREE.IcosahedronGeometry(1.7, 1);
    var shell = new THREE.LineSegments(
      new THREE.EdgesGeometry(icoGeo, 1),
      new THREE.LineBasicMaterial({ color: 0x22D3EE, transparent: true, opacity: 0.9 })
    );
    group.add(shell);
    var facet = new THREE.Mesh(icoGeo, new THREE.MeshBasicMaterial({ color: 0x0E1015, transparent: true, opacity: 0.55 }));
    facet.scale.setScalar(0.99); group.add(facet);

    function glowTex(rgba){
      var c = document.createElement('canvas'); c.width = c.height = 128;
      var ctx = c.getContext('2d');
      var g = ctx.createRadialGradient(64,64,0,64,64,64);
      g.addColorStop(0, rgba); g.addColorStop(0.25, rgba); g.addColorStop(1, 'rgba(0,0,0,0)');
      ctx.fillStyle = g; ctx.fillRect(0,0,128,128);
      return new THREE.CanvasTexture(c);
    }
    var coreHalo = new THREE.Sprite(new THREE.SpriteMaterial({ map: glowTex('rgba(50,215,245,0.9)'), blending: THREE.AdditiveBlending, transparent: true, depthWrite: false }));
    coreHalo.scale.setScalar(3.4); group.add(coreHalo);
    var core = new THREE.Mesh(new THREE.SphereGeometry(0.32, 24, 24), new THREE.MeshBasicMaterial({ color: 0x0AB4D6 }));
    group.add(core);
    var light = new THREE.PointLight(0x0AB4D6, 2, 20); light.position.set(0,0,0); scene.add(light);
    scene.add(new THREE.AmbientLight(0x223344, 0.6));

    renderer.render(scene, camera);
    if (fallback) fallback.classList.add('hidden');

    // share refs with later tasks
    HERO.THREE = THREE; HERO.scene = scene; HERO.camera = camera; HERO.renderer = renderer;
    HERO.group = group; HERO.shell = shell; HERO.core = core; HERO.coreHalo = coreHalo;
    HERO.glowTex = glowTex; HERO.wrap = wrap; HERO.canvas = canvas;

    window.addEventListener('resize', function(){
      var w = wrap.clientWidth, h = canvas.clientHeight || 420;
      camera.aspect = w/h; camera.updateProjectionMatrix(); renderer.setSize(w, h, false);
      renderer.render(scene, camera);
    });
  })();
  </script>
```

- [ ] **Step 2: Verify in browser**

Open the page. Expected: the hero shows the 3D cyan wireframe icosahedron with a glowing core (static, not animating); the SVG fallback has faded out; no console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Build static Three.js hero construct (shell, core, glow, light)"
```

---

## Task 5: Add orbiting workflow nodes, connector lines, and HTML labels

Add the 5 workflow nodes (BRIEFING, AI_COMPILATION, DB_STRUCTURE, SYSTEM_BUILD, PUSH) as glowing markers with connector lines, plus HTML overlay labels that show on hover/proximity. Positions are static this task (animation comes in Task 6).

**Files:**
- Modify: `index.html` — extend the heroScene IIFE; add label DOM elements into `#heroCanvasWrap`.

- [ ] **Step 1: Add label elements to the canvas wrap**

Inside `#heroCanvasWrap`, after the `.hero-fallback` div, add:
```html
            <div class="hero-node-label" data-i="0">BRIEFING</div>
            <div class="hero-node-label" data-i="1">AI_COMPILATION</div>
            <div class="hero-node-label" data-i="2">DB_STRUCTURE</div>
            <div class="hero-node-label" data-i="3">SYSTEM_BUILD</div>
            <div class="hero-node-label" data-i="4">PUSH</div>
```

- [ ] **Step 2: Build nodes + lines in the heroScene module**

In the IIFE, after the `light`/`AmbientLight` lines and before `renderer.render(...)`, insert:
```javascript
    var nodeGlowTex = glowTex('rgba(90,230,255,0.95)');
    var nodes = [], lines = [];
    var lineMat = new THREE.LineBasicMaterial({ color: 0x22D3EE, transparent: true, opacity: 0.18 });
    for (var i = 0; i < 5; i++){
      var n = new THREE.Group();
      var ang = (i/5) * Math.PI * 2, rad = 2.7 + (i % 2) * 0.5, depth = (i - 2) * 0.55;
      n.userData = { ang: ang, rad: rad, depth: depth, speed: 0.15 + i * 0.03 };
      n.add(new THREE.Mesh(new THREE.SphereGeometry(0.10, 16, 16), new THREE.MeshBasicMaterial({ color: 0x22D3EE })));
      var halo = new THREE.Sprite(new THREE.SpriteMaterial({ map: nodeGlowTex, blending: THREE.AdditiveBlending, transparent: true, depthWrite: false }));
      halo.scale.setScalar(0.7); n.add(halo);
      // static initial position (animated in Task 6)
      n.position.set(Math.cos(ang) * rad, depth, Math.sin(ang) * rad);
      group.add(n); nodes.push(n);

      var lg = new THREE.BufferGeometry();
      lg.setAttribute('position', new THREE.BufferAttribute(new Float32Array(6), 3));
      var ln = new THREE.Line(lg, lineMat); group.add(ln); lines.push(ln);
    }
    HERO.nodes = nodes; HERO.lines = lines;
    HERO.labels = Array.prototype.slice.call(document.querySelectorAll('.hero-node-label'));
```

- [ ] **Step 3: Verify in browser**

Open the page. Expected: 5 cyan glowing nodes sit around the construct with faint connector lines; labels are hidden (will wire up on hover next); no console errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add orbiting workflow nodes, connector lines, and label elements to hero"
```

---

## Task 6: Animate the scene + full 3D drag rotation + parallax + label projection

Add the render loop: idle spin, orbiting nodes, connector-line updates, pulsing core, full 3D drag rotation (yaw + pitch, accumulated), mouse parallax, and project node positions to screen to place the HTML labels (shown on proximity/hover).

**Files:**
- Modify: `index.html` — extend the heroScene IIFE; replace the single `renderer.render(...)` with the animation loop and interaction handlers.

- [ ] **Step 1: Replace the static render with the animation loop + interaction**

Remove the single `renderer.render(scene, camera);` line (keep the `fallback.classList.add('hidden')`). After the `HERO.labels = ...` line, add:
```javascript
    // --- interaction state: full 3D rotation ---
    var driftYaw = 0, tYaw = 0, tPitch = 0, dYaw = 0, dPitch = 0, curYaw = 0, curPitch = 0;
    var dragging = false, lastX = 0, lastY = 0;
    HERO.state = { get running(){ return running; } };

    function parallax(cx, cy){
      var r = wrap.getBoundingClientRect();
      tYaw = ((cx - r.left)/r.width - 0.5) * 0.8;
      tPitch = ((cy - r.top)/r.height - 0.5) * 0.6;
    }
    wrap.addEventListener('mousemove', function(e){
      parallax(e.clientX, e.clientY);
      if (dragging){ dYaw += (e.clientX-lastX)*0.012; dPitch += (e.clientY-lastY)*0.012; lastX=e.clientX; lastY=e.clientY; }
    });
    wrap.addEventListener('mousedown', function(e){ dragging = true; lastX=e.clientX; lastY=e.clientY; });
    window.addEventListener('mouseup', function(){ dragging = false; });
    wrap.addEventListener('touchstart', function(e){ if(e.touches.length){ dragging=true; lastX=e.touches[0].clientX; lastY=e.touches[0].clientY; } }, {passive:true});
    wrap.addEventListener('touchmove', function(e){ if(e.touches.length && dragging){ var t0=e.touches[0]; dYaw += (t0.clientX-lastX)*0.012; dPitch += (t0.clientY-lastY)*0.012; lastX=t0.clientX; lastY=t0.clientY; } }, {passive:true});
    window.addEventListener('touchend', function(){ dragging = false; });

    var t = 0, running = true, _proj = new THREE.Vector3();
    HERO.setRunning = function(v){ running = v; if (v) loop(); };

    function projectLabel(node, labelEl){
      _proj.setFromMatrixPosition(node.matrixWorld);
      var inFront = _proj.z < 1;
      _proj.project(camera);
      var r = wrap.getBoundingClientRect();
      var x = (_proj.x * 0.5 + 0.5) * r.width;
      var y = (-_proj.y * 0.5 + 0.5) * r.height;
      labelEl.style.left = x + 'px';
      labelEl.style.top = (y - 16) + 'px';
      return inFront;
    }

    function loop(){
      if (!running) return;
      requestAnimationFrame(loop);
      t += 0.016;
      driftYaw += 0.0022;
      curYaw   += ((tYaw + dYaw) - curYaw) * 0.08;
      curPitch += ((tPitch + dPitch) - curPitch) * 0.08;
      group.rotation.y = driftYaw + curYaw;
      group.rotation.x = curPitch;
      coreHalo.scale.setScalar(3.2 + Math.sin(t*2)*0.3);
      core.scale.setScalar(1 + Math.sin(t*2)*0.08);
      for (var k = 0; k < nodes.length; k++){
        var u = nodes[k].userData;
        var a = u.ang + t * u.speed;
        var x = Math.cos(a)*u.rad, z = Math.sin(a)*u.rad, y = u.depth + Math.sin(t*0.5 + k)*0.15;
        nodes[k].position.set(x, y, z);
        var pos = lines[k].geometry.attributes.position;
        pos.array[0]=0; pos.array[1]=0; pos.array[2]=0;
        pos.array[3]=x; pos.array[4]=y; pos.array[5]=z;
        pos.needsUpdate = true;
      }
      group.updateMatrixWorld();
      for (var m = 0; m < nodes.length; m++){
        var inFront = projectLabel(nodes[m], HERO.labels[m]);
        HERO.labels[m].classList.toggle('show', inFront);
      }
      renderer.render(scene, camera);
    }
    loop();
```

- [ ] **Step 2: Verify in browser**

Open the page. Expected: construct idle-spins; nodes orbit at varying depth and pass in front of / behind the shell; connector lines track them; labels float at each node and hide when a node rotates to the back; click-drag spins the construct on any axis (horizontal=yaw, vertical=pitch); moving the mouse without dragging gives subtle parallax; no console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Animate hero construct: orbit, full 3D drag rotation, parallax, projected labels"
```

---

## Task 7: Scroll director — camera dolly out + pause render offscreen

As the hero scrolls out of view, slowly dolly/tilt the camera; pause the render loop entirely when the hero canvas is offscreen (battery/CPU), resume when it returns.

**Files:**
- Modify: `index.html` — extend the heroScene IIFE (add scroll handler + IntersectionObserver).

- [ ] **Step 1: Add scroll dolly + visibility pause**

After the `loop();` call at the end of the IIFE body (before the closing `})();`), add:
```javascript
    // --- scroll director: dolly the camera as hero leaves ---
    var baseZ = 6;
    window.addEventListener('scroll', function(){
      var r = wrap.getBoundingClientRect();
      var progress = Math.min(Math.max(-r.top / (r.height || 1), 0), 1); // 0 at top, 1 when scrolled one panel past
      camera.position.z = baseZ + progress * 3;       // dolly back
      camera.position.y = progress * 0.8;             // drift up
      camera.lookAt(0, 0, 0);
    }, { passive: true });

    // --- pause render when offscreen ---
    if ('IntersectionObserver' in window){
      var io = new IntersectionObserver(function(entries){
        entries.forEach(function(en){
          if (en.isIntersecting){ if (HERO.setRunning) HERO.setRunning(true); }
          else { if (HERO.setRunning) HERO.setRunning(false); }
        });
      }, { threshold: 0.01 });
      io.observe(wrap);
    }
```

- [ ] **Step 2: Verify in browser**

Open the page and scroll. Expected: as you scroll past the hero, the construct eases back/up (camera dolly); scrolling back restores it. Open DevTools Performance/JS — confirm the rAF loop stops firing while the hero is fully scrolled off (no `loop` activity), and resumes on return. No console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add scroll-linked camera dolly and offscreen render pause to hero"
```

---

## Task 8: First-visit boot intro (assemble + HUD typing, skippable)

On first visit only (localStorage flag), play a short boot sequence: the construct fades/assembles in and the hero meta line types in. Skippable by click/tap; auto-completes. Repeat visits skip straight to idle.

**Files:**
- Modify: `index.html` — add CSS for the intro states; add a `bootIntro` block that coordinates with the hero meta text; gate the construct opacity.

- [ ] **Step 1: Add intro CSS**

Add to the stylesheet:
```css
    .boot-typing::after { content: '_'; color: var(--accent); animation: blink 1s steps(1) infinite; }
    #heroCanvas { opacity: 0; transition: opacity 0.9s ease; }
    #heroCanvas.booted { opacity: 1; }
    .boot-skip {
      position: fixed; bottom: 1.25rem; right: 1.25rem; z-index: 1200;
      font-family: var(--font-mono); font-size: 0.6rem; letter-spacing: 0.12em; text-transform: uppercase;
      color: var(--text-dim); border: 1px dashed rgba(255,255,255,0.12); padding: 0.4rem 0.7rem;
      background: rgba(6,7,9,0.8); cursor: pointer;
    }
    .boot-skip:hover { color: var(--accent); border-color: var(--accent-dim); }
  </style>
```
(Note: this closes `</style>` — make sure you append the rules *before* the existing closing `</style>`, not adding a second one.)

- [ ] **Step 2: Capture the hero meta original text and add a skip control + boot script**

The hero meta element is `<span class="hero-meta">ESTABLISHED 2026 / CAPE TOWN</span>`. Add a new `<script>` block at the very end of `<body>` (after the heroScene script):
```html
  <script>
  (function(){
    var KEY = 'gi_booted_v1';
    var canvas = document.getElementById('heroCanvas');
    var meta = document.querySelector('.hero-meta');
    var reduce = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    var fullText = meta ? meta.textContent : '';

    function finishInstant(){
      if (canvas) canvas.classList.add('booted');
      if (meta) meta.textContent = fullText;
    }

    // Repeat visit or reduced motion: skip the show
    if (reduce || localStorage.getItem(KEY)){
      finishInstant();
      return;
    }

    // First visit: play it
    if (meta) meta.textContent = '';
    var skip = document.createElement('div');
    skip.className = 'boot-skip'; skip.textContent = 'Skip intro';
    document.body.appendChild(skip);

    var done = false;
    function finish(){
      if (done) return; done = true;
      localStorage.setItem(KEY, '1');
      if (canvas) canvas.classList.add('booted');
      if (meta){ meta.classList.remove('boot-typing'); meta.textContent = fullText; }
      if (skip.parentNode) skip.parentNode.removeChild(skip);
    }
    skip.addEventListener('click', finish);

    // Sequence: fade construct in, then type the meta line
    setTimeout(function(){ if (canvas) canvas.classList.add('booted'); }, 400);
    var idx = 0;
    if (meta) meta.classList.add('boot-typing');
    var typer = setInterval(function(){
      if (done){ clearInterval(typer); return; }
      idx++;
      if (meta) meta.textContent = fullText.slice(0, idx);
      if (idx >= fullText.length){ clearInterval(typer); setTimeout(finish, 300); }
    }, 45);

    // Safety auto-complete
    setTimeout(finish, 4000);
  })();
  </script>
```

- [ ] **Step 3: Verify first visit**

Clear the flag in DevTools console: `localStorage.removeItem('gi_booted_v1')`, then reload. Expected: construct fades in (~0.4s), the meta line types out character-by-character with a blinking caret, a "Skip intro" control sits bottom-right; clicking it (or waiting) completes immediately and the caret disappears.

- [ ] **Step 4: Verify repeat visit**

Reload again (flag now set). Expected: no typing, construct visible immediately, no skip control.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add first-visit boot intro: construct fade-in, HUD typing, skippable"
```

---

## Task 9: Resilience guards — reduced motion, mobile, WebGL failure

Make the hero degrade gracefully: respect `prefers-reduced-motion` (static construct, no loop), reduce cost on mobile (already pixel-ratio capped; also lower geometry detail + skip if very small), and keep the static SVG fallback when WebGL is unavailable.

**Files:**
- Modify: `index.html` — guard branches at the top of the heroScene IIFE.

- [ ] **Step 1: Add reduced-motion + WebGL capability guard at the top of heroScene**

Immediately after `if (!THREE || !canvas || !wrap) return;` in the heroScene IIFE, add:
```javascript
    var reduceMotion = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    function webglOK(){
      try { var c = document.createElement('canvas');
        return !!(window.WebGLRenderingContext && (c.getContext('webgl') || c.getContext('experimental-webgl')));
      } catch(e){ return false; }
    }
    if (!webglOK()){ return; } // leave static SVG fallback visible
```

- [ ] **Step 2: Lower geometry detail on small screens**

Change the icosahedron detail based on width. Find:
```javascript
    var icoGeo = new THREE.IcosahedronGeometry(1.7, 1);
```
Replace with:
```javascript
    var isSmall = window.innerWidth < 768;
    var icoGeo = new THREE.IcosahedronGeometry(1.7, isSmall ? 0 : 1);
```

- [ ] **Step 3: Branch the render: static frame for reduced motion, loop otherwise**

Find the `loop();` call (the first invocation that starts the animation, at the end of the interaction block) and replace it with:
```javascript
    if (reduceMotion){
      group.updateMatrixWorld();
      for (var s = 0; s < nodes.length; s++){ projectLabel(nodes[s], HERO.labels[s]); HERO.labels[s].classList.add('show'); }
      renderer.render(scene, camera);
      running = false; // no animation loop
    } else {
      loop();
    }
```

- [ ] **Step 4: Verify reduced motion**

In DevTools, emulate `prefers-reduced-motion: reduce` (Rendering tab), clear `localStorage`, reload. Expected: construct renders a single static frame (no spin, no orbit), labels shown; boot typing skipped; no console errors.

- [ ] **Step 5: Verify WebGL-failure fallback**

In DevTools, block `three.min.js` (Network → block request URL) and reload. Expected: the static cyan SVG construct fallback stays visible; page fully functional; no uncaught errors.

- [ ] **Step 6: Verify mobile**

Toggle device toolbar to a phone viewport, reload. Expected: construct renders with lower detail, smooth; layout intact.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Add resilience guards: reduced-motion, mobile detail, WebGL fallback"
```

---

## Task 10: Cinematic reveals across the rest of the page

Extend the existing `reveal-on-scroll` system with 3D depth: services cards tilt-in on a perspective, and showcase cards get a focus/recede effect where hovering one sharpens it and dims/pushes back the others. Pure CSS + the existing observer; respects reduced motion.

**Files:**
- Modify: `index.html` — add CSS for 3D reveal + showcase focus; add classes to the services/showcase containers; small JS to toggle a `motion-ok` flag.

- [ ] **Step 1: Add a motion-ok gate and 3D reveal CSS**

Add to the stylesheet:
```css
    /* Cinematic 3D reveals (only when motion allowed) */
    .motion-ok .services-grid, .motion-ok .showcase-grid { perspective: 1200px; }
    .motion-ok .reveal-on-scroll.tilt3d {
      transform: translateY(40px) rotateX(12deg) scale(0.96);
      transform-origin: center bottom;
    }
    .motion-ok .reveal-on-scroll.tilt3d.active {
      transform: translateY(0) rotateX(0deg) scale(1);
    }
    /* Showcase focus/recede */
    .motion-ok .showcase-grid:hover .showcase-card { transition: transform 0.4s ease, opacity 0.4s ease, filter 0.4s ease; }
    .motion-ok .showcase-grid:hover .showcase-card:not(:hover) {
      transform: scale(0.97); opacity: 0.55; filter: saturate(0.6);
    }
    .motion-ok .showcase-grid:hover .showcase-card:hover {
      transform: scale(1.015) translateZ(20px);
    }
```

- [ ] **Step 2: Set the motion-ok flag (respect reduced motion)**

Add a small `<script>` at the end of `<body>`:
```html
  <script>
  (function(){
    var reduce = window.matchMedia && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    if (!reduce) document.body.classList.add('motion-ok');
  })();
  </script>
```

- [ ] **Step 3: Add the `tilt3d` class to the service + showcase reveal wrappers**

On each services card reveal wrapper (`<div class="reveal-on-scroll">` and `<div class="reveal-on-scroll delay-1">` inside `.services-grid`) and each showcase card reveal wrapper (`.showcase-card.reveal-on-scroll...`), add the `tilt3d` class. Example — find:
```html
        <div class="reveal-on-scroll">
          <div class="blueprint-card">
```
within the services grid and change the first line to:
```html
        <div class="reveal-on-scroll tilt3d">
          <div class="blueprint-card">
```
Apply the analogous `tilt3d` addition to all 2 service-card wrappers and all 4 showcase-card wrappers (do not alter the telemetry or hero reveals).

- [ ] **Step 4: Verify in browser**

Open and scroll to Services and Showcase. Expected: service/showcase cards tilt up out of perspective depth as they enter (not just fade); hovering one showcase card sharpens it while the others dim and recede; reduced-motion emulation disables both (cards just appear). No console errors; no layout shift on mobile (perspective only adds depth).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add cinematic 3D reveals: perspective tilt-in and showcase focus/recede"
```

---

## Task 11: Final QA pass + cleanup

Remove any now-dead code, confirm structure validity, and run the full manual QA checklist.

**Files:**
- Modify: `index.html` — remove dead CSS for the old `.schematic-svg`/`.svg-node`/`.svg-link` rules if no longer referenced; verify tag balance.

- [ ] **Step 1: Remove dead SVG-schematic CSS**

Search for `.svg-node`, `.svg-link`, `.schematic-svg`, `@keyframes flowPath` in the stylesheet. The hero SVG that used them was removed in Task 3. If no remaining markup references them (confirm with a grep for `class="svg-node"` etc. → no matches), delete those now-unused CSS rules. Keep `.schematic-box` and `.schematic-corner` (still used as the hero frame around the canvas).

- [ ] **Step 2: Tag-balance sanity check**

Run:
```bash
grep -c "<script" index.html; grep -c "</script>" index.html
grep -c "<div" index.html; grep -c "</div>" index.html
```
Expected: `<script` count == `</script>` count; `<div` count == `</div>` count.

- [ ] **Step 3: Confirm no stale references**

Run:
```bash
grep -niE "BFFF00|wa\.me|\+27|whatsapp interface|7A00FF" index.html
```
Expected: no matches (the only WhatsApp text allowed is the Stack Burger showcase copy/tag, which describes that demo — confirm those are the only hits if any).

- [ ] **Step 4: Full manual QA checklist (browser)**

Confirm all of the following with no console errors:
  - Desktop: hero construct renders, idle-spins, drag rotates on any axis, nodes orbit with labels, parallax works.
  - Scroll: camera dollies as hero leaves; rAF pauses offscreen; services/showcase cinematic reveals fire; showcase focus/recede works.
  - First visit (`localStorage.removeItem('gi_booted_v1')` + reload): boot intro plays, skippable.
  - Repeat visit: no intro.
  - Reduced motion: static construct, no intro, reveals disabled.
  - WebGL blocked: static SVG fallback visible.
  - Mobile viewport: lower-detail construct, smooth, layout intact, contact shows email only.
  - Contact section: business email only, no phone.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Final QA: remove dead schematic CSS, verify structure and palette"
```

---

## Self-Review Notes (author)

- **Spec coverage:** hero construct (T3–T6), nodes/labels (T5–T6), full 3D rotation (T6), scroll dolly + offscreen pause (T7), boot intro (T8), cyan palette (T2), cinematic-throughout reveals (T10), guards/reduced-motion/WebGL/mobile (T9), phone removal (T1), cleanup/verify (T11). All spec sections mapped.
- **No test runner:** intentional — static site; verification is browser-based + structural greps, stated up front.
- **Deferred-state note:** T1 commits an already-applied edit; T2 Step 3 keeps the page consistent even if T3 is deferred.
