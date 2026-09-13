# Deterministic Identity

`DeterministicIdentity<STATE>` is a variant designed for strict, bit‑stable simulation. It mirrors the lifecycle of `PredictedIdentity<STATE>` but uses deterministic math (`sfloat`) and can validate state equality across client/server when enabled.

***

**When to Use**

* Systems that can produce identical results on all machines from the same inputs (e.g., strategy sim, deterministic AI, fixed math gameplay).

If you don’t require strict bit‑determinism, prefer [`PredictedIdentity<STATE>`](predicted-identities.md) for simpler float‑based logic.

***

**Key Properties**

* Uses `sfloat` delta in `Simulate`/`LateSimulate` for deterministic time steps.
* History, rollback, and interpolation are the same pattern as stateful identities.
* Networking: it doesn’t send simulation state each tick; ownership metadata syncs when it changes, and the optional desync policy detects divergence from compact state hashes.

Prediction Manager settings (**Determinism** section):

* **Desync Policy**: how the server responds when a client’s deterministic state diverges from its own. `Ignore` (default) does no hashing and has zero overhead. `Report` raises `predictionManager.onDesyncDetected` on the server and `onLocalDesync` on the diverged client, with no automatic recovery. `Resync` re‑baselines the diverged client’s entire timeline with a full frame. `Correct` heals just the diverged identity by resending its authoritative state.
* **Desync Check Interval**: how often clients report tick‑salted hashes of settled deterministic state to the server, in seconds (default 0.25). Hashing only runs when at least one identity’s resolved policy is not `Ignore`.

Each `DeterministicIdentity` also has its own **Desync Policy** field (`Inherit` by default) that overrides the global setting; it is resolved once during setup. Hashes are compared at the same settled tick on both peers, so latency, jitter, and frame hitches never produce false positives.

Prediction policies:

* `FullPrediction`, `ServerRelay`, and `PredictedIfOwned` are supported without turning deterministic identities into ordinary per-tick state replication.
* `SoftCorrection` is not supported because deterministic identities do not receive the authoritative state deltas required to build a correction target. Selecting it keeps `SoftCorrection` as the configured policy, but the identity behaves as `ServerRelay` on clients. The non‑owner branch of `PredictedIfOwnedWithSoftFallback` falls back to `ServerRelay` the same way.

See [Prediction Policies](prediction-policies.md) for the client timeline behavior.

***

**Overrides**

* `protected virtual void GetUnityState(ref STATE state)` / `protected virtual void SetUnityState(STATE state)`
* `protected virtual void SimulationStart()`
* `protected virtual void Simulate(ref STATE state, sfloat delta)`
* `protected virtual void LateSimulate(ref STATE state, sfloat delta)`
* `protected virtual void UpdateView(STATE viewState, STATE? verified)`
* `protected virtual STATE Interpolate(STATE from, STATE to, float t)`

Note the `sfloat` delta: use deterministic math throughout your simulation. You can also mix in `FP` for fixed point math.

***

**STATE Requirements**

* `STATE : struct, IPredictedData<STATE>`, same as other identities.
* Provide deterministic operations via `IMath<STATE>` (used by default interpolation): `Add`, `Scale`, `Negate`.
* Avoid non‑deterministic data (raw `float` computation); prefer `sfloat` or integer/fixed‑point math inside state operations.

***

**Example**

```csharp
using PurrNet.Prediction;

public struct OscState : IPredictedData<OscState> {
    public sfloat phase; public sfloat speed; public sfloat amplitude;
    public void Dispose() {}
}

public class Oscillator : DeterministicIdentity<OscState>
{
    protected override OscState GetInitialState() => new OscState {
        speed = (sfloat)1.5f, amplitude = (sfloat)2f, phase = 0
    };

    protected override void GetUnityState(ref OscState s) { /* read if needed */ }
    protected override void SetUnityState(OscState s) { /* apply on rollback if needed */ }

    protected override void Simulate(ref OscState s, sfloat dt)
    {
        s.phase += s.speed * dt;
        var y = sfloat.Sin(s.phase) * s.amplitude;
        // use y for downstream deterministic effects; drive visuals in UpdateView
    }

    protected override void UpdateView(OscState view, OscState? verified)
    {
        // Convert to float for rendering only
    }
}
```

***

**Inputs and Convergence**

`DeterministicIdentity<INPUT, STATE>` mirrors `PredictedIdentity<INPUT, STATE>`: implement `GetFinalInput`, `UpdateInput`, and `SanitizeInput` the same way, and `Simulate(INPUT input, ref STATE state, sfloat delta)` receives the deterministic delta. Because every peer must feed identical inputs into the simulation, these inputs are delivered on a guaranteed transcript rather than best‑effort: the server includes every input tick since the client’s last acknowledged frame in each outgoing frame, up to a 32 tick window, so a lost packet is repaired by the next one.

If a client falls further behind than that window, the server sends a full frame that re‑anchors its deterministic timeline. Either way the simulation converges; packet loss can delay deterministic state but never permanently fork it.

***

**Best Practices**

* Use `sfloat` and integer math for all simulation‑impacting calculations.
* Avoid sampling Unity time or random APIs directly; use `PredictedTime` and `PredictedRandom`.
* Keep any conversions to `float` purely in `UpdateView`.
* Only mutate state from the simulated timeline: `SimulationStart`, `Simulate`, or events that fire from inside another identity's simulation (like `PredictedPlayers.onPlayerAdded`). Never mutate state from `Awake`, `LateAwake`, or other Unity callbacks; those run at setup time, which differs per peer, and deterministic state is never corrected afterwards. `SimulationStart` is the right place for one-time setup that reads other identities, since it runs at the first simulated tick and its executed flag is part of the rollback state.
* Leave **Desync Policy** on `Ignore` unless you need divergence detection. `Report` is a good development default; `Resync` or `Correct` are safe to ship when you want automatic recovery, at the cost of periodic hashing and reporting.
