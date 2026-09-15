# Views and Interpolation

PurrDiction separates simulation state from presentation. You simulate and reconcile state deterministically, then render a smooth view from that state. This page explains the view pipeline, interpolation, and how to customize smoothing and snapping.

***

**Update Flow**

* Simulation runs at the network tick rate, including local prediction and server‑verified replays.
* Each frame, the Prediction Manager calls `UpdateView(deltaTime)` on predicted identities in either `Update` or `LateUpdate` based on the Update View Mode setting.
* Once every identity has run `UpdateView`, the manager runs `LateUpdateView` on all of them in the same pass. This is the place for cameras, IK, and anything that reads another identity's freshly rendered pose.
* For `PredictedIdentity<STATE>`:
  * A small interpolation buffer smooths from the last verified state toward the latest predicted state.
  * `UpdateView(STATE viewState, STATE? verified)` receives the interpolated view plus the most recent verified snapshot, if available.

Key APIs on `PredictedIdentity<STATE>`:

* `public override void ResetInterpolation()`
* `public override void UpdateRollbackInterpolationState(float delta, bool accumulateError)`
* `protected virtual void ModifyRollbackViewState(ref STATE state, float delta, bool accumulateError)`
* `protected virtual void ViewStart(STATE viewState, STATE? verified)`
* `protected virtual void UpdateView(STATE viewState, STATE? verified)`
* `protected virtual void LateUpdateView(STATE viewState, STATE? verified)`

{% version range=">=1.4.0" %}

`StatelessPredictedIdentity` gets the same two hooks without arguments: `UpdateView()` and `LateUpdateView()`.

{% endversion %}

Terminology:

* `viewState`: The state you should render this frame (often interpolated and error‑corrected).
* `verified`: The last server‑verified state if present, otherwise null.

***

**Interpolation Buffer**

* A per‑identity interpolation helper buffers up to 0.1 s of ticks (`tickRate / 10`, min 3) to absorb jitter.
* Default interpolation blends linearly: `IMath<T>.Add/Scale/Negate` on `STATE`. You can override `Interpolate(STATE from, STATE to, float t)`.
* When reconciliation happens, the system snapshots and adjusts the buffer to smoothly approach the corrected timeline.

{% version range=">=1.4.0" %}

* The manager keeps one view clock for the whole scene. Every identity samples its own buffer at the same `predictionManager.viewTick`, so objects that spawned at different times still present the same moment; a player and the projectile they just fired never drift apart by a fraction of a tick.
* `viewTick` is also what [lag compensation](lag-compensation.md) reports to the server, so the rewind used for hit tests matches what was on screen.
* Per identity, `viewInterpolationBufferSize`, `viewAnchorTick`, `viewNextSampleTick`, and `hasPendingViewLatch` expose the buffer for debugging. A gap of more than one tick between the anchor and the next sample means the view is gliding across ticks that were never latched, for example after a lead jump.

{% endversion %}

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

**Soft Correction vs Rollback Interpolation**

These are separate correction paths:

* With `FullPrediction`, the simulation is restored and replayed. View interpolation hides the visual difference between the pre-reconcile and post-reconcile result.
* With `SoftCorrection`, the live client object is not restored. Verified state produces a pending pose or velocity error, and supported components consume that error during future live ticks.
* With `ServerRelay`, the view follows verified state because the identity has no speculative future on that client.

Built-in soft correction exposes an exponential correction rate and snap thresholds. Choose it only when smooth convergence is acceptable for gameplay; use full prediction when historical interaction correctness matters. See [Prediction Policies](prediction-policies.md).

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
