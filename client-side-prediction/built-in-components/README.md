# Built-in Components

PurrDiction ships components for common predicted gameplay. They are production building blocks as well as source examples for custom identities.

| Component | Use it for |
| --- | --- |
| [Predicted Transform](predicted-transform.md) | Rollback-aware position and rotation with visual interpolation and correction. |
| [Predicted Rigidbody / Rigidbody2D](predicted-rigidbody-2d-and-3d.md) | Unity physics bodies simulated by the Prediction Manager with rollback state and predicted collision events. |
| [Predicted Projectile 3D](predicted-projectile-3d.md) | Lightweight cast-based spherical projectiles with gravity, triggers, and bounce behavior. |
| [Predicted Physics Callbacks](predicted-physics-callbacks.md) | Rollback-aware 3D collision, trigger, and Character Controller hit events on predicted objects without a rigidbody. |
| [Network Identities in Predicted Prefabs](predicted-identity-spawner.md) | How ordinary PurrNet Network Identities inside a predicted prefab get spawned and follow the predicted object's lifetime. |
| [Predicted Parent](predicted-parent.md) | Opt-in predicted transform parenting: reparenting rolls back, replays, and reaches late joiners. |

Supporting systems include:

* `PredictedStateMachine` for modular predicted state logic.
* `PredictedRandom`, `PredictedTime`, `PredictedPlayers`, and `PredictedHierarchy`, enabled through the Prediction Manager's built-in systems.

Reference implementations for top-down character-controller and Rigidbody movement live under `Assets/PurrDiction/Runtime/Prebuilt/`. Copy the patterns you need rather than treating the samples as a general-purpose character controller.

Before choosing a component, decide whether it should use [full prediction, server relay, soft correction, or ownership-based prediction](../prediction-policies.md).
