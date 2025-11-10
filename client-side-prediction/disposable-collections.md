---
icon: recycling
---

# Disposable Collections

PurrNet provides pooled, disposable collections for GC‑friendly state and deterministic iteration:

- `DisposableList<T>`
- `DisposableDictionary<TKey, TValue>`
- `DisposableArray<T>`

Use these in your `STATE` structs and long‑lived prediction data. They integrate with packing/history to duplicate safely and dispose cleanly.

***

**Why Disposable Collections**

- Minimize allocations by renting from pools under the hood (`ListPool`, `DictionaryPool`, `ArrayPool`).
- Deterministic iteration for dictionaries via an internal stable key list.
- Codegen/packing support: deep copy via `Duplicate()` during history snapshots and deltas.

***

**General Rules**

- Always create via `Create(...)` factory; do not use `new List<T>()`, `new Dictionary<,>()`, or constructors.
- Always call `Dispose()` when you are done with the collection.
- If a collection lives inside a `STATE`, dispose it in `STATE.Dispose()`.
- Do not struct‑copy a disposable collection and dispose both; use `.Duplicate()` to create an independent copy.

***

**DisposableList<T>**

- Create: `var list = DisposableList<MyType>.Create(capacity);` or `Create()` or `Create(IEnumerable<T>)`.
- Use like a regular `List<T>` (`Add`, indexer, `Count`, etc.).
- Dispose: `list.Dispose();`
- In `STATE`, implement dispose:

```csharp
public struct InventoryState : IPredictedData<InventoryState>
{
    public DisposableList<int> items;
    public void Dispose() { items.Dispose(); }
}
```

***

**DisposableDictionary<TKey, TValue>**

- Create: `var dict = DisposableDictionary<PlayerID, PredictedObjectID>.Create();`
- Iteration order is deterministic using an internal key list.
- Use: `Add`, indexer, `TryGetValue`, `Remove`, `ContainsKey`.
- Example in a `STATE` (from PlayerSpawner): `players = DisposableDictionary<PlayerID, PredictedObjectID>.Create();`
- Dispose in `STATE.Dispose()`:

```csharp
public void Dispose() { players.Dispose(); }
```

Tip: When enumerating, `foreach (var (k, v) in dict)` is safe and stable. Avoid mutating structure while iterating.

***

**DisposableArray<T>**

- Fixed‑size, pooled array with optional `Resize(int)` growth.
- Create: `var arr = DisposableArray<byte>.Create(size);`
- No `Add/Remove/Insert/Clear`; indexer for read/write.
- Dispose: `arr.Dispose();`

***

**Copying and History**

- The prediction history uses packers that deep‑copy disposable collections by calling `Duplicate()` under the hood.
- This ensures snapshots are independent. You should still implement `Dispose()` on your `STATE` to release each snapshot when it is discarded.

Anti‑pattern:

- Avoid `var b = a; b.Dispose(); a.Dispose();` on disposable structs — both point to the same pooled container. Use `var b = a.Duplicate();` if you truly need a separate copy.

***

**Short‑Lived Temporaries**

- For per‑frame or function‑local scratch collections, prefer non‑disposable pools:
  - `var tmp = ListPool<T>.Instantiate(); ... ListPool<T>.Destroy(tmp);`
  - `var tmp = DictionaryPool<K,V>.Instantiate(); ... DictionaryPool<K,V>.Destroy(tmp);`
- These do not need to be part of state and should not be stored across frames.

***

**Leak Checks (Editor)**

- When `PURR_LEAKS_CHECK` is enabled in Editor, pooled allocations are tracked and usage is updated on access. This helps catch missed `Dispose()`/`Destroy()` calls during development.

***

**IDuplicate<T> and Performance**

- The packer copies state snapshots via `Packer.Copy<T>(value)`.
- If `T : IDuplicate<T>`, the packer calls `Duplicate()` directly instead of serializing/deserializing to clone.
- Implementing `IDuplicate<T>` on your custom structs nested inside `STATE` can significantly reduce GC and CPU during prediction history copies and reconciliation.

Example:

```csharp
using PurrNet.Packing;

public struct MySubState : IDuplicate<MySubState>
{
    public DisposableList<int> indices;
    public float weight;

    public MySubState Duplicate()
    {
        return new MySubState {
            indices = indices.Duplicate(), // deep copy pooled list
            weight = weight
        };
    }
}

public struct MyState : IPredictedData<MyState>
{
    public MySubState data;
    public void Dispose() { data.indices.Dispose(); }
}
```

Tips:

- Implement `IEquatable<T>` as well for fast equality checks (`Packer.AreEqual`) used in delta packing.
- Disposable collections already implement `IDuplicate<T>`; use and dispose them correctly to benefit automatically.
