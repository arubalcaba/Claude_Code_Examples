# Parallax — AI Landing Page Design Spec

## Overview

A single-file landing page for "Parallax," a computer vision / generative AI company. The page features a 3D animated hero section with a neural mesh rendered via Three.js and custom GLSL dither shaders, followed by a showcase-focused marketing flow.

**Tech stack:** Three.js, WebGL, GSAP (with ScrollTrigger), GLSL shaders — all loaded via CDN. Single `index.html` file, no build tools.

**Aesthetic:** Dark mode, futuristic, clean. Cyan/electric blue (#00e5ff) as the primary accent on a near-black (#0a0a0f) background. Ordered dithering used as a deliberate premium design texture, not a retro effect.

---

## Section 1: Navigation

A fixed top bar overlaid on all sections.

- **Layout:** Logo left, nav links center ("Capabilities", "Gallery", "Testimonials"), "Get Started" button right.
- **Hero state:** Fully transparent background.
- **Scrolled state:** Transitions to `rgba(10, 10, 15, 0.85)` with `backdrop-filter: blur(10px)` once the user scrolls past the hero. Transition animated with CSS.
- **Typography:** Geometric sans-serif (Inter or similar via Google Fonts). Logo in bold monospace.
- **Mobile:** Hamburger menu icon replacing center links at ≤768px.

---

## Section 2: Hero (100vh)

The visual centerpiece — a full-screen 3D neural mesh with dithered surfaces.

### 3D Scene

- **Geometry:** `THREE.IcosahedronGeometry` with detail level 3–4 (~320 triangular faces). Vertex positions displaced with 3D simplex noise for organic irregularity.
- **Wireframe layer:** A second mesh using `THREE.LineSegments` with `LineBasicMaterial` (cyan, low opacity) rendered on top of the solid geometry to create the lattice/network look.
- **Node points:** Small `THREE.Points` at each vertex with a custom point shader that renders glowing cyan dots.

### Dither Shader (GLSL)

- **Vertex shader:** Standard model-view-projection transform. Passes world-space position, normal, and UV to fragment shader.
- **Fragment shader:**
  - Computes basic diffuse + ambient lighting (single directional light from upper-right).
  - Applies an **8×8 Bayer matrix** ordered dither: compares the computed brightness at each fragment against the Bayer threshold at that screen pixel. Fragments above threshold render as the lit cyan color; below threshold render as near-black.
  - The dither threshold offset animates slowly over time (`uniform float u_time`), creating a subtle "breathing" pulse across the surface.
  - Color palette: lit pixels are `#00e5ff` (cyan) at varying brightness, unlit pixels are `#0a0a0f` (background).

### Mouse Interaction

- Mouse position is normalized to `[-1, 1]` range and passed as a uniform.
- The mesh rotates toward the mouse position with a max deflection of ±15° on each axis.
- Smooth interpolation via `THREE.MathUtils.lerp` with a factor of 0.05 per frame for fluid feel.
- **Mobile fallback:** Slow continuous auto-rotation (0.1 rad/s on Y axis, 0.05 on X).

### Post-Processing

- **Bloom:** `THREE.UnrealBloomPass` with low strength (0.3–0.5), targeting the bright node points and edge highlights.
- **Vignette:** A CSS radial gradient overlay on the canvas container (simpler than a shader pass, same visual result).
- No motion blur — dither patterns must remain crisp.

### Text Overlay

- HTML elements positioned absolutely over the canvas, centered vertically and horizontally.
- **Company name:** "PARALLAX" — bold, letter-spacing 6px, font-size ~48px, white.
- **Tagline:** "Vision Beyond Pixels" — cyan (#00e5ff), font-size ~16px, letter-spacing 3px.
- **CTA button:** "Get Started" — cyan border, transparent fill, fills cyan on hover with dark text. Rounded corners (6px).
- **Readability:** `text-shadow: 0 0 30px rgba(0, 0, 0, 0.8)` on all text elements.

### Scroll Transition

- As the user scrolls past the hero, GSAP ScrollTrigger fades the text overlay out and scales the mesh down slightly (0.95) with increased rotation, creating a "pulling away" effect.

---

## Section 3: Capabilities Showcase

A 4-card grid showcasing core capabilities.

### Layout

- Section label: "WHAT WE DO" in small cyan uppercase text, centered.
- Section heading: "Capabilities" in white, centered below label.
- **Grid:** 2×2 on desktop (max-width 900px, centered), stacking to 1-column on mobile.

### Cards

Each card contains:
1. **Icon** — Simple SVG icon in a rounded square container with a subtle cyan gradient background.
2. **Title** — Bold white text (e.g., "Image Synthesis").
3. **Description** — Gray secondary text, 1–2 lines.

Cards have a dark background (#0d1117), thin cyan-tinted border (`#00e5ff22`), 8px border radius.

### The Four Capabilities

1. **Image Synthesis** — "Generate photorealistic imagery from text descriptions with pixel-level control."
2. **Scene Understanding** — "Deep semantic parsing of visual scenes — objects, relationships, and context."
3. **Style Transfer** — "Apply and blend artistic styles across images while preserving structural integrity."
4. **Real-Time Processing** — "Sub-frame latency inference for live video streams and interactive applications."

### Animation

- Cards fade-up and stagger-in on scroll via GSAP ScrollTrigger (`y: 40 → 0`, `opacity: 0 → 1`, stagger: 0.15s).
- On hover: card border brightens to `#00e5ff44`, subtle `box-shadow: 0 0 20px rgba(0, 229, 255, 0.1)`.

---

## Section 4: Visual Demo Gallery

A stacked vertical gallery of generated visual demos.

### Layout

- Section label: "GALLERY" in small cyan uppercase text, centered.
- Section heading: "See What's Possible" in white, centered.
- **Gallery items** stacked vertically, max-width 700px, centered. 3 items.

### Gallery Items

Each item is a card with:
1. **Image area** (top) — Placeholder area with a dither-dot CSS pattern. In production, this would display actual generated imagery.
2. **Title** — Bold white text (e.g., "Architectural Visualization").
3. **Description** — Gray secondary text, one line.

Dark card background with subtle gradient, thin border.

### The Three Demos

1. **Architectural Visualization** — "Text-to-scene rendering with photorealistic lighting"
2. **Portrait Enhancement** — "AI-driven retouching that preserves natural character"
3. **Video Synthesis** — "Frame-consistent generation for motion content"

### Animation

- Each gallery item reveals on scroll via GSAP ScrollTrigger. The image area starts with a heavy dither-dot CSS pattern at full opacity and the content beneath at zero opacity. On trigger, the dither overlay fades out (`opacity: 1 → 0`, duration 0.8s) while the content beneath fades in, creating a "resolving from noise" effect.
- Staggered timing (0.2s between items) so they cascade down the page.

---

## Section 5: Testimonials

Minimal quote cards from satisfied teams.

### Layout

- Section label: "TRUSTED BY" in small cyan uppercase text, centered.
- Section heading: "What Teams Are Saying" in white, centered.
- **3 cards** in a horizontal row on desktop, stacking vertically on mobile.

### Card Structure

1. **Quote mark** — Large cyan opening quote character.
2. **Quote text** — Light gray, 14px, 1.6 line-height.
3. **Divider** — Thin horizontal line.
4. **Name** — White, bold.
5. **Title + Company** — Gray secondary text.

Dark background (#0d1117), very subtle border (`#ffffff0a`), 8px radius.

### The Three Testimonials

1. **Sarah Chen**, CTO at StudioForge — "Parallax cut our rendering pipeline from hours to seconds. The quality is indistinguishable from manual work."
2. **Marcus Rivera**, VP Engineering at StreamLab — "The real-time processing API changed what's possible for our live broadcast platform."
3. **Aiko Tanaka**, Lead ML Engineer at Voxel — "Best-in-class scene understanding. We integrated it in a weekend and haven't looked back."

### Animation

- Cards slide up from below with GSAP stagger (0.15s).
- Subtle cyan glow pulse on the quote mark (CSS animation, infinite loop, gentle).

---

## Section 6: CTA Footer

Final call-to-action and footer links.

### CTA Area

- Heading: "Ready to See Beyond Pixels?" — white, 28px, bold, centered.
- Subtext: "Join the teams building the future of computer vision." — gray, centered.
- **Two buttons** side by side, centered:
  - "Get Started" — cyan border, fills on hover (primary).
  - "View Docs" — subtle gray border (secondary).
- Background: faint dither-dot pattern (CSS) that subtly animates, echoing the hero.

### Footer

- Thin top border divider.
- Left: "© 2026 Parallax"
- Right: Social links — Twitter, GitHub, Blog.
- All in gray secondary text, minimal.

---

## Global Design Tokens

| Token | Value |
|---|---|
| Background | `#0a0a0f` |
| Surface | `#0d1117` |
| Primary accent | `#00e5ff` |
| Text primary | `#ffffff` |
| Text secondary | `#8892a4` |
| Text tertiary | `#c0c0d0` |
| Border subtle | `#ffffff0a` |
| Border accent | `#00e5ff22` |
| Border accent hover | `#00e5ff44` |
| Font body | Inter (Google Fonts) |
| Font mono/logo | JetBrains Mono or similar monospace |
| Border radius | 6–8px |

---

## CDN Dependencies

- **Three.js** (r160+) — 3D rendering, geometry, materials, post-processing
- **GSAP** (3.12+) + **ScrollTrigger plugin** — scroll-driven animations
- **Google Fonts** — Inter + monospace font for logo

No npm, no bundler, no build step.

---

## Responsive Behavior

- **Desktop (>1024px):** Full experience — mouse-reactive 3D, 2×2 capability grid, 3-column testimonials.
- **Tablet (768–1024px):** 2-column capability grid, 2-column testimonials, 3D scene rendered at reduced resolution.
- **Mobile (<768px):** Single column throughout, hamburger nav, 3D mesh auto-rotates (no mouse tracking), reduced geometry detail level (2 instead of 3–4) for performance.

---

## Performance Considerations

- Three.js renderer uses `antialias: false` (dithering replaces the need for AA).
- Canvas resolution capped at `devicePixelRatio` of 2 maximum.
- GSAP animations use `will-change` and GPU-composited properties only (transform, opacity).
- 3D scene pauses rendering when not visible (IntersectionObserver on the hero section).
- Gallery images use lazy loading.
