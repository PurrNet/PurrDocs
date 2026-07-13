# Predicted Transform

`PredictedTransform` stores position and rotation in prediction history, restores them during rollback, and renders a smoothed graphics transform after reconciliation. It works with a plain Transform and automatically bridges an attached 3D Rigidbody, Rigidbody2D, or Character Controller when present.

## Inspector settings

| Setting | Effect |
| --- | --- |
| Graphics | Optional visual transform updated by the view pass. Keep gameplay colliders and simulation on the root. |
| Float Accuracy | `Purrfect` sends full values, `Medium` uses compressed position/rotation, and `Low` uses half-precision state. Lower precision saves bandwidth at the cost of visible accuracy. |
| Interpolation Settings | `TransformInterpolationSettings` asset controlling normal rollback correction. A default asset ships with PurrDiction. |
| Unparent Graphics | Temporarily separates graphics from the root so predicted hierarchy pooling and replay enable/disable operations do not disturb the visual transform. |
| Character Controller Patch | Temporarily disables an attached Character Controller while applying restored state. |
| Soft Correction | Correction rate and snap thresholds used when the resolved [prediction policy](../prediction-policies.md) is `SoftCorrection`. |

## Graphics and simulation roots

The root transform is simulation state. Assign a child to **Graphics** when meshes, cameras, particles, or animation should be smoothed independently. Game logic and collision must read the predicted root, not the interpolated graphics child.

```text
PlayerRoot                  <- PredictedTransform and collision
└── Graphics                <- mesh/animator, assigned to Graphics
```

`updateGraphics` can temporarily disable view writes from code. Call `ResetInterpolation()` after a teleport or other intentional discontinuity to discard accumulated visual error.

## Interpolation asset

Create an asset through **Assets > Create > PurrNet > Purrdiction > InterpolationSettings**. Position and rotation each expose:

* **Correction Rate Min Max**: how quickly accumulated error is consumed.
* **Correction Blend Min Max**: error range used to move between the minimum and maximum rate.
* **Teleport Threshold Min Max**: ignore correction below the lower value and snap above the upper value.

Normal rollback interpolation and `SoftCorrection` solve different problems. The asset smooths visible error after full rollback; soft-correction settings converge an identity whose live client simulation was not rolled back.

## Policy ownership

When `PredictedTransform` shares a GameObject with `PredictedRigidbody`, `PredictedRigidbody2D`, or `PredictedProjectile3D`, that component owns the transform policy. Configure the policy on the physics component or on their shared scope so pose and velocity cannot end up on different client timelines.
