---
description: Automate converting your project from other networking libraries to PurrNet.
icon: wand-magic-sparkles
---

# Conversion Tool

Switching networking solutions mid-project is painful. The PurrNet Conversion Tool takes care of the heavy lifting for you, converting your code, prefabs, and scenes from other networking systems to PurrNet.

It works through a recipe system where each source networking library has its own set of mappings and conversion logic. Right now, **FishNet** is supported out of the box, with more converters coming.

## What it does

The tool handles three layers of conversion:

**Code** uses Roslyn to parse your C# files and apply type, namespace, method, property, and attribute mappings. It also handles trickier cases like merging FishNet's separate `OnStartServer`/`OnStartClient` into PurrNet's unified `OnSpawned(bool asServer)`.

**Prefabs** finds networking components on your prefabs (like `NetworkTransform`, `NetworkAnimator`, `NetworkObject`) and swaps them out for the PurrNet equivalents, copying over relevant settings.

**Scenes** does the same for scene objects, including full `NetworkManager` conversion with transport setup and tick rate transfer.

Each step runs independently so you stay in control of the process.

## Installation

### Via PurrNet Package Manager

The easiest way. If you already have PurrNet installed, open the **PurrNet Package Manager** in Unity and install the Conversion Tool with a single click.

### Via git URL

You can also add the package through the Unity Package Manager using this git URL:

```
https://github.com/PurrNet/PurrNet-Conversion-Tool.git?path=/Assets/PurrNet-Conversion
```

Or add it directly to your `Packages/manifest.json`:

```json
"dev.purrnet.conversion": "https://github.com/PurrNet/PurrNet-Conversion-Tool.git?path=/Assets/PurrNet-Conversion"
```

### Requirements

You need [PurrNet](https://github.com/PurrNet/PurrNet) installed in your project. The source networking library you're converting from (e.g. FishNet) also needs to be present, since the tool reads its components during prefab and scene conversion.

## How to use it

Open the tool from **Tools > PurrNet > Conversion Tool**.

1. Select the networking system you're converting from in the dropdown
2. Set which folders to include for scripts, prefabs, and scenes (defaults to the entire Assets folder)
3. Run each conversion step in order: **Convert Code**, then **Convert Prefabs**, then **Convert Scenes**

The conversion log at the bottom of the window shows you what changed. A `ConversionChangelog.txt` file is also written to your project root with timestamped details of every modification.

After conversion, you can remove the old networking library from your project.

{% hint style="warning" %}
Make sure your project is backed up prior to beginning any conversion!
{% endhint %}

{% hint style="info" %}
The tool also supports creating custom converters for other networking libraries. See [Creating your own converter](creating-your-own-converter.md) for more details.
{% endhint %}
