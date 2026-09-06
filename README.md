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

## Version

**SaveKit 1.0**

---

# SaveKit License

Copyright (c) 2026 Berke Kömür

SaveKit is proprietary software.

Except as expressly permitted by the applicable license terms, the SaveKit software, including its compiled assemblies and implementation, may not be copied, modified, redistributed, sublicensed, or resold separately.

When distributed through the Unity Asset Store, SaveKit is licensed to the customer under the applicable Unity Asset Store End User License Agreement.

## Extensions and Plugins

SaveKit provides public interfaces and APIs intended for third-party extensions.

Developers may create, distribute, and commercially sell independent plugins, extensions, data formats, transforms, storage implementations, and other integrations that use the public SaveKit API, provided that such products do not redistribute the SaveKit implementation itself.

Documentation and example code included in this repository may be used according to the terms specified for those materials.

## Third-Party Components

Third-party components, if any, are subject to their respective licenses. See `Third-Party Notices.txt` for applicable third-party licensing information.

## Disclaimer

SaveKit is provided "as is", without warranty of any kind, to the maximum extent permitted by applicable law.
