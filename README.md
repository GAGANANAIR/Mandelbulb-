# Bulb

A raymarched fractal instrument — 3D fractals rendered live in the browser using WebGL, by sphere-tracing a signed distance field. No polygons, no mesh: every pixel is computed directly from a distance function evaluated on the GPU.

**[View it live](https://gagananair.github.io/Mandelbulb-/)**

## What it does

- Renders **6 different fractal types**, switchable live (see below)
- Real-time controls: **Power** (reshapes the fractal), **Depth** (iteration count vs. speed), **Hue** (color palette), **Bloom**, **Depth of Field**
- Multi-pass rendering pipeline: scene pass → bloom (bright-pass + blur) → depth-of-field composite → filmic tonemap
- Soft shadows, ambient occlusion, and orbit-trap-driven coloring for a moody, cinematic look
- Mouse/touch orbit camera with scroll/pinch zoom
- Auto-drift camera motion (toggleable, or hit Space)
- **Microphone-reactive mode** — bass, mid, and treble subtly drive Power, Hue, and Bloom in real time
- **Save** button — downloads the current view as a PNG
- **Share** button — copies a URL with your exact camera angle and settings baked in
- Adaptive render scale — automatically trades resolution for frame rate to stay smooth on any GPU
- Live telemetry: rays per frame, average march steps, frame time, render scale, mic status

## Fractal modes

Click the mode button (bottom-right of the control deck) to cycle through:

| Mode | What it is |
|---|---|
| **Bulb** | Classic Mandelbulb — spherical-coordinate power iteration, adjustable exponent |
| **Julia** | Julia-set variant of the same formula (fixed additive constant instead of the sample position) — exposes extra **Julia C.x / C.y** sliders |
| **Box** | Mandelbox — alternating box-fold and sphere-fold |
| **Bulb2** | Power-2 Mandelbulb — smoother, rounder lobes than the default power-8 look |
| **Menger** | Menger Sponge — cube repeatedly carved with cross-shaped notches at increasing scale |
| **Kifs** | Kaleidoscopic IFS — space folded into a single wedge and scaled outward, producing intricate lattice structures |

## Running it

Just open `index.html` in any modern browser with WebGL support — no build step, no dependencies.

```bash
git clone https://github.com/GAGANANAIR/Mandelbulb-.git
cd Mandelbulb-
open index.html   # or just double-click it
```

## Controls

| Action | Effect |
|---|---|
| Drag | Orbit the camera |
| Scroll / pinch | Zoom in and out |
| Power slider | Changes the fractal's exponent / scale (its "bulbiness") |
| Depth slider | Trades rendering detail for performance |
| Hue slider | Shifts the color palette between warm and cool |
| Bloom slider | Controls how strongly bright areas glow |
| Depth of Field slider | Blurs areas away from the focal point |
| Mode button | Cycles through the 6 fractal types |
| Drift button (or Space) | Toggles automatic camera rotation |
| Reset button (or R) | Restores default camera and parameters |
| Save button | Downloads the current frame as a PNG |
| Mic button | Enables microphone-reactive visuals |
| Share button | Copies a URL with the current view's exact settings |

**Images**

<img width="946" height="410" alt="image" src="https://github.com/user-attachments/assets/dbb365fa-986b-4791-8f0b-8bb2ab60905b" />

<img width="931" height="404" alt="image" src="https://github.com/user-attachments/assets/8be29be9-39ad-4bf8-a3f9-45591a9bd8e8" />

## How it works

Each fractal is defined by an iterative formula (or, for Menger/Kifs, a space-folding operation), evaluated per pixel with no mesh at all. Instead of building geometry, the fragment shader marches a ray from the camera through 3D space, using a distance estimator (DE) to know how far it can safely step forward without missing the surface — this is sphere tracing. When the ray gets close enough to the surface, that point is shaded using a normal estimated from the DE gradient, plus soft shadows and ambient occlusion sampled the same way. Color comes from an orbit-trap accumulator tracked during each fractal's iteration, so hue follows the fractal's internal structure rather than a flat distance ramp.

The render pipeline runs in four passes: (1) render the scene with depth packed into the alpha channel, (2) extract bright pixels and blur horizontally, (3) blur vertically to finish the bloom, (4) composite everything together with a depth-of-field blur and a filmic tonemap.

## The Math

The Mandelbulb is the 3D analogue of the Mandelbrot set. Instead of iterating $z \leftarrow z^2 + c$ over complex numbers, we iterate over $\mathbb{R}^3$ using a spherical-coordinate generalization of the power operation.

**Iteration.** For a point $c \in \mathbb{R}^3$, starting from $z_0 = c$:

$$z_{n+1} = z_n^{\,p} + c$$

where the power operation $z^p$ is defined by converting $z = (x, y, z)$ to spherical coordinates,

$$r = |z|, \qquad \theta = \cos^{-1}\!\left(\frac{z_z}{r}\right), \qquad \phi = \text{atan2}(z_y, z_x)$$

then scaling the radius and multiplying the angles by the power $p$:

$$z^{p} = r^{\,p} \big(\sin(p\theta)\cos(p\phi),\ \sin(p\theta)\sin(p\phi),\ \cos(p\theta)\big)$$

A point $c$ belongs to the set if this sequence stays bounded (in `index.html` the loop bails out once $r > 2.2$). The **Power** slider is literally the exponent $p$ in this formula — at $p=8$ you get the "classic" Mandelbulb. The **Julia** mode uses the same formula but replaces $c$ (the additive term) with a fixed point, and the **Bulb2** mode locks $p = 2$ for a smoother look regardless of the slider.

**Sphere tracing needs a distance, not just in/out.** Since there's no mesh, the shader needs to know how far it can safely march the camera ray without stepping through the surface. This comes from a *distance estimator* (DE) derived from how fast the iteration's derivative grows. Alongside $z_n$, the shader tracks a running derivative magnitude $dr$:

$$dr_0 = 1, \qquad dr_{n+1} = p \, r_n^{\,p-1} \, dr_n + 1$$

and after $N$ iterations (or an early escape), the estimated distance to the fractal surface is:

$$\text{DE}(c) \approx \frac{1}{2} \, \frac{r_N \ln r_N}{dr_N}$$

That's the exact `0.5 * log(r) * r / dr` line in `mapDE()`. Sphere tracing then repeatedly steps the ray forward by this safe distance — never overshooting the surface, never wasting steps in empty space — until it converges (or the ray runs out of steps/distance).

**Menger Sponge** works differently: instead of an escape-time iteration, space itself is folded with `mod()` at increasing scale, and each fold carves a cross-shaped notch out of the current box using a standard box signed-distance function. **Kaleidoscopic IFS** folds space into a single symmetric wedge using `abs()` and axis swaps, then repeatedly scales outward from a fixed offset — a handful of these folds stacked together is enough to produce an intricate, self-similar lattice.

**Surface normals**, needed for shading, shadows, and ambient occlusion, come for free from the same DE: they're just its numerical gradient, sampled with tiny offsets around the hit point.

## Author

**Gagan A Nair**
- [Profile](https://gagagananair.netlify.app/)
- gagananair1@gmail.com
