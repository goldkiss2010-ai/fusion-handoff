# Status

Fusion Handoff is an experimental DCC Handoff adapter.

## Working path

- FLD1 v1 / point3-pv
- stable particle identity
- neighboring-sample cache
- cubic Hermite position and velocity reconstruction
- timeline mapping
- object position / pivot / XYZ rotation
- perspective / view scale / dolly / screen offset
- stable Density selection
- particle color / size / opacity
- speed brightness / depth cue
- velocity streak
- Dot / Sprite / Image Dots
- one shared Source Image
- View Depth Split / Focus Depth
- 20K and 1M development caches

## Deliberately not in the core path

- separate particle Mask Gate
- native Fusion 3D point-cloud conversion
- one host object per particle
- mandatory GPU/DVIP compute
- FLD1 path or scene embedding into a Resolve project

## Optimization candidates

These are not promises or release blockers:

- reduce Lua table overhead for very large particle counts
- screen culling
- avoid unnecessary velocity work when streak/speed brightness are disabled
- optimize Global Opacity so it does not require a full-frame post pass
- explicit GPU/DVIP implementation only if profiling justifies the complexity

Correctness of the cross-host rendering contract remains the first priority.
