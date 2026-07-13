# Predicted Identity Spawner

`PredictedIdentitySpawner` bridges a predicted object's lifetime to ordinary PurrNet `NetworkIdentity` components. Use it when predicted creation must also produce a normal networked object for RPCs, visibility, ownership, or other PurrNet systems.

Add the component to the predicted prefab and populate **Identities To Spawn** with Network Identities in that prefab hierarchy.

## What it coordinates

On the server, the spawner:

1. Reserves consecutive network IDs.
2. Performs early spawn for each configured identity.
3. Adds relevant observers.
4. Copies the predicted object's owner.
5. Finalizes the network spawns.

The first reserved network ID is stored in predicted state. During a verified client replay, clients use it to mirror early spawn and request observer status before finalizing the identities.

When the predicted object is removed, the spawner manually despawns its network identities. Observer changes are forwarded to the spawned identities as well.

## Constraints

The component is marked `[PredictionUnsafe]` because it performs normal networking side effects from predicted lifetime state. Keep the rest of the object's simulation deterministic and do not manually spawn the same Network Identities elsewhere.

{% hint style="warning" %}
Do not use this component for ordinary predicted projectiles or effects that only need predicted state. Each bridged identity enters the normal PurrNet hierarchy and carries the corresponding spawn, visibility, and ownership cost.
{% endhint %}

For purely predicted creation and deletion, use [Predicted Hierarchy](../predicted-hierarchy.md) instead.
