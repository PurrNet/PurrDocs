# Input Handling

Input handling in the Client-Side Prediction (CSP) system is straightforward and flexible. The primary requirement is to implement the `GetInput` method, which captures user input and returns it in a structured format (e.g., a struct). This input is then used to simulate the entity's behavior during prediction.

***

**Key Concepts**

1.  **Implementing `GetInput`**:

    * The `GetInput` method is where you define how user input is captured and structured.
    * It returns an `INPUT` struct (e.g., `SimpleWASDInput`) that contains all relevant input data for the current frame.

    **Example**:

    ```csharp
    protected override void GetFinalInput(ref Input input)
    {
        input.horizontal = Input.GetAxisRaw("Horizontal"),
        input.vertical = Input.GetAxisRaw("Vertical"),
        input.jump = Input.GetKey(KeyCode.Space),
        input.sprint = Input.GetKey(KeyCode.LeftShift)
    }
    ```

    * In this example, `SimpleWASDInput` is a struct that captures movement and action inputs.
2.  **Handling Key Down Events**:

    * For key-down events (e.g., pressing a key once), you need to cache the input during Unity's `Update` method and clear the cache after returning it in `GetInput`.
    * This ensures that key-down events are properly registered and not skipped during simulation.

    **Example**:

    ```csharp
    private bool _jumpKeyDown;

    private void Update()
    {
        // Cache key-down events in Unity's Update loop
        if (Input.GetKeyDown(KeyCode.Space))
            _jumpKeyDown = true;
    }

    protected override SimpleWASDInput GetFinalInput(SimpleWASDInput input)
    {
        input.horizontal = Input.GetAxisRaw("Horizontal"),
        input.vertical = Input.GetAxisRaw("Vertical"),
        input.jump = _jumpKeyDown,
        input.dash = Input.GetKey(KeyCode.LeftShift)

        // Clear the cached key-down event
        _jumpKeyDown = false;

        return input;
    }
    ```

    * Here, `_jumpKeyDown` is a cached variable that tracks whether the jump key was pressed during the current frame.

***

**Why This Works**

* **Flexibility**:
  * The `GetFinalInput` method allows you to define exactly how input is captured and structured, making it adaptable to different control schemes.
* **Key-Down Events**:
  * By caching key-down events in `Update` and clearing them in `GetFinalInput`, you ensure that these events are only processed once, preventing unintended behavior (e.g., multiple jumps from a single key press).
