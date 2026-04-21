# Blueprint Specification — Overview

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

> This document is the canonical overview of the Blueprint specification. It defines what a Blueprint is, the vocabulary it uses, the contract it imposes on conforming engines, and the model by which Blueprints are authored, composed, and verified.

---

## 0. Purpose

Blueprint exists so that software built with AI can enter consequential domains without dissolving the accountability those domains depend on.

The word "blueprint" is chosen deliberately. A blueprint in the architectural sense is a signed document. An engineer seals it. An inspector checks it against code. A contractor builds from it. If the building fails, investigators retrieve the blueprint, trace the signatures, and assign responsibility. The document persists. The chain of accountability is legible long after the building is occupied.

Software that runs utilities, hospitals, factories, grids, pipelines, logistics, and finance deserves the same treatment. For most of computing history it has not received it. Source code is rarely signed, rarely reviewed as a composable artifact, rarely durable across the platforms it targets, and rarely legible to the domain professionals whose work it governs. The arrival of capable AI code generation makes this gap urgent. AI can now produce software faster than any review process can audit it, in domains where unreviewed software causes measurable harm.

The common response inside the technology field is that AI will become trustworthy enough that human accountability can be relaxed. The Blueprint specification rejects this position. Capability and accountability are orthogonal. A more capable system without accountability has a larger blast radius, not a smaller one. Every prior wave of powerful technology — pharmaceuticals, aviation, automobiles, broadcast, finance — became safe through accountability infrastructure built around it: certifications, signatures, audits, trials, licensed professionals, and legal liability that lands on humans. None became safe by becoming smart. AI will be no different, and the longer the technology field waits for intelligence to substitute for accountability, the larger the eventual correction.

Blueprint is the accountability infrastructure for AI-assisted software in serious domains. It is not an anti-AI position and it is not a defensive one. It is a structural claim: AI's contribution to software is creative and therefore error-prone, which is a feature of creativity rather than a defect to be engineered away; errors from creative systems require accountability; accountability requires an agent that can be held responsible; AI cannot be held responsible in any meaningful sense; therefore a named human must remain responsible for every artifact that ships, and the tools we build must make that responsibility legible by construction rather than by discipline.

Every technical decision in this specification follows from that claim. The code-and-composition boundary (§1.4, [authoring-roles.md](authoring-roles.md)) exists because AI may compose freely over reviewed primitives but must not author unreviewed imperative code into production. Compounds exist as the unit at which a human engineer takes responsibility. Signatures and marks (§6, §7) exist so that responsibility is recorded, not remembered. The open, vendor-neutral specification exists because accountability infrastructure cannot be owned by one vendor and cannot be subject to a central authority. The cross-engine, cross-platform runtime contract exists because the organizations that most need this property cannot accept lock-in as a condition of safety. Grammar plurality (DAGs, state machines, behavior trees, scene graphs, event streams) exists because domain fidelity lowers the chance that AI misunderstands what it is composing.

Blueprint's purpose, stated directly: **to let the world adapt to AI deliberately rather than through disaster.** The specification does not pretend that AI should be kept out of consequential work. It takes for granted that AI will be used, and it insists — through structure rather than exhortation — that humans remain the accountable authors of what runs. This is the only position under which AI can responsibly participate in the domains that carry real weight.

The specification is open so that this infrastructure cannot be owned, captured, or revoked. It is implementable on any platform so that it cannot be lock-in. It is legible to domain professionals so that the people whose work is governed by the software can read, question, and refuse it. These are not features. They are conditions of the purpose.

---

## 1. Introduction

A **Blueprint** is a JSON document that describes an application.

The Blueprint specification defines a finite vocabulary for that document, a schema that constrains its shape, a runtime contract that any conforming engine must honor, and an authorship model that allows humans, AI agents, and tools to co-author the same document over time without coordinating through a central authority.

Blueprint is not a programming language, not a visual programming environment, not a low-code platform, and not a code-generation target. It is a **specification** — in the register of OpenAPI, JSON Schema, and CloudEvents — for the artifact that *is* an application.

### 1.1 What problem Blueprint solves

Applications are currently trapped in proprietary tools. Every major application platform uses a different file format, a different runtime, a different mental model, and a different set of primitives. There is no `.html` for applications. An application written in React cannot run on a Compose engine without rewriting. An application written in WPF cannot be audited by a tool that understands Flutter. An application exported from a low-code platform cannot be re-imported anywhere else.

This fragmentation exists for historical reasons. It is not a property of what applications *are*; it is a property of how applications have been *built*. Blueprint's thesis is that applications, like documents, have a describable structure that can be expressed in a common, vendor-neutral format, and that expressing them so unlocks the same compositional, distributive, and durable properties that HTML unlocked for documents.

### 1.2 What condition Blueprint creates

Applications become **co-authorable**. Humans, AI agents, and teams across time author into the same document. Contributions combine without central coordination. The artifact survives its authors. Any conforming engine, on any platform, can run it.

### 1.3 Why now

Three preconditions are in place that were not in place before:

1. **JSON is universal.** Every language has a parser. The carrier format question is settled.
2. **AI can produce structured output reliably.** Large language models can generate, modify, and reason over JSON documents at a register that was not possible until recently. Blueprint is designed to be a target for AI authorship, not only a target for human authorship.
3. **Cross-platform rendering engines exist.** React, Flutter, Compose, and their equivalents have converged on compatible component models. A single semantic vocabulary can now be rendered coherently across web, mobile, desktop, and embedded targets.

### 1.4 Code is written by engineers; compositions are authored by anyone

Blueprint distinguishes between **writing code** and **authoring a composition**. Code — the imperative implementation inside a compound, processor, connection, or widget — is the responsibility of a human engineer, who MAY use AI as a tool but remains accountable for what ships. A composition — the wiring of reviewed compounds into an application — MAY be authored by anyone, with or without AI assistance, because every compound it references is an audited artifact.

This is a safety property, not a UX preference. AI code generation outpaces human review; AI composition over reviewed primitives does not. The composition remains legible by construction: opening a Blueprint shows which compounds it uses and how they connect, and every compound traces to an engineer who approved it.

The full model — compound provenance, signatures, authoring roles, and the constraints that make this enforceable — is defined in [authoring-roles.md](authoring-roles.md).

---


## 2. Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals.

A **conforming Blueprint** is a JSON document that validates against the Blueprint JSON Schema for its declared version.

A **conforming engine** is an implementation that honors the runtime contract defined in this specification for all vocabulary terms it claims to support.

A **conforming authoring tool** is a tool that reads, writes, or edits Blueprints in a manner that preserves conformance and respects the authorship model.

---

## 3. The artifact

A Blueprint is a JSON object. It MUST declare its specification version at the top level through the `blueprint` field.

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:weather-dashboard",
  "title": "Weather Dashboard",
  "compounds": { },
  "marks": [ ]
}
```

### 3.1 Top-level fields

| Field | Required | Type | Purpose |
| --- | --- | --- | --- |
| `blueprint` | MUST | string | The specification version this document conforms to. |
| `id` | MUST | URI | A stable identifier for this Blueprint. |
| `title` | SHOULD | string | A human-readable title. |
| `description` | MAY | string | A human-readable description. |
| `compounds` | MUST | object | The compounds defined by this Blueprint, keyed by compound name. |
| `entry` | MUST | string | The name of the compound that is the Blueprint's entry point. |
| `schemas` | MAY | object | Schema definitions used by this Blueprint. |
| `imports` | MAY | array | External Blueprints or registries this Blueprint depends on. |
| `marks` | MAY | array | The authorship trace for this Blueprint. |
| `signatures` | MAY | array | Cryptographic proofs binding marks to verifiable authors. |
| `metadata` | MAY | object | Unstructured information that does not affect runtime behavior. |

A Blueprint document MUST NOT contain top-level fields not defined by this specification or by a registered extension. Engines MUST reject Blueprints that contain unrecognized top-level fields unless the fields are namespaced under a registered extension prefix.

### 3.2 The `blueprint` version field

The `blueprint` field identifies the specification version. Version strings follow [Semantic Versioning 2.0.0](https://semver.org/). Documents using a `-draft` suffix are targeting a draft specification and MUST NOT be deployed to production.

Engines MUST check the `blueprint` field before processing any other part of the document. Engines MAY refuse to process Blueprints declaring versions they do not support, and MUST clearly report the version mismatch when doing so.

### 3.3 Identifiers

The `id` field is a URI that stably identifies this Blueprint. It SHOULD be a URN. It MUST be unique within any registry or system that holds it. Two Blueprints with the same `id` but different content represent an error in the authorship tooling and MUST be treated as a conflict.

---

## 4. Vocabulary

Blueprint defines a small, finite vocabulary. Every term has a defined role and a defined runtime contract. Engines that claim conformance to this specification MUST implement all terms defined in this section.

### 4.1 Blueprint

The **Blueprint** is the artifact itself — the complete JSON document describing an application. Every Blueprint has an identity, a version, a set of compounds, and an entry compound.

Blueprints are the unit of distribution. A Blueprint is portable: it can be copied, transmitted, hashed, signed, and replayed on any conforming engine that supports its declared version and the features it uses.

### 4.2 Compound

A **compound** is a self-contained, independently-authored, independently-verifiable composition unit within a Blueprint. Compounds are the primary authorship surface: humans and AI agents draft compounds, registries publish compounds, reviewers approve compounds, and engines instantiate compounds at runtime.

A compound declares:

- An **interface**: the shape of its inputs, outputs, and events.
- An **implementation**: the compounds it composes, the channels it wires, and the state it maintains.
- A **scope**: the boundary within which its state and channels are addressable.
- Zero or more **marks**: the authorship trace for this compound.

Compounds compose recursively: a compound's implementation is expressed in terms of other compounds. The recursion terminates at **primitive compounds** defined by the specification (e.g., built-in rendering primitives) and at compounds provided by the engine or its extensions.

A compound MUST NOT depend on any state, channel, or identifier outside its declared scope except through its interface.

> **Lineage note.** The term *compound* is chosen for its use in chemistry and architecture (a coherent whole composed of parts). The compositional model itself draws on bookbinding, where a *signature* is a pre-assembled gathering of pages that is independently produced and then bound into the final volume. A compound in Blueprint plays the same structural role.

### 4.3 Channel

A **channel** is a named pathway through which compounds communicate. Channels carry typed values over time. A channel has a name, a schema, a direction (input, output, or bidirectional from the perspective of a compound), and a scope.

Channels are the only means by which compounds communicate across scope boundaries. A compound MUST NOT observe or mutate state inside another compound's scope directly; it MUST do so through a channel declared on the other compound's interface.

Channels are typed. Every value placed on a channel MUST validate against the channel's declared schema. Engines MUST reject messages that do not validate.

A channel's schema describes the **logical shape** of its payload, not a runtime wire encoding. Engines are free to represent payloads in whatever form is most efficient for the channel's context:

- **In-memory channels** (between compounds in the same engine process) use the engine's native runtime representation. Validation at channel boundaries is REQUIRED for correctness but MAY be elided when the producer's type system already guarantees conformance to the schema.
- **Streaming and binary channels** (audio, video, tensors, raw buffers) pass their payloads in binary form at runtime. Their schemas describe the logical shape using JSON Schema's `contentMediaType`, `contentEncoding`, and `$ref` facilities, optionally with Blueprint-registered extensions for format-specific metadata.
- **Cross-process channels** (between engines in different processes, hosts, or devices) require a wire encoding. JSON is the default wire encoding; engine pairs MAY negotiate a more compact encoding (MessagePack, CBOR, Protobuf, Avro, Arrow) provided the payload round-trips without loss and continues to validate against the channel's declared schema.

Canonical serialization (for the Blueprint document itself, for snapshots, and for signed marks) is JSON and is defined in the canonical-form section of the specification. Runtime channel encoding is an implementation concern, not part of canonical form.

### 4.4 State

**State** is the evolving data a compound carries over time. A compound's state is declared as a named collection of fields, each with a schema.

State is addressable only within the compound that declares it. State changes MUST be observable to the compound itself and MAY be exposed to other compounds through channels.

State MUST be serializable. A conforming engine MUST be able to snapshot the complete state of a Blueprint at any point and restore the Blueprint from that snapshot on any conforming engine of the same version. This property is the foundation of Blueprint's durability and portability guarantees.

### 4.5 Scope

A **scope** is the boundary within which state, channels, and marks are addressable. Every compound introduces a new scope. Scopes nest according to compound composition.

Scopes determine identity resolution, lifecycle, and capability inheritance. A name resolved within a scope MUST refer to the nearest enclosing declaration of that name. A compound's lifecycle is bounded by its scope: when a compound is instantiated, its scope begins; when it is destroyed, its scope ends, and all state and channels within the scope are released.

Scope also governs **capability propagation**: certain capabilities (access to a connection, permission to mutate state, authority to sign marks) flow from enclosing scopes to nested scopes unless explicitly restricted. Scope capability rules are defined in the capability section of the specification.

### 4.6 Schema

A **schema** is the typed shape of data. Blueprint uses [JSON Schema](https://json-schema.org/) (draft 2020-12) for all schema declarations: channel payloads, state fields, compound interfaces, and document-level schemas.

Schemas describe logical shape. They are not a prescription of runtime encoding. A value validates against a schema when its logical structure conforms to the schema's constraints, regardless of whether the value is currently represented as a JavaScript object, a Kotlin data class, a Python dict, a binary buffer, or a serialized byte stream.

Schemas MUST be declared inline or referenced by URI. Engines MUST validate all data crossing schema boundaries. Invalid data MUST be rejected with a diagnostic that identifies the offending field and the violated constraint.

### 4.7 Draft

**Draft** is the verb for authoring a Blueprint and the adjective for a Blueprint in active revision. A Blueprint in draft state MAY contain incomplete compounds, unresolved imports, or unsigned marks. Engines SHOULD NOT execute draft Blueprints in production environments; authoring tools and sandbox engines MAY execute them with appropriate warnings.

Draft state is not a separate file format. A Blueprint is in draft state when its top-level `status` field is absent or set to `"draft"`. A Blueprint transitions out of draft state by setting `status` to `"released"` and adding a release signature.

### 4.8 Mark

A **mark** is a coordination identifier stamped on compounds, channels, states, or the Blueprint itself. Marks carry authorship metadata: who made a change, when, why, and with what authority.

Every edit to a Blueprint SHOULD produce a mark. A mark is a JSON object with the following fields:

| Field | Required | Purpose |
| --- | --- | --- |
| `id` | MUST | A unique identifier for this mark (UUID or hash). |
| `target` | MUST | A pointer to the compound, channel, or element the mark is stamped on. |
| `author` | MUST | An identifier for the author (human, AI agent, or tool). |
| `timestamp` | MUST | RFC 3339 timestamp of when the mark was made. |
| `action` | MUST | What the author did: `create`, `modify`, `remove`, `approve`, `release`. |
| `rationale` | SHOULD | A human-readable explanation of the change. |
| `provenance` | MAY | A pointer to the prompt, conversation, ticket, or decision that produced this mark. |
| `signature` | MAY | A cryptographic signature for this mark (see §4.9). |

Marks form an ordered history. Conforming tools MUST preserve mark order across reads, writes, and forks. Marks MUST NOT be deleted; a revocation is itself a mark.

> **Lineage note.** The term *mark* is chosen from bookbinding, where a printer's mark (or *signature mark*) identifies a gathering of pages so that binders can assemble the book correctly. Blueprint's marks play the same coordination role: they tell any composer how pieces fit together and who is accountable for each piece, without requiring a central coordinator.

### 4.9 Signature

A **signature** is a cryptographic proof binding a mark (or a Blueprint as a whole) to a verifiable author. A signature is a JSON object with the following fields:

| Field | Required | Purpose |
| --- | --- | --- |
| `id` | MUST | A unique identifier for this signature. |
| `mark` | MUST | The identifier of the mark being signed, or `"blueprint"` for a whole-Blueprint signature. |
| `algorithm` | MUST | The signing algorithm (`ed25519`, `ecdsa-p256`, etc.). |
| `public_key` | MUST | The author's public key in a standard encoding. |
| `signature` | MUST | The signature bytes, base64url-encoded. |
| `key_binding` | SHOULD | A reference (URI, DID, or X.509 chain) establishing the author's identity. |

Engines MAY verify signatures at load time or on demand. Production deployments SHOULD require all marks of action `approve` or `release` to be signed. Unsigned marks are permitted but convey no authority.

The signable content for a mark is the canonical serialization of the mark with its `signature` field removed. The signable content for a whole Blueprint is the canonical serialization of the Blueprint with all signatures removed. Canonical serialization rules are defined in the canonical-form section of the specification.

---

## 5. Composition

Blueprints are composed from compounds. Composition is the central act of authorship and the defining capability of the specification.

### 5.1 Composition operators

Blueprint defines three composition operators:

- **Instantiate.** A compound `A` uses a compound `B` by instantiating it within `A`'s implementation. `B` becomes a child in `A`'s scope. This is the primary composition operator.
- **Import.** A Blueprint includes compounds from another Blueprint or from a registry through its `imports` field. Imported compounds are addressable by qualified name but otherwise behave identically to locally-defined compounds.
- **Overlay.** A compound MAY be specialized by overlaying field modifications on top of its base definition. Overlays MUST NOT change a compound's interface; they MAY adjust implementation details, defaults, and metadata. Overlays preserve the original compound's marks and add an overlay mark recording the modification.

### 5.2 Determinism

Composition is deterministic. Given the same Blueprint, the same engine version, and the same external inputs, the engine MUST produce equivalent runtime behavior on every execution.

This determinism is the foundation of portability. A Blueprint running on an engine in New York and an engine in Tokyo MUST produce the same output for the same input. Equivalence is defined up to scheduling order and non-observable internal state; externally visible behavior MUST match.

### 5.3 No external authority

Composition MUST NOT require a central authority at runtime. Marks coordinate composition, but marks are data within the Blueprint, not queries against a server. A Blueprint MUST be composable using only its own content and the content of its declared imports.

This property preserves offline operability and vendor independence. A Blueprint fetched from any source MUST be runnable without consulting any other service.

---

## 6. The runtime contract

A **conforming engine** is an implementation that honors the runtime contract defined in this section for the vocabulary terms it supports.

### 6.1 Load

On loading a Blueprint, an engine MUST:

1. Validate the document against the Blueprint JSON Schema for the declared version.
2. Resolve all imports and fail loudly if any cannot be resolved.
3. Validate every schema reference.
4. Verify signatures if signature verification is enabled.
5. Construct the compound graph rooted at the entry compound.

If any step fails, the engine MUST refuse to execute the Blueprint and MUST report a diagnostic identifying the failure.

### 6.2 Execute

At runtime, an engine:

- Instantiates compounds as their scope begins.
- Wires channels between compounds according to the Blueprint's composition.
- Maintains state inside each compound's scope.
- Delivers messages on channels according to channel-direction rules.
- Tears down compounds as their scope ends.

An engine MUST deliver channel messages in a manner consistent with the channel's declared delivery semantics (synchronous, asynchronous-ordered, asynchronous-unordered).

### 6.3 Persist

A conforming engine MUST be able to snapshot the complete state of a running Blueprint at any safe point and restore the Blueprint from that snapshot on any conforming engine of the same version. The snapshot format is defined in the persistence section of the specification.

### 6.4 Observe

A conforming engine SHOULD expose observability hooks for:

- Mark events: every runtime event that would produce a mark if authored.
- Channel activity: messages flowing on named channels.
- State transitions: changes to declared state fields.
- Lifecycle events: compound instantiation and destruction.

Observability is not required for conformance but is required for production-grade deployments.

---

## 7. Authorship

Blueprint is designed to be co-authored by humans, AI agents, and automated tools. The authorship model is defined by marks and signatures (§4.8, §4.9) and by the following rules.

### 7.1 Authors are first-class

Every change to a Blueprint SHOULD produce a mark identifying the author. Author identifiers are opaque strings; the specification does not prescribe the identity system. Common author forms include:

- Email addresses or usernames for humans.
- Model identifiers with operator attribution for AI agents (e.g., `ai:anthropic/claude-opus-4.7+operator:anton@example.com`).
- Tool identifiers for automated processes (e.g., `tool:eslint`, `tool:blueprint-cli`).

Identity binding is the role of signatures (§4.9).

### 7.2 AI and human marks are distinguishable

Marks produced by AI agents MUST carry author identifiers that clearly indicate the agent's nature. Conforming tools MUST display AI marks distinguishably from human marks when presenting the mark history.

### 7.3 Approval is an explicit action

A compound or Blueprint is **approved** when a mark of action `approve` is added by an author authorized to approve (as determined by the governance rules of the surrounding system; Blueprint does not prescribe an approval policy). Approval marks SHOULD be signed.

### 7.4 Release is a commitment

A Blueprint is **released** when its `status` is set to `"released"` and a signed mark of action `release` is added. Released Blueprints are immutable: any further change produces a new Blueprint with a new `id` or a new version.

---

## 8. Versioning and conformance

### 8.1 Specification versions

The Blueprint specification follows semantic versioning. Within a major version, changes are additive and backward-compatible. Across major versions, breaking changes are permitted and are accompanied by a migration specification.

### 8.2 Document versions

Every Blueprint declares a specification version in its `blueprint` field (§3.2). A Blueprint MAY declare its own application version in its `metadata` field; this is orthogonal to the specification version.

### 8.3 Conformance levels

An engine's conformance is described by:

- The **specification version** it targets.
- The **conformance class** it achieves: `core`, `full`, or a named subset.
- The **extensions** it supports.

The **core** conformance class covers the vocabulary and runtime contract defined in this document. The **full** conformance class covers core plus the feature packages listed in the conformance section of the specification. Named subsets (e.g., `read-only`, `validation-only`) are defined for tools that do not need full runtime execution.

---

## 9. Non-goals

Blueprint explicitly does not attempt to:

- **Prescribe a programming language.** Compounds are data; implementations are written in whatever language an engine chooses.
- **Prescribe a rendering technology.** Engines decide how to render compounds on their target platform.
- **Prescribe a persistence model.** Engines may persist state to any backing store that satisfies the serialization and restore contracts.
- **Replace developers.** Blueprint is a substrate for authorship, not a mechanism for eliminating the need for software expertise.
- **Compete with general-purpose programming languages.** Blueprint describes applications; general-purpose languages implement engines and tools.
- **Be a universal data format.** Blueprint is a format for applications. It is not a format for arbitrary data, documents, or messages.

---

## 10. Relationship to other specifications

Blueprint is designed to interoperate with, not replace, existing specifications.

- **JSON Schema.** Used directly for all schema declarations.
- **JSON-LD.** Not required. Blueprints MAY include JSON-LD context information in their `metadata` for tooling that wants to consume Blueprints as linked data.
- **OpenAPI.** Blueprints frequently consume APIs described by OpenAPI. A Blueprint compound MAY be automatically generated from an OpenAPI operation.
- **AsyncAPI.** Similar relationship to OpenAPI for asynchronous APIs.
- **CloudEvents.** Channel messages MAY be encoded as CloudEvents when emitted or consumed across system boundaries.
- **WebAssembly.** Blueprint engines MAY execute WebAssembly modules as compound implementations. The specification does not require it.
- **OCI.** Blueprint artifacts MAY be packaged and distributed as OCI artifacts.

Blueprint's OpenAPI analogy is structural: both are JSON-carried, schema-defined, vendor-neutral specifications for a class of artifact that was previously described inconsistently and proprietarily. OpenAPI did this for HTTP APIs; Blueprint does it for applications.

---

## 11. Governance

The Blueprint specification is developed openly. The long-term intent is to transfer stewardship to a neutral standards body. Until that transfer occurs, the specification is maintained in this repository under the Apache License 2.0.

A governance charter will be added when the specification reaches a stable 1.0 release.

---

## 12. Document status

This document is the overview of Blueprint specification version `0.1.0-draft`. Subsequent sections of the specification define:

- The complete JSON Schema for Blueprint documents.
- Detailed semantics for each vocabulary term.
- The canonical serialization rules.
- The capability and scope propagation rules.
- The snapshot and restore formats.
- The conformance test suite.

These sections are under active development and will be added to this directory as they stabilize.

---

*Authored by Anton Bursch. Apache License 2.0.*
