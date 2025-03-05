# Direct Local Execution of RPCs

PurrNet allows you to execute RPCs directly in the local context without sending them over the network. This can be useful when you want to bypass network transmission and handle the logic locally instead.

## **Scoped Local Execution with Compiler Flags**

To execute an RPC locally within a specific part of the method, use `PurrCompilerFlags.EnterLocalExecution()` and `PurrCompilerFlags.ExitLocalExecution()`.

```csharp
[ObserversRpc]
public void PerformAction()
{
    PurrCompilerFlags.EnterLocalExecution();

    // Any RPCs called here will bypass purrnet
    // They will be locally without sending it over the network

    PurrCompilerFlags.ExitLocalExecution();
    
    // Here we are back to sending RPCs over the network
}
```

* **What It Does**: Only the code between the `EnterLocalExecution` and `ExitLocalExecution` calls runs locally. The rest of the method behaves as a normal RPC.

## **Full Method Local Execution with LocalMode**

To make the entire method execute locally, use the `LocalMode` attribute.

```csharp
[ObserversRpc, LocalMode]
public void PerformAction()
{
    Debug.Log("This entire method runs locally without sending an RPC.");
}
```

* **What It Does**: The method runs locally and does not send any data over the network.

## Use Cases

* **Scoped Local Execution**: Use this if only a specific part of the method needs to bypass the network.
* **Full Method Local Execution**: Use this when the entire method should be local.

With these options, you can handle local execution of RPCs efficiently and flexibly.

## Disclaimer

These are compiler hints used by PurrNet to determine how the code should behave. They:

1. Do **not** execute or alter any logic directly.
2. Are scoped **only to the current function**. They do not propagate to other function calls.
3. Do **not** track the logical flow of the code. For instance, you can start local execution inside an `if` statement and end it outside the `if` block. In such cases, every instruction between `EnterLocalExecution` and `ExitLocalExecution` will still be processed, even if the `if` condition would not have been met.

Use these tools with care to ensure proper execution and avoid unintended behavior.
