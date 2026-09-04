# Data Model

SaveKit uses a structured data model as an intermediate representation between application data and its serialized representation.

The data model separates the logical structure of data from the way that data is stored in a file.

This allows SaveKit to organize, serialize, transform, and store data without requiring the application's runtime objects to be directly coupled to a specific file format.

---

## Overview

A simplified representation of the data flow is:

```text
Application Objects
        │
        ▼
   Data Model
        │
        ▼
Organization
        │
        ▼
Serialization
        │
        ▼
   Save Format
```

The data model describes **what the data is**, while serialization and file formats describe **how the data is represented and stored**.

---

# Why a Data Model?

A simple save system can serialize an object directly:

```text
C# Object
   ↓
JSON
   ↓
File
```

This approach can work well for simple projects, but it tightly couples the application's data structures to the storage format.

SaveKit introduces an intermediate representation:

```text
C# Object
   ↓
Data Model
   ↓
Serializer
   ↓
File Format
```

This additional layer provides greater flexibility.

For example, the same logical data can be serialized using different formats without requiring the application data itself to change.

---

# Data Model Structure

SaveKit represents data as a hierarchy of structured values.

Conceptually, a data model can contain:

```text
Root
├── Player
│   ├── Name
│   ├── Level
│   └── Health
│
├── Inventory
│   ├── Item_01
│   ├── Item_02
│   └── Item_03
│
└── Settings
    ├── MusicVolume
    └── SoundVolume
```

This structure allows related values to be grouped together while maintaining a clear hierarchy.

---

# Values

The data model consists of values that represent the information being stored.

Depending on the implementation, values can represent common types such as:

* Strings
* Integers
* Floating-point numbers
* Boolean values
* Null values
* Collections
* Objects
* Nested structures

For example:

```text
Player
├── Name: "Player"
├── Level: 10
├── Health: 85.5
└── IsAlive: true
```

The logical representation is independent from the final serialized representation.

---

# Objects and Fields

Structured data can contain named members.

For example:

```text
Player
├── Name
├── Level
├── Health
└── Inventory
```

A member can itself contain another structured value.

This allows complex data to be represented recursively.

```text
Player
└── Inventory
    ├── Sword
    │   ├── Damage
    │   └── Durability
    │
    └── Potion
        └── Amount
```

---

# Collections

Collections allow multiple values of the same or different types to be represented as a single logical structure.

For example:

```text
Inventory
├── Item
├── Item
├── Item
└── Item
```

Collections are particularly useful for game data such as:

* Inventories
* Character lists
* Quest lists
* Unlocked achievements
* World objects
* Save history entries

The collection representation is independent from the storage format.

---

# Records

Records can be used to represent structured groups of data.

For example:

```text
Character
├── Name
├── Level
├── Health
└── Position
```

A record can contain fields and other nested structures.

This makes it possible to represent complex application state without requiring the storage format to directly mirror a C# class.

---

# Root

Every data structure has a root from which its hierarchy can be traversed.

A simplified model can therefore be represented as:

```text
Root
├── Player
├── World
├── Inventory
└── Settings
```

The root acts as the entry point of the data structure.

It allows the rest of the model to be traversed and processed as a single structured representation.

---

# AST and Data Model

SaveKit separates the AST from the higher-level data model.

The AST represents the syntactic structure required for parsing and serialization.

The Data Model represents the logical structure of the data used by the rest of the system.

Conceptually:

```text
Serialized Data
      │
      ▼
   Tokenizer
      │
      ▼
    Parser
      │
      ▼
     AST
      │
      ▼
Data Model Builder
      │
      ▼
 Data Model
```

This distinction is important because parsing a file and understanding the data represented by that file are separate responsibilities.

---

# AST

The AST is closer to the serialized representation.

It can contain information necessary to preserve the structure of the parsed data.

For example:

```text
Serialized Text
      ↓
Tokenizer
      ↓
Tokens
      ↓
Parser
      ↓
AST
```

The AST can then be converted into the data model.

This makes the parser independent from higher-level application logic.

---

# Data Model

The Data Model is closer to the logical representation used by SaveKit.

For example, a parsed structure such as:

```text
Player {
    Name = "Player"
    Level = 10
}
```

can be represented conceptually as:

```text
Player
├── Name: "Player"
└── Level: 10
```

The Data Model can then be organized and serialized without requiring the parser to know how the application intends to use the data.

---

# Model Building

The conversion from AST to Data Model is handled by the model-building stage.

```text
AST
 │
 ▼
DataModelBuilder
 │
 ▼
Data Model
```

This stage translates the syntactic representation into the logical representation used by the rest of SaveKit.

Keeping this process separate makes the architecture easier to test and extend.

---

# Serialization

Serialization operates on the structured data model.

```text
Data Model
    │
    ▼
Serializer
    │
    ▼
Serialized Data
```

During loading, the process is reversed:

```text
Serialized Data
    │
    ▼
Parser / Deserializer
    │
    ▼
Data Model
```

The serializer therefore does not need to know how the resulting bytes will eventually be compressed, encrypted, or stored.

---

# Organization

The Data Model describes the structure of the data, while Organization determines how that data is grouped within a package.

For example:

```text
Data Model

Root
├── Player
├── Inventory
└── World
```

can be organized into logical sections:

```text
Package
├── Player
├── Inventory
└── World
```

Organization is therefore a separate concern from the underlying data representation.

---

# Data Model vs. Application Objects

SaveKit's data model should not be confused with the application's runtime objects.

For example, an application may have:

```csharp
public class Player
{
    public string Name;
    public int Level;
    public float Health;
}
```

The runtime object contains behavior and application-specific state.

The save data only needs the persistent information:

```text
Player
├── Name
├── Level
└── Health
```

This distinction prevents runtime implementation details from unnecessarily becoming part of the save format.

---

# Example

Consider a simple game state:

```text
Player
├── Name: "Alex"
├── Level: 12
├── Health: 74.5
│
└── Inventory
    ├── Sword
    ├── Potion
    └── Key
```

The same logical model could be serialized into a SaveKit format and then passed through additional transformations:

```text
Data Model
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
Save File
```

The data model itself does not need to know that these transformations will be applied.

---

# Design Goals

The SaveKit data model is designed around several goals.

## Format Independence

The logical data structure should not depend on a single physical file format.

## Structured Data

Complex hierarchical data should be representable without flattening everything into primitive values.

## Extensibility

New value types and structures should be possible without redesigning the entire save pipeline.

## Separation of Concerns

Parsing, modeling, organization, serialization, and transformation should remain independent responsibilities.

## Traversability

The model should allow structured data to be inspected and traversed programmatically.

## Testability

The data model can be tested independently from file I/O and transformation systems.

---

# When Should You Use the Data Model Directly?

Most users do not need to interact with the internal data model directly.

For normal save/load operations, SaveKit's higher-level APIs should be preferred.

Direct interaction with the data model becomes useful when you need to:

* Build data dynamically
* Inspect save data
* Modify structured data
* Create custom serializers
* Implement custom organizations
* Develop tools around SaveKit
* Manipulate save data without reconstructing application objects

---

# Summary

SaveKit's Data Model provides the logical representation of persistent data.

The overall relationship can be summarized as:

```text
Application Objects
        ↓
    Data Model
        ↓
   Organization
        ↓
   Serialization
        ↓
 Transformations
        ↓
   File Format
        ↓
       I/O
```

The Data Model is intentionally separated from the application's runtime objects and from the physical save format.

This separation allows SaveKit to provide a flexible persistence pipeline while keeping individual components independent and replaceable.
