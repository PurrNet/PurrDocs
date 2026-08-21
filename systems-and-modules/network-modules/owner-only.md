# Owner Only

Sometimes a value should only exist between the server and the owner of an object. A player's inventory, their hand of cards, a quest log, or anything else the other players have no business receiving. `ownerOnly` scopes a network module (and therefore any sync type) so its state is only ever sent to the owner of the parent identity.

Non-owner observers still see the object itself. They just never receive that module's data, and the data never leaves the server for them. This is real scoping, not client-side hiding.

### Usage

Every sync type accepts an `ownerOnly` flag in its constructor:

```csharp
private SyncVar<int> _gold = new(0, ownerOnly: true);
private SyncList<Item> _inventory = new(ownerOnly: true);
private SyncVar<int> _health = new(100); // Normal, everyone receives this
```

You can freely mix owner-only and normal sync types on the same identity, which is the main advantage over [Network Visibility](../network-manager/network-visibility/README.md): visibility rules scope the whole object, `ownerOnly` scopes a single module.

It combines with `ownerAuth` too. An `ownerAuth` + `ownerOnly` sync type lets the owner write the value while the server receives it and nobody else does:

```csharp
private SyncVar<string> _privateNote = new(default, ownerAuth: true, ownerOnly: true);
```

For your own custom modules, override the property:

```csharp
public class InventoryModule : NetworkModule
{
    public override bool ownerOnly => true;
}
```

### Behaviour

* **Ownership transfer:** the new owner is automatically caught up with the current state, and the old owner stops receiving updates. The old owner keeps the last value it saw locally; it is not cleared.
* **No owner:** if the identity has no owner, the module sends to nobody. The server still holds the state, and whoever becomes owner later is caught up at that point.
* **Late joiners:** a joining player who becomes an observer only receives the module's state if they are the owner, same as any other observer.
* **Host:** the host's local client shares the server's objects, so the host always sees every value locally. Nothing extra is sent over the network for it.

Under the hood this works by filtering per-module sends and by only firing `OnObserverAdded` / `OnObserverRemoved` on owner-only modules for the owner. An ownership transfer fires those same callbacks for the old and new owner, which is what drives the automatic catch-up. Custom modules that push state from `OnObserverAdded` are therefore owner-scoped correctly with no extra work.

### Limitations

* `ownerOnly` is meant to be a fixed property of the module. The constructor flag on sync types can't change after creation, and if you override the property on a custom module, don't return a value that changes at runtime: turning it off later won't retroactively send previously-hidden state to existing observers.
* Old owners keep their stale copy of the value after a transfer. If the value is sensitive enough that a stale copy matters, reset it server-side *before* changing ownership, so the reset is the last thing the old owner receives while they can still receive it.
