---
icon: diagram-project
---

# How it works

PurrLobby is built from three pieces that stay out of each other's way: an **orchestrator** asset that names your backend, a **menu scene** that runs the UI, and a **game scene** that receives the connection. Nothing is hardcoded to a specific backend, and the game scene needs almost no PurrLobby-specific setup.

## The orchestrator

`GameOrchestrator` is a ScriptableObject (**Create → PurrLobby → Menu Orchestrator**) holding four provider slots:

| Slot | Responsibility |
| ---- | -------------- |
| `sessionProvider` | Logging the player in and giving them an identity |
| `lobbyProvider` | Creating, joining, listing and leaving lobbies |
| `matchmakingProvider` | Ticket-based matchmaking |
| `gameAllocator` | Producing connection info and loading the game scene |

Mix and match freely. A Steam lobby with the generic matchmaker and Steam sockets is just as valid as PurrNet Services lobbies with PurrTransport.

The asset is shared between the menu and game scenes, so it doubles as the handoff channel. At runtime it also carries:

* `activeLobby`, the lobby the player is currently in, so the game scene can leave it on the way out.
* `lastExitReason`, why the player last left the game, which the menu reads when it comes back.
* `menuScene`, captured automatically when the game scene loads so the return trip knows where to go.

`GameOrchestrator.active` is a static pointing at whichever orchestrator booted the menu. That is how the game scene finds its way back without any wiring.

## Boot

`LobbyManager` drives startup. It holds the orchestrator and a PurrUI `ViewStack`, and on `Start` it runs:

1. Set `GameOrchestrator.active`.
2. `sessionProvider.Login(stack)`. The provider may push its own views here, which is how the device login screen appears when a backend needs credentials.
3. `Initialize()` on the lobby, matchmaking and allocator providers.
4. Subscribe to `onExternalJoinRequested` so platform invites work.
5. Push `MainMenuView`.

Everything after that is view-driven.

{% hint style="info" %}
External joins (an accepted Steam overlay invite, for example) are handled for you. `LobbyManager` leaves the current lobby, shows a loading view, joins the requested one, and opens the lobby screen. It is skipped silently if the provider does not advertise `JoinLobbyById`.
{% endhint %}

## Starting a game

When every player is ready, the lobby owner runs the launch flow:

1. `gameAllocator.AllocateGame(lobby)` returns a `ConnectionInfo` (server address, host id).
2. The connection info is written into lobby metadata so every member receives it.
3. `gameAllocator.LoadGame(lobby)` loads the game scene.
4. `gameAllocator.Connect(connection, shouldBeHost: true)` starts the host.

Clients see the metadata change, run steps 3 and 4 themselves, and connect with `shouldBeHost: false`. Both paths run behind a `LoadingView`, and a failure at any step toasts the error and resets the ready state instead of stranding the lobby.

## The scene load

`LoadGameScene` does more than call `LoadSceneAsync`:

* Records the current scene as `menuScene` for the return trip.
* Calls `NetworkManager.DisableFlags()`, globally suppressing auto-start until two frames after the load. This is what lets your game scene keep its auto-start flags and double as a testing environment: press play on it directly and it boots into a networked session, launch it through the lobby and the flags stand aside.
* Creates a `GameSession` in the loaded scene via `GameSession.EnsureInScene`.

That last point is why the game scene needs no PurrLobby component: the session is added for you.

## Connecting

`GameAllocatorProvider.Connect` is shared by every allocator and does the guard work in one place:

* Aborts if the scene's `GameSession` is already exiting.
* Requires a `NetworkManager` in the scene, and refuses to run if auto-start is still active. During a lobby-driven load it never is, since `LoadGameScene` has suppressed it.
* Downgrades a host request to a client when the allocator sets `supportsHosting` to false, which is what dedicated-server backends like Edgegap do.
* Calls the allocator's `ConfigureTransport`, which adds and configures the right transport component on the `NetworkManager`.
* Calls `StartHost()` or `StartClient()`.

Because `ConfigureTransport` adds the transport itself, you do not pre-assign one in the game scene. `PurrTransportGameAllocator` adds a `PurrTransport` and points it at the lobby id; `SteamGameAllocator` adds a `SteamTransport` aimed at the host's SteamID.

## Coming back

`GameSession` owns the return trip and covers three cases:

* The player chooses to leave.
* The server ends the game through `GameOverBroadcaster.EndGame()`.
* The connection drops unexpectedly, in which case clients retry for a configurable window before giving up.

In all three it leaves the lobby, loads `menuScene`, and records `lastExitReason` on the orchestrator.

## Next

* [Scene setup](scene-setup.md) for what actually goes in each scene.
* [Providers](providers.md) for backend-specific settings.
* [In the game scene](in-game.md) for the pause menu, game over and reconnect.
* [Customizing the UI](customizing-the-ui.md) for the view system.
