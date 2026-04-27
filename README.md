# Blueprint

**The application model runtime.**

Blueprint is a way to describe applications as graphs of small, self-contained units — and a runtime that can execute those graphs, compile them to other targets, and let them be authored by humans, AI, or both at the same time.

React renders views. Blueprint runs application models.

## What it is

A Blueprint is a declarative description of an application: its state, its behavior, its lifecycle, the channels through which its parts communicate, and the boundaries that contain them. The same description can be:

- **Run** directly by a Blueprint engine
- **Compiled** to platform-native code (web, mobile, desktop, embedded, accelerators)
- **Composed** with other Blueprints — every Blueprint is itself a unit that can be embedded in another, all the way down

The artifact is portable. The runtime is open. The contract is small enough to specify and large enough to express real applications.

## Status

Blueprint is in active design. This repository holds the specification.

The first round of spec work (currently visible on `main`) was written before any working engine existed. We've since built engines, compilers, and dozens of demos in [`blueprint-sandbox`](https://github.com/AntonBursch/blueprint-sandbox), and discovered that some of our early decisions don't survive contact with implementation.

**The spec is being rewritten on the `ir-first` branch** to reflect what we actually learned. The keystone document is [`spec/ir/ir-kinds.md`](https://github.com/AntonBursch/blueprint/blob/ir-first/spec/ir/ir-kinds.md), which defines the canonical intermediate representation that every engine, compiler, and tool must agree on.

The legacy specs in [`docs/specs/`](docs/specs/) are kept for history and will be replaced as the rewrite stabilizes.

## What's in this repository

```
docs/
  app-theory/   The thinking behind Blueprint (still relevant)
  essays/       Long-form writing
  specs/        Legacy spec drafts — being rewritten on `ir-first`
spec/           New spec (visible on `ir-first` branch)
```

Reference implementations, conformance tooling, and language bindings live in a separate repository. This repo is the contract.

## Vocabulary

Blueprint defines a small, finite vocabulary. The current working terms:

| Term | Role |
| --- | --- |
| **Blueprint** | The artifact: a declarative description of an application. |
| **Compound** | A self-contained, independently-authored, independently-verifiable composition unit. Compounds nest arbitrarily. |
| **Channel** | A named pathway through which compounds communicate. |
| **Scope** | The boundary within which state, channels, and marks are addressable. |
| **State** | The evolving data a Blueprint carries over time. |
| **Schema** | The typed shape of any data a Blueprint produces, consumes, or carries. |
| **Mark** | A coordination identifier stamped on compounds, carrying authorship and provenance. |
| **Signature** | A cryptographic proof binding a mark to a verifiable author. |

## Design principles

- **Portable.** A Blueprint produced by one author, in one tool, runs on any conforming engine without modification.
- **Composable.** Compounds from different authors combine deterministically. A Blueprint can contain other Blueprints, with no fixed depth.
- **Compilable.** The same model can be run by an engine or compiled to native code on any target — browser, mobile, server, embedded, accelerator.
- **Durable.** A Blueprint authored today should remain readable and runnable by any future conforming engine.
- **Legible.** The artifact is readable by humans, AI systems, and engines at the same register.
- **Neutral.** The specification is governed to remain free of any single vendor's control.

## Governance intent

Blueprint is designed to be stewarded by a neutral body. This repository is the staging ground for the specification; governance materials will be added when the project reaches the point where transfer to a foundation is appropriate.

## License

The specification is licensed under the [Apache License, Version 2.0](LICENSE). Implementations are free to choose their own licenses.

## Contact

This project is authored by Anton Bursch.
