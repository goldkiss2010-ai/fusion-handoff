# Fusion Handoff

Experimental FLD1 particle-state renderer for Blackmagic Fusion / DaVinci Resolve.

Fusion Handoff reads the same FLD1 state cache used by AE Handoff and reconstructs the presentation inside Fusion. The cache carries particle state; the DCC owns interpolation, view transforms, projection, rasterization, and compositing.

This repository is intentionally a thin host adapter. It may later be merged with the broader Handoff / FLD1 repositories, so host-independent behavior is documented separately from Fusion-specific code.

## Current implementation

`Fuses/FusionHandoff.fuse` currently includes:

- FLD1 v1 / `point3-pv` read path (`x y z vx vy vz visibility scalar0`)
- neighboring-sample cache and cubic Hermite position/velocity reconstruction
- Fusion timeline mapping with `Samples / Second` and `Sample Offset`
- object position, pivot, XYZ rotation
- perspective, view scale, dolly, screen offset
- stable Density selection
- particle color, size, speed brightness, depth cue
- velocity streaks
- particle and global opacity
- Dot, Sprite, and Image Dots render modes
- one shared `Source Image` connector: Sprite stamps it per particle; Image Dots samples its frame-0 XY color/alpha
- separate frame-0 XY `Mask Source` gate
- View Depth Split: Off / Front / Back around a world-space focus position

The Fuse is still experimental. The first priority is semantic correctness and cross-DCC behavior; optimization comes after the feature path is verified.

## Install

Copy `Fuses/FusionHandoff.fuse` into a Fusion/Resolve Fuse search directory, then restart Resolve after changes to `FuRegisterClass(...)` registration flags. A normal Fuse reload is usually enough for `Process()` edits, but registration metadata such as `REG_TimeVariant` may require a full Resolve restart.

Add **Fusion Handoff** from `Fuses > FLD1`, select an `.fld1` cache, and view the node or connect it downstream.

For image-driven modes, connect exactly one image to `Source Image`: `Render Mode = Sprite` stamps that image at every particle, while `Render Mode = Image Dots` samples its color/alpha using stable frame-0 XY particle mapping. `Mask Source` is a separate optional input used only by `Mask Gate`. Image-driven modes intentionally render transparent when `Source Image` is missing, so a wrong connection is not silently mistaken for ordinary dots.

## FLD1 time contract

New normalized caches use `sample_rate = 1`. Stored sample index is the state coordinate

```text
q = 0, 1, 2, ...
```

and stored velocity is `dx/dq` per stored sample interval. Fusion maps presentation time to state coordinate:

```text
q = SampleOffset + elapsed_seconds * SamplesPerSecond
```

The FLD1 file does not own presentation duration.

## Repository boundary

The deployable Fuse stays self-contained, but its sections follow the same conceptual boundary intended for other Handoff adapters:

```text
FLD1 reader
  -> state sampling / Hermite
  -> object + view transform
  -> projection
  -> presentation modes
  -> host image output
```

See `docs/RENDERER_CONTRACT.md` for the host-independent semantics and `docs/TEST_CHECKLIST.md` for the verification pass.

## Status

Prototype / experimental. Tested development path includes 20K and 1M-particle FLD1 caches in Fusion. Performance characteristics depend strongly on output resolution, frame rate, particle size, streaks, sprite size, and downstream Fusion processing.
