# Fully Local RPC

It's not uncommon that you might have functionality which you'd want to run fully locally without sending any data. This way an RPC method can be used both for networking, but also local logic.

Having the RPC call locally is simple, all you need to do, is flag your code to only run locally with the PurrCompilerFlags.

Here is an example below, with comments as to what code will send to the server and what will not:

```csharp
private void AnyMethod() 
{
    PurrCompilerFlags.EnterLocalExecution();
    // These 2 will ONLY run locally
    Test("Test 1");
    Test("Test 2");
    PurrCompilerFlags.ExitLocalExecution();

    // This will be sent to the server
    Test("Test 3");
}

[ServerRpc]
private void Test(string myText)
{
    Debug.Log(myText);
}
```
