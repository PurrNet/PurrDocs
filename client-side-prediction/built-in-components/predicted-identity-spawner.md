# Predicted Identity Spawner

**PredictedIdentitySpawner**

* Purpose: Spawns a set of `NetworkIdentity` objects deterministically from prediction.
* Usage:
  * Add component, list identities to spawn.
  * On server, spawns and assigns observers/ownership.
  * On clients, mirrors spawns during verified replays.
* Caveat: Marked `[PredictionUnsafe]` because it orchestrates networked side‑effects; keep core simulation deterministic.
* See: Predicted Hierarchy docs for details.
