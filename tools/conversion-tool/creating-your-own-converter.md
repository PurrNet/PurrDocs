# Creating your own converter

The Conversion Tool uses a recipe system that makes it straightforward to add support for other networking libraries. Each converter lives in its own folder and consists of three classes.

## 1. Mappings

Extend `NetworkSystemMappings` and define your mappings in the constructor:

```csharp
using PurrNet.ConversionTool;

public class MySystemMappings : NetworkSystemMappings
{
    public MySystemMappings()
    {
        SystemName = "MyNetworkLib";
        SystemIdentifiers = new List<string> { "MyNetworkLib" };

        NamespaceMappings = new Dictionary<string, string>
        {
            { "MyNetworkLib", "PurrNet" }
        };

        TypeMappings = new Dictionary<string, string>
        {
            { "NetConnection", "PlayerID" },
            { "NetObject", "NetworkIdentity" }
        };

        PropertyMappings = new Dictionary<string, string>
        {
            { "IsOwner", "isOwner" },
            { "IsServer", "isServer" }
        };

        MethodMappings = new Dictionary<string, string>
        {
            { "OnClientStart", "OnSpawned" }
        };

        // ... and so on for MemberMappings, MethodCallMappings,
        // AttributeMappings, AttributeParameterMappings, etc.
    }
}
```

For cases that don't fit into simple mappings, override `SpecialCaseHandler` to do custom Roslyn syntax tree transformations.

## 2. Prefab handling

Extend `NetworkPrefabHandling` and override `ConvertPrefab`:

```csharp
using PurrNet.ConversionTool;
using UnityEngine;

public class MySystemPrefabHandling : NetworkPrefabHandling
{
    public override bool ConvertPrefab(GameObject prefab)
    {
        // Find old components, add PurrNet equivalents, copy settings
        // Return true if anything was changed
        return false;
    }
}
```

## 3. Scene handling

Extend `NetworkSceneHandling` and override `ConvertSceneObject`:

```csharp
using PurrNet.ConversionTool;
using UnityEngine;

public class MySystemSceneHandling : NetworkSceneHandling
{
    public override bool ConvertSceneObject(GameObject sceneObject)
    {
        // Convert scene-specific objects like NetworkManager
        // Return true if anything was changed
        return false;
    }
}
```

## Assembly setup

Your converter folder needs its own `.asmdef` that references `PurrNet.ConversionTool` and the source networking library. Set it to **Editor only** and add a `defineConstraints` entry so it only compiles when the source library is present.

Take a look at the [FishNet converter on GitHub](https://github.com/PurrNet/PurrNet-Conversion-Tool/tree/dev/Assets/PurrNet-Conversion/Converters/FishNet%20Converter) for a full working example.

## Discovery

The tool automatically discovers converters through reflection. As long as your three classes are in the same folder and extend the right base classes, they will show up in the dropdown. No registration needed.
