# Collider Rollback

## What is collider rollback?

Imagine you're playing catch with a friend over the internet, but there's a small delay between when you throw the ball and when your friend sees it. Sometimes it looks like you missed them even though on your screen it was a perfect throw!

Collider rollback is like having a time machine that quickly goes back to check "Wait, where was my friend ACTUALLY standing when the ball was thrown?" This works best when there's a referee (the server) controlling where everyone can move - because then the referee knows exactly where everyone really was!

But if everyone can move wherever they want without checking with the referee first (client-sided movement), using this time machine doesn't make much sense - because we already trust what each player sees on their screen anyway!

So rollback is mostly useful in games where the server is in charge of all the movement, like many fighting games or competitive shooters.

You can think of collider rollback as server validating hits accounting for your ping.

## Setting up colliders for rollback

Add `ColliderRollback` component and you should be about done.\
You have two options from here, either leave it to it's default settings (auto add all children including the current game object) or you can manually specify which colliders to add.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption><p>Default Settings</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Custom collider list</p></figcaption></figure>

This is enough to start recording the history of your colliders.

## Validating hits

### From the client side

On the client side you need to send the time at which you acted, this is made available to `NetworkIdentity` via `rollbackTick` . But you can get it from the `NetworkManager` too via :

```csharp
networkManager.tickModule.rollbackTick
```

So here is an example of shooting a raycast on the client's end and sending it to the server for validation:

```csharp
var ray = _camera.ScreenPointToRay(Input.mousePosition);

if (Physics.Raycast(ray, out var hit))
{
    // react locally with some hitmark or sound, blood vfx etc
    // we should let server do actual killing/damaging
}

ShootOnServer(rollbackTick, ray);
```

Locally (aka on client side) you do raycasts as you normally would and then simply forward that to the server. In this scenario `ShootOnServer` is a `ServerRpc` .

### From the server side

Let's validate this hit on the server now! This will require re-shooting the same ray accounting for the time difference like this:

```csharp
[ServerRpc(requireOwnership: false)]
private void Shoot(double preciseTick, Ray ray)
{
    if (rollbackModule.Raycast(preciseTick, ray, out var hit))
    {
        // handle hit by damaging the player or something
    }
}
```

Notice how we don't use `Physics.Raycast` , this is because we should do the ray-casting through our module to  properly account for state.

Unlike other systems we don't physically move the colliders because this can trigger unwanted physics events leading to weird bugs if unaccounted for. Instead we bend the ray on a per collider basis and solve the hits that way so you don't have to worry about colliders moving about.
