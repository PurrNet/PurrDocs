# Network Visibility

In any multiplayer game with a larger world, you don't want every player receiving data about every single object at all times. If there are hundreds of objects on the other side of the map that a player can't even see, sending all that data is wasted bandwidth and can hurt performance. Network Visibility lets you control which objects each player actually receives, so you only send what matters.

Visibility is decided by the server, per player and per [Network Identity](../../network-identity/). A player who can see an identity is one of its **observers**. Observers have the object spawned on their end and receive its RPCs and synced state. Everyone else doesn't have the object at all.

## Two ways to control it

* [**Visibility rules**](visibility-rules.md) are assets that answer "can this player see this identity?". You group them in a rule-set and assign it to the [Network Manager](../), or to individual identities as an override. The built-in [Distance condition](distance-condition.md) is one of them.
* [**Whitelists and blacklists**](whitelist-and-blacklist.md) let your own code say "this player sees this identity" or "this player never sees it" directly, for example from trigger colliders. Nothing has to be checked per player, so this is the cheapest option when you already know who should see what.

Both can be used together. The lists always win over the rules.

{% hint style="warning" %}
Rules are not checked continuously. PurrNet evaluates visibility at a few fixed moments, like when an object spawns, and it is up to you to ask for a new evaluation when your game state changes. See [Evaluating visibility](using-the-visibility.md).
{% endhint %}

## How the server decides

When an identity is evaluated for a player, the first line that applies wins:

1. The identity's parent is hidden from the player: **hidden**.
2. The player is on the identity's whitelist: **visible**.
3. The player is on the identity's blacklist: **hidden**.
4. No rule-set applies to the identity, neither an override nor one on the Network Manager: **visible**.
5. The player owns the identity: **visible**.
6. At least one rule in the rule-set says yes: **visible**. Otherwise: **hidden**.

So with nothing set up, everyone sees everything.

## Observers belong to components

Visibility is tracked for each network component, not for the GameObject as a whole. Every `NetworkIdentity` on an object, which includes each `NetworkBehaviour`, `NetworkTransform` and any other network component you add, has its own observers and its own whitelist and blacklist.

Most of the time you won't notice. Components on the same GameObject usually go through the same checks with the same rules, so they end up with the same observers. It starts to matter once you treat them differently, for example by blacklisting a player on only one of them:

* **The GameObject stays as long as one component is visible.** A player has the object spawned while they observe at least one of its network components. It is only despawned for them once they observe none.
* **A component they stop observing goes quiet.** It still exists on their side, since it is part of the same GameObject, but from then on its `ObserversRpc` calls and the changes to its synced values no longer reach them. The other components on the object keep working as usual.
* **Observing it again catches it up.** When the player becomes an observer of that component again, they receive its current state. The object isn't respawned for this.
* **The lists talk about one component.** `observers`, `IsObserver`, `whitelist` and `blacklist` all describe the component you read them from, not its neighbours.
* **The callbacks fire in two situations.** `OnObserverAdded` and `OnObserverRemoved` run on every network component of an object when the object appears or disappears for a player, and on a single component when only that one gains or loses the player while the object stays.

Children work the other way around. They are checked after their parent, and a player who observes none of the parent's components can't see anything below it, whatever the children's own lists and rules say. A visible parent doesn't make its children visible though, each child still goes through the checks above.

The same goes for objects you move around: parenting an object under something a player can't see takes it away from that player, and moving it back out returns it.

{% hint style="info" %}
If all you want is to show or hide a whole object, make the same change on every network component of it. See [Whitelist and blacklist](whitelist-and-blacklist.md) for an example.
{% endhint %}

## What a visibility change does

|                         | For that player                                                                                                                             | On the server                 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| **Gaining visibility**  | The object is spawned for them with its current state, exactly as if it had just been spawned.                                              | `OnObserverAdded(player)`     |
| **Losing visibility**   | The object is despawned for them. It is destroyed, or returned to the pool if the prefab uses [pooling](../../network-identity/pooling.md). | `OnObserverRemoved(player)`   |

The object keeps existing and simulating on the server and for every other observer. Only that one player's copy comes and goes.

While a player isn't an observer of an identity:

* `ObserversRpc` calls don't reach them.
* A `TargetRpc` aimed at them is not sent, and logs an error.
* A `ServerRpc` they send to it is rejected.

{% hint style="info" %}
When you play as a host, the host's player shares its objects with the server. Hiding an object from the host removes them from the observers, but the object stays in the scene since the server still needs it.
{% endhint %}

## Reading the observers

On the server, every identity exposes who is currently observing it:

```csharp
IReadOnlyList<PlayerID> players = identity.observers;
bool canSee = identity.IsObserver(player);
```

You can react to changes by overriding `OnObserverAdded` and `OnObserverRemoved` on your `NetworkBehaviour`, or by subscribing to the `onObserverAdded` and `onObserverRemoved` events of an identity. These only run on the server.

```csharp
protected override void OnObserverAdded(PlayerID player)
{
    Debug.Log($"{player} can now see {name}");
}

protected override void OnObserverRemoved(PlayerID player)
{
    Debug.Log($"{player} can no longer see {name}");
}
```

While playing in the editor as the server, the inspector of a spawned identity also lists its current observers.
