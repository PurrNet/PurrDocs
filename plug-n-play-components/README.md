---
icon: unity
---

# Plug n' play components

Some networking tasks come up in almost every multiplayer game: syncing an object's position, keeping animations in sync, or replicating physics. Writing custom sync code for each of these every time would be tedious and error-prone. Plug n' play components let you skip that entirely.

Simply add a component to your object, and it automatically handles the networking for you. Normal examples of this are the [Network Transform](network-transform.md) allowing you to automatically network the position, rotation and scale of an object, or the [Network Animator](network-animator.md) allowing you to automatically sync animations.
