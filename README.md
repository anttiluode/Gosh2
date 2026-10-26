# ANTTI'S BRAIN — FLIES

The dumbflies left the webcam and moved into Antti's Brain.

One HTML file, no build, no dependencies. Open `index.html` in a WebGL2 browser, allow the camera, click in.

Live (GitHub Pages): https://anttiluode.github.io/Gosh2/

Do not hype. Do not lie. Just show.

---

## What it is

**The world** is the AnttisBrain2 descent engine, unchanged. It is an endless chain of hash-dreamed chambers. Crystal moons lens the light and act as mirrors in four optical modes. Your webcam is on the walls. A gold key crystal opens the red seal to the next level.

**The flies** are `dumbflies.py` (the webcam "Evolving Flies Simulation"), ported whole into that world.

* **Each fly has a real eye.** Every 1/30 s all fly eyes are rendered by the same raymarcher you see through, one 32×32 tile per fly, read back from the GPU without blocking. A fly therefore sees what the world actually looks like from where it is:
  * the walls (your camera),
  * the moons, lensing and mirroring,
  * the other flies,
  * and **you** — you carry an orb that wears your face.
* **The moons bend the flies by the same law that bends the light.** The heading change per step is `Σ m·dv/|dv| / (|dv|²+0.4) · ds · 0.055 · lensing`. That is the shader's ray-deflection formula, applied to the fly's flight path, so flies swing around heavy crystals. The *Lensing* slider affects both the light and the flies.
* **The mirrors multiply them.** The population is capped at 32 (default 20), because every fly costs shader time per pixel. In Boundary mode each fly also appears inside every moon. In Projection and Cascade mode the walls repeat them until the bounces run out.

## The mind is not simplified

Everything in `dumbflies.py`'s mind is here, with its original constants:

| dumbflies.py | here |
|---|---|
| 3 × `ThinkingField(32×32)`: perception 0.6, memory 0.5, decision 0.55 thresholds | same |
| `apply_input`: min-max normalise, add ×0.1 | same |
| `evolve`: drift 0.01 → 0.5, gaussian kicks sd 0.05 blurred σ=1, decay 0.01, clip [0,1] | same, including scipy's `reflect` edges and truncate-4 kernel |
| `detect_patterns`, 3×3 +0.02 `associate_patterns` chain P→M→D | same |
| decision centroid → direction; density > 0.1 eat, > 0.2 mate | same |
| energy: −0.05/frame; eat = brightness×2 + motion×10, capped 5; max 150 | same, computed on the fly's own eye |
| mating: both >80 energy, cooldown 0, 1-in-100 chance, −40 each, cooldown 100 | same |
| child: three fields = mean of parents' fields; traits inherited with 5% mutation and the original clamps | same |
| repulsion 30 px / force 2, detection 100 px, hover, random direction changes, momentum 0.5, π/8 turn clip | same, scaled to world units |
| min 5 / initial population, repopulate, birth ring, death dot, save / load population | same (birth rings and death dots now in 3D) |

There is a live panel showing the chosen fly's eye and all three fields as they evolve, plus its energy, traits, generation and eating/mating state.

## What changed, and why

These are changes forced by moving from a 2D frame to a 3D world. They are listed so nobody mistakes them for the original:

1. **The eye.** In `dumbflies.py` the "vision cone" was the *whole* frame masked by a triangle and resized to 32×32. The field was a downsampled world map, and the decision centroid was measured from the *frame's* centre. In 3D there is no frame, so each fly gets a true eye: a circular cone (its `vision_cone_angle` trait), with reach set by its `vision_cone_length` trait. The decision centroid's offset from the eye's centre becomes the turn it wants: yaw for left/right, pitch for up/down.
2. **Pitch.** 2D flies only had an angle. Here the original random-direction rule drives pitch as well, at half strength, clamped to ±63°.
3. **Edges.** Flies used to wrap around the screen edge. A brain has walls, so they bounce off walls and glance off moons. Flies stranded in a room that drops out of the live chain are re-placed in the room behind you.
4. **Scale.** One dumbflies pixel = 0.014 world units of speed (a fly crosses a room in about 3 s, as before) and 0.03 units of spacing. Bodies are drawn about 4× larger than that so you can see them.
5. **Two quirks kept on purpose.** `inherit_traits` is called with both parents in turn, so the second call wins: a child carries the partner's mutated traits, plus a true blend of both parents' fields. And the `turn_speed` trait mutates but the code always steers with the config value. Both behave exactly as in the original.
6. **Wings beat faster** than the original's slow `time*3` sweep; amplitude keeps the original rule (0.8 mating, 0.5 not). Compound eyes were added to the head.

## Not verified

* It has only been run under a software GPU (SwiftShader) here, at tiny resolutions. Real-GPU frame rate is unmeasured. The fly loop adds per-pixel cost on top of AnttisBrain2. If the frame rate drops below 30 the render scale lowers itself (shown next to the fps), down to 50%.
* Population behaviour over long runs is unstudied. Like the original, flies that look at bright, moving things eat well and tend to breed up to the cap.

## Controls

* **W A S D** move · **Q / E** down / up · **Shift** boost · **drag** to look (horizontal inverted, as tradition demands)
* **F** pick the fly nearest the centre of your view (or the next one)
* **R** ride the chosen fly (chase camera; press again to step off where it is)
* **H** hide all panels (for recording)
* **Modes:** Boundary · Projection · Cascade · Glass, as in AnttisBrain2
* **Fly sliders:** max population, energy decay, max speed (for new flies, as in dumbflies)
* **Save / Load flies:** whole minds as JSON. A `dumbflies.py` population save also loads.
* **Save / Save code / Load:** the descent itself (six numbers, as in AnttisBrain2)

Music: put `game1.mp3` … `game10.mp3` next to `index.html` (the AnttisBrain soundtrack) and it plays on first click.

## Lineage

* [`AnttisBrain2`](https://github.com/anttiluode/AnttisBrain2) — the engine.
* `dumbflies.py` (~2025) — the flies.
* [`FlyBench`](https://github.com/anttiluode/FlyBench) — the research bench built from the flies.
