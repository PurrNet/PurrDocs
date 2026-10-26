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

### Minimal Example

```csharp
// A state without input
public class IdleState : PredictedStateNode<MyState>
{
    public override void Enter() { /* start idle anim */ }
    protected override void StateSimulate(ref MyState s, float dt) { /* no-op */ }
}

// A state with input
public class MoveState : PredictedStateNode<MyInput, MyState>
{
    // Required by PredictedIdentity<TInput, TState>. Runs every tick, even when this node isn't active
    protected override void Simulate(MyInput input, ref MyState state, float delta) { }

    protected override void StateSimulate(in MyInput i, ref MyState s, float dt)
    { s.velocity = new Vector3(i.x, 0, i.y) * s.speed; }
}

// Somewhere in a predicted identity or node
void HandleTransitions()
{
    if (/* want next */) machine.Next();
    if (/* want specific */) machine.SetState(mySpecificNode);
}
```

Notes:

- `ViewEnter`/`ViewExit` fire for both `viewState` and `verified` transitions, allowing you to scope VFX/UI to predicted vs verified changes.
- Keep `StateSimulate` deterministic; drive visuals in `ViewEnter`/`ViewExit`/`UpdateView`.

### State and input across transitions

Every state node is its own predicted identity, with its own state, its own input and its own history. Nothing is handed from one node to the next when the machine changes state. The machine's own state only holds which node is active (plus a transition counter), so if `MoveState` and `JumpState` both have a `velocity` field, those are two separate values.

What happens to a node that isn't active:

* Its state is kept as it was, and it is still rolled back and reconciled like any other predicted identity. `Enter` doesn't reset it, so when you come back to a node, it continues from where it left off.
* For an input node, `GetFinalInput` is still called every tick on the owner, and the input is still sent and stored. If input extrapolation is on, a node extrapolates from its own input history, so a node you just entered doesn't start from an empty input.
* `StateSimulate` and `StateUpdateView` only run on the active node. The normal `Simulate` and `UpdateView` overrides of a predicted identity run on every node, every tick, so don't put state specific logic there.

Inputs are gathered for every identity before any simulation runs, so during `StateSimulate` the active node can read the current tick's input and state of any other predicted identity.

#### Sharing data between nodes

Put the data that has to survive a transition, like velocity, grounded state or the player's input, on one separate predicted identity, and let the nodes read and write it. The nodes then only need state for what is unique to them, like an attack timer.

```csharp
public struct PlayerInput : IPredictedData
{
    public Vector2 move;
    public bool jump;
    public void Dispose() { }
}

public struct MotorState : IPredictedData<MotorState>
{
    public Vector3 velocity;
    public bool grounded;
    public void Dispose() { }
}

// Gathers the input and holds the data every node shares
public class PlayerMotor : PredictedIdentity<PlayerInput, MotorState>
{
    protected override void GetFinalInput(ref PlayerInput input)
    {
        input.move = new Vector2(Input.GetAxisRaw("Horizontal"), Input.GetAxisRaw("Vertical"));
        input.jump = Input.GetKey(KeyCode.Space);
    }

    protected override void Simulate(PlayerInput input, ref MotorState state, float delta) { }
}

public struct NoState : IPredictedData<NoState>
{
    public void Dispose() { }
}

public class WalkState : PredictedStateNode<NoState>
{
    [SerializeField] private PlayerMotor _motor;
    [SerializeField] private JumpState _jumpState;
    [SerializeField] private float _speed = 5f;

    protected override void StateSimulate(ref NoState state, float delta)
    {
        var input = _motor.currentInput;
        ref var motor = ref _motor.currentState;

        motor.velocity = new Vector3(input.move.x, 0, input.move.y) * _speed;

        if (input.jump && motor.grounded)
            machine.SetState(_jumpState);
    }
}

public class JumpState : PredictedStateNode<JumpState.State>
{
    [SerializeField] private PlayerMotor _motor;
    [SerializeField] private float _jumpSpeed = 6f;

    public struct State : IPredictedData<State>
    {
        public float airTime;
        public void Dispose() { }
    }

    public override void Enter()
    {
        // The walking velocity is still on the motor, so it carries into the jump
        _motor.currentState.velocity.y = _jumpSpeed;
        _motor.currentState.grounded = false;
        currentState.airTime = 0;
    }

    protected override void StateSimulate(ref State state, float delta)
    {
        state.airTime += delta;
        _motor.currentState.velocity += Physics.gravity * delta;
    }
}
```

Your own ground check sets `grounded` on the motor, and a node can switch back with `machine.SetState` once it's true again.

If you'd rather keep input on the nodes, give every node that needs it the same input type and the same `GetFinalInput`. That works because inactive nodes keep gathering input, but each node then sends its own copy of the input every tick.
