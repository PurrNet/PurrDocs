---
icon: server
---

# Nakama

[Nakama](https://heroiclabs.com/) by Heroic Labs is an open-source game server offering authentication, lobbies, matchmaking, chat, leaderboards, storage and more. It runs anywhere: your laptop, your own infrastructure, or Heroic Cloud.

It is also the most capable backend PurrLobby ships with, and the one to reach for when you want your lobby layer and the rest of your live game to share a single server.

{% hint style="success" %}
Heroic Labs is a PurrNet sponsor. If you are choosing a backend, Nakama is well worth a look: the server is Apache 2.0 licensed and free to self-host, so you can build and ship on your own infrastructure with no vendor lock-in, and move to their managed [Heroic Cloud](https://heroiclabs.com/pricing/) later if you would rather not run it yourself.
{% endhint %}

## What PurrLobby uses

| Role | Asset | Backed by |
| ---- | ----- | --------- |
| Session | `NakamaSessionProvider` | Nakama device authentication |
| Lobby | `NakamaLobbyProvider` | Nakama relayed matches |
| Matchmaking | `NakamaMatchmakingProvider` | Nakama's ticket matchmaker |
| Game allocation | `NakamaGameAllocator` | Nakama relayed match as the transport |

Preset assets for all four, plus a pre-filled orchestrator, ship in `Assets/PurrLobby/Providers/Nakama/Preset`. Drag `Orchestrator.Nakama` onto your `LobbyManager` and the only thing left to configure is where your server lives.

Requires [Nakama Unity](https://github.com/heroiclabs/nakama-unity).

## Choosing a game allocator

{% hint style="warning" %}
**`NakamaGameAllocator` is not meant for fast-paced games.** It routes gameplay through Nakama's relayed match socket, which is a WebSocket carrying a custom message protocol. That is a good fit for turn-based games, card and board games, and anything else where a little latency does not hurt. It is the wrong tool for shooters, fighting games, racing, or anything else that needs tight networking.
{% endhint %}

This is not a problem, because the orchestrator's four slots are independent. Nakama is excellent at session, lobby and matchmaking, so keep it for those and swap **only** the game allocator for a lower-latency path:

| Instead of `NakamaGameAllocator` | Gameplay runs over |
| -------------------------------- | ------------------ |
| `PurrTransportGameAllocator` | PurrTransport relay |
| `SteamGameAllocator` | Steam relay sockets |
| `EdgegapGameAllocator` | A deployed dedicated server |
| Your own allocator | Whatever transport you configure, for example direct UDP |

A Nakama session, Nakama lobbies, the Nakama matchmaker, and PurrTransport for the actual match is a perfectly normal setup. Nothing in the menu flow changes; only the transport the game scene connects with does.

See [Custom providers](custom-providers.md) if none of the shipped allocators fit.

## Testing locally

Nakama runs locally in Docker, which is the fastest way to develop against it.

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
2. Make a folder for the server and drop in a `docker-compose.yml`. Heroic Labs publishes ready-made PostgreSQL and CockroachDB compose files in their [Docker install guide](https://heroiclabs.com/docs/nakama/getting-started/install/docker/).
3. From that folder, start it:

```bash
docker compose up
```

The server comes up on `127.0.0.1`:

| Port | Purpose |
| ---- | ------- |
| 7350 | Client API, which is what PurrLobby connects to |
| 7351 | Developer console |
| 7349 | gRPC |

Open [http://127.0.0.1:7351](http://127.0.0.1:7351) for the console and sign in with `admin` / `password`. The console is useful while building a lobby flow: you can watch matches appear and disappear, inspect accounts created by device login, and read server logs as players join.

The shipped `Nakama Config` preset already points at this local server, so a fresh Docker instance needs no changes on the Unity side.

{% hint style="warning" %}
The local defaults are development values. Change the server key before exposing an instance to anything other than your own machine, and use HTTPS in production.
{% endhint %}

## Pointing at a server

The endpoint lives on a `NakamaConfig` asset (**Create → PurrLobby → Nakama → Config**), which `NakamaSessionProvider` references:

| Field | Default | Notes |
| ----- | ------- | ----- |
| `Scheme` | `Http` | Switch to `Https` for any remote server |
| `Host` | `127.0.0.1` | Hostname or IP |
| `Port` | `7350` | Client API port. Usually `443` over HTTPS |
| `Server Key` | `defaultkey` | Must match your server's configured key |

For Heroic Cloud or your own hosted instance, set `Scheme` to `Https`, `Host` to the address from your Heroic Labs dashboard, `Port` to `443`, and `Server Key` to the key configured on that instance.

### Is the server key safe to ship?

{% hint style="info" %}
**Yes. The server key is designed to live in your client.** Every Nakama client SDK takes it as a constructor argument, so it ends up in your build where a determined player can read it. Its privileges are deliberately narrow: it gates only the authentication and session-refresh endpoints, and every other API call requires a session token. Someone holding just the key can create an account on your server, not act as an existing player.

Two rules still apply:

* **Change it from `defaultkey` before you go live**, and use `Https` so it is not readable in transit. Heroic Labs recommends regenerating `server_key`, `session.encryption_key` and `runtime.http_key` server-side, all to different random values.
* **Never ship Nakama's `http_key` or console credentials.** The runtime `http_key` bypasses session authentication and would expose every server RPC. `NakamaConfig` has no field for either, so it cannot carry them into a build by accident.
{% endhint %}

Real protection comes from server-side logic, not the key. If you add runtime RPCs, guard each one by checking for a user id in the context, since Nakama has no middleware-style policy layer.

Keeping separate config assets for local and hosted, and swapping which one the session provider references, is an easy way to move between them without editing fields.

## Provider settings

| Asset | Settings |
| ----- | -------- |
| `NakamaSessionProvider` | `Config`, `Session PlayerPref Key` |
| `NakamaLobbyProvider` | `Session Provider`, `Max Players`, `Snapshot Timeout Ms` (4000), `Query Limit` (100) |
| `NakamaMatchmakingProvider` | `Min Count` (2), `Max Count` (4) |
| `NakamaGameAllocator` | `Game Scene`, `Wait For Game Start Flag` (off) |

`Session PlayerPref Key` is where the device session token is cached so returning players skip the login step. `Snapshot Timeout Ms` is how long a joining player waits for the lobby owner's first state snapshot before failing the join. `Query Limit` caps how many matches the browser lists. `Wait For Game Start Flag` makes the host wait on a lobby metadata flag before connecting; leave it off unless your flow needs it, since most pre-load the scene and connect immediately.

## Capabilities, and how to lift them

The Nakama provider advertises `CreateLobby`, `JoinLobbyById`, `JoinLobbyByCode` and `QueryLobbies`.

{% hint style="info" %}
**These limits are ours, not Nakama's.** Nakama is fully capable of rich lobby listings, private lobbies and random join. Doing so needs a custom server module written against its Go, Lua or TypeScript runtime, and this provider deliberately targets a **stock Nakama server** so it works with no server-side setup and no extra dependencies.

The trade-off: relayed matches expose no name, metadata or max size to the browser, so entries show the match id, every match is publicly listed, and query filters are ignored.
{% endhint %}

To lift any of it, write a server RPC that returns the listing data you want and override `QueryLobbies` in a subclass to call it. Nakama's [server framework docs](https://heroiclabs.com/docs/nakama/server-framework/) cover writing runtime modules. Matchmaking is unaffected either way and uses Nakama's real ticket-based matchmaker, so quick-play works fully on a stock server.

## Further reading

* [Nakama documentation](https://heroiclabs.com/docs/nakama/)
* [Unity client guide](https://heroiclabs.com/docs/nakama/client-libraries/unity/)
* [Matchmaker](https://heroiclabs.com/docs/nakama/concepts/multiplayer/matchmaker/)
* [Relayed multiplayer](https://heroiclabs.com/docs/nakama/concepts/multiplayer/relayed/)
