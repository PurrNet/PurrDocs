---
icon: megaphone
---

# NetworkAudioSource

Drop-in networked wrapper for Unity's `AudioSource`. Add it to any GameObject with an AudioSource to sync audio across the network.

### Setup

1. Add an `AudioSource` component to your GameObject.
2. Add `NetworkAudioSource` to the same GameObject (it auto-assigns the AudioSource via `Reset()`).
3. If you plan to change clips at runtime or use `PlayOneShot`, register those `AudioClip` assets in your `NetworkAssets` ScriptableObject.

> If the clip is baked into the prefab/scene and never changes at runtime, you do **not** need to register it in NetworkAssets.

### Authority

The `_ownerAuth` toggle (Inspector) controls who can drive the audio:

* **Owner Auth (default):** The owning client controls playback. If no owner is set, the server controls it.
* **Server Auth:** Only the server can control playback.

All public setters and methods guard with `IsController(_ownerAuth)` and silently no-op on non-controllers.

### Public API

Use `NetworkAudioSource` the same way you would use Unity's `AudioSource`:

#### Properties

| Property       | Type        | Description                                                       |
| -------------- | ----------- | ----------------------------------------------------------------- |
| `clip`         | `AudioClip` | The clip to play. Must be in NetworkAssets if changed at runtime. |
| `volume`       | `float`     | Volume (0.0 - 1.0)                                                |
| `pitch`        | `float`     | Pitch multiplier                                                  |
| `loop`         | `bool`      | Whether to loop                                                   |
| `mute`         | `bool`      | Whether muted                                                     |
| `spatialBlend` | `float`     | 2D (0.0) to 3D (1.0) mix                                          |
| `minDistance`  | `float`     | Distance where volume stops increasing                            |
| `maxDistance`  | `float`     | Distance where attenuation stops                                  |
| `time`         | `float`     | Playback position in seconds                                      |
| `isPlaying`    | `bool`      | Read-only, current play state                                     |

#### Methods

| Method                          | Description                                      |
| ------------------------------- | ------------------------------------------------ |
| `Play()`                        | Start playback with the current clip             |
| `Play(AudioClip)`               | Set clip and start playback                      |
| `Stop()`                        | Stop playback                                    |
| `Pause()`                       | Pause playback                                   |
| `UnPause()`                     | Resume from pause                                |
| `PlayOneShot(AudioClip, float)` | Fire-and-forget SFX (does not affect play state) |

### How It Works

#### Dirty Flag System

Each property setter and method sets a bit in a `AudioDirtyFlags` bitmask. On the next network tick, only the flagged fields are serialized and sent. If nothing changed since the last tick, nothing is sent.

Example: calling only `volume = 0.5f` sends 2 bytes (flags) + 4 bytes (float) = **6 bytes**, not the full \~35 byte state.

#### Custom Serialization

The `AudioSourceDelta` struct implements `IPacked` for hand-written bit-level serialization. It writes the flags bitmask first, then only the fields whose bits are set. Booleans (`loop`, `mute`) are packed as single bits.

#### Channel Selection

The dirty flags are checked each tick to decide reliable vs unreliable delivery:

| What changed                            | Channel        | Reason                                                               |
| --------------------------------------- | -------------- | -------------------------------------------------------------------- |
| `Play`, `Stop`, `Pause`, `UnPause`      | **Reliable**   | Discrete state transition; a dropped packet means permanent desync   |
| `clip`                                  | **Reliable**   | Discrete change; must arrive                                         |
| `volume`, `pitch`, `spatialBlend`, etc. | **Unreliable** | Continuous tweaks (fades); next tick resends latest value if dropped |
| `PlayOneShot`                           | **Reliable**   | Fire-and-forget SFX; must not be missed                              |
| Late joiner reconcile                   | **Reliable**   | Full state snapshot; must arrive                                     |

If a tick contains both reliable and unreliable changes (e.g. `volume = 0.5f` + `Play()` in the same frame), the entire delta is sent reliably.

#### RPC Routing

Follows the same pattern as `NetworkAnimator`:

* **Server controller** -> `ObserversRpc(excludeSender: true)` directly to clients.
* **Client controller** -> `ServerRpc` to server -> server applies locally -> `ObserversRpc(excludeSender: true)` to other clients.
* **Late joiner** -> `OnObserverAdded` sends `TargetRpc` with full state to the new observer.

#### Playback Time

* Time is only sent when explicitly changed (via `Play()`, `UnPause()`, or the `time` setter).
* It is **not** auto-sent every tick while playing, avoiding constant bandwidth for background music.
* On receiving end, if a time value arrives while playing and the local playback has drifted more than 0.1s, it snaps to the received time.

#### AudioClip Serialization

AudioClips are serialized via PurrNet's `Packer<T>` for `UnityEngine.Object`, which looks up the clip's index in `NetworkAssets`. This sends a small integer index, not the audio data.

* If the clip **is not** in NetworkAssets, it resolves to `null` on the remote side.
* If you never change the clip at runtime, it is never serialized (the `Clip` dirty flag is never set), so it doesn't matter whether it's registered.
