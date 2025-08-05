# Predicted State Node

A predicted state node is a class you can inherit from in order to build your own states for the [predicted state machine](./).

There are multiple predicted state node types to inherit from:

* `PredictedStateNode<STATE>` This allows your statenode to hold and simulate state
* `PredictedStateNode<INPUT, STATE>` This allows your statenode to hold and simulate state w. input. Optimal for characters/players'

### Overrides

These can override a few different methods, in order for you to run functionality unique to the individual states:

* `Enter` This is optimal for running simulation logic upon entering the state
* `Exit` This is optimal for running simulation logic upon exiting the state
* `StateSimulate` This is essentially your Simulate method, but it only runs when the state is active
* `ViewEnter` Here you will run view/visual logic upon entering the state
* `ViewExit` Here you will run view/visual logic upon exiting the state

Mind you, you still have the normal predicted identity override as well!
