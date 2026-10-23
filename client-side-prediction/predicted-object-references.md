# Predicted Object References

Store a predicted ID when your input or state needs to refer to another predicted object. Resolve the ID when you need the object, so the lookup uses the current prediction world after rollback or replay.

Use `PredictedObjectID` to refer to a GameObject:

```csharp
using PurrNet.Prediction;

public struct TargetState : IPredictedData<TargetState>
{
    public PredictedObjectID? target;
    public void Dispose() { }
}
```

Given a `PredictedIdentity` named `targetIdentity`, store its object's ID in your state:

```csharp
state.target = targetIdentity.id.objectId;
```

Resolve it inside your predicted identity when needed:

```csharp
if (state.target.TryGetGameObject(predictionManager, out var target))
{
    // Use target.
}
```

The lookup returns `false` if the ID is null or the object no longer exists in the current simulation.

If you start with a GameObject, `predictionManager.hierarchy.TryGetId(gameObject, out var id)` gets the ID of that registered piece.

Use `PredictedComponentID` to refer to a specific `PredictedIdentity` component. This distinguishes between multiple predicted identities on the same GameObject. Add a field to your state:

```csharp
public PredictedComponentID? targetComponent;
```

Store the component's full `id` and resolve it with `TryGetIdentity`:

```csharp
state.targetComponent = targetIdentity.id;

if (state.targetComponent.TryGetIdentity(predictionManager, out var targetComponent))
{
    // Use targetComponent.
}
```

Use `TryGetIdentity<T>` when you need a particular `PredictedIdentity` subclass. The lookup returns `false` if the component is missing or has a different type.

Stable references to children of the same prefab, such as a camera or visual transform, can stay as serialized component fields. They do not need to be stored in predicted state.
