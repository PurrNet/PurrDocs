# Predicted State Node

A predicted state node is a class you can inherit from in order to build your own states for the [predicted state machine](./).

There are multiple predicted state node types to inherit from:

* `PredictedStateNode<STATE>` This allows your statenode to hold and simulate state
* `PredictedStateNode<INPUT, STATE>` This allows your statenode to hold and simulate state w. input. Optimal for characters/players'

### Overrides

These can override a few different methods, in order for you to run functionality unique to the individual states:

* `Enter` This is optimal for running simulation logic upon entering the state
* `Exit` This is optimal for running simulation logic upon exiting the state
* `StateSimulate` This is your per‑tick state simulation; runs only when this state is active
* `ViewEnter` Run view/visual logic upon entering the state (predicted or verified)
* `ViewExit` Run view/visual logic upon exiting the state (predicted or verified)

Mind you, you still have the normal predicted identity override as well! Those run on every node, every tick, even when the node isn't active. Each node keeps its own state and input, see [State and input across transitions](./#state-and-input-across-transitions).

### Snippet

```csharp
public class AttackState : PredictedStateNode<MyInput, MyState>
{
    public override void Enter() { /* start attack windup */ }

    // Required by PredictedIdentity<TInput, TState>. Runs every tick, even when this node isn't active
    protected override void Simulate(MyInput input, ref MyState state, float delta) { }

    protected override void StateSimulate(in MyInput i, ref MyState s, float dt)
    {
        // simulate attack timing using only s and i
        s.attackTimer = Mathf.Max(0, s.attackTimer - dt);
        if (s.attackTimer == 0) machine.SetState(nextState);
    }

    public override void ViewEnter(bool verified)
    { /* play VFX/SFX, camera shake, etc */ }
}
```
