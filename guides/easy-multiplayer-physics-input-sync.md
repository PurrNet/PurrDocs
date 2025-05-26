# Easy Multiplayer Physics (input Sync)

The easiest way (in my humble opinion) to add physics interactions in multiplayer, is to use Input Synchronizing. The most popular alternative is Client Side Prediction (CSP). Both have their pros and cons.

{% embed url="https://youtu.be/Nvc8CaAEJis" %}
Guide going over implementing Input Synchronization with PurrNet
{% endembed %}

### Input Sync VS Client Side Prediction

#### Client Side Prediction

✔️ Instant response for players\
✔️ Easy to cheat proof\
❌ Difficult logic and hard to work with

#### Input Sync

✔️ Easy workflow (nearly as easy as single player code)\
✔️ Easy to cheat proof\
❌ Sensitive to ping

Keep in mind, that even though it won't actually be as instant as something running locally, you can still [smoke and mirror](https://dictionary.cambridge.org/dictionary/english/smoke-and-mirrors) it to make it feel instant locally, by polishing to your game. This can be done with animations handled locally for example.

### Why can't we just do physics for each client?

If the physics in your game does not interact between players, then you can do that just fine. However, if you want interactions between players, there has to be a single point of truth.

This is why various techniques has to be used in order to properly network physics interactions. They all come with their pros and cons, and in the end, it's all about you picking something suitable for your game!

### How does it work

The idea and execution is very simple. Essentially it's fully a server auth simulation, that is conveyed to clients using the [Network Transform](../plug-n-play-components/network-transform.md).

Essentially: The client sends only it's input and intentions to the server, and the server does the actual action locally, and thus the server becomes the single point of truth for the whole game. This is what makes it cheat proof, and also able to handle physics interactions (because they are all simulated in one place)

### Simple Example

This is essentially the example code shown from the video, with comments added to explain what is going on.

```csharp
[SerializeField] private float moveForce = 10f;
[SerializeField] private float jumpForce = 10f;
[SerializeField] private float bounceForce = 10f;
[SerializeField] private Rigidbody rigidbody;
private bool _willJump;

protected override void OnSpawned(bool asServer)
{
    base.OnSpawned(asServer);
    if (asServer)
        return;

    //All clients set it to kinematic, so only the server runs physics!
    rigidbody.isKinematic = !isServer;
    //Only the owner has it enabled, as to run Update()
    enabled = isOwner;

    //Only the owner runs OnTick to send input to the server
    if (isOwner)
        networkManager.onTick += OnTick;
}

protected override void OnDestroy()
{
    base.OnDestroy();
    
    //Unsubcribing again for cleanup
    networkManager.onTick -= OnTick;
}

private void Update()
{
    //We have to store the input to be used during the next tick
    if (Input.GetKeyDown(KeyCode.Space))
        _willJump = true;
}

private void OnTick(bool asServer)
{
    //In case of a host setup, we don't want this to run twice.
    if (asServer)
        return;

    //We generate the input struct that will be sent to the server
    var input = new InputData()
    {
        movement = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical")),
        jump = _willJump
    };

    //Restting the jump bool back after we've now used it in a tick
    _willJump = false;
    
    //We send the input to the server
    Move(input);
}

//Server RPC with Unreliable channel to send the data more efficiently
[ServerRpc(Channel.Unreliable)]
private void Move(InputData inputData)
{
    //This is where you can also handle cheat detection on the inputData
    //You could for example normalize it, if the magnitude is above 1
    //From here the code is basically "single-player" code from the 
    //perspective of the server
    
    //We generate the movement vector from the given input.
    var movement = new Vector3(inputData.movement.x, 0, inputData.movement.y) * moveForce;
    
    rigidbody.AddForce(movement);
    
    if(inputData.jump)
        rigidbody.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
}

private void OnCollisionEnter(Collision other)
{
    //Other than the if-statement here, this is single-player code from the
    //perspective of the server
    if (!isServer)
        return;

    if (!other.gameObject.TryGetComponent(out PlayerPhysicsMovement otherPlayer))
        return;
    
    var direction = (transform.position - other.transform.position).normalized;
    rigidbody.AddForce(direction * bounceForce, ForceMode.Impulse);
}

//Struct in which we hold input data. This isn't necessary, just a clean approach
private struct InputData
{
    public Vector2 movement;
    public bool jump;
}
```

