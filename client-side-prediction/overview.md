# Overview

PurrDiction runs a tick-based simulation alongside PurrNet. The client stays ahead using locally available input, while the server produces the authoritative timeline. Verified server frames periodically bring the client back to a known state before it replays any newer local ticks.

## Core pieces

| Piece | Responsibility |
| --- | --- |
| `PredictionManager` | Owns the predicted world for a scene: ticks, input queues, history, reconciliation, built-in systems, physics simulation, and view updates. |
| `PredictedIdentity` | Base component for a unit of predicted behavior. It owns state history, optional input, lifecycle callbacks, and a prediction policy. |
| `PredictedModule` | Reusable predicted state and behavior attached to an identity without requiring another root identity. |
| `PredictedHierarchy` | Rollback-aware creation, deletion, lookup, and pooling of predicted objects. |
| `PredictedPrefabs` | Stable registry used by the hierarchy to identify and create predicted prefabs. |
| Built-in systems | Optional time, random, players, hierarchy, and 2D/3D physics systems registered by the manager. |

## State, input, and view

Simulation data is split deliberately:

* `INPUT` describes player intent for one tick. The client collects it, and the server sanitizes and simulates it.
* `STATE` contains every mutable value required to restore and replay the simulation.
* Unity components bridge to state through `GetUnityState` and `SetUnityState` when rollback needs to capture or restore external state.
* View code renders the current corrected state. It must not make gameplay decisions.

Both input and state are structs implementing the appropriate `IPredictedData` interfaces so PurrDiction can pack, copy, compare, and dispose of them safely.

## Client timelines

Not every object needs the same prediction cost. A [prediction policy](prediction-policies.md) selects the client timeline per identity:

* **Full Prediction** simulates ahead and participates in rollback and replay.
* **Server Relay** only follows verified server ticks on clients.
* **Soft Correction** keeps simulating locally and converges toward verified state without rewinding the live object.
* **Predicted If Owned** fully predicts the locally owned copy and relays remote-owned copies.

Policies only alter client behavior. The server always simulates every identity normally.

## Networking model

PurrDiction sends inputs and frames over unreliable channels. Clients resend recent input ticks until the server acknowledges them, and each server frame carries state as a delta against a tick the client has already verified. A lost frame is never retransmitted: the next frame that arrives re-simulates the missing ticks with the real inputs it carries, so packet loss costs replay work instead of a round trip. Full frames, used for joins and resyncs, are the only reliable traffic.

Each delta frame also carries the recent input history for every input-driven identity, so the client can replay gap ticks with real inputs. This section is compressed against the previous tick: identity ids are only restated when the set changes, and an input that did not change since the previous tick costs a single bit instead of a full payload. In practice this keeps frame sizes close to their low-latency cost even at high ping, since the redundancy window grows with round-trip time.

### Recovering from heavy packet loss

Sustained packet loss can outpace the delta stream: if frames keep getting lost, the client stops acknowledging progress, the redundancy window grows, and larger frames become even more likely to lose a fragment. PurrDiction detects this per client by watching how fast their acknowledged baseline advances. When a client falls below half the tick rate over a detection window, the server switches that client to reliable full frames, delivered one at a time per acknowledgement round trip. Bandwidth stays bounded, the client keeps predicting locally, and the moment acknowledgements flow normally again the client returns to the regular unreliable delta stream. Healthy connections never enter this path, no matter how high their latency is, because their baseline advances every tick.

## What PurrDiction does not do

PurrDiction is not a replacement for PurrNet's Network Identities, RPCs, visibility, ownership, or security rules. Predicted gameplay uses its own identities and state pipeline, but the Prediction Manager itself is networked through PurrNet and can deliberately bridge to normal networked objects when needed.

Prediction also does not guarantee that arbitrary Unity code is deterministic. Unity physics can be rolled back and replayed, but exact results may still diverge across platforms or physics configurations. Reconciliation is what keeps those simulations aligned.

## Continue reading

* [Execution Flow](flow.md) lists the exact callbacks and phases.
* [Best Practices](best-practices.md) covers replay-safe simulation design.
* [Security Model](security.md) explains what the server trusts.
* [Views and Interpolation](views-and-interpolation.md) separates correction from presentation.
