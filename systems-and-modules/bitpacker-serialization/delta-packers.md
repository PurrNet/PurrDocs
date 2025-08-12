---
description: Per-value delta serialization (auto + custom)
icon: box-open-full
---

# Delta Packers

Delta packers write only the differences between an old value and a new value. This reduces bandwidth compared to writing the full value every time.

PurrNet automatically generates delta serializers for your types. You can also define custom delta logic when you need more control. If something goes wrong, we fall back to a safe path that behaves like "changed flag + full value".

## When to use

Use delta packers when:
- Your values change slightly or sparsely (e.g., vectors, rotations, small structs).
- You want fewer bits on the wire than full value writes.
- You're fine with the automatic delta, or you want to override it with your own logic.

If a delta isn't applicable (or an error occurs), we'll automatically write a small "changed" marker and, if needed, the full value.

## API (per-type)

Only the essentials you'll typically call:

```csharp
// Emits a delta from oldValue -> newValue.
// Returns true if any bits were written.
bool DeltaPacker<T>.Write(BitPacker packer, T oldValue, T newValue)

// Reads a delta and applies it to oldValue, producing the result.
T DeltaPacker<T>.Read(BitPacker packer, T oldValue)
```

Notes:
- You don't need to register anything for built-in or auto-discovered static serializers.
- If no delta is available, a compact fallback is used automatically.

## How it works (high level)

- Write decides if anything changed and writes only what's needed.
- Read consumes exactly what Write produced to rebuild the current value from the old one.
- If a type doesn't have a delta serializer available, we write a single "changed" bit and, when true, the full value (safe fallback).

## Examples

### Float with a small dead-zone
Only send a delta when the change is meaningful; keep the rest free.

```csharp
// Writing
var wrote = DeltaPacker<float>.Write(packer, oldHealth, newHealth);

// Reading
var currentHealth = DeltaPacker<float>.Read(packer, oldHealth);
```

Tip: With auto-generated deltas, this already "just works." You only need your own logic if you want a specific threshold or quantization scheme.

### Vector3 (changed-components pattern)
Send a small mask to indicate which components changed, then only write those.

```csharp
// Writing
bool wrote = DeltaPacker<Vector3>.Write(packer, oldPosition, newPosition);

// Reading
var currentPosition = DeltaPacker<Vector3>.Read(packer, oldPosition);
```

This is a common pattern: bitmask first, followed by only the changed components.

## Custom delta (advanced, optional)

If you want to override the automatic behavior, define your own static delta serializer similar to how you'd do for normal packers. PurrNet will discover static serializers defined under your static class and use them automatically.

You don't need to manually register these—discovery is automatic.