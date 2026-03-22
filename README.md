---
description: Made by network game developers with experience.
---

# 🐈 Introduction

PurrNet is a free, [performant](readme/performance/) and highly modular networking library for Unity, built by developers who have spent years working with multiplayer games. Our focus is on giving you the best possible developer experience: no clutter, no baking, no core features locked behind a paywall. Just a networking solution that works the way Unity works.

[See our official website here](https://purrnet.dev/)

Check out the [ROADMAP](readme/roadmap.md) to see what we are currently planning to add.

[**Get it on the Unity Asset Store**](https://assetstore.unity.com/packages/slug/297320)\
[**Get it on GitHub**](https://github.com/BlenMiner/PurrNet)

{% embed url="https://youtu.be/VxCPSFBTAms" %}

### Our mission

PurrNet is 100% free with no pro or premium version. You can use it to ship your game, and we ask nothing in return. No revenue sharing, no up-front cost, no strings attached.

We built PurrNet because we felt that existing networking solutions fight against Unity's natural workflow instead of embracing it. You shouldn't have to learn a completely different way of working just because your game is multiplayer. With PurrNet, if you know how to use Unity and C#, you already know most of what you need.

Read the [Unique to PurrNet](readme/unique-to-purrnet.md) section to see what sets us apart!

Make sure to join our Discord here: [https://discord.gg/NP9tP9Qx9R](https://discord.gg/NP9tP9Qx9R)

{% hint style="info" %}
**Building a multiplayer game as a studio?** We offer hands-on project support, migration planning, architecture reviews, and custom feature development. Everything is tailored to what your team actually needs. [Learn more about our studio support](readme/for-studios.md) or [get in touch directly](https://purrnet.dev/studios).
{% endhint %}

### What makes PurrNet different

There are a few things that we think really set PurrNet apart. Here's a quick taste, and you can find the full list on the [Unique to PurrNet](readme/unique-to-purrnet.md) page.

**Spawning and despawning just works.** You call `Instantiate()` and `Destroy()` the same way you already do in Unity, and PurrNet handles the networking for you. No special spawn calls, no extra steps.

**You control the authority model.** Our [Network Rules](systems-and-modules/network-manager/network-rules.md) system lets you configure who can do what (spawn, despawn, call RPCs, sync data) without changing your code. Start with full client authority for fast prototyping, then tighten it to server authority when you're ready. One setting, not a rewrite.

**No baking, no limitations.** We don't bake components or scene IDs. You can nest prefabs, rearrange your hierarchy, and work with version control without running into the edge cases that baking creates.

**RPC innovations.** We support [Generic](systems-and-modules/remote-procedure-call-rpc/generic-rpc.md), [Static](systems-and-modules/remote-procedure-call-rpc/static-rpc.md), and [Awaitable](systems-and-modules/remote-procedure-call-rpc/awaitable-rpc.md) RPCs, giving you more flexibility in how you structure your networked code.

## Everyone vs Server Auth?

Throughout PurrNet and this documentation, you'll see two terms come up a lot: [Server Auth](terminology/server-auth-safe.md) and [Everyone](terminology/client-auth-everyone-unsafe.md). These define who is allowed to perform a given action.

**Server Auth** means only the server can perform the action. This is the safe option for competitive games, since clients can't cheat by doing things they shouldn't be allowed to.

**Everyone** means all clients and the server can perform the action. This makes development faster and easier, but does mean clients have more control, which could allow cheating. This is a great fit for co-op games, friendly PvP, or during early development when you just want things to work.

The important thing is that you set this through [Network Rules](systems-and-modules/network-manager/network-rules.md), not in your game code. So you can switch between the two at any time without rewriting anything.

## Persistent user data

PurrNet has built-in support for persistent player data through its cookie system. If a player disconnects and later reconnects, the server still recognizes them as the same player and can restore their data. You can configure how persistent this is: scoped to the connection, the process, or the machine.
