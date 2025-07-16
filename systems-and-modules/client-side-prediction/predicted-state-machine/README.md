# Predicted State Machine

The predicted state machine allows you to easily utilize the beautiful modularity and scalability of a finite state machine.&#x20;

In order to work with the predicted state machine, you first need some [predicted state nodes](predicted-state-node.md).

From the simulate loop of any predicted identity, you can easily reference the machine and call one of the following:

* &#x20;`Next();` allowing you to go to the next state in the list
* `Previous();` going backwards to the previous state node
* `SetState(PredictedStateNode state);` allowing you to easily go to a specific state node.

### Scene setup

In order to work with the predicted state machine, all you need it to set it up in the scene or on your desired prefab (this can also be utilized for player logic). And in the states list, simply add a reference to your states (which are typically on or childed to the state machine gameobject)

And that's it. The state machine will now start in the first state in the list, and you can easily navigate it using the methods above.
