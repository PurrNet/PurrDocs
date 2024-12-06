# Ownership

Ownership is relating to the relation between a [player ](../playerid-client-connection.md)and a [Network Identity](./). Upon spawning a new Network Identity, some level of ownership exists, whether that'd be no owner meaning the server has full control, or a [player/client](../playerid-client-connection.md) can be the owner.

Being the owner of a [Network Identity](./), gives you different levels of authorization, depending on the [Network Rules](../network-manager/network-rules.md) of the [network manager](../network-manager/). Keep in mind that being the owner of a network identity, doesn't mean you are directly the owner of every identity on the object. This is a setting that is specific per [network identity](./).

You can easily change ownership on your Network Identity by calling the `GiveOwnership` method, and feeding it the [PlayerID](../playerid-client-connection.md) to be the new owner

```csharp
networkIdentity.GiveOwnership(newOwnerPlayerId);
```

You can easily check for ownership by simply getting the owner. Be mindful that the owner and often other PlayerID references are nullable, meaning you should check whether they have a value or not.

```csharp
if(owner.HasValue)
{
    //This object has an owner
    Debug.Log($"Owner of the object is: {owner.Value}");
}
else
{
    //This object has no owner
}

//You  can also easily check if you are the owner
if(isOwner) 
{
    //If we are here, we are the owner of the object
}
```

An easy way to check whether you have "control" of the object, meaning one of the two statements below is true:\
\- There is an owner and I am the owner\
\- There is no owner and I am the server\
Is to use the `IsController` bool.

```csharp
if(IsController)
{
    //We are either the owner, or there is no owner and we're the server
    
}
else
{
    //We are not in control of this object
}
```
