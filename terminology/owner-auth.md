# Owner Auth

Owner Auth is a setting found across PurrNet that determines whether the owning player or the server has authority over a networked value or component.

When owner auth is **enabled**, the owning client is the one responsible for controlling and updating the value. This means only the owner can modify it, and changes are sent from the owner to everyone else. If the object has no owner, the server automatically takes over as the [controller](controller.md).

When owner auth is **disabled**, only the server can control the value, regardless of who owns the object.

## Where you'll see it

Owner auth shows up in two main places:

**Sync types** have it as a constructor parameter:

```csharp
// Server authed (default)
private SyncVar<int> health = new(100);

// Owner authed
private SyncVar<int> health = new(100, ownerAuth: true);
```

This works the same for [SyncList](../systems-and-modules/network-modules/sync-types/synclist.md), [SyncDictionary](../systems-and-modules/network-modules/sync-types/syncdictionary.md), [SyncEvent](../systems-and-modules/network-modules/sync-types/syncevent.md) and other sync types.

**Plug n' play components** like [Network Transform](../plug-n-play-components/network-transform.md) and [Network Rigidbody](../plug-n-play-components/network-rigidbody.md) expose it as a toggle in the inspector. Enabling it lets the owning player directly control the object's movement, while disabling it means the server is in charge.

## When to use it

Owner auth is great for things the owning player should directly control, such as player movement, camera-related values or personal settings. It gives the owner immediate responsiveness since they don't have to go through the server first.

For anything gameplay-critical like scoring, health in competitive games, or shared world state, you're better off keeping owner auth disabled and letting the server be the authority. See [Server Auth (Safe)](server-auth-safe.md) and [Client Auth/Everyone (Unsafe)](client-auth-everyone-unsafe.md) for more on that.

## Related

* [Controller](controller.md) - How PurrNet decides who is "in control" of an object
* [Ownership](../systems-and-modules/network-identity/ownership.md) - How ownership itself works
* [Network Rules](../systems-and-modules/network-manager/network-rules.md) - Configuring authorization at a broader level
