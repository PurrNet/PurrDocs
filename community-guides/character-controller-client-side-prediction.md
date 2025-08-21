---
description: by Shelby
---

# Purrdicted Character Controller

## Introduction
While PurrDiction's main flagship feature is supporting `Rigidbody` controllers and interaction, we can still predict with the `CharacterController` as well (as well as everything else)!

Before we begin, it's highly recommended to read through the [PurrDiction](../tools/client-side-prediction/README.md) docs to get a better understanding of how the system works.

For further reading, you should also check out a series of articles on [Client Side Prediction](client-side-prediction.md) by Neotime, which provides a far more in-depth look at Client Side Prediction principles.

This guide will assume you have a basic understanding of how PurrDiction works, as well as a basic understanding of PurrNet.

## Getting Started
To get started, let's create a new script called `PredictedCharacterController` and inherit from `PredictedIdentity`. As well as that, let's create the `STATE` and `INPUT` structs, and implement the `Simulate` and `UpdateInput` methods.

`UpdateInput` is called every **frame** on the client and is where we will gather our input to later simulate. 
`Simulate` is called every **tick**, and is where we will apply our input to the state of our player.

```csharp
using PurrNet.Prediction;
using UnityEngine;

public struct PredictedCharacterControllerInput : IPredictedData<PredictedCharacterControllerInput>
{
    public void Dispose() { }
}

public struct PredictedCharacterControllerState : IPredictedData<PredictedCharacterControllerState>
{
    public void Dispose() { }
}

[RequireComponent(typeof(CharacterController))]
public class PredictedCharacterController : PredictedIdentity<PredictedCharacterControllerInput, PredictedCharacterControllerState>
{
    protected override void UpdateInput(ref PredictedCharacterControllerInput input) {}

    protected override void Simulate(PredictedCharacterControllerInput input, ref PredictedCharacterControllerState state, float delta) { }
}
```

## INPUT
The `INPUT` struct is what holds all of the input that we want to be able to check and use inside `Simulate`. For now, let's keep things extremely simple and check for Movement, and jumping, which we can do by adding a `Vector3` for movement and a `bool` for jumping.

```csharp
public struct PredictedCharacterControllerInput : IPredictedData<PredictedCharacterControllerInput>
{
    public Vector3 Movement;
    public bool Jump;

    public void Dispose() { }
}
```

## STATE
The `STATE` struct is what holds the current state of our player. This can be a lot of things, but since we are just working on simple movement, we can just store our current Velocity. You may wonder why we are storing Velocity instead of Position, and that is because we will be using the `PredictedTransform` to handle our position and rotation state.

```csharp
public struct PredictedCharacterControllerState : IPredictedData<PredictedCharacterControllerState>
{
    public Vector3 Velocity;

    public void Dispose() { }
}
```

## Gathering Input
Now that we have our `INPUT` and `STATE` structs, we can begin gathering input to later simulate. To do this, we use the `UpdateInput` method, which is called every frame on the client. You can gather input in any way you like, but for simplicity, we will use Unity's old input system.

```csharp
protected override void UpdateInput(ref PredictedCharacterControllerInput input)
{
    input.Movement = new Vector3(Input.GetAxisRaw("Horizontal"), 0, Input.GetAxisRaw("Vertical"));
    input.Jump |= Input.GetKeyDown(KeyCode.Space); // identical to: input.Jump = Input.GetKeyDown(KeyCode.Space) || input.Jump;
}
```

You may be curious to why we are using `|=` for `input.Jump` instead of just `=`, and that is because while `UpdateInput` is called every frame, `Simulate` is called every tick, which means that if we press the jump button between ticks, we may miss the jump input.

## Simulating with Our Input
Now that we have gathered our input, we can simulate! To do this, we can override the `Simulate` method, which is called every tick. If you are familiar with writing a basic `CharacterController`, the rest of this will be very familiar to you!

This code is modified from the Unity documentation on [CharacterController.Move](https://docs.unity3d.com/ScriptReference/CharacterController.Move.html).

```csharp
// Variables for speed, gravity, and jump height
[SerializeField] float speed = 5f;
[SerializeField] float gravity = -10f;
[SerializeField] float jumpHeight = 1f;

protected override void Simulate(PredictedCharacterControllerInput input, ref PredictedCharacterControllerState state, float delta)
{
    bool groundedPlayer = controller.isGrounded;
    if (groundedPlayer && state.Velocity.y < 0)
    {
        state.Velocity.y = 0f;
    }

    // Read input
    Vector3 move = new Vector3(input.Movement.x, 0, input.Movement.z);
    move = Vector3.ClampMagnitude(move, 1f);

    // Jump
    if (input.Jump && groundedPlayer)
    {
        state.Velocity.y = Mathf.Sqrt(jumpHeight * -2.0f * gravity);
    }

    // Apply gravity
    state.Velocity.y += gravity * delta;

    // Combine horizontal and vertical movement
    Vector3 finalMove = (move * speed) + (state.Velocity.y * Vector3.up);
    controller.Move(finalMove * delta);
}
```

## Final Script
Putting it all together, our final script looks like this:

```csharp
using PurrNet.Prediction;
using UnityEngine;

public struct PredictedCharacterControllerInput : IPredictedData<PredictedCharacterControllerInput>
{
    public Vector3 Movement;
    public bool Jump;

    public void Dispose() { }
}

public struct PredictedCharacterControllerState : IPredictedData<PredictedCharacterControllerState>
{
    public Vector3 Velocity;

    public void Dispose() { }
}

[RequireComponent(typeof(CharacterController))]
public class PredictedCharacterController : PredictedIdentity<PredictedCharacterControllerInput, PredictedCharacterControllerState>
{
    [SerializeField] float speed = 5f;
    [SerializeField] float gravity = -10f;
    [SerializeField] float jumpHeight = 1f;

    CharacterController controller;

    protected void Awake()
    {
        controller = GetComponent<CharacterController>();
    }

    protected override void UpdateInput(ref PredictedCharacterControllerInput input)
    {
        input.Movement = new Vector3(Input.GetAxisRaw("Horizontal"), 0, Input.GetAxisRaw("Vertical"));
        input.Jump |= Input.GetKeyDown(KeyCode.Space);
    }

    protected override void Simulate(PredictedCharacterControllerInput input, ref PredictedCharacterControllerState state, float delta)
    {
        bool groundedPlayer = controller.isGrounded;
        if (groundedPlayer && state.Velocity.y < 0)
        {
            state.Velocity.y = 0f;
        }

        // Read input
        Vector3 move = new Vector3(input.Movement.x, 0, input.Movement.z);
        move = Vector3.ClampMagnitude(move, 1f);

        // Jump
        if (input.Jump && groundedPlayer)
        {
            state.Velocity.y = Mathf.Sqrt(jumpHeight * -2.0f * gravity);
        }

        // Apply gravity
        state.Velocity.y += gravity * delta;

        // Combine horizontal and vertical movement
        Vector3 finalMove = (move * speed) + (state.Velocity.y * Vector3.up);
        controller.Move(finalMove * delta);
    }
}
```

## Creating the Player Prefab
1. Inside of Unity, Create a new Empty `GameObject` and name it **Player**. 
2. Add the `PredictedCharacterController` component we just created, and also add the `PredictedTransform` component mentioned earlier to handle the Position and Rotation state. 
3. Under the **Player**, Create a new `Capsule` and name it **Graphics**. This will be our graphical representation of our player. Make sure to remove the `CapsuleCollider` from the **Graphics** object, as having colliders on Graphical objects is not allowed. 
4. Finally, assign the **Graphics** GameObject to the `PredictedTransform` component's `Graphics` field.

## Testing
1. Create a new scene and add a plane for the player to walk on and create a `Camera` in the scene so we can see our player.
2. Create an empty `GameObject` and name it **NetworkManager**. Add the `NetworkManager` component to it.
3. Create an empty `GameObject`, and name it **Prediction Manager**. Add the `PredictionManager` component to it. Click on *New* under the `PredictedPrefabs` field to create a new `PredictedPrefabs` asset.
4. Add the `PredictedPlayerSpawner` component to the **Prediction Manager** and assign the **Player** prefab to the `Player Prefab` field.
5. Run the scene and confirm you can jump and move around!

## Conclusion
Congratulations! You (hopefully) have a capsule that can run around and jump! Fire up a client and use the [PurrTransport](../systems-and-modules/transports/purr-transport.md) and try it out with real world latency!

As long as you always modify your state inside `Simulate`, and gather your input inside `UpdateInput` (or `GetFinalInput`), you'll notice the workflow becomes very similar to writing singleplayer code.

If you have any questions, feel free to ask in the [Discord server](https://discord.gg/HnNKdkq9ta)!