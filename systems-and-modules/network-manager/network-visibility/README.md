# Network Visibility

Overall this system deciphers whether the server should send data from a Network Identity to a specific client. The [Network Manager ](../)takes in a network visibility rule-set, and this rule-set consists of rules/conditions.

In the [Network Rules ](../network-rules.md)you can modify the behavior of this as to whether the object is destroyed, de-spawned or simply stays without receiving or sending data.

A simple example of this is the [Distance Condition](distance-condition.md). If the distance is set to 50 units, if the player is further away from an object than 50 units, they won't receive any data. This is great for performance, especially in larger environments and games, so we don't have to send unnecessary data.

The network visibility rules/conditions, can be added as global conditions, but they can also be added on a per [Network Identity ](../../network-identity/)basis, by adding a custom rule-set as an override.

You can also custom make your own condition for cases where the pre-made conditions aren't fitting. All you need to do is inherit from **INetworkVisibilityRule** and implement the **HasVisibility** rule, and simply return true or false depending on whether it should send data or not.&#x20;
