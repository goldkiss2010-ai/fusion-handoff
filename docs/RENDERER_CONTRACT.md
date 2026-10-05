# Renderer contract

This document separates the Handoff rendering semantics from Fusion-specific API details so the implementation can later be merged or shared with AE Handoff without making FLD1 host-specific.

## 1. State boundary

FLD1 v1 base profile is eight float32 values per particle per stored state:

```text
x y z vx vy vz visibility scalar0
```

`scalar0` remains generic. A renderer must not assign a mandatory physical meaning to it.

For normalized caches, adjacent stored states are one `q` unit apart and velocity is `dx/dq`.

## 2. Presentation time

The DCC maps its own timeline to FLD1 state coordinate:

```text
q = SampleOffset + elapsed_seconds * SamplesPerSecond
```

Clamp `q` to the stored state range. `SamplesPerSecond = 0` makes `SampleOffset` the direct state coordinate.

## 3. Interpolation

Between stored states `a` and `b`, use cubic Hermite interpolation with stored velocity as the tangent. The derivative of the Hermite curve is the continuous display velocity used by speed-dependent rendering and streaks.

Visibility and `scalar0` are linearly interpolated.

## 4. Object / view transform

FLD1 coordinates remain unchanged. Presentation transforms belong to the DCC adapter.

The common order is:

```text
p1 = Rxyz(p - pivot) + pivot
p2 = p1 + object_position
camera_z = p2.z - dolly
projection factor = mix(1, camera_distance / (camera_distance + camera_z), perspective)
screen = center + screen_offset + p2.xy * view_scale * projection_factor
```

Host image-coordinate conventions may require a Y sign change only at the final screen projection.

Current reference camera distance is `10.0`.

## 5. Rendering controls

`Density` is a stable particle-ID subset. The same particle ID must remain selected across time; changing Density must not create temporal reshuffling.

Dot radius is multiplied by the perspective factor. Sprite long-side size is also multiplied by the perspective factor.

`Speed Brightness` mixes base brightness with a clamped speed scalar. `Depth Cue` mixes base brightness with a camera-depth scalar.

`Velocity Streak` uses the continuous interpolated velocity and draws from

```text
tail = position - velocity * streak_length
```

through the same object/view/projection path as the particle. The reference streak color multiplier is `0.42`.

At full particle opacity, Dot/Streak rendering uses channel-wise max behavior. Below full opacity, use premultiplied source-over.

## 6. Image Dots

Image Dots uses stable frame-0 XY mapping, not current-frame XY. This keeps particle/image membership stable while the particle field moves.

Frame-0 XY is normalized over the frame-0 XY bounds. Each host adapter converts normalized V to its own image-coordinate convention.

Image Dots samples RGBA from the image. Sampled RGB becomes particle color and sampled alpha multiplies visibility.


## 7. View Depth Split

View Depth Split is presentation-only and does not modify FLD1.

The split plane is always parallel to the image/sensor plane. It has one degree of freedom only: `Focus Depth`, expressed directly as camera-space Z. Particles are transformed normally, then classified by camera depth before perspective projection:

```text
focus_camera_z = Focus Depth
Front: particle_camera_z <= focus_camera_z
Back:  particle_camera_z >= focus_camera_z
```

There is no focus X/Y position. Changing Focus Depth slides the plane only along the viewing axis.

Two Handoff nodes/layers reading the same FLD1 can therefore bracket an ordinary 2D DCC object:

```text
Back particles
2D object
Front particles
```

This is a focus-plane split, not a general Z-buffer or mesh occlusion system.

## 8. Host-specific allowances

The following may differ between AE and Fusion without changing FLD1:

- exact UI layout
- image Y-axis convention
- pixel format and compositing implementation
- cache implementation
- stable Density hash implementation, unless exact cross-host subset parity becomes a requirement
- CPU/GPU execution strategy

The state contract and transform/render semantics above should remain portable.
