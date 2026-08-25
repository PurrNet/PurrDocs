---
icon: plug
---

# Providers

A provider is a ScriptableObject implementing one of four roles. You assign one of each to a [`GameOrchestrator`](how-it-works.md#the-orchestrator), and PurrLobby never talks to a backend directly.

Every backend ships **preset assets** under `Assets/PurrLobby/Providers/<backend>/Preset`, including a pre-filled orchestrator. Drag `Orchestrator.PurrNet`, `Orchestrator.Steam` or `Orchestrator.Nakama` onto your `LobbyManager` and you are running.

## Capabilities

Lobby providers declare what they support through `LobbyCapabilities`:

`CreateLobby`, `JoinLobbyById`, `JoinLobbyByCode`, `JoinRandom`, `QueryLobbies`, `PrivateLobbies`.

The menu reads these and hides what a backend cannot do, so a provider without `QueryLobbies` shows no browser button and one without `PrivateLobbies` shows no visibility toggle. Providers default to `All`, so a custom provider only overrides the property when it needs to restrict something.

## PurrNet Services

The smallest complete backend, and the best reference when writing your own.

| Asset | Settings |
| ----- | -------- |
| `PurrNetSessionProvider` | None. Uses PurrServices device login, pushing a login view when needed. |
| `PurrNetLobbyProvider` | `Max Players` (default 4) |
| `PurrTransportGameAllocator` | `Game Scene` |
| `LobbyMatchmaker` (generic) | `Lobby Provider`, `Lobby Name Prefix` |

Requires PurrServices. Game traffic runs over PurrTransport relay, with the lobby id used as the room name.

## Steam

Full capability set, including private lobbies and the friends and overlay flow.

| Asset | Settings |
| ----- | -------- |
| `SteamSessionProvider` | None. Uses the running Steam client. |
| `SteamLobbyProvider` | `Max Players` (default 4) |
| `SteamGameAllocator` | `Game Scene` |
| `LobbyMatchmaker` (generic) | `Lobby Provider`, `Lobby Name Prefix` |

Requires [Steamworks.NET](https://github.com/rlabrecque/Steamworks.NET). Game traffic runs over Steam relay sockets to the lobby owner's SteamID. Accepted overlay invites arrive through `onExternalJoinRequested` and are handled without any work on your side.

## Nakama

The most capable backend PurrLobby ships with, and the one to use if you want your lobby layer and the rest of your live game on one server. Covers session, lobby, matchmaking and game allocation in a single stack, self-hosted or on Heroic Cloud.

| Asset | Settings |
| ----- | -------- |
| `NakamaConfig` | `Scheme` (Http), `Host` (127.0.0.1), `Port` (7350), `Server Key` (defaultkey) |
| `NakamaSessionProvider` | `Config`, `Session PlayerPref Key` |
| `NakamaLobbyProvider` | `Session Provider`, `Max Players`, `Snapshot Timeout Ms` (4000), `Query Limit` (100) |
| `NakamaMatchmakingProvider` | `Min Count` (2), `Max Count` (4) |
| `NakamaGameAllocator` | `Game Scene`, `Wait For Game Start Flag` (off) |

Nakama runs locally in Docker, so you can develop against a real server from the start.

{% hint style="warning" %}
`NakamaGameAllocator` runs gameplay over Nakama's relayed match socket, a WebSocket with a custom message protocol. It suits turn-based games, not fast-paced ones. Keep Nakama for session, lobby and matchmaking and swap just the allocator for `PurrTransportGameAllocator`, `SteamGameAllocator` or `EdgegapGameAllocator` when you need tighter networking.
{% endhint %}

{% hint style="success" %}
See the dedicated [**Nakama**](nakama.md) page for running a server locally, pointing PurrLobby at it, and extending the lobby browser with a server module.
{% endhint %}

Requires [Nakama Unity](https://github.com/heroiclabs/nakama-unity).

## Edgegap

Edgegap is matchmaking plus managed dedicated servers, so there is no lobby provider at all.

| Asset | Settings |
| ----- | -------- |
| `EdgegapMatchmakingProvider` | Configured through its custom inspector |
| `EdgegapGameAllocator` | `Game Scene`, `Transport` (UDP), `Port Name`, `Deployment Timeout Ms` (300000), `Poll Interval Ms` (2000) |

The matchmaker forms the match **and** deploys the server, so a found match already carries usable connection info and there is no separate allocation step. Pair `EdgegapMatchmakingProvider` with `EdgegapGameAllocator` so both agree on transport and port selection.

`Port Name` is optional; leave it empty to use the first port whose protocol matches the chosen transport.

Because players connect to a dedicated server, this allocator sets `supportsHosting` to false. A host request is downgraded to a client connection automatically.

Requires the [Edgegap Unity plugin](https://github.com/edgegap/edgegap-unity-plugin) and PurrServices.

## The generic matchmaker

`LobbyMatchmaker` gives lobby-based backends matchmaking without a dedicated matchmaking service. It quick-joins an open lobby when the provider supports `JoinRandom`, otherwise it queries and joins, and creates one named with `Lobby Name Prefix` if nothing suitable exists.

Settings: `Lobby Provider`, `Lobby Name Prefix` (default `Matchmaking`).

## Missing SDKs

Provider code is wrapped in compile guards keyed to its SDK (`STEAMWORKS`, `NAKAMA`, `PURR_SERVICES`). A backend whose package is not installed compiles out entirely, so you only need the SDKs for the backends you actually use.

Providers also carry `[ProviderDependency]`, which surfaces a clear inspector warning naming the missing package instead of a compile error.

## Writing your own

See [Custom providers](custom-providers.md).
