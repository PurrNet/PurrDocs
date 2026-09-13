# Simulating latency

In order to test locally and still have a realistic scenario, we need to be able to simulate latency and packet loss instead of relying on a perfect localhost connection.

### Built-in network simulation

The UDP Transport and Purr Transport have a `Network Simulation` section in their inspector:

* **Simulate Latency** - Holds every packet for a random duration between **Min Latency** and **Max Latency** (in milliseconds) before delivering it. Note that this delay applies in each direction, so a value of 100 results in a round trip time of roughly 200ms.
* **Simulate Packet Loss** - Drops a random percentage of packets, controlled by **Packet Loss Chance**. Reliable traffic is still delivered, since the loss is applied below the reliability layer and lost packets get retransmitted, just like on a real connection.
* **Include In Build** - By default the simulation only applies in the editor. Enable this to also apply it in builds.

{% hint style="warning" %}
The simulation is compiled out of release builds entirely. It works in the editor and in **Development builds**, or in any build when the `SIMULATE_NETWORK` scripting define is set. A release build silently runs with a clean connection even if **Include In Build** is enabled, so double check your build settings before concluding that a test passed under latency.
{% endhint %}

You can also drive it from code, for example to wire it to command line arguments for automated tests:

```csharp
udpTransport.networkSimulation = new NetworkSimulation
{
    includeInBuild = true,
    simulateLatency = true,
    minLatency = 40,
    maxLatency = 80,
    simulatePacketLoss = true,
    packetLossChance = 5
};
```

### Throttling the whole process

The built-in simulation only affects PurrNet traffic. For a true to world experience, including everything else your game does over the network, you can throttle the whole process (Unity or builds) or even the whole system. For this there are plenty of tools that can work, we personally use [Clumsy](https://github.com/jagt/clumsy).
