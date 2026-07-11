# Channels

Channels define the delivery guarantees PurrNet requests from the active transport. Reliable channels retry lost data and therefore cost more bandwidth and can add latency. Unreliable channels do not retry, which makes them a good fit for frequently refreshed data where a newer value makes an older one irrelevant.

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

## Messages larger than the MTU

A transport can only place a limited amount of data in one packet. This limit is its **maximum transmission unit (MTU)**. The Network Manager's **MTU Exceeded Behaviour** setting decides what PurrNet does when a message sent on `Unreliable` or `UnreliableSequenced` is too large:

* **Fragment** (default) splits the message into MTU-sized unreliable packets. The receiver delivers the message only after every fragment arrives. Fragments are not retransmitted, so losing any fragment drops the whole message.
* **Upgrade To Reliable** changes that message to `ReliableOrdered`, allowing reliable fragmentation and delivery. PurrNet logs a warning when this happens.
* **Drop** discards the message and logs an error, preserving strictly unreliable behavior without the extra fragment traffic.

`Fragment` does not make an unreliable message reliable. Larger messages need more fragments and therefore have a greater chance that at least one fragment is lost. Prefer keeping frequently sent unreliable messages below the MTU when practical.

`UnreliableSequenced` keeps its sequencing behavior when fragmentation is enabled. A newer message on the same stream supersedes an older incomplete fragmented message, and an older message cannot overwrite a newer completed one.

See [Network Manager](../systems-and-modules/network-manager/README.md#unreliable-messages-and-the-mtu) for configuration guidance.
