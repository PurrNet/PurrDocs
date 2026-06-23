# Host

Host means that you are running as both the server and the client.

This is the main thing to keep in mind when working with host logic in PurrNet. A host is not a special third side. It is a server side and a client side running on the same machine.

Because of that, anything in PurrNet that gives you an `asServer` parameter can run twice on a host. This applies to callbacks, ticks, events, and other hooks where PurrNet gives you that bool:

```csharp
protected override void OnSpawned(bool asServer)
{
    base.OnSpawned(asServer);

    if (asServer)
        return;
}
```

One call is for the server side. The other call is for the client side. The `asServer` value tells you which side that specific call belongs to.

`asServer` does not mean "is this machine a host?". It means "is this call currently running as the server side?".

On a dedicated server, that same callback only runs as server. On a normal remote client, it only runs as client. On a host, you get both, because the host is both.

This is why host bugs often show up as logic running twice, inputs being sent twice, events being subscribed twice, or visual/audio logic also running on the server side.

Use `asServer` to split the work:

```csharp
protected override void OnSpawned(bool asServer)
{
    base.OnSpawned(asServer);

    if (asServer)
        SetupServerSide();
    else
        SetupClientSide();
}
```

For server-authoritative gameplay, spawning, validation, damage, score, inventory changes, and similar state changes, you usually want the server side.

For input, camera, UI, audio, local VFX, and other player-facing logic, you usually want the client side.

If the same code should run on both sides, that is also fine. Just make sure it is intentional, because on a host that means it will run twice.
