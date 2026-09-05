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


## Logging

## Metadata

## Summary
