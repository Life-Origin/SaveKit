# Advanced Features

This guide covers features and usage patterns that go beyond the basic SaveKit workflow.

If you are new to SaveKit, it is recommended to read [Getting Started.md] first. This guide assumes that you are already familiar with creating packages, defining sections, mapping data, and performing basic save and load operations.

For information about how these features are implemented internally, see [System Architecture.md].

---

## Multiple Sections

A SaveKit package can contain multiple sections.

Sections allow different types of data to be stored separately within the same package. For example, a visual novel may use separate sections for character data, inventory data, settings, and history.

A package might therefore contain sections such as:

```text
Game.savekit
├── Characters
├── Inventory
├── Settings
└── History
```

Each section can have its own schema and category.

This allows related data to remain organized without requiring a separate save file for every type of data.

### Example

The following example creates a package containing multiple sections:

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit")
    .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data).Build())
    .DefineSection(SectionBuilder.DefineSection("Inventory", SectionSchemas.Entry, SectionCategories.Collection).Build())
    .DefineSection(SectionBuilder.DefineSection("Settings", SectionSchemas.Field, SectionCategories.Config).Build())
    .DefineSection(SectionBuilder.DefineSection("History", SectionSchemas.Entry, SectionCategories.History).Build())
    .Build();
```

Multiple sections are particularly useful when a save contains several logically independent groups of data.

For a complete example, see [Examples/...].

---

## Multiple Packages

SaveKit does not require all data to be stored in a single package.

Multiple packages can be useful when different types of data have different lifetimes or purposes.

For example:

```text
Save1.savekit
    ├── Characters
    ├── Inventory
    └── History

Config.savekit
    └── Configurations

Logs.savekit
    └── Logs
```

A game might keep configuration data and logs separate from the main game state.

Using multiple packages can also make it easier to manage data that should be loaded or saved independently.

### Example

```csharp
SaveKit.InitializeWithDefault();

int CurrentSavePackage = 1001, ConfigurationsPackage = 1002, LogsPackage = 1003;

PackageBuilder.DefinePackage("Debug/Save1.savekit", ID: CurrentSavePackage)
    .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data).Build())
    .DefineSection(SectionBuilder.DefineSection("Inventory", SectionSchemas.Entry, SectionCategories.Collection).Build())
    .DefineSection(SectionBuilder.DefineSection("History", SectionSchemas.Entry, SectionCategories.History).Build())
    .Build();

PackageBuilder.DefinePackage("Debug/Configurations.savekit", ID: ConfigurationsPackage)
    .DefineSection(SectionBuilder.DefineSection("Settings", SectionSchemas.Field, SectionCategories.Config).Build())
    .Build();

PackageBuilder.DefinePackage("Debug/Logs.savekit", ID: LogsPackage)
    .DefineSection(SectionBuilder.DefineSection("Logs", SectionSchemas.Entry, SectionCategories.Log).Build())
    .Build();
```

When deciding whether to use one or multiple packages, consider whether the data belongs to the same logical save state and whether it needs to be loaded and saved together.

---

## Collections

Save data often contains collections rather than individual values.

For example, a game may have:

```text
Characters
    ├── Life
    ├── Nebula
    └── Another Character

Inventory
    ├── Potion
    ├── Sword
    └── Key
```

SaveKit can be used with collection-based data, allowing multiple values to be stored as part of the save state.

### Save Example

```csharp
public static class Debugger
{
    public static void Main()
    {
        Console.Clear();
        SaveKit.InitializeWithDefault();
        SaveKit.AutoRegister();

        int CurrentSavePackage = 1001;

        PackageBuilder.DefinePackage("Debug/Save1.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Inventory", SectionSchemas.Entry, SectionCategories.Collection).Build())
            .Build();
            
        Inventory[0] = new Item("Star Laser Summoner");
        Inventory[8] = new Item("Strawberry", 10);

        SaveKit.Save(CurrentSavePackage);
    }
    [MapCollection("Inventory")]
    public static Item[] Inventory = new Item[10];
}

public class Item
{
    public string Name;
    public int Count = 1;
    public Item(string Name) => this.Name = Name;
    public Item(string Name, int Count)
    {
        this.Name = Name;
        this.Count = Count;
    }
    public Item(){}
}
```
As a result, the contents of your file will be as follows.

```savekit
[SaveKit 1.0]
00D600000094"Section.Name": "Inventory";
"Section.Schema": "Entry";
"Section.Category": "Collection";
"Section.Checksum": "SHA256";
"Section.Checksum.Value": "699D82E27CD9F756A7EBBA74C938FFF8D6DB40B99DB9BD390F7D0605CBDA6B30";
{
    "Name": "Star Laser Summoner",
    "Count": 1
};
Null;
Null;
Null;
Null;
Null;
Null;
Null;
{
    "Name": "Strawberry",
    "Count": 10
};
Null;
```

### Load Example

```csharp
Console.Clear();
SaveKit.InitializeWithDefault();
SaveKit.AutoRegister();

int CurrentSavePackage = 1001;

SaveKit.LoadFile("Debug/Save1.savekit", ID: CurrentSavePackage);

for (int i = 0; i < Inventory.Length; i++)
{
    if (Inventory[i] == null) Console.WriteLine($"{i}: Null");
    else Console.WriteLine($"{i}: {Inventory[i].Name} x {Inventory[i].Count}");
}
```

As a result, the console output will be as follows.

```text
0: Star Laser Summoner x 1
1: Null
2: Null
3: Null
4: Null
5: Null
6: Null
7: Null
8: Strawberry x 10
9: Null
```

When using collections, consider whether the collection itself or its individual elements should be mapped.

For a complete collection example, see [Examples/Collection.md].

---

## History

Normal save data represents the current state of an application.

History can instead be used when previous states or changes need to be retained.

For example:

```text
Current State

Opinion = 20
     ↓
Opinion = 30
     ↓
Opinion = 50
```

A history system can be useful for tracking changes that occurred during a game session or application lifetime.

### Example

```csharp
Console.Clear();
SaveKit.InitializeWithDefault();
SaveKit.AutoRegister();

int CurrentSavePackage = 1001;

PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage).Build();

// You don't need to create a custom section for the History class.
// If a section with the specified name doesn't exist, it will create one automatically.

History history = new History("Player History", CurrentSavePackage);
history.Deserialize();

history.Push(HistoryEntryBuilder.Define("Give", "Nebula").SetDetail("Item", "Strawberry").SetDetail("For", "Gift").Build());
history.Push(HistoryEntryBuilder.Define("Talk", "Life").SetDetail("Type", "Compliment").SetDetail("Topic", "Hair.Color").Build());

history.Serialize();
SaveKit.Save(CurrentSavePackage);
```

History is especially useful when the application needs to inspect or restore previous changes rather than only the latest state.

For a complete example, see [Examples/Log and History.md].

---

## Metadata

Metadata provides a way to store additional information associated with a package.

Metadata provides a way to store additional information associated with a package.
Unlike the main save data, metadata can be used for information that describes, configures, or supplements the package.

Examples include:

```text
Language = English
Mode = Debug
Version = 1.0
```

Metadata is useful when this information should remain associated with the save package but does not belong to the main application state.
For example, it's suitable for storing achievements, counters, flags, and custom characters.

### Example

```csharp
public static class Debugger
{
    public static void Main()
    {
        Console.Clear();
        SaveKit.InitializeWithDefault();
        SaveKit.AutoRegister();

        int CurrentSavePackage = 1001;

        PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage).Build();

        // You don't need to create a custom section for the Meta class.
        // If a section with the specified name doesn't exist, it will create one automatically.

        Meta achievements = new Meta("Achievements", CurrentSavePackage);
        achievements.Deserialize();
        
        achievements["First Dead"] = true;
        achievements["God Core"] = false;
        achievements["The Goddess Strawberry's Love"] = false;
        achievements["Highest score"] = 12345;

        achievements.Serialize(true, false);
        
        Meta SavedCharacters = new Meta("Saved Characters", CurrentSavePackage);
        SavedCharacters.Deserialize();

        SavedCharacters["Nebula"] = new Character("Nebula", "Nebula Aura", 6, Color.Indigo, true);
        
        SavedCharacters.Serialize(true, false);

        SaveKit.Save(CurrentSavePackage);

        Console.WriteLine(achievements["Highest score", typeof(int)]);
        Console.WriteLine(SavedCharacters["Nebula", typeof(Character)]);
    }
}

public class Character
{
    public string Name;
    public string FullName;
    public int Age;
    public Color TextColor;
    public bool Female;
    public Character(string name, string fullName, int Age, Color color, bool Female)
    {
        Name = name;
        FullName = fullName;
        this.Age = Age;
        TextColor = color;
        this.Female = Female;
    }
    public Character(){}
    public override string ToString()
    {
        return $"{Name}\n Full Name: {FullName}\n Age: {Age}\nText Color: {TextColor}\n Female: {Female}";
    }
}
```

For a basic metadata example, see [Getting Started.md].

---

## Logging

SaveKit provides logging functionality through the `Logger` class.

Logs can contain information such as:

* The message
* The source of the message
* The log level
* The timestamp

SaveKit provides several predefined log levels and also supports custom levels.

A logging section can therefore contain structured entries such as:

```text
Info
Debug
Warning
Error
Trace
Fatal
Custom
```

### Example

```csharp
Console.Clear();
SaveKit.InitializeWithDefault();
SaveKit.AutoRegister();

int CurrentSavePackage = 1001;

PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage).Build();

// You don't need to create a custom section for the Logger class.
// If a section with the specified name doesn't exist, it will create one automatically.

Logger logger = new Logger("Logs", CurrentSavePackage);
logger.Deserialize();

logger.Info("Game", "Money +100");
logger.Info("Game", "Day 15");
logger.Warning("Game", "You don't have enough money.");
logger.Error("Game", "Not Implemented");
logger.Error("Game", "ArgumentException");
logger.Trace("Actions.cs", "Line 123");
logger.Debug("Relations", "Life.Opinion: 95");

logger.Serialize();

SaveKit.Save(CurrentSavePackage);
```

Logging can be useful during development, debugging, and when diagnosing problems in a released application.

For a complete logging example, see the logging example in [Getting Started.md] or [Examples/...].

---

## Section Types

A section is defined by a combination of a **Schema** and a **Category**.

Although schemas and categories are designed as independent concepts, SaveKit's default categories are associated with compatible schemas. This design reduces configuration complexity and helps prevent invalid section configurations.

When creating a section, you can choose a compatible combination of schema and category. These combinations can be considered Section Types.

For example:

```text
Section Type
├── Schema
└── Category
```

The available default Section Types and their intended use are described in the [Section Types.md] file.

For more information about how schemas and categories are implemented internally, see [Package Architecture.md].

---

# Transforms

SaveKit supports transformations that can be applied to serialized data before it is stored.

Transforms can provide functionality such as:

* Encoding
* Compression
* Encryption
* Integrity verification

These operations can be combined to create a processing pipeline appropriate for the application.

---

## Encoding

Encoding transforms data into another representation.

Encoding can be useful when data needs to be converted into a specific representation before being processed or stored.

### Example

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
            .SetEncoding("UTF8").Build())
            .Build();
```

---

## Compression

Compression reduces the amount of space required to store data.

This can be particularly useful for large save files or applications that generate a significant amount of save data.

### Example

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
            .SetCompression("Brotli").Build())
            .Build();
```

When choosing compression settings, consider the trade-off between file size and processing time.

---

## Encryption

Encryption protects the contents of a save file from being directly read.

This can be useful when the application stores data that should not be easily inspected or modified by users.

Encryption does not prevent all forms of cheating or tampering, especially when the encryption key is available to the application itself.

### Example

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
            .SetEncryption("Aes").Build())
            .Build();
```

Encryption should not be confused with integrity verification. Encryption protects confidentiality, while a checksum or other integrity mechanism can be used to detect modifications.

---

## Checksum

A checksum can be used to verify whether stored data has been modified or corrupted.

For example:

```text
Original Data
     ↓
Checksum
     ↓
Stored Data
```

When the data is loaded, the checksum can be used to detect changes to the protected data.

### Example

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
            .SetChecksum("SHA256").Build())
            .Build();
```

A checksum does not provide encryption or confidentiality.

---

## Combining Transforms

Multiple transformations can be combined into a single processing pipeline.

For example:

```text
Serialized Data
      ↓
Encoding
      ↓
Checksum
      ↓
Compression
      ↓
Encryption
      ↓
Storage
```

The order of transformations can affect the resulting data and the way it must be processed during loading.

The loading process applies the corresponding operations in the reverse order:

```text
Stored Data
      ↓
Decryption
      ↓
Decompression
      ↓
Checksum
      ↓
Decoding
      ↓
Deserialization
      ↓
Data
```

### Example

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
            .SetEncoding("UTF8")
            .SetCompression("Gzip")
            .SetEncryption("Aes")
            .SetChecksum("SHA256").Build())
            .Build();
```

The exact processing pipeline depends on the transforms configured for the package or section.

For an implementation-level explanation of the transformation pipeline, see [System Architecture.md].

---

## Data Formats

SaveKit currently supports the **Ask** data format.

Additional data formats, including **JSON** and **XML**, are planned for future releases. The `IDataFormat` abstraction allows these formats to be added without changing the rest of the save pipeline.

```text
Data Format
└── Ask
```

Future releases may provide:

```text
Data Format
├── Ask
├── JSON
└── XML
```

For information about the Ask format itself, see [Package Architecture.md].

For creating your own data format, see [Examples/How to Create Your Own Data Formats.md].

---

### Using a Data Format

```csharp
PackageBuilder.DefinePackage("Debug/Example.savekit", ID: CurrentSavePackage)
            .DefineSection(SectionBuilder.DefineSection("Characters", SectionSchemas.Field, SectionCategories.Data)
            .SetDataFormat("Ask")
            .Build())
            .Build();
```

For information about the Ask format itself, see [Package Architecture.md].

For creating your own data format, see [Examples/How to Create Your Own Data Formats.md].

---

# Custom Components

One of SaveKit's main design goals is extensibility.

Applications can provide their own components when the built-in functionality does not meet a particular requirement.

Possible extension points include:

```text
Custom Transform
Custom Data Format
Custom Storage
Custom Section Category
```

Each component should implement the appropriate SaveKit abstraction and then be registered with the system.

---

## Custom Transforms

A custom transform can be used when an application requires a transformation that is not provided by SaveKit.

Examples might include application-specific encoding, compression, or data processing.

```csharp
public interface IEncodingTransform
{
    ReadOnlyMemory<char> Decode(
        ReadOnlyMemory<byte> source);

    ReadOnlyMemory<byte> Encode(
        ReadOnlyMemory<char> source);
}
public interface ICompressionTransform
{
    public ReadOnlyMemory<byte> Compress(ReadOnlyMemory<byte> Input);
    public ReadOnlyMemory<byte> Decompress(ReadOnlyMemory<byte> Input);
}
public interface IEncryptionTransform
{
    public ReadOnlyMemory<byte> Encrypt(ReadOnlyMemory<byte> Input, ReadOnlyMemory<byte> Password);
    public ReadOnlyMemory<byte> Decrypt(ReadOnlyMemory<byte> Input, ReadOnlyMemory<byte> Password);
}
public interface IChecksumTransform
{
    public ReadOnlyMemory<byte> Compute(ReadOnlyMemory<byte> Input);
}
```

For a complete implementation, see [Examples/How to Create Your Own Transformations.md].

---

## Custom Data Formats

A custom data format can be created when the application needs to serialize data using a format that is not already supported.

```csharp
public interface IFormat
{
    public AstNode Deserialize(ReadOnlySpan<char> Text);
    public string Serialize(AstNode Root);
}
```

For a complete implementation, see [Examples/How to Create Your Own Data Formats.md].

---

## Custom Storage

SaveKit can be extended with custom storage implementations when the default storage behavior does not fit the application's requirements.

For example, an application could use a custom storage system for a specific platform or external storage mechanism.

```csharp
public interface IOTarget
{
    public Result Save(string Path, ReadOnlyMemory<byte> Bytes);
    public Result Load(string Path, out ReadOnlyMemory<byte> Bytes);
}
```

For a complete implementation, see [Examples/How to Create Your Own Storage.md].

---

## Custom Section Categories

Section categories can be extended when an application needs to distinguish a new type of section from the built-in categories.

For a complete implementation, see [Examples/How to Create Your Own Section Category.md].

---

# Development vs. Production

The appropriate SaveKit configuration can depend on whether the application is being developed or released.

During development, features such as detailed logging and easily inspectable save data can be useful.

For a production application, you may instead want to prioritize:

* Smaller save files
* Encryption
* Integrity verification
* Reduced logging
* Appropriate transformation pipelines

A possible development configuration might look like:

```text
Readable Data
     +
Detailed Logging
     +
Checksum
```

While a production configuration might use:

```text
Serialization
     ↓
Compression
     ↓
Encryption
     ↓
Checksum
     ↓
Storage
```

The appropriate configuration depends on the requirements of the application.

---

# Performance Considerations

Save systems can become more expensive as the amount of stored data increases.

When designing a large save system, consider:

* How frequently data is saved.
* How much data is serialized.
* The number and size of sections.
* The size of collections.
* Compression costs.
* Encryption costs.
* Logging volume.

Avoid saving data more frequently than necessary, especially when the amount of serialized data is large.

For performance-sensitive applications, measure the actual cost of serialization, transformations, and storage rather than assuming that one configuration will always be faster.

---

# Best Practices

When designing a SaveKit-based save system, consider the following practices.

### Keep Save Data Focused

Only store data that is necessary to reconstruct the application state.

Runtime-only or temporary values should not automatically become part of the save data.

### Organize Data into Sections

Use sections to separate logically different types of data.

For example:

```text
Characters
Inventory
Settings
History
Logs
```

### Separate Configuration from Game State

Configuration data and game state often have different purposes and lifetimes.

Keeping them separate can make the save system easier to manage.

### Use Transforms Purposefully

Do not enable every transform simply because it is available.

Choose transformations based on the requirements of the application.

### Keep Extensions Isolated

When creating custom SaveKit components, keep application-specific behavior inside the custom component instead of modifying unrelated parts of the save pipeline.

---

# Summary

SaveKit provides more than basic save and load operations. Its sections, collections, history, metadata, logging, transformations, data formats, and extension points allow a save system to be adapted to different application requirements.

The main concepts covered in this guide are:

```text
Data Organization
       ↓
Sections and Packages
       ↓
Serialization
       ↓
Transforms
       ↓
Storage
```

For implementation details, see [System Architecture.md].

For the physical structure of SaveKit packages and files, see [Package Architecture.md].

For practical implementations of individual features, see the files in the `Examples` directory.
