---
icon: users
---

# PurrLobby (Lobbies & matchmaking)

**PurrLobby** is a drop-in lobby and matchmaking front-end for PurrNet. It gives you the multiplayer menu flow most games need: create a lobby, share a code, browse open lobbies, chat, ready up, matchmake, and move everyone into a game scene together.

The UI is already built. You choose a backend, assign your game scene, and customize the prefabs as needed.

{% hint style="info" %}
PurrLobby replaces the older [Lobby System](../../addons/lobby-system.md) addon. New projects should use PurrLobby.
{% endhint %}

## Requirements

* Unity **2022.3** or newer.
* [PurrNet](https://github.com/PurrNet/PurrNet) and [PurrUI](https://github.com/PurrNet/PurrUI).
* PurrServices when using the PurrNet Services lobby provider.
* Optional, depending on backend:
  * [Steamworks.NET](https://github.com/rlabrecque/Steamworks.NET) for the Steam providers.
  * [Nakama Unity](https://github.com/heroiclabs/nakama-unity) for the Nakama providers.

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

A backend is four ScriptableObjects assigned to a `GameOrchestrator`: session, lobby, matchmaking and game allocation. PurrLobby ships working sets for **PurrNet Services**, **Steam** and **Nakama**, each with preset assets and a pre-filled orchestrator.

The slots are independent, so you can take the best part of each. Providers also advertise their optional lobby actions through `LobbyCapabilities`, and the menu hides unsupported buttons automatically, so a backend without lobby browsing simply shows no browser entry point.

{% hint style="warning" %}
`NakamaGameAllocator` runs gameplay over a WebSocket relay, which suits turn-based games rather than fast-paced ones. A common setup is Nakama for session, lobby and matchmaking with `PurrTransportGameAllocator` or `SteamGameAllocator` for the match itself.
{% endhint %}

Full settings for every backend are on the [Providers](providers.md) page.

## Getting started

Open `Assets/PurrLobby/Scenes/MenuScene.unity`, add it and `GameScene.unity` to Build Settings, and press play. From there, [Getting started](getting-started.md) walks through pointing it at your own game scene.

## Customizing

Every screen is a PurrUI view prefab, and every colour comes from a shared `ColorPalette` asset. Restyling can be as small as swapping the palette or as deep as replacing view prefabs and adding your own screens.

See [Customizing the UI](customizing-the-ui.md), and read [Updating](./#updating) first.

## Writing your own backend

Implement the provider base classes in `Assets/PurrLobby/Runtime/Providers`: `SessionProvider`, `LobbyProvider`, `MatchmakingProvider`, `GameAllocatorProvider`.

The PurrNet provider under `Assets/PurrLobby/Providers/PurrNet` is the smallest complete example. Nakama is a fuller reference for relayed-match lobby state.

If all you want is a different network path, a custom `GameAllocatorProvider` swaps the transport while leaving the rest of PurrLobby untouched. See [Custom providers](custom-providers.md).
