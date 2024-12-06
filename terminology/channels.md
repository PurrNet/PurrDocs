# Channels

Channels are for the developer to decide how the data is sent. Typically the reliable channels are the most commonly used, but comes at a higher data cost. So if you are sending something all the time and you don't necessarily care if you lose a packet or two, the **Unreliable** channel might be a good option for you.

```csharp
public enum Channel : byte
{
    /// <summary>
    /// It ensures that the data is received but the order is not guaranteed.
    /// </summary>
    ReliableUnordered,
    
    /// <summary>
    /// Packets are guaranteed to be in order but not guaranteed to be received.
    /// </summary>
    UnreliableSequenced,
    
    /// <summary>
    /// Packets are guaranteed to be received in order. -- Default channel
    /// </summary>
    ReliableOrdered,
    
    /// <summary>
    /// Only the last sent packet is guaranteed to be received.
    /// </summary>
    ReliableSequenced,
    
    /// <summary>
    /// Packets are not guaranteed to be received nor in order.
    /// </summary>
    Unreliable
}
```
