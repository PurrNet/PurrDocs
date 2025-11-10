# Views and Interpolation

PurrDiction separates simulation state from presentation. You simulate and reconcile state deterministically, then render a smooth view from that state. This page explains the view pipeline, interpolation, and how to customize smoothing and snapping.

***

**Update Flow**

* Simulation runs at the network tick rate, including local prediction and server‑verified replays.
* Each frame, the Prediction Manager calls `UpdateView(deltaTime)` on predicted identities in either `Update` or `LateUpdate` based on the Update View Mode setting.
* For `PredictedIdentity<STATE>`:
  * A small interpolation buffer smooths from the last verified state toward the latest predicted state.
  * `UpdateView(STATE viewState, STATE? verified)` receives the interpolated view plus the most recent verified snapshot, if available.

Key APIs on `PredictedIdentity<STATE>`:

* `public override void ResetInterpolation()`
* `public override void UpdateRollbackInterpolationState(float delta, bool accumulateError)`
* `protected virtual void ModifyRollbackViewState(ref STATE state, float delta, bool accumulateError)`
* `protected virtual void UpdateView(STATE viewState, STATE? verified)`

Terminology:

* `viewState`: The state you should render this frame (often interpolated and error‑corrected).
* `verified`: The last server‑verified state if present, otherwise null.

***

**Interpolation Buffer**

* A per‑identity interpolation helper buffers several ticks (`~tickRate/10`, min 2) to absorb jitter.
* Default interpolation blends linearly: `IMath<T>.Add/Scale/Negate` on `STATE`. You can override `Interpolate(STATE from, STATE to, float t)`.
* When reconciliation happens, the system snapshots and adjusts the buffer to smoothly approach the corrected timeline.

Tip: If your `STATE` contains non‑linear fields (e.g., quaternions), override `Interpolate` and use slerp or domain‑specific blending.

***

**PredictedTransform**

`PredictedTransform` demonstrates a robust, out‑of‑the‑box interpolation and correction strategy:

* Uses `TransformInterpolationSettings` to control smoothing and snap thresholds for both position and rotation.
* Accumulates error on reconcile and gradually corrects it over multiple frames.
* Snaps (teleports) when error exceeds an upper threshold; skips correction when below a lower threshold.

Adjustable settings via `TransformInterpolationSettings`:

* `useInterpolation`: Toggle smoothing corrections.
* `positionInterpolation` and `rotationInterpolation` (`PredictedInterpolation`):
  * `correctionRateMinMax`: Base correction speed range.
  * `correctionBlendMinMax`: Scales correction rate based on error magnitude.
  * `teleportThresholdMinMax`: Lower bound to ignore tiny error; upper bound to snap immediately.

When you need to reset visual drift (e.g., teleporting gameplay), call `ResetInterpolation()` to clear accumulated error and re‑align view with prediction.

***

**Owner vs. Controller**

* `isOwner`: True when the identity’s owner matches `PredictionManager.localPlayer`.
* `isController`: True for the owner on clients; on server, also true for bots/AI.
* Views should be purely visual; avoid branching simulation on these flags. Use them to toggle camera, UI, or local effects.

***

**Tuning and Debugging**

* Choose `Update` vs `LateUpdate` from the Prediction Manager (Update View Mode) if visuals depend on post‑physics order.
* Log and compare `viewState` against `verified` to validate correction behavior.
* If you see oscillation, widen lower thresholds or increase correction rate.
* For large warps (portals, respawns), prefer snapping by calling `ResetInterpolation()`.
