# Built-in Components

PurrDiction ships components for common predicted gameplay. They are production building blocks as well as source examples for custom identities.

| Component | Use it for |
| --- | --- |
| [Predicted Transform](predicted-transform.md) | Rollback-aware position and rotation with visual interpolation and correction. |
| [Predicted Rigidbody / Rigidbody2D](predicted-rigidbody-2d-and-3d.md) | Unity physics bodies simulated by the Prediction Manager with rollback state and predicted collision events. |
| [Predicted Projectile 3D](predicted-projectile-3d.md) | Lightweight cast-based spherical projectiles with gravity, triggers, and bounce behavior. |
| [Predicted Identity Spawner](predicted-identity-spawner.md) | A deliberate bridge from a predicted object to one or more ordinary PurrNet Network Identities. |

Supporting systems include:

* `PredictedPhysicsCallbacks` for rollback-aware 3D collision and trigger events on custom predicted objects.
* `PredictedStateMachine` for modular predicted state logic.
* `PredictedRandom`, `PredictedTime`, `PredictedPlayers`, and `PredictedHierarchy`, enabled through the Prediction Manager's built-in systems.

Reference implementations for top-down character-controller and Rigidbody movement live under `Assets/PurrDiction/Runtime/Prebuilt/`. Copy the patterns you need rather than treating the samples as a general-purpose character controller.

Before choosing a component, decide whether it should use [full prediction, server relay, soft correction, or ownership-based prediction](../prediction-policies.md).
