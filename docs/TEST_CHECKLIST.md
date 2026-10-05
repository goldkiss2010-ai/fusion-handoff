# Fusion Handoff verification checklist

Run this after replacing the Fuse. For changes to `FuRegisterClass(...)`, fully restart Resolve before testing.

## A. Load / timeline

- [ ] Node appears as **Fusion Handoff** under `Fuses > FLD1`.
- [ ] Selecting the known 20K Vortex FLD1 produces particles with no Console error.
- [ ] Known header reads as FLD1 v1, 20,000 particles, 49 samples, stride 8.
- [ ] `Samples / Second = 12`, `Sample Offset = 0` plays the 49-sample cache across 4 seconds.
- [ ] Timeline scrub changes the state without touching a control.
- [ ] `Samples / Second = 0` freezes timeline-driven state progression.
- [ ] With `Samples / Second = 0`, keyframing `Sample Offset` drives the state directly.
- [ ] End-of-cache clamps cleanly without read errors.

## B. Transform / projection

- [ ] Object Position X/Y/Z translates the field in world space.
- [ ] Object Pivot changes the center of rotation rather than merely shifting the image.
- [ ] Rotation X, Y, Z each work independently.
- [ ] Perspective = 0 gives orthographic-size behavior.
- [ ] Perspective > 0 makes near particles larger and far particles smaller.
- [ ] Dolly changes viewpoint depth without rewriting FLD1.
- [ ] View Scale changes framing scale.
- [ ] Screen Offset X/Y moves only the final 2D framing.

## C. Rendering

- [ ] Density 100% shows the full set.
- [ ] Density 50% shows a stable subset with no temporal flicker.
- [ ] Changing Density upward adds particles rather than reshuffling the existing subset.
- [ ] Particle Color changes Dot/Streak color.
- [ ] Particle Size follows perspective.
- [ ] Speed Brightness changes brightness according to reconstructed motion.
- [ ] Depth Cue changes brightness according to camera depth.
- [ ] Particle Opacity below 100% reveals overlap rather than behaving like opaque max-only dots.
- [ ] Global Opacity affects the completed output.

## D. Velocity Streak

- [ ] Velocity Streak = 0 produces no line.
- [ ] Increasing Velocity Streak produces a tail opposite the interpolated velocity direction.
- [ ] Rotating the object rotates the streak correctly in 3D before projection.
- [ ] Streak follows Hermite-interpolated motion between stored samples.
- [ ] Sprite mode does not draw velocity streaks.

## E. Sprite

- [ ] Connect a small RGBA image to Sprite Source.
- [ ] Render Mode = Sprite stamps the image at particle positions.
- [ ] Sprite alpha is respected.
- [ ] Sprite Size changes the long side.
- [ ] Sprite size follows perspective.
- [ ] Disconnecting Sprite Source does not crash; output simply has no sprites.

## F. Image Dots

- [ ] Connect a color image to Image Source.
- [ ] Render Mode = Image Dots colors particles from the image.
- [ ] Mapping stays attached to particle IDs while the field evolves.
- [ ] Image alpha suppresses/fades corresponding particles.
- [ ] Image orientation is visually correct; if vertically flipped, only the Fusion frame-0 V mapping should be changed.

## G. Mask Gate

- [ ] Connect a grayscale/alpha image to Mask Source.
- [ ] Mask Gate = Frame 0 XY filters stable particle IDs.
- [ ] Mask Threshold changes membership.
- [ ] Mask Invert reverses the gate.
- [ ] Particle membership does not crawl as the FLD1 animation moves.

## H. View Depth Split

- [ ] Set View Depth Split = Front and note only particles nearer than the focus plane remain.
- [ ] Set View Depth Split = Back and verify the complementary side remains.
- [ ] Front + Back in two Fusion Handoff nodes reconstructs the full field when composited together.
- [ ] Put a 2D image/text layer between Back and Front; particles visibly pass behind and in front of it.
- [ ] Rotate the particle view: the split plane follows the current view automatically.
- [ ] Animate Focus Position Z and verify the split plane moves through the field.

## I. Scale test

- [ ] 20K cache: normal interactive behavior.
- [ ] 100K cache: no correctness regression.
- [ ] 300K cache: no correctness regression.
- [ ] 1M cache: Dot mode loads and plays/scrubs without crash.
- [ ] Repeat 1M at 3840x2160 / 60 fps and note actual behavior; do not compare performance against AE unless resolution, fps, particle size, streak, opacity, and viewer/cache state match.

## J. Failure behavior

- [ ] Empty FLD1 path returns transparent output.
- [ ] Invalid/non-FLD1 file reports a Console error but does not crash Resolve.
- [ ] Missing optional Sprite/Image/Mask inputs do not crash.
- [ ] Seeking outside cache range clamps safely.
- [ ] Japanese/non-ASCII FLD1 path still opens.
