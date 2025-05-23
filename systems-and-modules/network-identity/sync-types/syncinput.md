# SyncInput

The SyncInput sync types is good for owner authorized input synchronization with server authorized gameplay. This might sounds advanced, but is actually very simple!

The idea is: The server is responsible for all gameplay, and simply the necessary input is sent from client to server and from there the server will use it.

This is easy enough to do by utilizing RPC's, but can become non-performant quite easily by utilizing too many unnecessary input calls and too much data utilized. So PurrNet luckily provides a super easy to use module for this!

### Why utilize Input Synchronization?

There are really only a few good ways of going about player on player interactions in Multiplayer. \
1\. [Client Side Prediction](../../client-side-prediction/) - Difficult to work with and understand, but most responsive outcome\
2\. Input Synchronization - Easy to work with, but ping = input delay

Essentially the big strength of Input Synchronization is the workflow. It's extremely easy to understand and get working, given that all the game logic just has to run on the server and be conveyed to the clients. That's the whole idea. This is utilized in games such as:

* Gang Beasts
* Totally Accurate Battle Simulator (TABS) Multiplayer
* Human: Fall Flat
* Stick Fight: The Game

### The type

`SyncInput<T>` takes any unmanaged or IEquatable type, meaning that you can not only use simple types, but also custom structs or classes as long as they can be compared with the equatable setup.

### Simple usage example

```csharp
[SerializeField] private SyncInput<Vector2> _input = new();

private void Awake()
{
    //Subscribe to the input change event
    _input.onChanged += OnInputChanged;
}

private void OnDestroy()
{
    //Unsubscribe again
    _input.onChanged -= OnInputChanged;
}

private void OnInputChanged(Vector2 newInput)
{
    //Only the server will receive this event
    //The event will be called whenever input changes
    Debug.Log($"New input: {newInput}");
}

private void Update()
{
    if (!isOwner)
        return;
    
    //The owner will consistently send input - Only the necessary input changes will be sent
    //Rest is automatically filtered by the SyncInput
    var input = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));
    _input.value = input;
}
```

### Movement example:

This setup requires a Rigidbody, a collider and a network transform component with owner auth toggled off. This will give you players that can collider cleanly using physics!

```csharp
[SerializeField] private Rigidbody _rigidbody;
[SerializeField] private float _moveSpeed = 5f;
[SerializeField] private float _jumpForce = 5f;

[SerializeField] private SyncInput<Vector2> _moveInput = new();
[SerializeField] private SyncInput<bool> _jumpInput = new();

private bool _jump;

private void Awake()
{

    _jumpInput.onChanged += OnJump;
    _jumpInput.onSentData += OnSentData;
}

private void OnDestroy()
{
    _jumpInput.onChanged -= OnJump;
    _jumpInput.onSentData -= OnSentData;
}

private void OnSentData()
{
    _jump = false;
}

private void OnJump(bool newInput)
{
    _rigidbody.AddForce(Vector3.up * _jumpForce, ForceMode.Impulse);
}

private void Update()
{
    if (!isOwner)
        return;
    
    _moveInput.value = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));
    
    if(!_jump)
        _jump = Input.GetKeyDown(KeyCode.Space);
    _jumpInput.value = _jump;
}

private void FixedUpdate()
{
    if (!isServer)
        return;
    
    Vector3 move = new Vector3(_moveInput.value.x, 0, _moveInput.value.y).normalized;
    _rigidbody.AddForce(move * _moveSpeed);
}
```
