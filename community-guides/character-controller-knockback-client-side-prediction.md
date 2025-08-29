---
description: by Shelby
---

# Purrdicted Character Controller Knockback

## Introduction
This guide will follow on from the [Purrdicted Character Controller](character-controller-client-side-prediction.md) guide and will show you how to implement knockback into your character controller! All the prerequisites from the previous guide still apply, so make sure to check that out first if you haven't already.

## Getting Started
To get started, let's create a new class `PredictedKnockback`, and we'll go ahead and inherit from `PredictedIdentity` again. This time, however, we will only be creating a `STATE` struct, as we won't be needing any additional input for this.

```csharp
using PurrNet.Prediction;
using UnityEngine;

public struct PredictedKnockbackState : IPredictedData<PredictedKnockbackState>
{
    public void Dispose() { }
}

public class PredictedKnockback : PredictedIdentity<PredictedKnockbackState>
{
    protected override void Simulate(ref PredictedKnockbackState state, float delta) { }
}
```

## STATE
For our state, let's keep track of our `CurrentKnockbackVelocity`. We can use this to apply a knockback force to our character controller.

```csharp
public struct PredictedKnockbackState : IPredictedData<PredictedKnockbackState>
{
    public Vector3 CurrentKnockbackVelocity;

    public void Dispose() { }
}
```

## Preparing to Simulate
Just like in the previous guide, we will want to get a reference to our `CharacterController`, as well as create a `drag` variable.

```csharp
[RequireComponent(typeof(CharacterController))]
public class PredictedKnockback : PredictedIdentity<PredictedKnockbackState>
{
    [SerializeField] float drag = 5f;
    CharacterController controller;

    protected void Awake()
    {
        controller = GetComponent<CharacterController>();
    }

    // ... rest of the code remains the same
}
```

## Simulate
Now, let's implement the `Simulate` method. In here, we will apply the knockback velocity to our character controller. We'll do this by just moving the character controller by the `CurrentKnockbackVelocity` multiplied by `delta`. As well as that, we will reduce the `CurrentKnockbackVelocity` over time using `Vector3.MoveTowards` (you can probably use Lerp for this as well, or any other method).


```csharp
protected override void Simulate(ref PredictedKnockbackState state, float delta)
{
    state.CurrentKnockbackVelocity = Vector3.MoveTowards(state.CurrentKnockbackVelocity, Vector3.zero, drag * delta);
    controller.Move(state.CurrentKnockbackVelocity * delta);
}
```

## Applying Knockback
Now that we have our knockback system set up, we need a way to apply knockback to our player. We can do this by creating a method called `ApplyKnockback`, which takes in a direction and a force. This method will be called on the server and will update the `CurrentKnockbackVelocity` in our state.

```csharp
public void ApplyKnockback(Vector3 velocity)
{
    currentState.CurrentKnockbackVelocity += velocity;
}
```

## Final Script
Putting it all together, our final script looks like this:
```csharp
using PurrNet.Prediction;
using UnityEngine;

public struct PredictedKnockbackState : IPredictedData<PredictedKnockbackState>
{
    public Vector3 CurrentKnockbackVelocity;

    public void Dispose() { }
}

[RequireComponent(typeof(CharacterController))]
public class PredictedKnockback : PredictedIdentity<PredictedKnockbackState>
{
    [SerializeField] float drag = 5f;
    CharacterController controller;

    protected void Awake()
    {
        controller = GetComponent<CharacterController>();
    }

    protected override void Simulate(ref PredictedKnockbackState state, float delta)
    {
        state.CurrentKnockbackVelocity = Vector3.MoveTowards(state.CurrentKnockbackVelocity, Vector3.zero, drag * delta);
        controller.Move(state.CurrentKnockbackVelocity * delta);
    }

    public void ApplyKnockback(Vector3 velocity)
    {
        currentState.CurrentKnockbackVelocity += velocity;
    }
}
```

## Testing
As a very simple test, let's head back to our `PredictedCharacterController` and add a simple test to apply knockback when we press the `Shift` key.
1. Create a new `Dash` boolean in our `PredictedCharacterControllerInput` struct.
```csharp
public struct PredictedCharacterControllerInput : IPredictedData<PredictedCharacterControllerInput>
{
    public Vector3 Movement;
    public bool Jump;
    public bool Dash;

    public void Dispose() { }
}
```

2. Check for input in `UpdateInput`
```csharp
protected override void UpdateInput(ref PredictedCharacterControllerInput input)
{
    input.Movement = new Vector3(Input.GetAxisRaw("Horizontal"), 0, Input.GetAxisRaw("Vertical"));
    input.Jump |= Input.GetKeyDown(KeyCode.Space);
    input.Dash |= Input.GetKeyDown(KeyCode.LeftShift);
}
```

3. In `Simulate`, check if `input.Dash` is true, and if so, apply knockback to ourselves.
```csharp
protected override void Simulate(PredictedCharacterControllerInput input, ref PredictedCharacterControllerState state, float delta)
{
    // ... rest of the code remains the same

    if (input.Dash)
        GetComponent<PredictedKnockback>().ApplyKnockback(transform.forward * 10f);
}
```
4. Add the `PredictedKnockback` component to the same GameObject as the `PredictedCharacterController` and `CharacterController` components (Our Player prefab).
5. Run the scene, and confirm hitting `Shift` applies "knockback" to the player!
6. (Optional) Try calling `ApplyKnockback` from another script you may have!

## Conclusion
Congratulations! You (hopefully) have a capsule that can now be knocked around! Fire up a client and use the [PurrTransport](../systems-and-modules/transports/purr-transport.md) and try it out with real world latency!

If you have any questions, feel free to ask in the [Discord server](https://discord.gg/HnNKdkq9ta)!