---
icon: robot
---

# Bots

A bot is a player that only exists on the server. It gets a real [PlayerID](../terminology/playerid-client-connection.md), shows up in the player list on the server and on every client, and can own objects, but there is no client or connection behind it. Use bots for AI players, filling empty slots in a match, or testing with more players than you have.

## Creating a bot

Call `CreateBot()` on the player module, on the server:

```csharp
using PurrNet;
using UnityEngine;

public class BotSpawner : NetworkBehaviour
{
    [SerializeField] private NetworkIdentity botPrefab;

    protected override void OnSpawned(bool asServer)
    {
        if (!asServer)
            return;

        // Server only. Calling this on a client throws an InvalidOperationException.
        PlayerID bot = networkManager.playerModule.CreateBot();

        var botObject = Instantiate(botPrefab);
        botObject.GiveOwnership(bot);
    }
}
```

`CreateBot()` returns the bot's `PlayerID`, with `isBot` set to `true`. Bots take their number from the same counter as real players, so they never get the same id as a client. No client can claim a bot's `PlayerID`, because bots have no cookie to reconnect with.

When a bot is created, `onPlayerJoined` fires on the server and on every client, just like for a real player (`isReconnect` is `false`). In public scenes, the bot is added to the scene and counts as loaded right away.

{% hint style="warning" %}
Don't build bot ids yourself with `new PlayerID(1000 + n, true)`. `PlayerID` equality only compares the number, not `isBot`, so a hand-made id is "equal" to whichever real player or bot already has that number. A made-up id also isn't registered as a player, so PurrNet won't treat it as one. Always use the `PlayerID` that `CreateBot()` gives you.
{% endhint %}

## What a bot doesn't have

A bot has no client, so:

* `playerModule.TryGetConnection(bot, out _)` always returns `false`, and `IsPlayerConnected(bot)` is `false`.
* Anything sent to a bot (broadcasts, Target RPCs, Observers RPCs) is skipped. Nothing runs "on the bot's side".
* Nobody is `isOwner` of a bot's objects, not even the server. Drive bots from server code (check `isServer`) and make their network components server authoritative, instead of relying on `isOwner` or owner authority.
* Host migration doesn't wait for bots to rejoin.

## Checking for bots

Use `isBot` on any `PlayerID`. For an object's owner, check that it has one first:

```csharp
var owner = identity.owner;

if (owner.HasValue && owner.Value.isBot)
{
    // Owned by a bot
}
```

The same works in player events:

```csharp
networkManager.onPlayerJoined += (player, isReconnect, asServer) =>
{
    if (player.isBot)
        return; // e.g. don't spawn a camera or UI for bots
};
```

The host's own player is a normal player, not a bot: use `networkManager.localPlayer` on the host to get it.

## Removing a bot

Kick it from the server:

```csharp
networkManager.playerModule.KickPlayer(bot);
```

This removes the bot from the player list and fires `onPlayerLeft` on the server and on the clients, like a player disconnecting.
