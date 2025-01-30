# Predicted Hierarchy

The `Predicted Hierarchy` system provides a set of methods for dynamically creating, deleting, and managing objects within the simulation. These methods allow developers to interact with the hierarchy in a way that feels natural and Unity-like, while ensuring that all changes are predictable and synchronized with the server.

**Key Methods**

1. **Object Creation**:
   *   **`Create` Methods**:

       * Instantiates objects in the simulation.
       * Supports creating objects from prefab IDs or `GameObject` references.
       * Optionally allows specifying position and rotation.
       * Returns a `PredictedObjectID` to uniquely identify the created object.

       ```csharp
       public PredictedObjectID? Create(int prefabId);
       public PredictedObjectID? Create(int prefabId, Vector3 position, Quaternion rotation);
       public PredictedObjectID? Create(GameObject prefab);
       public PredictedObjectID? Create(GameObject prefab, Vector3 position, Quaternion rotation);
       ```
   *   **`TryCreate` Methods**:

       * Attempts to create an object and returns a boolean indicating success.
       * Outputs the `PredictedObjectID` of the created object.
       * Fails if prefab id is invalid or prefab isn't part of the registered prefab list.

       ```csharp
       public bool TryCreate(int prefabId, out PredictedObjectID id);
       public bool TryCreate(GameObject prefab, out PredictedObjectID id);
       ```
2. **Object Deletion**:
   *   **`Delete` Methods**:

       * Removes objects from the simulation using their `PredictedObjectID`.
       * Handles both pooled and non-pooled objects.

       ```csharp
       public void Delete(PredictedObjectID id);
       public void Delete(PredictedObjectID? id);
       ```
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

       * Retrieves the `PredictedObjectID` associated with a `GameObject`.

       ```csharp
       public bool TryGetId(GameObject go, out PredictedObjectID id);
       ```
   *   **`TryGetGameObject`**:

       * Retrieves the `GameObject` associated with a `PredictedObjectID` and returns a boolean indicating success.

       ```csharp
       public bool TryGetGameObject(PredictedObjectID? id, out GameObject go);
       ```
