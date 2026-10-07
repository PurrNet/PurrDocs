# Evaluating visibility

Visibility is not checked every tick, and this is for good reason!

Rules can be expensive, and most of the time nothing relevant has changed. If we evaluated every second and you had 1.000 identities all standing still, that would be 1.000 evaluations per player, every second, that could have easily been avoided. So PurrNet only evaluates at the moments it knows something changed, and leaves the rest to you.

## When PurrNet evaluates on its own

* **An object spawns.** It is evaluated for every player in its scene.
* **A player loads a scene**, which includes joining the game. Every object in that scene is evaluated for them.
* **An identity changes parent.**
* **A player is added to or removed from a** [**whitelist or blacklist**](whitelist-and-blacklist.md)**.** The identity is evaluated for that player on the next tick.

Nothing else triggers an evaluation. In particular:

* Objects and players moving around doesn't, so the [Distance condition](distance-condition.md) only does its job if you evaluate regularly.
* Changing ownership doesn't. A new owner who couldn't see the object before only gets it once it is evaluated for them.
* Anything your own rules read doesn't.
* Assigning a different rule-set or adding rules at runtime doesn't.

## Evaluating it yourself

Call `EvaluateVisibility` on any identity, from the server:

```csharp
// This identity and its networked children, for every player in the scene
identity.EvaluateVisibility();

// The same, but for a single player
identity.EvaluateVisibility(player);
```

Observers are added and removed right away, and the spawns and despawns that follow are sent to the affected players. Calling it on a client does nothing.

When it is a player that changed rather than an object, you can evaluate everything that is spawned for that one player:

```csharp
using PurrNet.Modules;

networkManager.GetModule<HierarchyFactory>(true).EvaluateVisibilityForPlayer(player);
```

{% hint style="info" %}
While testing, you can right-click a Network Identity component in the inspector and pick `PurrNet > Evaluate Visibility` to run an evaluation by hand.
{% endhint %}

## Deciding when to evaluate

This depends entirely on your game. The goal is to evaluate when the answer could have changed, and not more often than that.

For anything distance based there are two sides to it. When an object moves, the players who can see **it** may change. When a player moves, what **they** can see may change, and that includes objects that never move at all. Evaluating only the things that moved isn't enough: a player walking up to a chest that has been standing still all game would never get it.

The component below covers both. Put it on the player's character, and on anything else that moves around:

```csharp
using PurrNet;
using PurrNet.Modules;
using UnityEngine;

public class VisibilityRefresher : NetworkBehaviour
{
    [SerializeField] private float _moveThreshold = 2f;

    private Vector3 _lastPosition;

    private void FixedUpdate()
    {
        if (!isServer)
            return;

        var position = transform.position;

        if ((position - _lastPosition).sqrMagnitude < _moveThreshold * _moveThreshold)
            return;

        _lastPosition = position;

        // Who can see me?
        EvaluateVisibility();

        // What can I see?
        if (owner.HasValue)
            networkManager.GetModule<HierarchyFactory>(true).EvaluateVisibilityForPlayer(owner.Value);
    }
}
```

Nothing is evaluated while the object stands still, and the threshold decides how far it has to travel before it's worth another look. Keep the threshold well below the range of your rule, or objects will show up noticeably late.

`EvaluateVisibility()` asks the rules once per player in the scene, and `EvaluateVisibilityForPlayer` once per spawned object. Neither is free with a lot of players or objects. If evaluations start showing up in the profiler, raise the threshold, or move the objects that matter most to [whitelists and blacklists](whitelist-and-blacklist.md), which don't need rules to be evaluated at all.
