---
icon: puzzle-piece
---

# Built-in Components

Quick references for the most commonly used PurrDiction components you can drop into scenes.

**PredictedTransform**

- Purpose: Predicts and reconciles position/rotation; renders a smoothed view.
- Works with: bare `Transform`, `Rigidbody`, `Rigidbody2D`, and optionally `CharacterController`.
- Key fields:
  - Graphics: Optional child transform to move for visuals only.
  - Unparent Graphics: Avoid disable/enable artifacts on reconcile by unparenting.
  - Character Controller Patch: Temporarily disable the controller when applying rollback state.
  - Interpolation Settings: Assign a `TransformInterpolationSettings` asset; a default asset ships at `Assets/PurrDiction/Runtime/Transform/DefaultInterpolation.asset`.
  - Float Accuracy: `Purrfect` (full), `Medium` (compressed), `Low` (half) network packing for state.
- Tips:
  - For characters, enable Unparent Graphics and place all renderers under Graphics.
  - Tune correction thresholds and rates in the interpolation settings to reduce snapping.

***

**PredictedRigidbody (3D)**

- Purpose: Predicts `Rigidbody` motion with deterministic simulation and reconciles state.
- Requirements: Enable 3D physics in `PredictionManager` via `BuiltInSystems.Physics3D` and select a physics provider that includes Unity 3D physics.
- Notes:
  - PurrDiction switches Unity to script-driven simulation for ticks; avoid using `FixedUpdate` for physics.
  - For callbacks, see `IPredictedPhysicsCallbacks` and `PredictedPhysicsCallbacks` in the UnityPhysics runtime folder.

**PredictedRigidbody2D (2D)**

- Purpose: Predicts `Rigidbody2D` motion similarly to 3D.
- Requirements: Enable 2D physics in `PredictionManager` via `BuiltInSystems.Physics2D` and include the Unity 2D physics provider.

***

**PredictedIdentitySpawner**

- Purpose: Spawns a set of `NetworkIdentity` objects deterministically from prediction.
- Usage:
  - Add component, list identities to spawn.
  - On server, spawns and assigns observers/ownership.
  - On clients, mirrors spawns during verified replays.
- Caveat: Marked `[PredictionUnsafe]` because it orchestrates networked side‑effects; keep core simulation deterministic.
- See: Predicted Hierarchy docs for details.

***

**Prebuilt Samples**

- Located under `Assets/PurrDiction/Runtime/Prebuilt/`.
- Examples: `TopDownMovement_CC`, `TopDownMovement_RB`, `RigidbodyShooter`, `RigidbodyKnockback`, `RigidbodyJump`.
- Intended as reference implementations; adapt logic to your project’s states/inputs.

***

**Related Systems**

- `PredictedStateMachine`: Modular state logic with view hooks. See the State Machine docs.
- `PredictedRandom`: Deterministic RNG for predicted code paths.
- `PredictedPlayers`, `PredictedTime`, `PredictedHierarchy`: Built‑ins registered by `PredictionManager` to support common gameplay needs.
