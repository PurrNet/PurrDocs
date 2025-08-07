# Client Spawning Validation

Client side spawning is great for instant feedback on the client's end but if you want to avoid abuse in a more controlled fashion we allow you to register validators. These validators only run on the server and allow you to deny spawns entirely. If a spawn is denied the original client will despawn it on their end.

One thing to keep in mind is that `SpawnPacket` in `PurrNet` can be partial because every identity is independent. To ensure that a spawn is as simple as a prefab spawn we provide a `TryGetRawPrefab` method that validates it is a simple spawn and also gives you the prefab used.

Here is a bare bones example of validating all normal (not partial) prefab spawns:

```csharp
using PurrNet;
using PurrNet.Logging;
using PurrNet.Modules;
using UnityEngine;

public class TestValidator : MonoBehaviour
{
    [SerializeField] private NetworkManager _networkManager;

    private void Awake()
    {
        _networkManager.onClientSpawnValidate += ValidateSpawn;
    }

    private void OnDestroy()
    {
        if (_networkManager != null)
            _networkManager.onClientSpawnValidate -= ValidateSpawn;
    }

    private bool ValidateSpawn(PlayerID player, SpawnPacket data)
    {
        // TryGetRawPrefab outputs the prefab used to trigger this spawn packet
        // it also validates that it is complete (we have partial spawns in PurrNet)
        if (data.TryGetRawPrefab(_networkManager, out var prefab))
        {
            PurrLogger.Log($"Spawn validated for player {player}: {data.prototype}\nPrefab: {prefab.name}");
            return true;
        }

        return false;
    }
}
```
