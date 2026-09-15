# Network Identities in Predicted Prefabs

Predicted objects sometimes need to be ordinary PurrNet objects too: a predicted player pawn that also wants RPCs, a `SyncVar` for a display name, PurrNet visibility, or any other system that works on `NetworkIdentity`. This page covers how a `NetworkIdentity` inside a predicted prefab gets spawned and kept in sync with the predicted object's lifetime.

{% version range=">=1.4.0" %}

## Automatic mirroring

Any `NetworkIdentity` component inside a predicted prefab is spawned and despawned automatically. There is nothing to add and nothing to configure: put the component on the prefab, register the prefab in your Predicted Prefabs asset, and create it through the hierarchy as usual.

The server drives the mirror:

* When an instance enters the authoritative topology, the server reserves one block of network ids for every `NetworkIdentity` in the prefab and spawns them.
* The predicted owner is copied to the network identities and follows later ownership changes.
* Each player observes the network identities exactly while the predicted root is visible to them, and only after they have acknowledged a frame that contains the instance, so buffered RPCs are never sent to a client that cannot route them yet.
* When the instance leaves the topology or goes back to the pool, the identities are despawned.

Clients only mirror what the server has verified. A speculatively created instance, or one that gets rolled back, never touches PurrNet at all. Once the instance is part of a verified frame, the client spawns the identities under the ids the server allocated, so both sides agree on which component has which `NetworkID`.

Every peer collects the identities in the same order, which is the deterministic hierarchy order: the components on the piece itself, then the ones on descendants that are not pieces of their own. Nested pieces own their own identities, so a prefab made of several pieces gets one contiguous id block split across them. If a piece ends up with a different number of `NetworkIdentity` components than the prototype expects, for example because you added or removed one at runtime, the mirror logs an error and skips that piece.

## Who is in charge

The predicted world is. The mirror uses PurrNet's manual spawn path, so the mirrored identities are flagged as manually spawned and PurrNet's own spawn rules and visibility rules never run for them:

* Lifetime follows the predicted instance. You never call `Spawn` or `Despawn` on them yourself, and PurrNet will not despawn them on its own.
* Observers are set by the mirror from predicted visibility. PurrNet's visibility evaluation skips manually spawned identities, so visibility conditions and rules on them are ignored. To stop a player from seeing one, hide the predicted root from that player with `predictionManager.HideFrom(player, rootId)`; the mirrored identities follow. See [Predicted Visibility](../predicted-visibility.md).
* Ownership is copied from the predicted owner when the identities spawn and again whenever the predicted owner changes. Ownership you set on the identity directly survives only until the next predicted ownership change, so treat the predicted owner as the source of truth.
* PurrNet's despawn-on-owner-disconnect rule also skips manually spawned identities. Their teardown is entirely the predicted instance being deleted.

What PurrNet still provides is everything that hangs off a live `NetworkIdentity` with an id, an owner, and observers: RPCs and their ownership checks, `SyncVar` and the other sync types, `OnSpawned` and `OnObserverAdded` callbacks, and lookups by `NetworkID`. Use the mirrored identity for that, and keep gameplay decisions in predicted state.

{% hint style="warning" %}
No spawn packet is sent for mirrored identities, since every peer materializes them from the predicted topology, but each one still carries observer bookkeeping, buffered RPC delivery, and sync traffic. Do not put a `NetworkIdentity` on high-volume predicted prefabs such as projectiles or debris unless you really need PurrNet features on them.
{% endhint %}

## Upgrading from PredictedIdentitySpawner

`PredictedIdentitySpawner` is obsolete and no longer does anything. Remove it from your prefabs; the identities it used to list are now picked up automatically. The old component is hidden from the Add Component menu and only kept so existing prefabs still load.

{% endversion %}

{% version range="<1.4.0" %}

## Predicted Identity Spawner

`PredictedIdentitySpawner` bridges a predicted object's lifetime to ordinary PurrNet `NetworkIdentity` components. Use it when predicted creation must also produce a normal networked object for RPCs, visibility, ownership, or other PurrNet systems.

Add the component to the predicted prefab and populate **Identities To Spawn** with Network Identities in that prefab hierarchy.

### What it coordinates

On the server, the spawner:

1. Reserves consecutive network IDs.
2. Performs early spawn for each configured identity.
3. Adds relevant observers.
4. Copies the predicted object's owner.
5. Finalizes the network spawns.

The first reserved network ID is stored in predicted state. During a verified client replay, clients use it to mirror early spawn and request observer status before finalizing the identities.

When the predicted object is removed, the spawner manually despawns its network identities. Observer changes are forwarded to the spawned identities as well.

### Constraints

The component is marked `[PredictionUnsafe]` because it performs normal networking side effects from predicted lifetime state. Keep the rest of the object's simulation deterministic and do not manually spawn the same Network Identities elsewhere.

{% hint style="warning" %}
Do not use this component for ordinary predicted projectiles or effects that only need predicted state. Each bridged identity enters the normal PurrNet hierarchy and carries the corresponding spawn, visibility, and ownership cost.
{% endhint %}

{% endversion %}

For purely predicted creation and deletion, use [Predicted Hierarchy](../predicted-hierarchy.md) instead.
