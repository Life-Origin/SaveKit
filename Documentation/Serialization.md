# Serialization

Serialization is the process of converting SaveKit's structured data into a serialized representation that can be stored or transmitted.

Deserialization performs the reverse operation, reconstructing structured data from the serialized representation.

SaveKit keeps serialization separate from file storage, compression, encryption, and other transformations.

---

## Overview

The basic serialization flow is:

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

Serialization defines how structured data is represented. It does not determine where the resulting data is stored or which transformations are applied afterward.

---

# Serialization Pipeline

SaveKit's serialization system can be viewed as several stages:

```text
Application Data
       │
       ▼
   Data Model
       │
       ▼
    Serializer
       │
       ▼
 Serialized Data
```

For formats that require lexical and syntactic processing, the deserialization path can be represented as:

```text
Serialized Data
       │
       ▼
    Tokenizer
       │
       ▼
     Tokens
       │
       ▼
     Parser
       │
       ▼
      AST
       │
       ▼
DataModelBuilder
       │
       ▼
   Data Model
```

Each stage has a different responsibility.

---

# Serialization vs. File Format

Serialization and file format are related but separate concepts.

**Serialization** determines how the data structure is converted into a serialized representation.

**File format** determines how that representation and its associated metadata are packaged and stored.

For example:

```text
Data Model
    │
    ▼
Serialization
    │
    ▼
Serialized Data
    │
    ▼
File Format
    │
    ▼
Save File
```

This separation allows the serialization system to evolve independently from the physical file representation.

---

# Serializer

A serializer converts a structured data representation into serialized data.

Conceptually:

```text
Data Model
    │
    ▼
Serializer
    │
    ▼
Serialized Data
```

For example, a structured model such as:

```text
Player
├── Name: "Alex"
├── Level: 10
└── Health: 85.5
```

could be converted into a textual representation such as:

```text
Player {
    Name = "Alex"
    Level = 10
    Health = 85.5
}
```

The exact syntax depends on the serialization format being used.

---

# Deserialization

Deserialization reconstructs structured data from serialized input.

For a text-based representation, this commonly involves several stages:

```text
Serialized Text
      │
      ▼
Tokenizer
      │
      ▼
Tokens
      │
      ▼
Parser
      │
      ▼
AST
      │
      ▼
Data Model
```

The parser is responsible for understanding the syntactic structure.

The model-building stage is responsible for converting that structure into the logical representation used by SaveKit.

---

# Tokenization

Tokenization converts serialized input into a sequence of tokens.

For example:

```text
Player {
    Level = 10
}
```

can conceptually become:

```text
Identifier("Player")
OpenBrace
Identifier("Level")
Equals
Integer(10)
CloseBrace
```

The tokenizer does not need to understand the meaning of the complete data structure.

Its responsibility is to identify the individual lexical elements required by the parser.

---

# Parsing

The parser consumes tokens and constructs an Abstract Syntax Tree.

```text
Tokens
  │
  ▼
Parser
  │
  ▼
AST
```

For example:

```text
Player {
    Level = 10
}
```

can produce an AST similar to:

```text
Record: Player
└── Field: Level
    └── Value: 10
```

The AST preserves the syntactic structure needed by later stages.

---

# AST to Data Model

The AST is an intermediate representation.

After parsing, SaveKit can convert the AST into its logical Data Model:

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
DataModelBuilder
      │
      ▼
 Data Model
```

This distinction prevents the parser from having to understand higher-level application concepts.

The parser understands syntax.

The Data Model represents structured data.

---

# Why Use an AST?

A direct parser-to-model approach can be sufficient for very simple formats, but an AST provides a useful intermediate layer.

It can provide:

* Clear separation between syntax and data
* Easier parser testing
* Better error reporting
* Intermediate inspection
* Potential format transformations
* A reusable representation for tooling

For example:

```text
Serialized Input
       ↓
     Tokens
       ↓
      AST
       ↓
 Data Model
```

Each stage can be inspected and tested independently.

---

# Serialization and Transformations

Serialization produces the data that can then be processed by transformation stages.

For example:

```text
Data Model
    │
    ▼
Serialization
    │
    ▼
Encoding
    │
    ▼
Compression
    │
    ▼
Encryption
    │
    ▼
Checksum
    │
    ▼
File
```

The serializer does not need to know whether compression or encryption will be applied.

This is an intentional separation of responsibilities.

---

# Why Compression Is Not Part of Serialization

Compression changes the representation of serialized data, but it does not define the logical structure of that data.

For example:

```text
Data Model
    ↓
Serialization
    ↓
Serialized Data
    ↓
Compression
    ↓
Compressed Data
```

The same serialized data could be compressed using different algorithms without changing the underlying data model or serializer.

This makes compression a transformation rather than a serialization responsibility.

---

# Why Encryption Is Not Part of Serialization

Encryption similarly operates on the serialized representation.

```text
Data Model
    ↓
Serialization
    ↓
Serialized Data
    ↓
Encryption
    ↓
Encrypted Data
```

Keeping encryption separate allows different encryption implementations to be used without modifying the serializer.

---

# Serialization and Encoding

Encoding and serialization are also different concepts.

Serialization converts structured data into a representation.

Encoding converts that representation into a specific byte representation.

For example:

```text
Data Model
    ↓
Serialization
    ↓
Text
    ↓
UTF-8 Encoding
    ↓
Bytes
```

This distinction is useful because a serializer can produce a logical serialized representation without being tightly coupled to a specific character encoding.

---

# Example

Consider the following data:

```text
Player
├── Name: "Alex"
├── Level: 12
└── Health: 74.5
```

A serialization process may produce:

```text
Player {
    Name = "Alex"
    Level = 12
    Health = 74.5
}
```

The complete save pipeline could then be:

```text
Data Model
    ↓
Serialization
    ↓
UTF-8 Encoding
    ↓
Brotli Compression
    ↓
AES Encryption
    ↓
SHA-256 Checksum
    ↓
Save Format
    ↓
File
```

The serializer only needs to be concerned with the serialization step.

---

# Custom Serialization

SaveKit's serialization architecture is designed to allow different serialization implementations.

A custom serializer can be useful when:

* A project requires a custom syntax
* A legacy format must be supported
* A specialized data representation is required
* A different balance between readability and size is desired
* A custom toolchain needs to consume the serialized data

A custom implementation should remain independent from unrelated transformations whenever possible.

The resulting pipeline can then be composed with the existing SaveKit infrastructure.

---

# Serialization and Versioning

Serialized data may change as a project evolves.

For example, an early version may contain:

```text
Player
├── Name
└── Level
```

while a later version may contain:

```text
Player
├── Name
├── Level
└── Experience
```

Serialization and versioning should therefore be considered separately.

The serializer is responsible for representing the current data structure.

Versioning or migration mechanisms are responsible for handling differences between data versions.

This separation prevents version-specific logic from becoming unnecessarily coupled to the serializer.

---

# Error Handling

Serialization errors can occur when data cannot be represented correctly.

Deserialization errors can occur when serialized input is invalid or incompatible with the expected structure.

Typical causes include:

* Invalid syntax
* Unexpected tokens
* Invalid values
* Unsupported types
* Missing required data
* Incompatible versions
* Corrupted input

Errors should be detected as early as possible in the pipeline.

For example:

```text
Serialized Data
      │
      ▼
   Tokenizer
      │
      ▼
    Parser
      │
      ├── Syntax Error
      │
      ▼
     AST
      │
      ▼
DataModelBuilder
      │
      ├── Model Error
      │
      ▼
 Data Model
```

Keeping these stages separate also makes it easier to determine where an error originated.

---

# Testing

Serialization should be tested independently from file I/O and transformations.

Useful tests include:

### Serialization Tests

Verify that a data model produces the expected serialized representation.

### Deserialization Tests

Verify that valid serialized data produces the expected data model.

### Round-Trip Tests

Verify that:

```text
Data Model
    ↓
Serialize
    ↓
Deserialize
    ↓
Data Model
```

produces an equivalent result.

### Parser Tests

Verify that valid and invalid syntax is handled correctly.

### Tokenizer Tests

Verify that input is converted into the expected token sequence.

### Error Tests

Verify that malformed input produces the expected errors.

---

# Round-Trip Serialization

One of the most important serialization tests is the round-trip test.

Given an initial data model:

```text
Model A
   │
   ▼
Serialize
   │
   ▼
Serialized Data
   │
   ▼
Deserialize
   │
   ▼
Model B
```

The goal is:

```text
Model A ≡ Model B
```

where equivalence is defined according to the semantics of the data model.

This verifies both sides of the serialization process together.

---

# Design Principles

SaveKit's serialization architecture follows several principles.

## Separation of Concerns

Tokenization, parsing, modeling, serialization, and transformation have separate responsibilities.

## Format Independence

The data model should not be tightly coupled to a single serialized representation.

## Composability

Serialization should work as one stage within a larger persistence pipeline.

## Extensibility

New serialization implementations should be possible without modifying unrelated components.

## Testability

Each stage should be independently testable.

## Predictability

Serializing and deserializing the same logical data should produce equivalent results.

---

# Summary

SaveKit treats serialization as one stage of a larger data persistence pipeline.

The core relationship is:

```text
Data Model
    ↓
Serialization
    ↓
Serialized Data
    ↓
Transformations
    ↓
File Format
    ↓
Storage
```

For structured text formats, deserialization can be represented as:

```text
Serialized Data
    ↓
Tokenizer
    ↓
Parser
    ↓
AST
    ↓
DataModelBuilder
    ↓
Data Model
```

By separating serialization from parsing, data modeling, transformations, and storage, SaveKit can support extensible serialization pipelines without coupling the entire save system to a single representation.
