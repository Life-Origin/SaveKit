# Troubleshooting

This guide lists common issues that may occur when using SaveKit and their possible solutions.

## `System.MissingMethodException`

If a `System.MissingMethodException` is thrown when SaveKit tries to create or deserialize an object:

1. Make sure the class has a **parameterless constructor**.
2. If the class uses a custom constructor, add a parameterless constructor that SaveKit can use for deserialization.

Example:

```csharp
public class Character
{
    public string Name;

    public Character()
    {
    }

    public Character(string name)
    {
        Name = name;
    }
}
```

---

## An Object Remains `null`

If an object remains `null` after loading or retrieving data:

1. Make sure `SaveKit.AutoRegister();` is called during initialization.
2. Make sure the value was registered and saved correctly.
3. Make sure the requested key, field, or property exists in the saved data.
4. Make sure the object is being retrieved using the correct type.

---

## File Not Found

If SaveKit cannot find a save file:

1. Make sure `SaveKit.InitializeWithDefault();` is called before using SaveKit.
2. Make sure the specified file path is correct.
3. Make sure the file exists in the expected location.
4. If using custom storage, make sure the storage module is registered correctly.

---

## Save Operation Fails

If saving fails and the following message is returned:

```text
The given key '...' was not present in the dictionary.
```

check the following:

1. Make sure `SaveKit.InitializeWithDefault();` is called during initialization.
2. Make sure the required module or plugin is registered.
3. Make sure the name or identifier being requested is registered correctly.
4. If using a custom component, make sure it has been registered before saving.

---

## General Initialization Issues

Many SaveKit components depend on the SaveKit initialization and registration process.

A typical initialization sequence is:

```csharp
SaveKit.InitializeWithDefault();
SaveKit.AutoRegister();
```

Make sure these methods are called before creating or loading SaveKit components that depend on them.

---

## Still Having Problems?

If the problem persists:

1. Check the relevant documentation and examples.
2. Verify that the required components have been registered.
3. Check the exception message and stack trace.
4. Make sure you are using a compatible version of SaveKit.

If the issue appears to be a bug, report it through the SaveKit GitHub repository with the error message, stack trace, SaveKit version, and a minimal example that reproduces the issue.
