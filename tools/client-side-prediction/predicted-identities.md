# Predicted Identities

Predicted identities are the core entities managed by the CSP system. They are normal Unity components that can be used to define the behavior of objects in a predicted environment. These identities are responsible for simulating their state based on user input or environmental factors, ensuring that the client can predict and reconcile their behavior accurately.

**Key Concepts**

1. **`PredictedIdentity` Abstract Class**:
   * The base class for all predicted entities.
   * Not meant to be used directly; instead, it provides the foundation for more specialized implementations.
   * Handles the lifecycle of prediction, simulation, and reconciliation.
2. **Generic Variants**:
   * **`PredictedIdentity<INPUT, STATE>`**:
     * Used for entities that are controlled by user input.
     * `INPUT` represents the user input (e.g., movement keys, actions).
     * `STATE` represents the entity's state (e.g., position, rotation, velocity).
   * **`PredictedIdentity<STATE>`**:
     * Used for entities that are not directly controlled by user input but are affected by the environment or time (e.g., physics objects, AI-controlled entities).
     * `STATE` represents the entity's state.
