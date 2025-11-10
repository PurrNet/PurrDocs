# Overview

The **Client-Side Prediction (CSP)** system is designed to provide a seamless and responsive multiplayer experience by predicting game states locally on the client. This system is built around a modular architecture that allows for flexible and extensible design, enabling systems that behave as if the game were single-player, while still synchronizing with the server automatically.

**Key Components**

1. **PredictionManager**:
   * Acts as the central "world" for client-side prediction.
   * Manages all predicted entities and systems.
   * Handles the lifecycle of predicted states, including prediction, reconciliation, and view updates.
2. **PredictedIdentity**:
   * Unity components that define the behavior of predicted entities.
   * Created by users to handle specific functionalities, such as movement, physics, or custom logic.
3. **PredictedHierarchy**:
   * Provides a prediction compatible version of Unity's Instantiate and Destroy methods.

**Design Philosophy**

* **Decoupled from Traditional Networking**:
  * This system is completely disconnected from the usual `NetworkIdentity` setup.
  * **RPCs (Remote Procedure Calls)** are not supported or needed in this architecture, as prediction handles state synchronization naturally.
  * Logic is executed locally on the client, mimicking a single-player experience, while still maintaining consistency with the server.
  * This approach simplifies development, as developers can focus on writing game logic without worrying about networking intricacies.

**Benefits**

* **Responsive Gameplay**: Predictions provide immediate feedback to the player, reducing the perceived latency.
* **Modularity**: Systems can be easily added or modified, allowing for flexible and scalable game design.
* **Consistency**: Reconciliation ensures that the client’s state aligns with the server’s authoritative state, maintaining a consistent game world.

**Limitations**

* **Prediction Errors**: Incorrect predictions may require corrections, which can occasionally result in visual "snapping" or adjustments.
* **Complexity**: While the system simplifies networking, it introduces new challenges in managing predicted states and reconciliation.

