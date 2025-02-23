# State Machine (Auto Networked)

The Auto networked state machine of PurrNet makes it very easy to align state machines even with custom state data if necessary.

{% embed url="https://youtu.be/rJm0YPoLBGs" %}

The state machine also have a custom editor allowing for easy debugging and runtime manipulation of the active state. It also attempts to serialize custom node data.

By default, the state machine is server auth, but you can use the bool in the inspector, to change whether you'd rather have it be owner auth. The main difference is who is allowed to change states on the state machine. The "controller" (whether that is the server or owner), will always handle state changes local to make it fast and responsive.

The state machine is extremely useful at keeping games aligned, and we personally use it for game state handling, which could look something like:\
&#x31;_. Countdown state_\
&#xNAN;_&#x32;. Spawning enemies state_\
&#xNAN;_&#x33;. Spawning boss state_\
&#xNAN;_&#x34;. Round end state_\
&#xNAN;_&#x35;. Shop state_

_They can also be used for other things like the state of your player, in the case of using owner auth._

You can move between states as you please, but also have an easy way of moving sequentially through state.

### Scene setup

The scene setup is easy! All you have to do, is add the State Machine to your scene. I recommend making a new gameobject for these, to keep it structured.

For every **StateNode** that you make, you can add that to your scene as well. I typically do it as a child of the **StateMachine** gameobject.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Example setup of the State Machine</p></figcaption></figure>

### State Nodes

State nodes are what you know as your states. There are 2 kinds of state notes. Those that expect data, and those that don't.

First off, in order to have a state node, you need a new script which inherits from **StateNode** or **StateNode\<T>** depending on whether the state expects data or not.

```csharp
using PurrNet.StateMachine;

public class TestState : StateNode
{
    
}
```

The StateNode is a network behavior as well, so you can act with it exactly as you would any other networked script. The main difference are some of the virtual methods which you can override, as well as the ease access to the state machine in order to change states.

```csharp
public override void Enter(bool asServer)
{
    //Runs when the state is entered
}

public override void Exit(bool asServer)
{
    //Runs when the state is exited
}

public override void StateUpdate(bool asServer)
{
    //Runs every frame while the state is active
}
```

### Changing state

Changing states with the networked state machine is very easy. Only thing you need is a reference to the state machine. **StateNodes** automatically have that reference by simply typing `machine`.

There are 3 ways of changing the state. `Next`, `Previous` or `SetState` .

```csharp
public StateNode specificState;

private void StateChangeShowcase()
{
    //Goes to the next state in the StateMachine list
    machine.Next();
    //Goes to the previous state in the StateMachine list
    machine.Previous();
    //Goes to a specific state in the StateMachine list
    machine.SetState(specificState);
}
```

### State Nodes with data

Working with state nodes which require data is easy as well. You work with it super similar to a regular state node, except you make it generic with the type of required data:

```csharp
public class TestState : StateNode<int>
{
    public override void Enter(int data, bool asServer)
    {
        Debug.Log($"I received data: {data}");
    }
}
```

Beware that if the state is switched to without being fed data, it will run the normal Enter method without the data type. This way you can always use that as a fail safe in case a state might be entered wrongly.

In order to send data with the state switch, you simply feed it when changing state

```csharp
//Goes to the next state in the StateMachine list while feeding data
machine.Next(5);

//Goes to a specified state while feeding data
machine.SetState(specificState, 5);
```
