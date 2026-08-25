---
icon: gamepad
---

# In the game scene

Once the game scene loads, PurrLobby steps back. It keeps ownership of exactly one thing: getting everyone back to the menu cleanly. Everything else is optional and opt-in.

## GameSession

`GameSession` is created automatically when the allocator loads your scene, so there is nothing to add. It handles three exits:

* The player chooses to leave.
* The server ends the game.
* The connection drops, in which case clients retry before giving up.

In all three it leaves the lobby, records why on the orchestrator, and loads the menu scene.

### Settings

Select the `GameSession` object in the loaded scene to adjust:

| Field | Default | Meaning |
| ----- | ------- | ------- |
| `Orchestrator` | auto | Optional override. Resolved from `GameOrchestrator.active` when empty. |
| `Menu Scene` | auto | Optional override. Captured from the launch scene when empty. |
| `Reconnect Timeout` | 15s | How long clients retry before returning to the menu. |
| `Reconnect Interval` | 2s | Delay between attempts. |

Reconnect applies to clients only. A listen host losing its own server has nothing to reconnect to.

### API

```csharp
// Leave voluntarily. Same path the pause menu uses.
GameSession.instance.LeaveToMenu();

// Per-scene lookup, preferred when you know the scene.
if (GameSession.TryGet(gameObject.scene, out var session))
{
    session.onReconnectingChanged += reconnecting => _spinner.SetActive(reconnecting);
    session.onExiting += reason => Debug.Log($"Leaving because {reason}");
}
```

`isReconnecting` and `isExiting` are readable at any time. `onExiting` fires once with a `GameExitReason`:

| Reason | Meaning |
| ------ | ------- |
| `None` | Normal launch, nothing to report |
| `LeftByChoice` | The player left |
| `GameOver` | The server ended the game |
| `ConnectionLost` | Disconnected and reconnects were exhausted |

The menu reads `GameOrchestrator.active.lastExitReason` after the return, so you can show a "connection lost" message on the main menu without passing state yourself.

## Pause menu

Drop the **SessionManager** prefab into your scene and you have a working pause menu. It carries a `ViewStack` for the in-game canvases and a `PushPauseMenu` component wired to it.

`PushPauseMenu` is the sole Escape owner in the game scene, and it is deliberately careful about it:

* Nothing on the stack: Escape opens `PauseMenuView`.
* Pause menu topmost: Escape closes it.
* Any other view topmost: the press is left alone, because that view's own back handler owns it.

The `BackInput` latch guarantees a single consumer per frame regardless of `Update` order, so your own views can use Escape without fighting the pause menu.

### Reacting to pause

Two UnityEvents sit on the `PushPauseMenu` component, `On Pause Opened` and `On Pause Closed`, which is the no-code route to freezing input or muting audio.

From script, `PauseMenuView` raises statics that fire no matter what pushed or popped the menu:

```csharp
private void OnEnable()
{
    PauseMenuView.onOpened += DisablePlayerInput;
    PauseMenuView.onClosed += EnablePlayerInput;
}

private void OnDisable()
{
    PauseMenuView.onOpened -= DisablePlayerInput;
    PauseMenuView.onClosed -= EnablePlayerInput;
}
```

The pause menu also manages the cursor through `CursorScope`, unlocking it while open and restoring your in-game state on close. When the scene is going away it abandons the saved state instead, so you land in the menu with a usable cursor.

## Ending the game

`GameOverBroadcaster` is a networked scene object included in the `SessionManager` prefab. From server code:

```csharp
if (GameOverBroadcaster.TryGet(gameObject.scene, out var broadcaster))
    broadcaster.EndGame();
```

Every player returns to the menu with `GameExitReason.GameOver`. Calling it on a client, or before it has spawned, logs an error and does nothing.

## Lobby players and voice

`LobbyPlayerSpawner` spawns a `PurrLobbyPlayer` per player, server-side, as players load the scene, and destroys it when they leave. Each `PurrLobbyPlayer` knows its `ILobby`, its `IPlayer`, and its row in the lobby UI.

When [PurrVoice](../purrvoice-voice-chat/) is installed, the `PURR_VOICE` define activates the integration on the same prefab:

* The local player's `PurrVoicePlayer` follows the lobby's microphone toggle, starting muted and unmuting when the player enables their mic.
* Remote players are unmuted on spawn so you hear them.
* With a `PurrLipSync` component assigned, phoneme changes are forwarded to the player's UI row, driving mouth movement on the avatar.

Without PurrVoice installed the whole integration compiles out, so there is no cost and nothing to configure.
