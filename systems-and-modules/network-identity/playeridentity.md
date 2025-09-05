# PlayerIdentity

The player identity allows you to very easily have access to your players based on ownership. The PlayerIdentity inherits from NetworkIdentity, so you still have all networking capabilities available, it's simply a layer on top to make your life easier!

You can easily utilize any of these:

**bool TryGetLocal(out T player)** -> Will output the local player\
**bool TryGetPlayer(PlayerID playerId, out T player)** -> Will output the player with the given playerId\
**allPlayers** ->  returns a readonly dictionary of all the registered players

Simple example usage:

```csharp
public class PlayerMovement : PlayerIdentity<PlayerMovement>
{
    public void Teleport(Vector3 destination) {
        //Do whatever to teleport here
    }
}

public class GameManager : NetworkIdentity 
{
    [SerializeField] private Transform teleportDestination;
    
    public void TeleportPlayer(PlayerID targetPlayer) 
    {
        //Returns true if it finds the player
        if(!PlayerMovement.TryGetPlayer(targetPlayer, out var player)
            return;
            
        player.Teleport(teleportDestination.position);
    }

    public void TeleportLocalPlayer() 
    {
        //Returns true if it finds the local player
        if(!PlayerMovement.TryGetLocal(out var player)
            return;
            
        player.Teleport(teleportDestination.position);
    }
    
    public void TeleportAllPlayers() 
    {
        //allPlayers gives you a dictionary of all the players registered
        foreach(var player in PlayerMovement.allPlayers) {
            player.value.Teleport(teleportDestination.position);
        }
    }
}
```
