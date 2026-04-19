# Blueprint

**An open specification for authoring applications.**

Blueprint is a vocabulary, schema, and runtime contract for portable applications, expressed in JSON. It is designed to be the shared medium in which humans, AI agents, and conforming engines co-author software.

Applications described in Blueprint are portable across engines, composable across authors, durable across time, and legible to every participant — human or machine.

## Status

Blueprint is in active design. The specification is being drafted on the `main` branch. The first stable release is targeted at version 1.0.

This repository contains the specification. Reference implementations, tooling, and governance materials will live alongside it as the project matures.

## What's in this repository

```
docs/
  specs/        The Blueprint specification
```

Additional top-level directories will be added as the project grows:

- `engines/` — reference implementations of the Blueprint runtime contract
- `schema/` — JSON Schema definitions for validating Blueprint documents
- `examples/` — example Blueprint documents demonstrating the vocabulary
- `governance/` — charter, working-group process, and contribution guidelines

## Vocabulary

Blueprint defines a small, finite vocabulary. The current working terms:

| Term | Role |
| --- | --- |
| **Blueprint** | The artifact: a JSON document describing an application. |
| **Draft** | The verb for authoring a Blueprint; the state of a Blueprint under active revision. |
| **Compound** | A self-contained, independently-authored, independently-verifiable composition unit. |
| **Mark** | A coordination identifier stamped on compounds and Blueprints, carrying authorship and provenance. |
| **Signature** | A cryptographic proof binding a mark to a verifiable author. |
| **Channel** | A named pathway through which compounds communicate. |
| **Scope** | The boundary within which state, channels, and marks are addressable. |
| **State** | The evolving data a Blueprint carries over time. |
| **Schema** | The typed shape of any data a Blueprint produces, consumes, or carries. |

The specification defines how each term behaves, composes, and persists.

## Design principles

- **Portable.** A Blueprint produced by one author, in one tool, must run on any conforming engine without modification.
- **Composable.** Compounds from different authors, at different times, combine deterministically through marks.
- **Durable.** A Blueprint authored today must remain readable and runnable by any future conforming engine.
- **Legible.** The artifact is readable by humans, AI systems, and engines at the same register.
- **Neutral.** The specification is governed to remain free of any single vendor's control.

## Governance intent

Blueprint is designed to be stewarded by a neutral body. This repository is the staging ground for the specification; governance materials will be added as the project reaches the point where transfer to a foundation is appropriate.

Until then, the specification is published under a permissive license to maximize adoption and implementation.

## License

The specification is licensed under the [Apache License, Version 2.0](LICENSE). Implementations are free to choose their own licenses.

## Contact

This project is authored by Anton Bursch.
