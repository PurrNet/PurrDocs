# Predicted Parent

`PredictedParent` makes a piece's transform parent part of predicted state. Reparenting a piece that has this component is rolled back, replayed, and delivered to late joiners like any other state change.

Without the component, the simulation always considers a piece attached to its authored prefab parent. You can still reparent it locally for visuals, but that is client-local decoration: it never affects the simulation, and a development build logs a warning to make the intent explicit.

***

**Usage**

Add the component to the piece you want to reparent at runtime. From then on, just set the transform parent from simulation code:

```csharp
public class PickupItem : PredictedIdentity<ItemState>
{
    protected override void Simulate(ref ItemState state, float delta)
    {
        if (state.pickedUpBy.HasValue &&
            predictionManager.hierarchy.TryGetGameObject(state.pickedUpBy, out var holder))
        {
            transform.SetParent(holder.transform);
        }
    }
}
```

The component captures the new parent into its state at the end of the tick. Rollbacks restore the previous attachment, replays reapply it, and late joiners receive the current link with the rest of the world state.

* The parent can be any predicted object, including pieces of other instances. If you parent under a plain child transform inside a piece, the link resolves to the nearest predicted ancestor and every peer attaches the piece to that object's own transform. The nested placement is not part of the state, so parent directly to the predicted object's transform when you want identical results everywhere.
* A `null` parent means the piece sits at the scene root.
* Setting the parent back to the authored default is recognized and stored as such.

To spawn an object already attached to something, use the `Create` overloads that take a parent instead of creating and reparenting in two steps.

***

**Behavior Details**

* Delete cascades follow the simulated attachment. A piece reparented away from its dying instance survives the delete; a foreign piece parented under a dying instance is detached and kept alive.
* If the target parent does not exist yet at the moment a correction is applied (for example the parent spawns in the same frame), the component keeps the target pending and attaches as soon as it resolves. A warning is logged if it cannot attach right away.
* If you restructure plain transforms between the piece and its predicted ancestor without changing the piece's own parent, call `RefreshParentLink()` so the link is re-resolved on the next capture.

***

**Physics and Scale**

* `PredictedRigidbody` state stays in world space, so reparenting a physics piece is safe, but a dynamic body will not follow its parent. Use kinematic bodies for carried or mounted objects.
* Attached kinematic colliders follow the parent with the usual one tick of physics latency.
* Avoid non-uniform scale on predicted parents. View interpolation ignores scale, so heavily scaled parents will make interpolated children drift visually.

***

**Best Practices**

* Only add the component to pieces that actually change parents at runtime. Everything else stays at its authored default for free, with zero state cost.
* Reparent from simulation code only, so the change is captured on the simulated timeline and replays correctly.
* Store the target as a `PredictedObjectID` or `PredictedComponentID` in your own state and resolve it when applying, rather than caching transforms across ticks.
