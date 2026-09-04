# SaveKit Architecture

SaveKit is designed as a modular data persistence pipeline.

Instead of treating saving as a single serialization operation, SaveKit separates data representation, organization, serialization, transformation, and file handling into independent layers.

This architecture allows individual components to be replaced, extended, or configured without requiring changes to the entire system.

---

## Overview

A simplified representation of the SaveKit architecture is:

```text
                    ┌─────────────────┐
                    │   Application   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Data Model   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Organization   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Serialization  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Transformation │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │      Format     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │       I/O       │
                    └─────────────────┘
```

Each layer has a specific responsibility.

---

# Core Components

## Data Model

The Data Model represents the logical data that the application wants to persist.

SaveKit does not require the physical save file to directly represent application objects.

Instead, application data can be converted into SaveKit's structured representation before it is serialized.

This separation allows the storage format to evolve independently from the application's runtime objects.

---

## AST

SaveKit uses an Abstract Syntax Tree (AST) as a structured representation of serialized data.

The AST provides an intermediate representation between parsing/serialization and the higher-level data model.

A simplified flow is:

```text
Input
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
Data Model
```

For loading, the process can be reversed:

```text
Data Model
  │
  ▼
AST
  │
  ▼
Serializer
  │
  ▼
Serialized Data
```

Using an intermediate representation makes the serialization system less dependent on a specific physical file format.

---

# Organization

The Organization layer determines how data is structured within a package.

For example, data can be separated into logical sections:

```text
Package
├── Characters
├── Player
├── Inventory
└── Logs
```

Organization is separate from serialization.

This means that the way data is grouped does not have to determine how that data is encoded into a file.

The separation also allows different organization strategies to be implemented independently.

---

# Serialization

The Serialization layer converts structured data into a serialized representation and reconstructs structured data from serialized input.

Conceptually:

```text
Structured Data
      │
      ▼
Serialization
      │
      ▼
Serialized Data
```

and during loading:

```text
Serialized Data
      │
      ▼
Deserialization
      │
      ▼
Structured Data
```

Serialization is deliberately separated from transformations such as compression and encryption.

This prevents the serializer from having to know how the resulting data will be stored or protected.

---

# Transformation Pipeline

Transformations operate on serialized data.

A transformation can modify the representation without changing the logical data model.

For example:

```text
Serialized Data
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
Final Data
```

SaveKit defines transformation interfaces so that individual implementations can be replaced or extended.

Examples include:

* Encoding
* Compression
* Encryption
* Checksum

The pipeline can therefore be configured according to the needs of the application.

---

# Why Transformations Are Separate

Compression, encryption, and integrity verification are not serialization responsibilities.

Keeping them separate provides several advantages:

### Replaceability

A compression implementation can be replaced without changing the serializer.

### Composition

Multiple transformations can be combined into a single pipeline.

### Testability

Each transformation can be tested independently.

### Extensibility

New transformation types can be added without modifying unrelated components.

For example, a project could use:

```text
Serialization
    ↓
Brotli Compression
    ↓
AES Encryption
    ↓
SHA-256 Checksum
```

while another project could use only:

```text
Serialization
    ↓
File
```

---

# File Formats

SaveKit separates the logical data pipeline from the physical file format.

A file format is responsible for defining how the resulting information is represented as a file.

For example, a SaveKit file may contain information describing:

```text
Header
├── Version
├── Compression
├── Encryption
├── Checksum
└── Organization

Data
```

This allows metadata about the file and its processing pipeline to be stored alongside the actual data.

The format layer can therefore evolve without requiring the application data model to change.

---

# I/O

The I/O layer is responsible for reading and writing the resulting data.

Its purpose is to keep filesystem or other storage-specific operations separate from the rest of the pipeline.

Conceptually:

```text
SaveKit Pipeline
      │
      ▼
Serialized / Transformed Data
      │
      ▼
I/O
      │
      ▼
Storage
```

This separation makes it possible to adapt SaveKit to different storage mechanisms without coupling the core data processing system to a particular storage API.

---

# Saving Pipeline

A simplified save operation can be represented as:

```text
Application Data
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
Format
       │
       ▼
I/O
       │
       ▼
Save File
```

Not every stage is required.

A minimal configuration may look like:

```text
Data
 ↓
Serialization
 ↓
Format
 ↓
I/O
```

A more advanced configuration may use the complete pipeline.

---

# Loading Pipeline

Loading performs the reverse operations where applicable:

```text
Save File
       │
       ▼
I/O
       │
       ▼
Format
       │
       ▼
Checksum Verification
       │
       ▼
Decryption
       │
       ▼
Decompression
       │
       ▼
Decoding
       │
       ▼
Deserialization
       │
       ▼
Organization
       │
       ▼
Data Model
       │
       ▼
Application Data
```

The exact order depends on the configured pipeline and the requirements of the format.

---

# Package Architecture

SaveKit is divided into modules with distinct responsibilities.

A conceptual package structure is:

```text
SaveKit
├── IO
├── Organization
├── Transform
├── Formats
├── Data
│   ├── AST
│   ├── Model
│   └── Serialization
└── Common
```

This modular structure is intended to minimize coupling between components.

For example, adding a new compression algorithm should not require changes to the AST or data model.

---

# Design Principles

SaveKit's architecture follows several principles.

## Separation of Concerns

Each component should have a clearly defined responsibility.

## Composition Over Coupling

Independent components can be combined to construct a complete save pipeline.

## Extensibility

Core interfaces allow applications to provide custom implementations where necessary.

## Testability

Components are designed so that individual parts of the pipeline can be tested independently.

## Format Independence

Application data should not be tightly coupled to a single physical file representation.

## Explicit Processing

Operations such as compression, encryption, and checksum calculation are represented as explicit pipeline stages rather than hidden behavior inside the serializer.

---

# Example

Consider a game that wants to store player data securely while keeping save files small.

The application may use:

```text
Player Data
     │
     ▼
SaveKit Data Model
     │
     ▼
Organization
     │
     ▼
Serialization
     │
     ▼
Brotli
     │
     ▼
AES
     │
     ▼
SHA-256
     │
     ▼
Save Format
     │
     ▼
File System
```

The game does not need to implement compression, encryption, checksum generation, file organization, and serialization as one tightly coupled system.

Each responsibility remains isolated within the SaveKit architecture.

---

# Extending SaveKit

The architecture is designed to allow custom components to be introduced where appropriate.

For example, a project could provide its own:

* Serialization format
* Compression algorithm
* Encryption implementation
* Checksum implementation
* Organization strategy
* Storage backend
* Data processing component

Custom components can then participate in the existing pipeline without requiring the rest of SaveKit to be rewritten.

---

# Summary

SaveKit treats persistence as a pipeline rather than a single serialization operation.

The main responsibilities are separated into:

```text
Data
 ↓
Organization
 ↓
Serialization
 ↓
Transformation
 ↓
Format
 ↓
I/O
```

This separation is the foundation of SaveKit's extensibility.

It allows developers to start with a simple save pipeline and gradually add capabilities such as compression, encryption, integrity verification, custom formats, and custom processing without rebuilding the entire save system.
