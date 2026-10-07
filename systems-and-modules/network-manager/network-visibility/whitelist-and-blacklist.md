# Whitelist and blacklist

Every Network Identity carries two lists of players that override whatever the [visibility rules](visibility-rules.md) would say:

* **Whitelist**: these players always see the identity.
* **Blacklist**: these players don't see the identity, unless they are on the whitelist too.

They let you manage visibility straight from your own code. Since nothing has to be worked out per player, this is cheaper than rules and the better fit whenever your game already knows who should see what: trigger volumes, rooms, teams, an ability that reveals something for a few seconds.

## The API

```csharp
identity.WhitelistPlayer(player);
identity.RemoveWhitelistPlayer(player);

identity.BlacklistPlayer(player);
identity.RemoveBlacklistPlayer(player);
```

All four can only be called on the server, on an identity that is spawned. They return `true` when the list actually changed.

You don't need to do anything else. On the next tick the server re-evaluates the identity for that player and spawns or despawns it for them.

The current content of the lists can be read through `identity.whitelist` and `identity.blacklist`.

If an object should never reach a player at all, not even for a moment, fill the lists in `OnEarlySpawn`. It runs on the server before the object is sent to anyone:

```csharp
protected override void OnEarlySpawn(bool asServer)
{
    if (asServer)
        BlacklistPlayer(somePlayer);
}
```

## How the lists behave

* **The whitelist wins.** A player who is on both lists sees the identity.
* **Whitelisting doesn't hide anything.** It only guarantees visibility for the players on it. Everyone else is still decided by the rules, and without rules everyone sees everything. To get "only these players", combine it with a rule-set that hides the object, as shown below.
* **Blacklisting beats ownership.** A blacklisted player can't see the identity even if they own it.
* **Removing a player hands them back to the rules.** They aren't forced hidden or visible, the usual checks simply apply again.
* **The lists don't survive a despawn.** They are cleared when the identity despawns, so a pooled object starts clean.
* **Children follow their parent.** Hiding a parent hides everything below it, and a whitelisted child only shows up once the player can see its parent too.

## One list per component

The lists belong to a single network component. If your object has several of them, for example a `NetworkTransform` next to your own `NetworkBehaviour`, or networked children, each one has its own lists and is checked on its own. See [Observers belong to components](./#observers-belong-to-components).

In practice this means:

* **Blacklisting a player on one component doesn't hide the object.** It only silences that component for them. The GameObject stays spawned on their side for as long as another component on it is still visible to them.
* **Whitelisting a player on one component only guarantees that component.** Whether the rest of the object comes along depends on the rules that apply to the other components.

{% hint style="warning" %}
To show or hide a whole object, change the lists on every network component of it, children included, like the example below does.
{% endhint %}

## Only visible to the players you pick

1. Create a rule-set containing the **No Visibility** rule. See [Visibility rules](visibility-rules.md).
2. Assign it as the **Visibility Override** on the first network component of your prefab's root, the top one in the inspector. The components below it and the children use it too. Nobody but the owner sees the object now.
3. Whitelist the players that should see it, on every network component of the object.

## Example: visible while in range

This is the manual version of interest management. Give the object a trigger collider that covers the area it should be seen from, and whitelist players as their character walks in and out. The physics engine does the distance checks for you, and nothing runs while nobody moves across the edge.

The prefab uses a No Visibility override as described above.

```csharp
using PurrNet;
using UnityEngine;

// Goes on the root of the networked object, together with a trigger collider
public class ProximityVisibility : NetworkBehaviour
{
    private NetworkIdentity[] _identities;

    protected override void OnSpawned(bool asServer)
    {
        if (asServer)
            _identities = GetComponentsInChildren<NetworkIdentity>(true);
    }

    private void OnTriggerEnter(Collider other)
    {
        if (isServer && TryGetPlayer(other, out var player))
            SetVisible(player, true);
    }

    private void OnTriggerExit(Collider other)
    {
        if (isServer && TryGetPlayer(other, out var player))
            SetVisible(player, false);
    }

    private static bool TryGetPlayer(Collider other, out PlayerID player)
    {
        var identity = other.GetComponentInParent<NetworkIdentity>();

        if (identity && identity.owner.HasValue)
        {
            player = identity.owner.Value;
            return true;
        }

        player = default;
        return false;
    }

    private void SetVisible(PlayerID player, bool visible)
    {
        for (var i = 0; i < _identities.Length; i++)
        {
            if (visible)
                _identities[i].WhitelistPlayer(player);
            else
                _identities[i].RemoveWhitelistPlayer(player);
        }

        EvaluateVisibility(player);
    }
}
```

`SetVisible` updates every network component of the object, then calls `EvaluateVisibility(player)` on the root so the whole object is spawned or despawned for that player in one go instead of waiting for the next tick.

If a player can have more than one collider inside the trigger at the same time, count them per player and only remove the player from the whitelist when the last one leaves.
