---
icon: user-police
---

# Network Ownership Toggle

This component allows you to enable/activate components/gameobjects based on [ownership](../systems-and-modules/network-identity/ownership.md).\
Notice that there is a toggle on the far right of every entry, this toggle reads as `Activate if I'm the owner, otherwise deactivate`.

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

In the example above, `PlayerMovement` would be enabled **only** for the owner and disabled for others while the `Text` GameObject would be disabled for the owner but active for others.

This is quite handy to quickly setup a networked prefab without having to write too much code.
