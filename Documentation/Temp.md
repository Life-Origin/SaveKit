# Getting Started

This guide covers the general usage of SaveKit. Only basic features and simple use cases are covered here.

For an overview of the system architecture, see [System Architecture.md].
For the file structure, see [Package Architecture.md].
For advanced features and detailed usage, see [Advanced Features.md].
To create your own components, see [Plugins.md].
You can refer to the [Troubleshooting.md] file to resolve the issues.

---

## Installation

Add SaveKit to your project and make sure the required SaveKit assemblies are available.

For Unity projects, import the SaveKit package into your project and allow Unity to compile the assemblies.

After installation, you can start using the SaveKit API from your C# scripts.

```csharp
using Savekit;
```

> The exact namespace may differ depending on the SaveKit version and package configuration.

---

## Your First Save

Let's start with a simple scenario.

Our goal is to save the states of two characters in a visual novel.

First, let's define what information we want to store for each character.

For this example, we will use the following class:

```csharp
public class Character
{
    public string Name;
    public string FullName;
    public int Opinion;
    public Color TextColor;
    public bool RuntimeValue;
    public Character(string name, string fullName, int opinion, Color color, bool runtime)
    {
        Name = name;
        FullName = fullName;
        Opinion = opinion;
        TextColor = color;
        RuntimeValue = runtime;
    }
    public Character(){}
}
```

Now, let's create the two characters that we want to save.

We will store them in static fields. The reason for using static fields will become clear later in this guide.

**Warning**: To avoid receiving a System.MissingMethodException error during loading, be sure to add parameterless constructor methods to the classes you will be using.

```csharp
public static Character Life;
public static Character Nebula;

public static void Main()
{
    Life = new Character("Life", "Life Aura", 20, Color.Aquamarine, true);
    Nebula = new Character("Nebula", "Nebula Aura", 50, Color.Indigo, false);
}
```

The values we want to save are now ready.

However, SaveKit needs to know which values should be included in the save data. We can define this using the `MapData` attribute.

For now, ignore the attribute parameters. They will be explained later in this guide.

```csharp
[MapData("Characters", "Life")]
public static Character Life;

[MapData("Characters", "Nebula")]
public static Character Nebula;
```

Now that our characters have been mapped, we can create a save package and save them.

For this example, we will use the `Main` method. The details of package and section configuration are not important yet and will be explained later.

```csharp
public static void Main()
{
    Console.Clear();
    SaveKit.InitializeWithDefault();
    SaveKit.AutoRegister();

    // The file path might cause an error on Windows devices.
    // You can change it to "Debug\\Example.savekit".
    int SavePackageID = PackageBuilder.DefinePackage("Debug/Example.savekit")
        .DefineSection(
            SectionBuilder
                .DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
                .Build()
        )
        .Build();

    Life = new Character(
        "Life",
        "Life Aura",
        20,
        Color.Aquamarine,
        true
    );

    Nebula = new Character(
        "Nebula",
        "Nebula Aura",
        50,
        Color.Indigo,
        false
    );

    SaveKit.Save(SavePackageID);
}
```

When this code is executed, SaveKit creates a file containing the serialized data.

The resulting file will look similar to this:

```savekit
[SaveKit 1.0]
00D100000130"Section.Name": "Characters";
"Section.Schema": "Field";
"Section.Category": "Data";
"Section.Checksum": "SHA256";
"Section.Checksum.Value": "D3EE243330F9469742D80977DCA79AB0D2A3F452860DCF641C2853AF287FE3CA";
"Life": {
    "Name": "Life",
    "FullName": "Life Aura",
    "Opinion": 20,
    "TextColor": [Rgb] 0xFFFFFFFFFF7FFFD4,
    "RuntimeValue": True
};
"Nebula": {
    "Name": "Nebula",
    "FullName": "Nebula Aura",
    "Opinion": 50,
    "TextColor": [Rgb] 0xFFFFFFFFFF4B0082,
    "RuntimeValue": False
};
```

If you look carefully at the serialized data, you can see the values we wanted to save.

For example, the `Life` object contains:

```text
Name         = "Life"
FullName     = "Life Aura"
Opinion      = 20
TextColor    = [Rgb] 0xFFFFFFFFFF7FFFD4
RuntimeValue = True
```

and the `Nebula` object contains its corresponding values.

This is the basic SaveKit workflow:

```text
Define Data
    ↓
Map Data
    ↓
Create Package
    ↓
Register Data
    ↓
Save
```

In the following sections, we will look at how to load this data and how to work with more complex save scenarios.

---

## Loading Your Data

Now, let's see how to load the values we've saved. All you need is the following code.

```csharp
public static void Main()
{
    Console.Clear();
    SaveKit.InitializeWithDefault();
    SaveKit.AutoRegister();

    SaveKit.LoadFile("Debug/Example.savekit");
    
    Console.WriteLine(Life.FullName);
}
```

The output should be:

```text
Life Aura
```
---

## Detailed Explanation

In the previous sections, we created and loaded a simple save file. This section explains what each part of the example does and why it is required.

### Initializing SaveKit

Before using SaveKit, it must be initialized.

```csharp
SaveKit.InitializeWithDefault();
```

`InitializeWithDefault()` initializes SaveKit using its default configuration.

This should be done before performing save or load operations.

---

### Registering Data

After initializing SaveKit, we register the data used by our save system.

```csharp
SaveKit.AutoRegister();
```

`AutoRegister()` automatically registers the types and mapped data required by SaveKit.

This allows SaveKit to discover and process the data defined with `MapData`.

---

### Mapping Data

SaveKit needs to know which values should be included in the save data.

We define this using the `MapData` attribute:

```csharp
[MapData("Characters", "Life")]
public static Character Life;
```

The first parameter specifies the section where the data will be stored.

The second parameter specifies the name of the mapped value.

In this example, the `Life` object is stored in the `Characters` section under the name `Life`.

The same applies to the `Nebula` object:

```csharp
[MapData("Characters", "Nebula")]
public static Character Nebula;
```

As a result, both characters are mapped to the same section.

---

### Creating a Package

Before saving data, we need to define a save package.

```csharp
int savePackageId = PackageBuilder
    .DefinePackage("Debug/Example.savekit")
```

`DefinePackage()` creates a package definition and specifies the file where the package will be stored.

The returned package builder allows us to configure the package before it is built.

---

### Defining a Section

The package needs to contain a section for our character data.

```csharp
.DefineSection(
    SectionBuilder
        .DefineSection(
            "Characters",
            SectionSchemas.Field,
            SectionCategories.Data
        )
        .Build()
)
```

Here, we define a section named `Characters`.

`SectionSchemas.Field` specifies the schema used by the section, while `SectionCategories.Data` specifies its category.

`Build()` creates the section definition, which is then added to the package by `DefineSection()`.

A package can contain multiple sections, allowing different types of data to be organized separately.

---

### Building the Package

After configuring the package, we build it:

```csharp
.Build();
```

This completes the package definition and returns its identifier.

The identifier is stored in `savePackageId`:

```csharp
int savePackageId = PackageBuilder
    .DefinePackage("Debug/Example.savekit")
    ...
    .Build();
```

This identifier is later passed to `SaveKit.Save()`.

---

### Saving the Data

Once the package has been created and the data has been registered, we can save the data:

```csharp
SaveKit.Save(savePackageId);
```

SaveKit uses the package definition to determine where and how the mapped data should be stored.

The resulting file contains the section information and the serialized values of the mapped objects.

---

### Loading the Data

Loading the data is simpler because the package does not need to be defined again.

```csharp
SaveKit.LoadFile("Debug/Example.savekit");
```

`LoadFile()` reads the specified save file and restores the mapped values.

After loading, the `Life` field contains the data stored in the file:

```csharp
Console.WriteLine(Life.FullName);
```

The output is:

```text
Life Aura
```

This demonstrates that the serialized object was successfully reconstructed and assigned to the mapped field.

---

### Why Parameterless Constructors Are Required

The `Character` class contains two constructors:

```csharp
public Character(
    string name,
    string fullName,
    int opinion,
    Color color,
    bool runtime
)
{
    Name = name;
    FullName = fullName;
    Opinion = opinion;
    TextColor = color;
    RuntimeValue = runtime;
}

public Character() { }
```

The parameterless constructor is required when SaveKit creates an instance of the class during loading.

Without it, loading the serialized object may result in a `System.MissingMethodException`.

For classes used with SaveKit, make sure an accessible parameterless constructor is available when required by the deserialization process.

---

### Why Static Fields Are Used

The example uses static fields for the mapped objects:

```csharp
public static Character Life;
public static Character Nebula;
```

This allows the example to access the same mapped fields from both the save and load operations without creating another instance of the containing class.

The use of static fields is only for simplifying this example. SaveKit's data mapping system can be used in more complex application structures as well.

---

### What Happens During a Save?

At a high level, the save operation follows this sequence:

```text
Mapped Data
    ↓
Data Registration
    ↓
Package Definition
    ↓
Section Selection
    ↓
Serialization
    ↓
Transformations
    ↓
File Output
```

The exact internal processing is covered in [System Architecture.md].

---

### What Happens During a Load?

Loading follows the reverse direction:

```text
File Input
    ↓
Transformations
    ↓
Deserialization
    ↓
Data Reconstruction
    ↓
Mapped Data
```

The complete loading pipeline and the individual components involved are explained in [System Architecture.md].

---

## Logging

SaveKit provides a `Logger` class for creating and storing log entries inside a save package.

A logger can create entries with different log levels, such as `Info`, `Debug`, `Warning`, `Error`, `Trace`, and `Fatal`. Custom log levels can also be created when needed.

### Creating Logs

First, initialize SaveKit and create a package for the logs.

```csharp
public static void Main()
{
    Console.Clear();
    SaveKit.InitializeWithDefault();

    int logPackage = 1001;

    PackageBuilder
        .DefinePackage("Debug/Example2.savekit", ID: logPackage)
        .Build();

    Logger logger = new Logger("Logs", logPackage);

    logger.Info("Characters", "Created Character: Life");
    logger.Debug("Characters", "FullName: Life Aura");
    logger.Warning("Characters", "Nebula.FullName is Empty");
    logger.Error("Characters", "The Character Nebula could not be created.");
    logger.Trace("Characters.cs", "Line: 100");
    logger.Fatal("Characters", "Nebula Not Found");
    logger.Custom("System", "Characters module is being restarted.", "Restart");

    logger.Serialize();

    SaveKit.Save(logPackage);
}
```

The first parameter of the `Logger` constructor specifies the name of the section where the logs will be stored. The second parameter specifies the package that contains the section.

```csharp
Logger logger = new Logger("Logs", logPackage);
```

The `Logger` class provides several methods for creating log entries:

```text
Info
Debug
Warning
Error
Trace
Fatal
Custom
```

Each method receives a source and a message. `Custom()` additionally receives the name of the custom log level.

After creating the entries, `Serialize()` converts the logger's entries into the format used by the corresponding section.

Finally, `SaveKit.Save()` writes the package to the specified file.

### Log Data

The resulting section contains each log entry as a separate object.

```savekit
[SaveKit 1.0]
00CA00000401"Section.Name": "Logs";
"Section.Schema": "Entry";
"Section.Category": "Log";
"Section.Checksum": "SHA256";
"Section.Checksum.Value": "1AD4378735BECA238C8162A7E4E03FFA1EF3035C88F19AFF466BB643CCA2D1BB";
{
    "Message": "Created Character: Life",
    "Timestamp": "2026-09-06T06:24:14.2240492Z",
    "Level": "Info",
    "Source": "Characters"
};
{
    "Message": "FullName: Life Aura",
    "Timestamp": "2026-09-06T06:24:14.2240918Z",
    "Level": "Debug",
    "Source": "Characters"
};
{
    "Message": "Nebula.FullName is Empty",
    "Timestamp": "2026-09-06T06:24:14.2241299Z",
    "Level": "Warning",
    "Source": "Characters"
};
{
    "Message": "The Character Nebula could not be created.",
    "Timestamp": "2026-09-06T06:24:14.2241671Z",
    "Level": "Error",
    "Source": "Characters"
};
{
    "Message": "Line: 100",
    "Timestamp": "2026-09-06T06:24:14.2242041Z",
    "Level": "Trace",
    "Source": "Characters.cs"
};
{
    "Message": "Nebula Not Found",
    "Timestamp": "2026-09-06T06:24:14.2242427Z",
    "Level": "Fatal",
    "Source": "Characters"
};
{
    "Message": "Characters module is being restarted.",
    "Timestamp": "2026-09-06T06:24:14.2242964Z",
    "Level": "Restart",
    "Source": "System"
};
```

Each entry contains four pieces of information:

* `Message` — the log message.
* `Timestamp` — the time at which the entry was created.
* `Level` — the log level.
* `Source` — the source associated with the entry.

### Loading Logs

Previously saved logs can be loaded in the same way as other SaveKit data.

```csharp
public static void Main()
{
    Console.Clear();
    SaveKit.InitializeWithDefault();

    int logPackage = 1001;

    SaveKit.LoadFile("Debug/Example2.savekit", ID: logPackage);

    Logger logger = new Logger("Logs", logPackage);
    logger.Deserialize();

    List<string> logs = logger.GetStrings();

    for (int i = 0; i < logs.Count; i++)
        Console.WriteLine(logs[i]);
}
```

After loading the package, a `Logger` is created for the same section and package.

`Deserialize()` reconstructs the logger entries from the saved section.

`GetStrings()` returns the log entries as formatted strings, which can then be displayed or processed by the application.

The output is:

```text
6.09.2026 06:24:14: Info from Characters = Created Character: Life
6.09.2026 06:24:14: Debug from Characters = FullName: Life Aura
6.09.2026 06:24:14: Warning from Characters = Nebula.FullName is Empty
6.09.2026 06:24:14: Error from Characters = The Character Nebula could not be created.
6.09.2026 06:24:14: Trace from Characters.cs = Line: 100
6.09.2026 06:24:14: Fatal from Characters = Nebula Not Found
6.09.2026 06:24:14: Restart from System = Characters module is being restarted.
```

This demonstrates the complete logging workflow:

```text
Create Log Entries
        ↓
Serialize
        ↓
Save Package
        ↓
Load Package
        ↓
Deserialize
        ↓
Get Log Strings
```
---

## Metadata

SaveKit provides the `Meta` class for storing additional information associated with a package.

Metadata can be used for values such as configuration settings, language selection, game mode, version information, or other small pieces of information that do not belong to the main save data.

### Writing Metadata

Metadata can be loaded from an existing package, modified, and saved again.

```csharp
public static void Main()
{
    Console.Clear();
    SaveKit.InitializeWithDefault();

    int logPackageId = 1001;

    SaveKit.LoadFile("Debug/Example2.savekit", ID: logPackageId);

    Meta meta = new Meta("Configurations", logPackageId);
    meta.Deserialize();
    // Deserialize() also retrieves previously saved metadata, if any exists.

    meta["Language"] = "English";
    meta["Mode"] = "Debug";
    meta["PlayTime"] = "0";

    meta.Serialize();

    SaveKit.Save(logPackageId);
}
```

The `Meta` constructor takes the name of the metadata section and the ID of the package containing that section.

```csharp
Meta meta = new Meta("Configurations", logPackageId);
```

Metadata can be accessed using a key:

```csharp
meta["Language"] = "English";
meta["Mode"] = "Debug";
meta["PlayTime"] = "0";
```

If the package already contains metadata, calling `Deserialize()` retrieves the existing records before they are modified.

After modifying the metadata, `Serialize()` prepares the metadata for saving, and `SaveKit.Save()` writes the package to the file.

### Reading Metadata

Previously saved metadata can be read using the same key-based syntax.

```csharp
public static void Main()
{
    Console.Clear();
    SaveKit.InitializeWithDefault();

    int logPackageId = 1001;

    SaveKit.LoadFile("Debug/Example2.savekit", ID: logPackageId);

    Meta meta = new Meta("Configurations", logPackageId);
    meta.Deserialize();

    Console.WriteLine(meta["Language"]);
    Console.WriteLine(meta["Mode"]);

    meta.Serialize();

    SaveKit.Save(logPackageId);
}
```

The output is:

```text
English
Debug
```

This allows metadata to be used as persistent package-level information without mixing it with the main save data.

For example, a game could use metadata to store information such as:

```text
Language = English
Mode     = Debug
PlayTime = 0
```

Metadata is particularly useful when this information needs to remain associated with a save package but does not need to be part of the primary game state.

---

## Summary

In this guide, we covered the basic workflow of SaveKit and created a simple save system from start to finish.

We learned how to:

* Initialize SaveKit with the default configuration.
* Map data using the `MapData` attribute.
* Create packages and sections.
* Register mapped data.
* Save data to a SaveKit file.
* Load previously saved data.
* Use `Logger` to create and store log entries.
* Use `Meta` to store additional package information.
* Serialize and deserialize data before saving and loading.

The basic save workflow can be summarized as:

```text
Define Data
    ↓
Map Data
    ↓
Register Data
    ↓
Create Package
    ↓
Save
```

Loading follows the opposite direction:

```text
Load File
    ↓
Deserialize
    ↓
Restore Data
```

These examples demonstrate the basic concepts needed to start using SaveKit. More advanced features, internal architecture, and customization options are covered in the other documentation sections.

For a deeper understanding of how these components work together, see [System Architecture.md].

For information about the structure of SaveKit files and packages, see [Package Architecture.md].

