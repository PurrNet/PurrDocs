---
icon: magic-wand
---

# Development Shortcuts ("Abusing" The System)

Sometimes you want to move fast and "just make it work," even if it isn’t fully deterministic yet. PurrDiction’s reconciliation can paper over some differences: the server remains authoritative and clients will correct. This page outlines pragmatic shortcuts, when they’re acceptable, and how to minimize the jank.

We still recommend keeping simulation deterministic whenever possible. Use these techniques as stepping‑stones, then graduate to deterministic implementations.

***

**What This Means**

- You perform an action only on the server (or with server‑only branches) that the client didn’t predict.
- The server’s authoritative state diverges from the client’s predicted state.
- Clients receive the verified frame and correct via rollback/interpolation (possible pops or snaps).

Acceptable when:

- Prototyping features and iterating quickly.
- Visual differences are minor and short‑lived.
- You rely on interpolation to hide most of the correction.

***

**Common Shortcuts**

- Server‑Only State Change
  - Branch on server during simulation to adjust state (e.g., grant a buff, change velocity) without an equivalent predicted path.
  - Clients will snap/adjust to the verified state; use Interpolation Settings to smooth.

- Server‑Only Spawn/Despawn
  - Create or delete predicted objects only on the server. Clients will not create them until verified; expect a visible pop.
  - Mitigation: Use PredictedIdentitySpawner to mirror verified spawns deterministically; tune smoothing on affected visuals.

- Server‑Only Teleport
  - Teleport or re‑position on the server in response to a trigger. Clients will hard‑correct.
  - Mitigation: Call Reset Interpolation on the affected identity (e.g., PredictedTransform) to avoid dragging back.

- Clamp Inputs Only On Server
  - Skip Sanitize Input on clients while enforcing it on the server. Expected behavior wins on server; minor local feel differences may occur.

***

**Mitigations For Visual Jank**

- Interpolation Settings
  - Increase correction rates and thresholds to eat small errors; snap only when exceeding a larger threshold.

- Predicted Events For Feedback
  - Fire a Predicted Event to give immediate local SFX/VFX/UI before verification arrives.
  - Events are view‑only and won’t cause double‑fires during replay.

- Hide Pop‑Ins
  - Spawn with a short fade‑in or particle puff to mask late appearance.
  - Defer critical camera cuts until after verification.

***

**Caveats & Risks**

- Player Experience
  - Overuse creates a "floaty" or inconsistent feel. Favor deterministic paths for core control and combat.

- Debugging Cost
  - Server‑only branches can hide logic errors until later. Add logs/telemetry during development and remove branches as you stabilize.

- Accumulated Error
  - Continuous server‑only changes can constantly fight interpolation. Prefer one‑off authoritative corrections with clear visuals.

***

**Graduating To Determinism**

- Replace server‑only branches with shared deterministic logic in Simulate/Late Simulate.
- Move triggers to inputs or states owned by the controller; derive server results from the same inputs.
- Use Predicted Hierarchy to pre‑create expected objects locally and let verification finalize them.
- For strict stability, consider Deterministic Identity and Validate Deterministic Data during QA.

