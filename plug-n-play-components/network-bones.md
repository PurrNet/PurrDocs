---
icon: bone
---

# Network Bones

This component synchronizes the bone transforms of skinned meshes across the network. If your characters use skeletal animation and you need precise bone positions on all clients (for example, attaching items to hands or doing IK-driven animations), Network Bones is a straightforward way to get that working.

It automatically gathers all bones from the `SkinnedMeshRenderer` components on the object and its children.

## Setup

1. Add a `NetworkBones` component to the same GameObject that has your `SkinnedMeshRenderer`.
2. That's it. It will find the bones for you.

If you have additional transforms that aren't part of a skinned mesh but still need syncing (like weapon attachment points), you can add them to the **Extra Bones** list in the inspector.

## Settings

| Setting | Description |
| --- | --- |
| Owner Auth | If true, the owner sends bone data. If false, the server does. |
| Send Rate | How many updates per second to send (1 to 128). |
| Extra Bones | Additional transforms to sync beyond the skinned mesh bones. |
| Position Accuracy | Compression tolerance for position values. Lower is more precise but costs more bandwidth. Default: 0.01 |
| Rotation Accuracy | Compression tolerance for rotation in degrees. Default: 0.5 |
| Scale Accuracy | Compression tolerance for scale values. Default: 0.05 |
| Min Buffer Size | Minimum interpolation buffer. Default: 1 |
| Max Buffer Size | Maximum interpolation buffer. Default: 3 |

## How It Works

Network Bones compresses each bone's position, rotation and scale using the accuracy thresholds you configure. It then uses delta encoding, meaning it only sends values that have actually changed since the last update. On the receiving end, it interpolates between updates for smooth visual playback.

This keeps bandwidth low even on characters with many bones, since bones that aren't moving don't cost anything to sync.

## When to Use This vs Network Animator

[Network Animator](network-animator.md) syncs animator parameters (floats, bools, triggers) and lets the animation system drive the bones locally on each client. This works great when all clients have the same animation clips.

Network Bones syncs the actual bone positions directly. Use it when:

- Your animations are driven by IK or procedural logic rather than clips
- You need precise bone positions for gameplay (hit detection on specific body parts, item attachment)
- Clients might not have identical animation data
