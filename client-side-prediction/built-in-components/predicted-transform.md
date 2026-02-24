# Predicted Transform

* Purpose: Predicts and reconciles position/rotation; renders a smoothed view.
* Works with: bare `Transform`, `Rigidbody`, `Rigidbody2D`, and optionally `CharacterController`.
* Key fields:
  * Graphics: Optional child transform to move for visuals only.
  * Unparent Graphics: Avoid disable/enable artifacts on reconcile by unparenting.
  * Character Controller Patch: Temporarily disable the controller when applying rollback state.
  * Interpolation Settings: Assign a `TransformInterpolationSettings` asset; a default asset ships at `Assets/PurrDiction/Runtime/Transform/DefaultInterpolation.asset`.
  * Float Accuracy: `Purrfect` (full), `Medium` (compressed), `Low` (half) network packing for state.
