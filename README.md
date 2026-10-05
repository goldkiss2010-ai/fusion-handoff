![Fusion Handoff](branding/key-visual.jpg)

# Fusion Handoff

**FLD1 particle-state handoff for Blackmagic Fusion / DaVinci Resolve.**

Fusion Handoff reads externally computed particle state from FLD1 and reconstructs the presentation inside Fusion.

The producer computes **what the particles are doing**. Fusion keeps control of **time, viewpoint, appearance, and composition**.

Fusion Handoff is the Fusion / DaVinci Resolve implementation of the [DCC Handoff](https://github.com/goldkiss2010-ai/dcc-handoff) architecture. [AE Handoff](https://github.com/goldkiss2010-ai/ae-handoff) reads the same FLD1 state contract in After Effects.

## Status

Experimental, but the complete first rendering path is working.

Verified development path includes:

- FLD1 v1 / `point3-pv` decoding
- 20K-particle and 1M-particle caches
- timeline-driven state sampling
- cubic Hermite position and velocity reconstruction
- 3D object/view transform and perspective
- Dot / Sprite / Image Dots
- velocity streak
- density, color, size, opacity, speed brightness, depth cue
- View Depth Split / Focus Depth

The current implementation is a self-contained Fuse:

```text
Fuses/FusionHandoff.fuse
```

No Python runtime is required to play an existing FLD1 cache.

For testing, the host-independent [FLD1 Asset Pack v01](https://github.com/goldkiss2010-ai/ae-handoff/releases/download/v1.15-preview.1/FLD1_Asset_Pack_v01.zip) contains five 20K fields and a 1M-particle Vortex Ring. The same files are usable by AE Handoff and Fusion Handoff.

## Architecture

```text
external computation
        |
        | particle state
        v
       FLD1
        |
        v
 Fusion Handoff
        |
        +-- sample selection
        +-- Hermite reconstruction
        +-- object / view transform
        +-- perspective projection
        +-- Dot / Sprite / Image Dots
        +-- streak / opacity / depth presentation
        |
        v
      RGBA
        |
        v
 normal Fusion flow
```

The cache remains 3D, but the current adapter performs projection and rasterization itself. Fusion receives the completed image rather than one native Fusion object per particle.

That is intentional. A million-particle cache remains a cache plus one renderer node instead of becoming a million host objects.

## Install

Copy:

```text
Fuses/FusionHandoff.fuse
```

into a Fusion / Resolve Fuse search directory, then restart Resolve.

Add **Fusion Handoff** from:

```text
Fuses > FLD1
```

Select an `.fld1` cache and view the node or connect it downstream.

A normal Fuse reload is often enough after changing `Process()`, but changes to `FuRegisterClass(...)` registration metadata such as `REG_TimeVariant` may require a full Resolve restart.

See [Installation](docs/INSTALLATION.md).

## Rendering controls

Current implementation includes:

- FLD1 file selection
- Samples / Second
- Sample Offset
- Object Position X/Y/Z
- Object Pivot X/Y/Z
- Rotation X/Y/Z
- Perspective
- View Scale
- Dolly
- Screen Offset X/Y
- Density
- Particle Color
- Particle Size
- Speed Brightness
- Depth Cue
- Velocity Streak
- Particle Opacity
- Global Opacity
- Dot / Sprite / Image Dots
- one shared Source Image input
- View Depth Split: Off / Front / Back
- Focus Depth

## Time contract

New normalized FLD1 caches use saved sample index as the state coordinate:

```text
q = 0, 1, 2, ...
```

Stored velocity is `dx/dq`.

Fusion maps its timeline to state coordinate:

```text
q = SampleOffset + elapsed_seconds * SamplesPerSecond
```

The FLD1 file does not own presentation duration.

Set `Samples / Second = 0` when you want `Sample Offset` itself to be keyframed or expression-driven.

## Interpolation

Between neighboring stored states, Fusion Handoff reconstructs position with cubic Hermite interpolation using stored velocity as the tangent.

The derivative of that Hermite curve is also reconstructed and used by speed-dependent presentation and velocity streaks.

This is a state reconstruction path, not frame blending.

## Image modes

Fusion Handoff exposes one optional image connector: **Source Image**.

**Sprite** stamps Source Image at each particle position. Source alpha is respected, and sprite size follows perspective.

**Image Dots** samples Source Image using stable frame-0 XY particle mapping. Sampled RGB becomes particle color and sampled alpha multiplies visibility.

There is intentionally no separate particle-mask image input. Whole-output masking remains an ordinary Fusion compositing operation.

## View Depth Split

`View Depth Split = Front / Back` compares transformed particle camera-space Z with one scalar `Focus Depth`.

```text
Front: camera_z <= Focus Depth
Back:  camera_z >= Focus Depth
```

The split plane is always parallel to the image / sensor plane. It has no Focus X/Y position.

This allows a normal Fusion image, text, or other 2D branch to sit between two Handoff renders:

```text
Back Handoff
2D image / text / object
Front Handoff
```

This is a presentation split, not a general Z-buffer or mesh occlusion system.

## Why not native Fusion particles first?

The current goal is not to reproduce the FLD1 cache as a large graph of native host objects. The first goal is to preserve the same compact state-to-presentation model across DCCs.

A future native-3D adapter is possible, but it would be a different adapter strategy. FLD1 itself does not require either approach.

## Performance

The current Fuse prioritizes semantic correctness and inspectability. Performance depends strongly on:

- particle count
- output resolution
- frame rate
- particle size
- velocity streak
- sprite size
- opacity/compositing path
- downstream Fusion processing
- viewer/cache state

20K has been used as the normal interactive development case, with 1M-particle caches used as scale tests.

Do not infer that the Lua particle loop is GPU-computed merely because Resolve/Fusion shows GPU activity. The current Fuse does not implement an explicit DVIP compute path.

## Repository boundary

Fusion Handoff is intentionally a host adapter.

```text
dcc-handoff
  cross-host architecture

fld1
  state contract

fusion-handoff
  Fusion adapter

ae-handoff
  After Effects adapter
```

The host implementations do not need a shared runtime library to share semantics.

See [Renderer contract](docs/RENDERER_CONTRACT.md) for the portable behavior and [Verification checklist](docs/TEST_CHECKLIST.md) for the current test pass.

## 日本語

Fusion Handoffは、外部で計算した粒子状態をFLD1でFusion / DaVinci Resolveへ渡し、時間・視点・見た目・合成をFusion側に残すためのFuseです。

AE Handoffと同じFLD1を読み、Hermite補間、3D変換、透視投影、Dot / Sprite / Image Dots、Velocity Streak、View Depth SplitをFusion側で再構成します。

現状は実験版です。特に性能については、同じ解像度・fps・粒子サイズ・ストリーク・Viewer条件を揃えずにAE版との速度比較を行わない方針です。

## License

MIT License. See [LICENSE](LICENSE).

The DCC Handoff / Fusion Handoff branding image is presentation material and is not granted under the repository MIT license; see [branding/README.md](branding/README.md).
