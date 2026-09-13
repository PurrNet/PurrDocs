---
icon: palette
---

# Customizing the UI

Every screen in PurrLobby is a [PurrUI](https://github.com/PurrNet/PurrUI) view prefab pushed onto a `ViewStack`. There is no bespoke menu framework to learn: if you know PurrUI, you already know how to bend PurrLobby's UI.

{% hint style="warning" %}
Before you edit any shipped prefab, read [Updating](./#updating). Duplicating what you plan to change into your own folder keeps your work safe from package updates.
{% endhint %}

## Views and the stack

Each screen derives from PurrUI's `MonoView`:

| View | Base | Role |
| ---- | ---- | ---- |
| `MainMenuView` | `MonoView` | Entry point |
| `LobbyView` | `MonoView` | Player list, chat, ready-up, start |
| `MatchmakingView` | `MonoView` | Ticket progress |
| `LoadingView` | `MonoView` | Blocking spinner with a label |
| `PauseMenuView` | `MonoView` | In-game pause |
| `CreateLobbyView` | `SlidePageView` | Lobby creation form |
| `JoinWithCodeView` | `SlidePageView` | Code entry |
| `LobbyBrowserView` | `SlidePageView` | Lobby list |

`SlidePageView` is a `MonoView` that slides in as a page rather than appearing as an overlay.

The `ViewStack` component on the `LobbyManager` prefab owns them. Its fields:

* **Prefab Collections**, a list of `ViewCollection` assets. PurrLobby's is `PurrLobby.Generic` in `Assets/PurrLobby/Prefabs/Views`.
* **Color Palette**, the `ColorPalette` asset driving theming.
* **Push On Start**, an optional view pushed automatically on start.
* **Order Offset**, the sorting order base for this stack.

## Driving the stack

Views are pushed by type, and the stack resolves the prefab from its collections:

```csharp
// Push a new view.
parentStack.Push<LobbyBrowserView>().Setup(_orchestrator);

// Swap this view for another, or push if this one is no longer on the stack.
parentStack.ReplaceOrPush<LobbyView>(this).Setup(lobby, _orchestrator);

// Close the view that called this.
PopMe();

// Inspect the stack.
var top = _stack.top;
var existing = _stack.GetFirstView<LobbyView>();
```

`ReplaceOrPush<T>(this)` is the pattern used throughout PurrLobby. The generic argument is the view being **created**; the argument is the view being **replaced**. They are usually different types, which is exactly the point when moving from a create or browse screen into the lobby.

If a type is not registered in any collection, the stack logs `No window prefab of type X found in WindowPrefabs` and does nothing.

Views expose two lifecycle hooks worth overriding:

```csharp
public override void OnPushed() { /* view became visible */ }
public override void OnPopped() { /* view is going away */ }
```

`PauseMenuView` uses these to push and pop a `CursorScope`, which is how the cursor unlocks for the menu and returns to its in-game state afterwards.

## Theming

The `ColorPalette` asset is a single source of colour for every view. It defines nine roles, each with a matching contrast colour where it makes sense:

`Black`, `White`, `Muted`, `Background`, `Surface`, `Accent`, `Success`, `Warning`, `Danger`.

Read and write them at runtime:

```csharp
palette.SetColor(ColorType.Accent, myBrandColor);
var accent = palette.GetColor(ColorType.Accent);
var onAccent = palette.GetContrast(ColorType.Accent);
```

The palette raises `onChange` when anything is set, and the stack repaints. Editing the asset in the inspector during play mode updates the UI live, which makes it easy to dial in a theme.

The fastest way to rebrand PurrLobby is to duplicate the palette asset, change the colours, and assign it to your `ViewStack`. No prefab edits required.

## Replacing a screen

To change how a built-in screen looks without touching its logic, duplicate its prefab from `Assets/PurrLobby/Prefabs/Views`, restyle the copy, and point your `ViewCollection` at it. The type stays the same, so `Push<LobbyView>()` picks up your version.

The smaller building blocks (player rows, chat entries, toasts, context menu items) live in `Assets/PurrLobby/Prefabs/Elements`.

## Adding your own screen

1. Write a class deriving from `MonoView` (or `SlidePageView` for a page).
2. Build a prefab for it.
3. Add the prefab to a `ViewCollection` on your `ViewStack`. Your own collection alongside PurrLobby's works fine.
4. Push it: `parentStack.Push<MySettingsView>()`.

Your view now takes part in the same back-button handling, cursor scoping and theming as the built-in ones.

## Back and pause input

`BackInput` centralizes the Escape and Back keys with a per-frame latch, so exactly one handler consumes a press:

```csharp
if (BackInput.WasBackPressed() && BackInput.TryConsume())
    PopMe();
```

In the game scene, `PushPauseMenu` is the sole Escape owner. See [In the game scene](in-game.md#pause-menu).
