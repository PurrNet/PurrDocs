# Predicted Rigidbody (2D & 3D)

**PredictedRigidbody (3D)**

* Purpose: Predicts `Rigidbody` motion with deterministic simulation and reconciles state.
* Requirements: Enable 3D physics in `PredictionManager` via `BuiltInSystems.Physics3D` and select a physics provider that includes Unity 3D physics.
* Notes:
  * PurrDiction switches Unity to script-driven simulation for ticks; avoid using `FixedUpdate` for physics.
  * For callbacks, see `IPredictedPhysicsCallbacks` and `PredictedPhysicsCallbacks` in the UnityPhysics runtime folder.

**PredictedRigidbody2D (2D)**

* Purpose: Predicts `Rigidbody2D` motion similarly to 3D.
* Requirements: Enable 2D physics in `PredictionManager` via `BuiltInSystems.Physics2D` and include the Unity 2D physics provider.
