---
version: ">=1.3.0"
---

# Predicted Visibility

By default every player receives every predicted object. Predicted visibility lets the server decide, per player and per object, who receives it at all.

Hiding an object from a player does three things for that player only:

* The object is removed from their world.
* Its state and inputs are no longer sent to them.
* When you show it again, it arrives like a new spawn, with full state.

Use it for stealth, fog of war, per-team objects, interior cells, or anything where sending an object to a player would leak information or waste bandwidth.

{% hint style="info" %}
Predicted visibility is separate from PurrNet's [Network Visibility](../systems-and-modules/network-manager/network-visibility/) rules. Those rules decide who observes a `NetworkIdentity`. Predicted visibility decides who receives a predicted object.
{% endhint %}

## Which objects can be hidden

Every predicted object the [Predicted Hierarchy](predicted-hierarchy.md) tracks:

* Instances created with `hierarchy.Create`.
* Objects placed in the scene.

Both use the same calls and follow the same rules. The only difference is what happens on the hidden player's client, covered in [What the hidden player sees](#what-the-hidden-player-sees).

## The API

All four calls live on `PredictionManager`. Each takes the target player and the `PredictedObjectID` of the object:

```csharp
// Hide the object from one player.
predictionManager.HideFrom(player, objectId);

// Stop hiding it. The object goes back to the default: visible.
predictionManager.ShowTo(player, objectId);

// Same as ShowTo, spelled for intent.
predictionManager.ResetVisibility(player, objectId);

// Force it visible until the handle is disposed, even if it is hidden.
IDisposable handle = predictionManager.AcquireVisibility(player, objectId);
handle.Dispose();
```

`HideFrom` and `ShowTo` return `true` when they changed something and `false` when the object was already in that state.

Call these on the server. Only the server sends frames to players, so calling them on a client has no effect.

### Getting the object id

| You have | Use |
| --- | --- |
| The result of `hierarchy.Create` | The returned `PredictedObjectID` |
| A `PredictedIdentity` on the object | `identity.rootObjectId` |
| A GameObject, including one placed in the scene | `predictionManager.hierarchy.TryGetId(gameObject, out var objectId)` |

Visibility always applies to a whole object, never to part of one. If you pass the id of a child piece, it resolves to the root of that object.

## When a hidden object stays visible

`HideFrom` is a request, not a hard cut. Two things keep an object visible to a player even while it is hidden from them:

* **Ownership.** If the player owns any predicted identity inside the object, they keep receiving it. A player always receives what they control. Once they lose ownership, the object is hidden from them again.
* **Acquisitions.** An active `AcquireVisibility` handle forces the object visible. Handles can overlap: the object stays visible until the last handle is disposed. Disposing a handle twice does nothing.

Attachments between objects matter too. When an object is attached under another through [Predicted Parent](built-in-components/predicted-parent.md), hiding the parent hides everything attached beneath it. An owned or acquired child stays visible anyway, and so do the parents it is attached to, because the child cannot exist on the client without them.

## When changes take effect

A change applies the next time the server sends a frame to that player. If you hide and show the same object several times within one tick, only the final result counts and nothing is sent for the changes in between.

You can hide an object from a player as soon as they join, before they have received anything. It is then left out of the first state they receive.

## What the hidden player sees

Once the change takes effect, the player stops receiving anything about the object:

* It is no longer part of the hierarchy state sent to them, so their client removes it.
* Its state and inputs are skipped.
* Collision and trigger events are dropped when either side of the contact is hidden.

What "removed" means on their client depends on where the object came from:

| | Created with `hierarchy.Create` | Placed in the scene |
| --- | --- | --- |
| When hidden | Deactivated and kept for two seconds, in case a later frame brings it back | Deactivated and kept for as long as it stays hidden |
| After two seconds | Destroyed, or returned to the prefab pool if the prefab is pooled. `Destroyed()` fires now, not when it disappeared | Nothing changes. `Destroyed()` does not fire |
| When shown again | A new instance, or the kept one if it came back within the two seconds | The same scene object is reactivated |

In every case the object keeps its `PredictedObjectID`, and the server sends the full state of every identity inside it, because the client has nothing recent to delta against. Plan for a hide and show pair to cost about as much as a spawn.

{% hint style="warning" %}
A scene object is part of the scene every client loads, so a hidden player's build still contains it. Hiding it keeps its live state from them, such as position, health, and inputs. It does not keep the object's existence secret. For objects whose existence must stay secret, create them at runtime with `hierarchy.Create`.
{% endhint %}

Everyone else is unaffected. Visibility is tracked separately for each player, and hidden objects keep simulating normally on the server and on every client that can see them. Hiding changes what a player receives, not what the simulation does.

## Example: only receive the enemies you can see

```csharp
public class Revealer : PredictedIdentity<RevealState>
{
    protected override void Simulate(ref RevealState state, float delta)
    {
        if (!isServer || !owner.HasValue)
            return;

        // state.enemies holds the PredictedObjectIDs this player could run into.
        foreach (var enemy in state.enemies)
        {
            if (CanSee(enemy)) // your own line of sight check
                predictionManager.ShowTo(owner.Value, enemy);
            else
                predictionManager.HideFrom(owner.Value, enemy);
        }
    }
}
```

Calling `HideFrom` on an object that is already hidden, or `ShowTo` on one that is already visible, does nothing. It is fine to repeat the decision every tick like this: only real changes are sent.

For a temporary reveal on top of a longer hide, use an acquisition instead of `ShowTo`:

```csharp
// Server side: an ability reveals the target for a moment.
var reveal = predictionManager.AcquireVisibility(caster, targetId);
// ...later, when the effect ends:
reveal.Dispose();
```

Your `HideFrom` stays in place the whole time and applies again once every acquisition on that object is released. You do not have to remember whether the target was hidden before the reveal.

## Good to know

* **What you hid is server-side bookkeeping.** It is not part of predicted state and does not roll back.
* **Keep ids, not GameObjects.** A hidden instance can be destroyed and recreated on the client, but its `PredictedObjectID` stays the same.
* **A player who leaves is forgotten.** When a player stops observing the Prediction Manager, for example because they disconnected, everything you hid from them and every acquisition for them is dropped. Disposing a handle you still hold for them is safe and does nothing. If the same player id is reused by a new connection, that player starts with everything visible.
* **Network identities inside predicted prefabs follow along.** A player observes the mirrored `NetworkIdentity` components exactly while they can see the predicted object. See [Network Identities in Predicted Prefabs](built-in-components/predicted-identity-spawner.md).
