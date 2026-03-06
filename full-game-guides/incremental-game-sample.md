---
description: PurrNet networking sample for an incremental game
---

# 📈 Incremental Game Sample

### Introduction

This sample includes a basic gameplay loop + simple polish (audio & visual) of a multiplayer incremental game where you collect resources, bring them to your base, upgrade and work towards an end goal that can be purchased in the shop. You utilize your dashing to move through resources to break them and gather their sweet sweet loot!

The setup is built so it's easy to scale on the core elements such as:

* Upgrades that are based on scriptables.&#x20;
* Player which is ran by state machine logic
* Resource spawning which is fully modular and layer based.

**Used in this sample:**

* Kenney.nl for visual assets
* Cinemachine
* New Unity input system
* PurrNet

{% embed url="https://youtu.be/aX1gUZgG3Ig" %}

### Install instructions

1. Clone or download the [Github repository](https://github.com/PurrNet/PurrNet-Incremental-Sample) ([https://github.com/PurrNet/PurrNet-Incremental-Sample](https://github.com/PurrNet/PurrNet-Incremental-Sample))
2. Open the project in Unity
3. Open `Scenes/GameScene`
4. Press Play. You should see your player spawn, be able to run around, chop trees, suck up loot and dump it at your base



### Written guide

When you first spin up the project, you'll see a familiar structure of folders, the most relevant for now being:\
\- Art\
\- Prefabs\
\- Scripts

But first, we gotta go into the Scenes folder and open the `GameScene`. Given everything is correct off the bat, you should be able to just press play, and the project should work. You should see your player spawn, be able to run around, chop trees, suck them up and put them in your base to purchase upgrades.

You should also be able to spin up a clone and test with that as well!

#### Network Manager

The [Network Manager](../systems-and-modules/network-manager/) is setup in the most basic way, currently utilizing the [UDP transport](../systems-and-modules/transports/udp-transport.md). This is something you'd want to change later in case you want to setup with Steam or even a dedicated server (Read the guide here).

The [Network Prefabs](../systems-and-modules/network-manager/network-prefabs.md) are already setup and located in your Prefabs folder, and the [NetworkAssets](../systems-and-modules/network-manager/network-assets.md) are already setup to support the audio of the game and are located in the root Assets folder.

The [Network Rules](../systems-and-modules/network-manager/network-rules.md) are also setup as the unsafe rules, as this is generally not the type of game where we care about cheating, and it makes the workflow overall simpler.

#### Player setup

The player is spawned using the PurrNet `PlayerSpawner` which is located on the "PlayerSpawner" game object in the scene.

If you open the Player prefab, you'll see that the hierarchy holds the root player object, a "Visuals" object which holds a "Bodies" object which holds the actual character meshes. The reason for this is that the animations run on the "Visuals", whilst the code animations (squishing the player when dashing and sucking) are running on the "Bodies" transform.&#x20;

We also have the Camera object which holds the Cinemachine camera logic including the `PlayerCamera` component. This component is responsible for the dynamic camera and zooming effects when adding wood to the base.

Going back to the root of the Player prefab, you'll notice that the player is running on the [Networked State Machine](../plug-n-play-components/state-machine-auto-networked.md) of PurrNet. This makes it very easy for us to network which state the player is currently in. It runs like any finite state machine, and depending on Input will switch between the states (this is defined in the individual states)

The players states are currently:\
\- Player Movement\
\- Player Dash\
\- Player Suck

Before we delve into these states, it's important to mention that the player also consists of a [Network Transform](../plug-n-play-components/network-transform.md) which is [owner](../systems-and-modules/network-identity/ownership.md) authed, a normal Rigidbody, Box Collider and Audio source.

#### Player States

**Player Movement**

The Player movement is a very basic take on some simple Rigidbody movement. It's only 100 lines of code, and that's including the state machine handling as well.&#x20;

**Player Dash**

The Player Dash will handle dashing in a given direction. As soon as it enters it's state it'll start tracking the amount of time it has dashed. It'll then set it's collider to a trigger and continue to dash in a given direction. By the end of it's dash, it'll ensure that it's not inside any collider, and if it is, move it out. Afterwards, it'll re-enable it's own collider and exit the state back to the movement state.

**Player Suck**

The sucking state will attempt to suck up any Loot found in a given cone in front of it. First it'll take ownership of said loot. Then it'll utilize physics forces on the given loot and apply those to go back towards the player, and once close enough it'll be added to the players inventory and disabled from the world (not yet despawned/destroyed)

#### Base

The base is the core of most of the gameplay loop. It holds and syncs the wood count which is also what's displayed in the top right corner of the screen. It'll "suck out" loot from the players inventory and into the base, handling some visual effects on the fly. As soon as that loot hits the base, it'll properly despawn/destroy the loot.&#x20;

#### Resources (BaseMineable, IMineable)

Resources, in this case the trees, are setup to have x-amount of health. This is synced, and whoever hits the resource, takes ownership and changes the health sync immediately. This means that everything feels nice and responsive for the player hitting the resource, regardless of host or not.&#x20;

When they despawn, they will spawn some amount of loot, depending on the players upgrades and what's defined in editor.

#### Resource Spawning

Resource spawning is handled by the `ResourceSpawner` component found on the GameManager in the scene. This is a layer based system, which would allow you to spawn any types of resources with various conditions and rarities.

It is spawned utilizing noise, and limiting the amount of resources to spawn per frame in order to avoid stuttering individual frames. This component also tracks all the players on the server, so when players move, resources will spawn around them, giving the idea of an infinite world. And of course it also cleans up behind itself, so when there are no players in a given radius of a resource, it'll despawn.&#x20;

Lastly, the component also handles respawning of resources after a set interval of being harvested.&#x20;

#### Loot

This is currently seen as the currency, and just registered as wood. I went a simple route on this, to keep things linear, but it'd be easy to expand on if needed be. These are dropped of resources being harvested through dashing.

#### Object Pooling

This setup utilizes PurrNet [object pooling](../systems-and-modules/network-identity/pooling.md) which requires nearly no customization what so ever. This is enabled in the Network Prefabs scriptable. This is utilized for the trees in the world and the loot.

#### Upgrades

The upgrades are the core of most gameplay mechanics. There is currently built in upgrades for:\
\- Movement speed\
\- Damage\
\- Dash distance\
\- Inventory space\
\- Wood per tree\
\- Win the game

These are utilized to populate the shop through the `PlayerUpgrades` component found on the GameManager in the scene.

#### Network Audio

Some of the audio are modularized to be called over the network using the `NetworkAudioHelper`. These are for example the wood hitting audio. As a tree is cut down, we'd still want to hear the audio, so it's instead handled using pooling by the `NetworkAudioHelper`.

#### UI Handling

The UI of this setup is extremely bare bones. There is a "HUDView" and a "ShopView", which are both found nested under the Canvas in the scene. They handle exactly what their name indicates, and various scripts communicate with them utilizing the [InstanceHandler](../systems-and-modules/instance-handler.md) of PurrNet.
