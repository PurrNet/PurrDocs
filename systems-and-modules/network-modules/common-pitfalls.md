# Common Pitfalls

## Common Pitfalls

### 1. Modules Must Be Fields or Properties

Network modules **must** be declared as a field or property on a `NetworkIdentity`.\
**Do not** create them dynamically or store them in collections.

```csharp
public class MyNetworkObject : NetworkIdentity
{
    public SyncVar<int> health; // ✅ Correct
    private SyncList<string> names; // ✅ Correct

    // ❌ Incorrect: Do not use lists of modules
    public List<SyncVar<int>> invalidList; 
}
```

***

### 2. Editor Serialization

Modules can be serialized and shown in the Unity Editor.\
Just mark them with `[SerializeField]` or make them `public`.

```csharp
[SerializeField]
private SyncVar<int> health; // Will show up in the Inspector

public SyncList<string> names; // Also visible if public
```

***

### 3. Initialization: Use `OnInitializeModules()`

You can create, override, or initialize modules in the\
`protected virtual void OnInitializeModules()` callback.\
This runs **before** networking sends data or assigns IDs.

```csharp
protected override void OnInitializeModules()
{
    health = new SyncVar<int>(100);
    // Custom initialization here
}
```

***

### 4. Do Not Replace Modules After Spawn

**Once spawned, never assign a new module instance to a field/property.**\
Doing so will break networking or cause undefined behavior.\
Modules are meant to be **constant** after spawn.

```csharp
// ❌ Don't do this after spawn:
health = new SyncVar<int>(200); // Will break networking!
```

***

### 5. Nesting Modules

You can nest modules—meaning a network module can use other modules like `SyncVar` as fields or properties.

```csharp
public class MyModule : NetworkModule
{
    public SyncVar<int> score; // ✅ Allowed
}
```

**Do not create circular dependencies.**\
If Module A uses Module B, and Module B uses Module A, it will break.\
Nesting must form a tree, not a loop.

```csharp
// ❌ Not allowed: Circular reference
public class ModuleA : NetworkModule
{
    public ModuleB b;
}

public class ModuleB : NetworkModule
{
    public ModuleA a; // This creates a cycle and will cause compiler error
}
```

***

### 6. No Lists of Modules

You **cannot** create a list or array of modules.\
Modules must be declared as fields or properties at compile time.

```csharp
// ❌ Not allowed:
public List<SyncVar<int>> moduleList;

// ✅ Allowed:
public SyncVar<int> health;
public SyncVar<int> mana;
```

***

### 7. Module Events

Modules have similar events to `NetworkIdentity`, such as:

* `OnSpawned`
* `OnOwnerChanged`
* `OnDespawned`

You can use these for custom logic inside your modules.

```csharp
public class MyModule : NetworkModule
{
    protected override void OnSpawned() { /* ... */ }
    protected override void OnOwnerChanged(..) { /* ... */ }
    protected override void OnDespawned() { /* ... */ }
}
```
