---
icon: flag-checkered
---

# Getting started

The fastest way to see PurrLobby working is to open the shipped scenes and press play. Once that runs, point it at your own game scene.

## Run the sample

1. Open `Assets/PurrLobby/Scenes/MenuScene.unity`.
2. Add both `MenuScene` and `GameScene` to **File → Build Settings**.
3. Press play.

You get the main menu, and from there you can create a lobby, ready up, and launch into `GameScene`. The scene ships wired to a working backend, so nothing needs configuring to try it.

To test with a second player, use PurrNet's [multiplayer play mode or a build](../../guides/test-with-friends/). Two clients in the same lobby is enough to see the ready-up, chat and start flow.

## Point it at your game

1. Select the **LobbyManager** in your menu scene and assign a `GameOrchestrator`. Preset orchestrators live under `Assets/PurrLobby/Providers/<backend>/Preset`, so dragging `Orchestrator.PurrNet` or `Orchestrator.Steam` in fills every slot at once.
2. Open the orchestrator's **Game Allocator** asset and set its `Game Scene` to your gameplay scene.
3. Make sure that scene has a `NetworkManager`.
4. Add your scene to Build Settings.

That is the whole loop. The allocator adds the transport, starts the host or client, and creates the `GameSession` that brings everyone back to the menu afterwards.

{% hint style="success" %}
Auto-start flags on your `NetworkManager` can stay on. PurrLobby suppresses them for the duration of its own scene load, so your game scene still works as a direct-play testing environment while the lobby owns the launch.
{% endhint %}

## Add the pause menu

Drop the **SessionManager** prefab from `Assets/PurrLobby/Prefabs/Setup` into your game scene. It carries an in-game `ViewStack`, the Escape handling, and a `GameOverBroadcaster` so the server can end the match for everyone.

## Where to go next

| If you want to | Read |
| -------------- | ---- |
| Understand the moving parts | [How it works](how-it-works.md) |
| Set up scenes properly | [Scene setup](scene-setup.md) |
| Pick or configure a backend | [Providers](providers.md) |
| Restyle the menus | [Customizing the UI](customizing-the-ui.md) |
| Handle pause, game over, reconnect | [In the game scene](in-game.md) |
| Target a backend we do not ship | [Custom providers](custom-providers.md) |
