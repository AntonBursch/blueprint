# Blueprint Specification

> **⚠️ Being rewritten.** These drafts predate the first working Blueprint engines and compilers. Implementing them surfaced decisions that don't hold up — vocabulary that needed to change, structure that needed to be inverted, and a missing layer (the canonical IR) that everything else has to be built on.
>
> The rewrite is happening on the [`ir-first`](https://github.com/AntonBursch/blueprint/tree/ir-first) branch, anchored on [`spec/ir/ir-kinds.md`](https://github.com/AntonBursch/blueprint/blob/ir-first/spec/ir/ir-kinds.md). The documents below are kept for history and will be replaced section by section as the rewrite stabilizes.
>
> If you want to know where Blueprint is going, read the `ir-first` branch. If you want to know how it got here, read on.

---

This directory contains the Blueprint specification, version 0.1.0-draft.

## Read the spec

- [overview.md](overview.md) — the specification at a glance
- [ROADMAP.md](ROADMAP.md) — authoring plan, doc status, working policy

## Structure

```
docs/specs/
├── README.md                 This file
├── ROADMAP.md                Authoring plan (internal)
├── overview.md               Specification at a glance
│
├── vocabulary/               Definitions of each term (normative)
│   ├── schema.md
│   ├── scope.md
│   ├── state.md
│   ├── channel.md
│   ├── compound.md
│   ├── mark.md
│   ├── signature.md
│   ├── draft.md
│   └── blueprint.md
│
├── semantics/                Runtime behavior (normative)
│   ├── composition.md
│   ├── lifecycle.md
│   ├── scope-propagation.md
│   ├── channel-delivery.md
│   ├── determinism.md
│   └── authorship.md
│
├── schema/                   Machine-readable JSON Schemas (normative)
│   ├── blueprint.schema.json
│   ├── mark.schema.json
│   └── signature.schema.json
│
├── conformance/              Conformance and contracts (normative)
│   ├── conformance.md
│   ├── engine-contract.md
│   ├── tool-contract.md
│   └── test-suite.md
│
├── canonical-form.md         Byte-level serialization (normative)
├── versioning.md             Version declaration and evolution (normative)
├── extensions.md             Extension mechanism (normative)
│
├── rationale/                Why the spec is shaped this way (informative)
│   ├── ambition.md
│   ├── co-authorship.md
│   ├── openapi-precedent.md
│   ├── prior-art.md
│   └── non-goals.md
│
├── primers/                  Audience-specific introductions (informative)
│   ├── quick-start.md
│   ├── for-developers.md
│   ├── for-enterprise-architects.md
│   └── for-ai-systems.md
│
└── examples/                 Worked examples (informative)
    └── README.md
```

## Document status

The specification is under active drafting. Documents carry one of three
status banners:

- **Draft** — work in progress, may change materially
- **Candidate** — stable enough for review, breaking changes unlikely
- **Stable** — part of a released specification version

At present, only [overview.md](overview.md) contains substantive content;
all other documents are placeholders. See [ROADMAP.md](ROADMAP.md) for the
authoring plan and order.

## Conformance language

Normative documents use the conformance terminology of RFC 2119 and
RFC 8174 (MUST, SHOULD, MAY, etc.). Informative documents use plain prose.

## License

The specification is published under the [Apache License, Version 2.0](../../LICENSE).
