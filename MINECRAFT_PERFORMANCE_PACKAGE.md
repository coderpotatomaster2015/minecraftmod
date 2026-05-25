# Minecraft Java Performance Optimization Package (Fabric)

## 1) Resource Pack File Tree (Low-Visual-Overhead Pack)

```text
fps-pack-lite/
├─ pack.mcmeta
├─ pack.png
└─ assets/
   └─ minecraft/
      ├─ textures/
      │  ├─ block/
      │  │  ├─ stone.png
      │  │  ├─ dirt.png
      │  │  ├─ grass_block_top.png
      │  │  ├─ oak_planks.png
      │  │  ├─ water_still.png
      │  │  ├─ water_flow.png
      │  │  ├─ lava_still.png
      │  │  ├─ lava_flow.png
      │  │  └─ ...
      │  ├─ item/
      │  │  ├─ iron_sword.png
      │  │  ├─ diamond_pickaxe.png
      │  │  └─ ...
      │  ├─ entity/
      │  │  ├─ zombie/
      │  │  │  └─ zombie.png
      │  │  ├─ creeper/
      │  │  │  └─ creeper.png
      │  │  └─ ...
      │  ├─ gui/
      │  │  ├─ icons.png
      │  │  ├─ widgets.png
      │  │  ├─ options_background.png
      │  │  └─ ...
      │  └─ particle/
      │     ├─ particles.png
      │     └─ ...
      ├─ optifine/   # optional; omit if pure vanilla/Fabric compatibility is desired
      └─ font/
         └─ default.json
```

> Build two variants:
> - `fps-pack-lite-8x` (safer readability)
> - `fps-pack-ultra-4x` (maximum GPU/VRAM reduction)

---

## 2) `pack.mcmeta`

Use the correct `pack_format` for your target Minecraft version.

```json
{
  "pack": {
    "pack_format": 34,
    "description": "FPS Pack Lite: 8x/4x textures, reduced visual noise"
  }
}
```

If targeting a different version, update only `pack_format` accordingly.

---

## 3) Texture Replacement Strategy (Resource Pack Layer)

## Core Rule
Resource pack changes are **visual only**. They do **not** alter hotkeys, UI scale behavior, CPU logic, AI, or tick logic.

## Strategy by category

### A. Block textures (major GPU/VRAM impact)
- Downscale common blocks from 16x to 8x (or 4x): stone, dirt, logs, leaves, ores, deepslate, netherrack, end stone.
- Keep strong contrast edges for readability in caves.
- Remove fine noise from repeating textures (reduces shimmering/aliasing perception).

### B. Item textures
- Replace detailed gradients with flat-color silhouettes + one highlight tone.
- Preserve unique item identity by color coding (tools/ores/food).

### C. Entity textures
- Simplify mob skins to fewer color zones.
- Keep eyes/high-contrast features so hostile mobs stay recognizable at distance.

### D. GUI textures
- Simplify `widgets.png`, `icons.png`, inventory backgrounds.
- Reduce ornamentation and alpha-heavy overlays.

### E. Particles and animations
- Optional “low-noise” variant:
  - static water/lava/fire textures (or very low-frame animation)
  - reduced-contrast particle sheet
- Keep critical gameplay particles (hit, redstone feedback) visible but minimal.

### F. Fonts (optional)
- Keep default font for readability unless your audience accepts compact pixel fonts.

---

## 4) Fabric Mod List (Performance Core)

Required core mods:

1. **Sodium**
   - Primary rendering engine optimization.
   - Improves frame pacing, chunk mesh/render pipeline efficiency.
   - Main FPS booster (GPU + render thread).

2. **Lithium**
   - Optimizes game physics, AI/pathing, block ticking internals.
   - Reduces CPU load and MSPT spikes.

3. **FerriteCore**
   - Reduces memory footprint of block states/models and related structures.
   - Lowers RAM usage and GC pressure.

4. **Starlight**
   - Replaces/optimizes lighting engine behavior.
   - Faster chunk lighting updates and reduced chunk-gen stutter.

5. **Entity Culling**
   - Skips rendering entities hidden behind walls/occlusion.
   - Cuts unnecessary GPU/CPU render work in caves/bases.

6. **ImmediatelyFast**
   - Optimizes immediate-mode and GUI-related rendering paths.
   - Improves UI and certain world-render hotspots.

Optional helper:

7. **Sodium Extra** (optional)
   - Adds advanced Sodium-tunable options for quality/perf tradeoffs.

Debug/utility optional:

8. **MiniHUD** (optional)
   - Adds technical overlays (useful while tuning).

9. **Debugify** (optional)
   - Fixes vanilla inconsistencies/bugs that can affect debugging quality.

---

## 5) Optimized In-Game Settings (Configuration Layer)

Recommended baseline for weak-to-mid hardware:

- **Render Distance:** `8–12 chunks`
  - Lower chunk draw volume = less GPU fill + fewer chunk meshes.

- **Simulation Distance:** `4–6 chunks`
  - Major CPU savings; fewer entities/redstone/ticks simulated.

- **Graphics Mode:** `Fast` (or lowest equivalent in Sodium options)
  - Reduces expensive visual effects.

- **Particles:** `Minimal`
  - Reduces particle count and transparency overdraw.

- **Entity Distance:** `50%` (or lower if needed)
  - Fewer entities processed for rendering.

- **Biome Blend:** `0–2 chunks`
  - Lower color blending radius reduces per-frame terrain color processing.

- **VSync:** `Off` (default performance setting)
  - Removes refresh lock; can raise FPS ceiling and reduce latency.
  - If severe tearing occurs, enable VSync or external frame cap.

- **Clouds:** `Off`
- **Shadows (if exposed by mod settings):** `Low/Off`
- **Mipmap Levels:** `0–2` (use `0` on very weak GPUs)

Practical tuning workflow:
1. Start with low settings above.
2. Raise render distance by +2 until FPS drops below target.
3. Keep simulation distance low unless gameplay needs larger redstone/entity range.

---

## 6) Debug / Pie Chart Layer

### Open profiler
- Press **F3 + Shift** in-game to open the pie chart profiler.

### What it measures
- Real-time breakdown of frame/tick workload categories (rendering, game loop sections, etc.).
- Useful for spotting whether bottleneck is render-heavy vs tick-heavy.

### How to interpret quickly
- If rendering/chunk sections dominate: lower render distance, entity distance, particles; verify Sodium settings.
- If tick/entity-related sections dominate: lower simulation distance; reduce mob density and farms; verify Lithium is loaded.
- If stutters align with chunk updates/lighting: verify Starlight and reduce exploration speed on weak CPUs.

### Important limitation
- **Pie chart resizing is not supported in vanilla Minecraft.**
- To change debug UI behavior/visibility, you need mods or a custom UI rendering overhaul.

---

## 7) Installation Steps

1. **Install Fabric Loader** for your Minecraft version.
2. **Install Fabric API** (if required by selected mods).
3. Put performance mods in `.minecraft/mods/`:
   - Sodium
   - Lithium
   - FerriteCore
   - Starlight
   - Entity Culling
   - ImmediatelyFast
   - (optional) Sodium Extra, MiniHUD, Debugify
4. Put your resource pack folder/zip in `.minecraft/resourcepacks/`.
5. Launch Minecraft with Fabric profile.
6. Enable the resource pack in Options → Resource Packs.
7. Apply the optimized video/settings profile listed above.
8. Join a representative world and validate with F3 metrics + pie chart.

---

## 8) Troubleshooting

- **Game crashes at launch**
  - Confirm all mods match exact Minecraft + Fabric Loader version.
  - Remove one optional mod at a time to isolate incompatibility.

- **No FPS gain**
  - Ensure Sodium is actually loaded (check Mods menu).
  - Disable shader packs (if any).
  - Re-test in same scene/time/weather.

- **RAM still high**
  - Confirm FerriteCore is loaded.
  - Lower texture resolution further (8x → 4x).
  - Reduce background apps and JVM max heap if over-allocated.

- **CPU spikes / lag in bases**
  - Lower simulation distance.
  - Reduce entity cramming, hopper clocks, and high-frequency redstone.

- **Visual readability too low at 4x**
  - Switch to 8x variant while keeping mod+settings optimizations.

---

## Deliverable Checklist

- [x] Resource pack file tree
- [x] `pack.mcmeta`
- [x] Texture replacement strategy
- [x] Fabric mod list + purpose of each
- [x] Optimized settings config
- [x] Installation steps
- [x] Troubleshooting
