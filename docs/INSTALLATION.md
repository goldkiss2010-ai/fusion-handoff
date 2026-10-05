# Installation

Fusion Handoff is currently distributed as a single Fuse source file.

## File

```text
Fuses/FusionHandoff.fuse
```

Copy the file into a Fuse search directory used by Fusion / DaVinci Resolve.

The exact search path can differ by installation and platform. Use a directory that Resolve/Fusion is actually scanning; a manually created directory with the right-looking name is not sufficient if it is outside the active search path.

After installation, Fusion Handoff should appear under:

```text
Fuses > FLD1
```

## Reload behavior

For edits inside `Process()`, a Fuse reload may be enough.

For changes to registration metadata in `FuRegisterClass(...)`, especially `REG_TimeVariant`, fully restart Resolve. Registration metadata may remain cached even when the Fuse body itself reloads.

## First test

1. Add Fusion Handoff.
2. Select a known FLD1 cache.
3. Start with Render Mode = Dot.
4. Use Samples / Second = 12 and Sample Offset = 0 for the 49-sample / 4-second 20K demo caches.
5. Scrub the timeline and verify that state changes without touching a control.
6. Change Rotation and Perspective.
7. Test View Depth Split = Front / Back and move Focus Depth.

For Sprite or Image Dots, connect exactly one image to `Source Image`.

## Troubleshooting

If the node does not appear, first verify the actual Fuse search directory.

If the node appears but timeline scrubbing does not request new frames after changing registration flags, restart Resolve completely.

If an invalid or unsupported FLD1 file is selected, the Fuse should report an error without crashing Resolve.

See [TEST_CHECKLIST.md](TEST_CHECKLIST.md) for the full verification pass.
