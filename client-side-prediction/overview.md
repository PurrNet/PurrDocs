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
* **Predicted If Owned With Soft Fallback** fully predicts the locally owned copy and soft-corrects everyone else's.

Policies only alter client behavior. The server always simulates every identity normally.

## Networking model

PurrDiction sends inputs and frames over unreliable channels. The client uploads each input tick several times inside a short redundancy window derived from its input-margin band, so input bandwidth stays flat as round-trip time grows: an unchanged input costs about one bit per extra copy, and the server drops fully duplicate uploads before parsing them.

Each server frame carries state as a delta against a tick the client has already verified, plus the newest input for each input-driven identity, written as a delta against the input block that client last acknowledged: an input that did not change costs a single repeat bit. Inputs for [deterministic identities](deterministic-identity.md) instead ride a guaranteed transcript that is re-sent until acknowledged; if a client falls further behind than the transcript window, the server re-anchors it with a full frame, so deterministic state always converges.

A lost frame is never retransmitted: the next frame that arrives re-simulates the missing ticks with the inputs it carries, so packet loss costs replay work instead of a round trip. Full frames, used for joins and resyncs, are the only reliable traffic.

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
