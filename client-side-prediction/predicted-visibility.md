---
version: ">=1.3.0"
---

# Predicted Visibility

By default every player receives every predicted object. Predicted visibility lets the server decide, per player and per spawned object, who gets it at all. A hidden object is not just muted for that player: it is removed from their world, its state and inputs stop being sent, and it comes back as a fresh spawn when it is shown again.

Use it for fog of war, per-team objects, interior cells, or anything where sending an object to a player would leak information or waste bandwidth.

{% hint style="info" %}
Predicted visibility is separate from PurrNet's [Network Visibility](../systems-and-modules/network-manager/network-visibility/) rules. It only governs predicted objects created through the hierarchy. Scene objects are always visible to everyone.
{% endhint %}

## The API

All four calls live on `PredictionManager` and take the target player and a `PredictedObjectID`:

```csharp
// Hide a whole predicted object from one player.
predictionManager.HideFrom(player, objectId);

// Restore the default-visible policy for it.
predictionManager.ShowTo(player, objectId);

// Same as ShowTo, spelled for intent.
predictionManager.ResetVisibility(player, objectId);

// Force it visible until the handle is disposed, regardless of the hidden policy.
IDisposable handle = predictionManager.AcquireVisibility(player, objectId);
handle.Dispose();
```

`HideFrom` and `ShowTo` return `true` when they changed the policy and `false` when it was already in that state.

You can pass the id of any piece of an instance; it resolves to the instance's root while that piece exists. Visibility is always decided for whole roots, never for individual pieces.

Only the server applies visibility, because only the server writes frames to players. Calling these on a client has no effect.

## Rules that override the hidden policy

Hiding is a policy, not a hard cut. Two things keep an object visible to a player even while it is hidden from them:

* **Ownership.** If any predicted identity inside the root is owned by that player, the root stays visible to them. A player always receives what they control. Losing ownership re-applies the hidden policy.
* **Acquisitions.** An active `AcquireVisibility` handle forces the root visible. Handles overlap safely: the root stays visible until the last handle is disposed, and disposing a handle twice does nothing.

Attachments between instances matter too. When a root is attached under another root through [Predicted Parent](built-in-components/predicted-parent.md), hiding the ancestor hides everything attached beneath it. An owned or acquired descendant stays visible anyway and pulls in only the ancestors it needs to be represented.

## When changes take effect

Calls are cheap and just mark the player dirty. The change is committed when the next frame for that player is prepared, so hiding and showing the same object several times within one tick collapses to the final state, and nothing is sent for the churn in between.

## What the hidden player sees

For the player it is hidden from, a root is projected out of the frame entirely:

* The hierarchy state written to them no longer lists the instance, so their client removes it, the same way it applies any other verified removal. The object is deactivated and parked in the rollback pool first, in case the next frame brings it back, and after two seconds of ticks it is destroyed, or returned to the prefab pool when the prefab opts into pooling. `Destroyed()` fires at that point, not at the moment it disappears.
* State sections and input sections skip every identity inside the root.
* Physics collision and trigger events are dropped from their frame when either side of the contact is hidden.

Other players are unaffected; each receiver has its own visibility timeline.

When the root is shown again, the client cannot delta against anything it has, so the server sends the identities inside it as full state and the client recreates the instance. Plan for a hide and show pair to cost about as much as a spawn.

Hidden objects keep simulating normally on the server and on every client that can see them. Hiding changes what a player receives, not what the simulation does.

## Example: reveal an enemy while it is in your line of sight

```csharp
public class Revealer : PredictedIdentity<RevealState>
{
    [SerializeField] private float _revealSeconds = 1f;

    protected override void Simulate(ref RevealState state, float delta)
    {
        if (!isServer || !owner.HasValue)
            return;

        // Hide every enemy from this player by default...
        foreach (var enemy in state.enemies)
            predictionManager.HideFrom(owner.Value, enemy);

        // ...and only let the ones we can actually see through.
        foreach (var enemy in state.enemies)
        {
            if (CanSee(enemy))
                predictionManager.ShowTo(owner.Value, enemy);
        }
    }
}
```

`HideFrom` on an already hidden root and `ShowTo` on an already visible one are no-ops, so this style of "reassert every tick" code does not generate traffic.

For a temporary reveal that should not fight a longer-lived hidden policy, prefer an acquisition:

```csharp
// Server side: an ability shows the target for a moment.
var reveal = predictionManager.AcquireVisibility(caster, targetId);
// ...later, when the effect ends:
reveal.Dispose();
```

The hidden policy you set with `HideFrom` is untouched and becomes effective again once every acquisition on that root is released.

## Interaction with other systems

* [Network Identities in Predicted Prefabs](built-in-components/predicted-identity-spawner.md) follow predicted visibility: a player observes the mirrored `NetworkIdentity` components exactly while they can see the predicted root.
* Store `PredictedObjectID` values you hide, not GameObjects, and treat the hidden set as server-side bookkeeping. It is not part of predicted state and does not roll back.
* When a player stops observing the Prediction Manager, for example because they disconnected, their hidden set, acquisitions, and visibility timeline are all dropped. A handle you still hold for that player becomes inert; disposing it is safe and does nothing. If the same player id is reused for a new connection, it starts from the default-visible policy.
