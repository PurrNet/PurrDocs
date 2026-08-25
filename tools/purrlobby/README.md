---
icon: users
---

# PurrLobby (Lobbies & matchmaking)

**PurrLobby** is a drop-in lobby and matchmaking front-end for PurrNet. It gives you the multiplayer menu flow most games need: create a lobby, share a code, browse open lobbies, chat, ready up, matchmake, and move everyone into a game scene together.

The UI is already built. You choose a backend, assign your game scene, and customize the prefabs as needed.

{% hint style="info" %}
PurrLobby replaces the older [Lobby System](../../addons/lobby-system.md) addon. New projects should use PurrLobby.
{% endhint %}

## In this section

| Page | What it covers |
| ---- | -------------- |
| [How it works](how-it-works.md) | The orchestrator, the boot sequence, and the menu to game handoff |
| [Scene setup](scene-setup.md) | What goes in the menu scene and the game scene |
| [Providers](providers.md) | Every backend and its settings |
| [Nakama](nakama.md) | Running a server locally and pointing PurrLobby at it |
| [Customizing the UI](customizing-the-ui.md) | Views, the stack, theming, adding your own screens |
| [In the game scene](in-game.md) | Pause menu, game over, reconnect, voice |
| [Custom providers](custom-providers.md) | Writing your own backend or transport |

## Requirements

* Unity **2022.3** or newer.
* [PurrNet](https://github.com/PurrNet/PurrNet) and [PurrUI](https://github.com/PurrNet/PurrUI).
* PurrServices when using the PurrNet Services lobby provider or the Edgegap game allocator.
* Optional, depending on backend:
  * [Steamworks.NET](https://github.com/rlabrecque/Steamworks.NET) for the Steam providers.
  * [Nakama Unity](https://github.com/heroiclabs/nakama-unity) for the Nakama providers.
  * [Edgegap Unity plugin](https://github.com/edgegap/edgegap-unity-plugin) for Edgegap workflows.

Provider code is compiled out when its SDK is absent, so you only need the packages for the backends you actually use.

## Installing

Install through the PurrNet package manager.

1. Open **Tools → PurrNet → PurrNet Packages** (`Ctrl+Shift+Alt+P`).
2. Find **PurrLobby** in the list.
3. Pick a version from the dropdown and hit **Install**.

PurrUI is listed as a dependency and is installed alongside it. The package page is at [purrnet.dev/packages/purrlobby](https://purrnet.dev/packages/purrlobby).

## Updating

{% hint style="warning" %}
**Updating replaces the contents of the package.** Any edits you have made to the shipped prefabs, scenes, materials, or scripts are overwritten.
{% endhint %}

Two ways to keep your customizations:

* **Deselect your files during import.** PurrLobby imports into `Assets/PurrLobby`, so updating opens Unity's interactive import window. Uncheck anything you have modified to keep your version and take the rest of the update.
* **Duplicate before you edit.** Copy any prefab or asset you plan to change into your own folder outside `Assets/PurrLobby` and point your scenes at the copy. Updates then never touch your version.

The second approach is worth doing up front if you expect to restyle the UI heavily, since it keeps your work fully separate from the package.

## What's included

* Menu flow: main menu, create lobby, join by code, lobby browser, matchmaking, and an in-lobby view with a player list and chat.
* Ready-up and owner-driven game start.
* A scene handoff that loads the game scene, connects the network transport, and returns to the menu on leave, game over, or connection loss.
* A ready-made pause menu and server-driven game over for the game scene.
* Swappable provider interfaces for sessions, lobbies, matchmaking, and game allocation.
* Prefabs and sample scenes under `Assets/PurrLobby`.

## Providers

| Provider         | Lobbies                   | Lobby Browser     | Matchmaking                  | Game Allocation       |
| ---------------- | ------------------------- | ----------------- | ---------------------------- | --------------------- |
| PurrNet Services | yes                       | yes               | via generic lobby matchmaker | PurrTransport         |
| Steam            | yes                       | yes               | via generic lobby matchmaker | Steam sockets         |
| Nakama           | create/join by id or code | basic (ids only)  | yes                          | Nakama relayed match  |
| Edgegap          | no                        | no                | yes                          | managed server assignment |

Providers advertise their optional lobby actions through `LobbyCapabilities`. The menu hides unsupported buttons automatically, so a backend without lobby browsing will not show the browser entry point.

{% hint style="info" %}
**The Nakama row describes this provider, not Nakama.** Nakama is fully capable of everything in the table, including rich lobby listings, private lobbies and random join, through a custom server module. This provider deliberately targets a stock Nakama server so it works with no server-side setup. See the [Nakama page](nakama.md) for the details and how to lift the limits.
{% endhint %}

Full settings for every backend are on the [Providers](providers.md) page.

## Getting started

1. Open `Assets/PurrLobby/Scenes/MenuScene.unity` for a working example, and `GameScene.unity` for the receiving end.
2. Select the **LobbyManager** in the scene and assign a `GameOrchestrator`. Preset orchestrators live under `Assets/PurrLobby/Providers/.../Preset`.
3. Choose the session, lobby, matchmaking, and game allocator providers for your backend.
4. Set the allocator's `Game Scene` to your gameplay scene.
5. Make sure your game scene has a `NetworkManager`. Auto-start flags can stay on; PurrLobby suppresses them for its own load so the scene still works as a direct-play test environment.

[Scene setup](scene-setup.md) walks through both scenes properly.

## Customizing

Every screen is a PurrUI view prefab, and every colour comes from a shared `ColorPalette` asset. Restyling can be as small as swapping the palette or as deep as replacing view prefabs and adding your own screens.

See [Customizing the UI](customizing-the-ui.md), and read [Updating](./#updating) first.

## Writing your own backend

Implement the provider base classes in `Assets/PurrLobby/Runtime/Providers`: `SessionProvider`, `LobbyProvider`, `MatchmakingProvider`, `GameAllocatorProvider`.

The PurrNet provider under `Assets/PurrLobby/Providers/PurrNet` is the smallest complete example. Nakama is a fuller reference for relayed-match lobby state.

If all you want is a different network path, a custom `GameAllocatorProvider` swaps the transport while leaving the rest of PurrLobby untouched. See [Custom providers](custom-providers.md).
