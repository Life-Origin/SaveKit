# Getting Started

This guide will help you get started with SaveKit and create your first save and load operations.

You don't need to understand SaveKit's internal architecture to follow this guide. Once you have a working save system, you can explore the advanced features and customize the pipeline to fit your project.

---

## 1. Installation

Add SaveKit to your project and make sure the required SaveKit assemblies are available.

For Unity projects, import the SaveKit package into your project and allow Unity to compile the assemblies.

After installation, you can start using the SaveKit API from your C# scripts.

```csharp
using SaveKit;
```

> The exact namespace may differ depending on the SaveKit version and package configuration.

---

## 2. Your First Save

A save operation starts with your application data.

For example, a game might need to store:

```csharp
public class PlayerData
{
    public string Name;
    public int Level;
    public float Health;
}
```

Create your data and pass it to SaveKit's save pipeline.

```csharp
var player = new PlayerData
{
    Name = "Player",
    Level = 10,
    Health = 85.5f
};

// Save the data using SaveKit.
```

The exact save API depends on the SaveKit configuration you are using. See the API documentation for the available save methods and options.

---

## 3. Loading Your Data

Loading follows the reverse process.

```text
Save
Data
 ↓
Serialization
 ↓
Transforms
 ↓
File
```

When loading:

```text
File
 ↓
Transforms
 ↓
Deserialization
 ↓
Data
```

This allows the same data pipeline to be used consistently for both saving and loading.

---

## 4. Working With Multiple Values

SaveKit is designed to store structured data rather than limiting a save to a single object.

For example, a save can contain several sections:

```text
Save
├── Player
│   ├── Name
│   ├── Level
│   └── Health
│
├── Characters
│   ├── Character_01
│   └── Character_02
│
└── Logs
    ├── Entry_01
    └── Entry_02
```

This makes it possible to organize data according to the needs of your application.

---

## 5. Understanding the Save Pipeline

SaveKit separates the different responsibilities involved in saving data.

A typical pipeline can be represented as:

```text
Data
  ↓
Organization
  ↓
Serialization
  ↓
Encoding
  ↓
Compression
  ↓
Encryption
  ↓
Checksum
  ↓
File
```

Not every project needs every stage.

For example, a simple development save might only require:

```text
Data
 ↓
Serialization
 ↓
File
```

A production save could use:

```text
Data
 ↓
Organization
 ↓
Serialization
 ↓
Compression
 ↓
Encryption
 ↓
Checksum
 ↓
File
```

Each stage can be configured independently where supported.

---

## 6. Compression

Compression can reduce the size of save files.

For example, SaveKit supports compression transforms such as Brotli.

A compressed pipeline can be configured as:

```text
Data
 ↓
Serialization
 ↓
Compression
 ↓
File
```

Compression is useful when save files contain large amounts of structured data.

---

## 7. Encryption

Encryption can be added when save data should not be directly readable.

For example:

```text
Data
 ↓
Serialization
 ↓
Encryption
 ↓
File
```

SaveKit supports encryption as a separate transformation stage, allowing it to be combined with other transformations.

> Encryption should not be treated as a replacement for authentication or server-side security. Choose the appropriate security model for your game.

---

## 8. Checksum and Data Integrity

A checksum can be used to detect corrupted or modified data.

For example:

```text
Data
 ↓
Serialization
 ↓
Checksum
 ↓
File
```

When the file is loaded, the checksum can be verified.

If the calculated checksum does not match the stored checksum, the data should be considered invalid.

This is particularly useful for detecting corrupted save files.

---

## 9. Combining Transformations

Compression, encryption, and checksum can be combined into a single pipeline.

For example:

```text
Data
 ↓
Serialization
 ↓
Compression
 ↓
Encryption
 ↓
Checksum
 ↓
File
```

The exact order of transformations matters.

SaveKit's transform system is designed so that individual transformation components can be added, removed, or replaced without changing the rest of the save architecture.

---

## 10. Save File Formats

SaveKit supports structured file formats instead of requiring every project to rely on a generic format such as JSON.

A SaveKit file can contain structured information about its version, organization, transformations, and data.

For example:

```text
[ASKH]
Version
Format
Organization
...
```

The file format and data pipeline are separate concepts.

This allows the same underlying data to be processed through different transformations without tightly coupling your application data to the physical file representation.

---

## 11. Development vs. Production

During development, a simple configuration is often preferable:

```text
Data
 ↓
Serialization
 ↓
File
```

It is easier to inspect and debug.

For production, you may want additional transformations:

```text
Data
 ↓
Serialization
 ↓
Compression
 ↓
Encryption
 ↓
Checksum
 ↓
File
```

Choose the pipeline according to your project's requirements rather than enabling every feature by default.

---

## 12. Where to Go Next

Once you have completed the basic save/load workflow, the following topics are recommended:

* [Architecture](Architecture.md)
* [Data Model](DataModel.md)
* [Serialization](Serialization.md)
* [Data Formats](DataFormats.md)
* [Transforms](Transforms.md)
* [Compression](Compression.md)
* [Encryption](Encryption.md)
* [Checksums](Checksums.md)
* [Examples](../Examples/)
* [API Reference](API.md)

These documents explain how to customize SaveKit and build more advanced save pipelines.

---

## Troubleshooting

### The save file cannot be loaded

Check:

* Whether the file exists.
* Whether the file was written completely.
* Whether the save format matches the expected format.
* Whether the required transformation configuration is the same.
* Whether the checksum validation fails.
* Whether the save version is supported.

### Checksum validation fails

A checksum mismatch generally means that the data being verified is different from the data used to generate the original checksum.

This can happen if the file was corrupted, modified, truncated, or processed with an incompatible pipeline.

### The save file is larger than expected

Consider enabling compression if your data contains enough repetitive or structured content to benefit from it.

---

## Next Step

You are now ready to use SaveKit in your project.

Start with the simplest possible pipeline, verify that saving and loading work correctly, and then add compression, encryption, checksums, or other transformations as your project requires.
