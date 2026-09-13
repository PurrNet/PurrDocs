---
icon: layer-group
---

# Scene setup

PurrLobby needs two scenes: a menu scene running the UI, and your game scene. The menu scene does the work. The game scene needs one component you almost certainly already have.

## Menu scene

Open `Assets/PurrLobby/Scenes/MenuScene.unity` for a working reference, or build your own:

1. Drop in the **LobbyManager** prefab from `Assets/PurrLobby/Prefabs/Setup`. It carries the `LobbyManager` component and a PurrUI `ViewStack` with the view collection already assigned.
2. Create a `GameOrchestrator` asset (**Create → PurrLobby → Menu Orchestrator**) and assign it to the `LobbyManager`.
3. Fill the orchestrator's four provider slots. Preset assets ship under `Assets/PurrLobby/Providers/<backend>/Preset`, so for most backends you can drag the presets straight in. Each backend also ships a pre-filled orchestrator, so dragging `Orchestrator.PurrNet` or `Orchestrator.Steam` onto the `LobbyManager` covers steps 2 and 3 at once.

That is the whole menu scene. The main menu, lobby browser, create-lobby, join-by-code, matchmaking and lobby screens are all view prefabs pushed on demand.

## Game scene

`Assets/PurrLobby/Scenes/GameScene.unity` is the shipped example. Your game scene needs a `NetworkManager`, and that is it.

The `PurrNet-PurrTransport` prefab in `Assets/PurrLobby/Prefabs/Networked` is a `NetworkManager` already set up the way PurrLobby expects, so dropping it in saves you the configuration pass. Your own `NetworkManager` works just as well.

You do **not** need to:

* Assign a transport. The allocator adds and configures the correct one during `Connect`.
* Add a `GameSession`. `LoadGameScene` creates one in the scene automatically.
* Add anything to return to the menu. `GameSession` handles leave, game over and disconnect.
* Turn off the auto-start flags. See below.

### Auto-start flags

Leave them set however suits you. PurrLobby globally suppresses auto-start for the duration of its own scene load and restores it two frames later, so the lobby always owns starting the host or client and your flags never race it.

{% hint style="success" %}
This is a convenience, not a restriction. Your game scene can double as your testing environment: keep the `Editor` start flags on, press play on the game scene directly, and it boots straight into a networked session with no lobby in the way. Launch that same scene through the lobby and the flags quietly stand aside.
{% endhint %}

`Connect` does refuse to run while auto-start is genuinely active, but that only happens if the scene was entered outside the lobby flow, which is the case where racing would actually bite.

### Optional: pause menu and game over

Drop the **SessionManager** prefab from `Assets/PurrLobby/Prefabs/Setup` into the game scene. It is a self-contained bundle carrying a `ViewStack` for the in-game canvases, a `PushPauseMenu` that owns the Escape key, and a `GameOverBroadcaster` that lets the server end the match for everyone.

No wiring needed. See [In the game scene](in-game.md) for what it gives you and how to hook into it.

## Pointing at your game scene

The game scene is set **on the game allocator asset**, not on the `LobbyManager`. Every allocator has a `Game Scene` field:

| Allocator | Asset menu |
| --------- | ---------- |
| `PurrTransportGameAllocator` | Create → PurrLobby → PurrNet → Game Allocator |
| `SteamGameAllocator` | Create → PurrLobby → Steam → Game Allocator |
| `NakamaGameAllocator` | Create → PurrLobby → Nakama → Game Allocator |

The field uses a scene picker, so you select the scene asset rather than typing a name. Leaving it empty throws a clear error when a launch is attempted rather than failing silently.

Add both scenes to **File → Build Settings**. Unity can only load scenes that are in the build list, including in the editor.

## Checklist

* Menu scene has `LobbyManager` with an orchestrator assigned.
* Orchestrator has its provider slots filled.
* Game allocator has `Game Scene` set.
* Game scene has a `NetworkManager`. Auto-start flags can stay on.
* Game scene has the `SessionManager` prefab, if you want the pause menu and game over.
* Both scenes are in Build Settings.
