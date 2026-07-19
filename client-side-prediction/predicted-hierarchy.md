# Predicted Hierarchy

The `Predicted Hierarchy` system provides a set of methods for dynamically creating, deleting, and managing objects within the simulation. These methods allow developers to interact with the hierarchy in a way that feels natural and Unity-like, while ensuring that all changes are predictable and synchronized with the server.

***

**Pieces**

Every GameObject inside a spawned prefab that carries at least one `PredictedIdentity` is a piece with its own `PredictedObjectID`. The prefab root is always a piece, and nested identity-bearing children each get their own id too. This means you can address, delete, and reparent parts of a prefab individually instead of treating the whole instance as one unit.

* `Create` returns the id of the root piece. Ids for the remaining pieces of the same instance are allocated in a contiguous block right after it, in the prefab's hierarchy order.
* Spawning always instantiates the complete prefab and then prunes any pieces the current state says are deleted. Because all surviving pieces come from the same instantiation, serialized references between them keep working on every peer, including late joiners.
* Scene objects follow the same rules: nested identity-bearing children of a scene object are individually addressable pieces.

***

**Key Methods**

1. **Object Creation**:
   *   **`Create` Methods**:

       * Instantiates objects in the simulation.
       * Supports creating objects from prefab IDs or `GameObject` references.
       * Optionally allows specifying position, rotation, and owner.
       * Returns the `PredictedObjectID` of the root piece.

       ```csharp
       public PredictedObjectID? Create(int prefabId, PlayerID? owner = null);
       public PredictedObjectID? Create(int prefabId, Vector3 position, Quaternion rotation, PlayerID? owner = null);
       public PredictedObjectID? Create(GameObject prefab, PlayerID? owner = null);
       public PredictedObjectID? Create(GameObject prefab, Vector3 position, Quaternion rotation, PlayerID? owner = null);
       ```
   *   **Parented `Create` Overloads**:

       * Instantiates an object already attached to a parent, with a local pose relative to it.
       * The parent link is part of the spawn, so late joiners and replays reproduce it.
       * The `Transform` overloads resolve the predicted object themselves. The transform must be on a GameObject carrying a `PredictedIdentity`; passing a plain nested transform logs an error, attaching at arbitrary child transforms is not supported.

       ```csharp
       public PredictedObjectID? Create(int prefabId, Vector3 localPosition, Quaternion localRotation, PredictedComponentID parent, PlayerID? owner = null);
       public PredictedObjectID? Create(GameObject prefab, Vector3 localPosition, Quaternion localRotation, PredictedComponentID parent, PlayerID? owner = null);
       public PredictedObjectID? Create(GameObject prefab, Vector3 localPosition, Quaternion localRotation, PredictedIdentity parent, PlayerID? owner = null);
       public PredictedObjectID? Create(int prefabId, Vector3 localPosition, Quaternion localRotation, Transform parent, PlayerID? owner = null);
       public PredictedObjectID? Create(GameObject prefab, Vector3 localPosition, Quaternion localRotation, Transform parent, PlayerID? owner = null);
       ```
   *   **`TryCreate` Methods**:

       * Attempts to create an object and returns a boolean indicating success.
       * Outputs the `PredictedObjectID` of the created object.
       * Fails if prefab id is invalid or prefab isn't part of the registered prefab list.

       ```csharp
       public bool TryCreate(int prefabId, out PredictedObjectID id, PlayerID? owner = null);
       public bool TryCreate(GameObject prefab, out PredictedObjectID id, PlayerID? owner = null);
       ```
2. **Object Deletion**:
   *   **`Delete` Methods**:

       * Removes pieces from the simulation. Any piece can be deleted, not just prefab roots.
       * Deleting a piece also deletes the same-instance pieces attached under it. Pieces that were reparented away survive, and foreign pieces that were physically nested under it are detached and kept alive.
       * Deleting a root while some of its pieces live elsewhere promotes the survivors to standalone objects.
       * Handles both pooled and non-pooled objects.

       ```csharp
       public void Delete(PredictedObjectID? id);
       public void Delete(PredictedIdentity pid);
       public void Delete(GameObject go);
       ```

       `Delete(GameObject)` performs a real delete of that piece. If you only want a cheap reversible toggle, use `PredictedGameObject.SetActive` instead.
3. **Object Retrieval**:
   *   **`GetGameObject`**:

       * Retrieves the `GameObject` associated with a `PredictedObjectID`.

       ```csharp
       public GameObject GetGameObject(PredictedObjectID? id);
       ```
   *   **`GetComponent`**:

       * Retrieves a specific component from the `GameObject` associated with a `PredictedObjectID`.

       ```csharp
       public T GetComponent<T>(PredictedObjectID? id) where T : Component;
       ```
   *   **`TryGetId`**:

       * Retrieves the `PredictedObjectID` associated with a `GameObject`. Works for any piece, including nested ones.

       ```csharp
       public bool TryGetId(GameObject go, out PredictedObjectID id);
       ```
   *   **`TryGetGameObject`**:

       * Retrieves the `GameObject` associated with a `PredictedObjectID` and returns a boolean indicating success.

       ```csharp
       public bool TryGetGameObject(PredictedObjectID? id, out GameObject go);
       ```
4. **Instance Queries**:
   *   **`TryGetRootId`** resolves the root piece of the instance a piece belongs to.
   *   **`SameInstance`** checks whether two pieces were spawned as part of the same instance.
   *   **`PredictedIdentity.rootObjectId`** is a convenience shortcut for the root piece id of the identity's instance.

       ```csharp
       public bool TryGetRootId(PredictedObjectID pieceId, out PredictedObjectID rootId);
       public bool SameInstance(PredictedObjectID a, PredictedObjectID b);
       public bool SameInstance(PredictedIdentity a, PredictedIdentity b);
       ```

***

**Reparenting**

Transform parents are not synchronized by default. A piece without a [`PredictedParent`](built-in-components/predicted-parent.md) component always belongs to its authored prefab parent as far as the simulation is concerned, and physically reparenting it is treated as client-local decoration. Add `PredictedParent` to a piece to make its parent part of predicted state, so reparenting rolls back, replays, and reaches late joiners like any other state change.

Delete cascades follow the simulated graph, meaning `PredictedParent` state where present and the authored default otherwise. Local-only decoration never changes what gets deleted.

***

**Prefabs and Pooling**

* Predicted object creation uses a central `PredictedPrefabs` asset referenced by `PredictionManager`.
* Prefabs can opt into pooling. Deleted pieces are kept in a pool for the rollback window so a mispredicted delete can resurrect the exact same objects, serialized references intact.
* When reconciling, pooled objects may be disabled/enabled; avoid parenting critical visuals to the root identity if you rely on `SetActive` state, or use `PredictedTransform`'s optional graphics unparenting.

***

**Spawning Network Identities**

* Use `PredictedIdentitySpawner` to mirror a set of `NetworkIdentity` objects between server and clients based on prediction state.
* The spawner coordinates early spawn, observer assignment, ownership, and finalization in both server and verified client passes.

***

**Best Practices**

* Only create/destroy from within simulation code paths (or via machine/state transitions) to keep behavior deterministic under replay.
* Prefer `TryCreate` and check the boolean to handle invalid or unregistered prefabs gracefully.
* Store and pass around `PredictedObjectID` rather than `GameObject` to remain resilient across rollbacks.
* When you need to know if two ids belong to the same spawned prefab, use `SameInstance` or `TryGetRootId` instead of comparing raw ids.

***

**Upgrade Notes**

Older versions assigned one id per spawned prefab. Now every identity-bearing GameObject gets its own id:

* `id.objectId` identifies the piece, not the whole instance. Code that compared object ids to group components of the same prefab should use `SameInstance` or `rootObjectId`.
* `TryGetId` now succeeds for nested piece GameObjects, not just prefab roots.
* `Delete(GameObject)` used to reroute to `PredictedGameObject.SetActive(false)`. It now deletes the piece for real.
* The wire and state formats changed, so all peers must run the same version.
