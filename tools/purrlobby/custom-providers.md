---
icon: code
---

# Custom providers

Every backend in PurrLobby is a ScriptableObject implementing one of four base classes. Writing your own is the supported way to target a service we do not ship, or to swap the transport without touching the UI.

The PurrNet provider under `Assets/PurrLobby/Providers/PurrNet` is the smallest complete example. Nakama is a fuller reference for relayed-match lobby state.

## The four roles

```csharp
public abstract class SessionProvider : ScriptableObject
{
    public abstract bool isLoggedIn { get; }
    public abstract string playerId { get; }
    public abstract string playerName { get; }
    public abstract Task Login(ViewStack stack);
    public abstract Task Logout();
}
```

`Login` receives the menu's `ViewStack`, so a provider needing credentials can push its own view and await the result. That is how the shipped device-login screen appears.

```csharp
public abstract class LobbyProvider : ScriptableObject
{
    public abstract int maxPlayer { get; }
    public virtual LobbyCapabilities capabilities => LobbyCapabilities.All;
    public virtual Task Initialize() => Task.CompletedTask;
    public abstract Task<LobbyResponse> CreateLobby(LobbySettings settings);
    public abstract Task<LobbyResponse> JoinLobby(string lobbyId);
    public abstract Task<LobbyResponse> JoinLobbyByCode(string code);
    public abstract Task<LobbyResponse> JoinRandom(LobbyQuery query = null);
    public abstract Task<LobbyCollectionResponse> QueryLobbies(LobbyQuery query = null);
}
```

Override `capabilities` to restrict what your backend supports and the UI hides the rest. `MatchmakingProvider` and `GameAllocatorProvider` round out the set.

Lobby objects returned from these methods implement the contracts in `Assets/PurrLobby/Runtime/Contracts`, chiefly `ILobby`, `IPlayer`, `IMetadata` and `ILobbyChat`. Deriving from `LobbyBase<TPlayer>`, `LobbyPlayerBase`, `LobbyMetadataBase` and `LobbyChatBase` gives you most of the bookkeeping for free.

Add `[CreateAssetMenu]` so your provider can be created as an asset, and `[ProviderDependency("com.some.package", "Display Name")]` if it needs an SDK, which produces a helpful inspector warning instead of a compile error.

## Custom game allocation

`GameAllocatorProvider` is where the transport decision lives, and it is the class to write if you want everything else PurrLobby offers but a different network path.

Three members matter:

```csharp
// Produce the connection info the lobby will share with every member.
public abstract Task<GameStartResponse> AllocateGame(ILobby lobby);

// Load the game scene. Call the provided helper rather than loading yourself.
public abstract Task LoadGame(ILobby lobby);

// Add and configure the transport. Return false to abort the connection.
protected abstract bool ConfigureTransport(NetworkManager manager, ConnectionInfo connection, bool asHost);
```

`ConnectionInfo` is deliberately transport-agnostic:

| Field | Meaning |
| ----- | ------- |
| `serverAddress` | Host address for direct transports, room or match id for relayed ones, SteamID for Steam |
| `serverPort` | Port, where the transport uses one |
| `connectionToken` | Opaque token, for example a matchmaker ticket. Transports may ignore it |
| `hostId` | Player id of the hosting player, when a player hosts |

Two hooks are worth knowing:

* `supportsHosting`, returning false for dedicated-server backends. `Connect` then always starts a client, downgrading a host request rather than failing.
* `LoadGameScene(sceneName)`, which records the menu scene, suppresses auto-start for the load, creates the `GameSession`, and awaits completion. Always use it instead of `SceneManager.LoadSceneAsync`.

### Example: direct UDP instead of PurrTransport

A player-hosted allocator using a direct UDP connection rather than the PurrTransport relay:

```csharp
using System.Threading.Tasks;
using PurrNet.Transports;
using UnityEngine;

namespace MyGame.Lobby
{
    [CreateAssetMenu(menuName = "MyGame/Lobby/UDP Game Allocator")]
    public class UdpGameAllocator : GameAllocatorProvider
    {
        [SerializeField, PurrScene] private string _gameScene;
        [SerializeField] private ushort _port = 5000;

        public override Task<GameStartResponse> AllocateGame(ILobby lobby)
        {
            // Whatever your backend knows about how to reach the host.
            var address = lobby.lobbyData.GetData("host_ip");

            if (string.IsNullOrEmpty(address))
                return Task.FromResult(GameStartResponse.Failure("The lobby has no host address."));

            return Task.FromResult(GameStartResponse.Success(new ConnectionInfo
            {
                serverAddress = address,
                serverPort = _port,
                hostId = lobby.owner?.id,
            }));
        }

        public override Task LoadGame(ILobby lobby) => LoadGameScene(_gameScene);

        protected override bool ConfigureTransport(NetworkManager manager, ConnectionInfo connection, bool asHost)
        {
            var transport = manager.transport as UDPTransport
                            ?? GetOrAddComponent<UDPTransport>(manager.gameObject);

            manager.transport = transport;
            transport.address = connection.serverAddress;
            transport.serverPort = connection.serverPort;
            return true;
        }
    }
}
```

Assign it to your orchestrator's `Game Allocator` slot and the rest of PurrLobby is unchanged. The menu, lobby, chat, ready-up, scene handoff, pause menu and return-to-menu all work exactly as before.

The shipped allocators are the same shape. `PurrTransportGameAllocator` adds a `PurrTransport` and sets `roomName` to the lobby id. `SteamGameAllocator` adds a `SteamTransport` aimed at the host's SteamID. Neither is more privileged than yours.

{% hint style="info" %}
`GetOrAddComponent<T>` is provided by the base class. Reusing an existing transport component when one is already present, as above, avoids stacking duplicates when a player returns to the menu and launches again.
{% endhint %}

### Dedicated servers

For a backend that deploys servers rather than having a player host:

```csharp
protected override bool supportsHosting => false;
```

`AllocateGame` then returns the deployed server's address, and every player connects as a client. `EdgegapGameAllocator` works this way.

## Testing

`Assets/PurrLobby/Tests` contains editor tests for the abstract layer, including `LobbyBaseTests`, `LobbyMetadataBaseTests` and `MatchmakingProviderTests`, plus the test doubles they run against. Those doubles are a compact reference for the minimum a provider has to implement.
