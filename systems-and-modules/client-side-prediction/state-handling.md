# State Handling

State handling is at the core of the Client-Side Prediction (CSP) system. The `STATE` struct represents the current state of your entity, and it is the primary source of truth for all simulation logic. Here’s what you need to know about state handling:

***

**Key Concepts**

1. **`STATE` as the Source of Truth**:
   * The `STATE` struct contains all the data that defines the entity's current state (e.g., position, rotation, velocity, flags like `isJumping`).
   * This is the data you should rely on for all simulation logic.
2. **Automatic Reconciliation**:
   * The CSP system automatically reconciles the `STATE` with the server's authoritative state.
   * If the client's predicted state differs from the server's state, the system will adjust the `STATE` to match the server's version.
3. **Proxying Unity State**:
   * If your `STATE` affects Unity components (e.g., `Transform`, `Rigidbody`), use the following overrides to synchronize them:
     * **`protected override void GetUnityState(ref STATE state)`**:
       * Updates the `STATE` struct with data from Unity components (e.g., reading the `Transform.position`).
     * **`protected override void SetUnityState(STATE state)`**:
       * Applies the `STATE` to Unity components (e.g., setting the `Transform.position`).
