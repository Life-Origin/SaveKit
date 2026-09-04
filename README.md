# SaveKit

SaveKit is a modular and extensible save system for C# and Unity, designed to provide a structured and reliable way to store, load, transform, and manage application data.

Unlike simple serialization-based save systems, SaveKit provides a complete data pipeline with its own structured data model, serialization system, custom file formats, and transformation layers.

## Features

* Modular save/load pipeline
* Structured data model and AST
* Custom file formats
* Serialization and deserialization
* Compression support
* Encryption support
* Checksum and data integrity verification
* Extensible transformation system
* Support for organized and multi-section data
* Designed for C# and Unity
* Testable and extensible architecture

## Architecture

SaveKit separates the save process into independent layers:

**Data → Organization → Serialization → Transformation → File**

This allows individual components to be replaced or extended without requiring changes to the entire system.

For example, compression, encryption, and checksum operations can be configured independently from the underlying data format and serialization logic.

## Why SaveKit?

Save systems often start as a simple serialization method and gradually become tightly coupled to the application's data structures.

SaveKit is designed to avoid this by treating saving as a data pipeline rather than a single serialization operation.

This makes it possible to build custom formats and processing pipelines while keeping the core system modular and extensible.
