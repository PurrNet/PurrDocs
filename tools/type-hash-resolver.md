---
icon: hashtag
---

# Type Hash Resolver

PurrNet identifies networked types with stable 32-bit hashes. Logs can sometimes contain only that numeric ID, especially when a received type is not registered. The Type Hash Resolver maps the ID back to matching types in the assemblies currently loaded by the Unity Editor.

Open **Tools > PurrNet > Analysis > Type Hash Resolver**, enter the hash from the log, and select **Resolve**. The input accepts:

* An unsigned decimal value, such as `3153982458`
* A signed decimal representation, such as `-1140984838`
* A hexadecimal value prefixed with `0x`

Each result shows the full type name, assembly, hash, and whether the type is registered with PurrNet. Use **Copy** for the full type name or **AQN** for its assembly-qualified name.

The resolver scans loaded assemblies, so it can find a type that exists in the project even when PurrNet has not registered it. If there is no match, confirm that the assembly containing the type compiled successfully and is loaded in the current Editor session.

{% hint style="info" %}
The tool diagnoses the ID; it does not register the type or change project files.
{% endhint %}
