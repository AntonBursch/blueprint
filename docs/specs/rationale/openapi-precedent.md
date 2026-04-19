# The OpenAPI precedent

**Status:** Draft. Informative, not part of the normative specification.

---

## Why this document exists

Blueprint is a new specification, but it is not an unprecedented one. The
closest successful analogue is the OpenAPI Specification (originally
Swagger), which since roughly 2011 has established that:

- a portable, human- and machine-readable artifact describing software can
  be a meaningful object of engineering;
- such a specification can be maintained by a neutral body without any one
  vendor owning the outcome;
- a rich and durable tooling ecosystem can form around a shared contract
  without fragmenting it;
- non-technical stakeholders can meaningfully participate in a design
  described in a structured document that is not itself code.

The Blueprint project draws on this precedent deliberately. Understanding
what Blueprint takes from OpenAPI, what it does differently, and what it
is attempting with no precedent at all is the easiest way to describe the
shape of Blueprint's ambition to a reader who already knows how OpenAPI
works.

This document is informative. It does not impose requirements.

---

## What Blueprint takes from OpenAPI

### 1. The artifact is the product

The OpenAPI Specification is not a tool. It is a document format. Everything
that makes OpenAPI valuable — Swagger UI, codegen, linters, mock servers,
gateway integrations, security auditing — exists because the artifact is
stable and widely understood. The ecosystem forms around the specification,
not around any particular implementation.

Blueprint takes the same stance. The normative thing is the specification
and the artifact it describes. Engines, authoring tools, validators, and
stores are implementations of the specification and compete with each
other. None of them *is* Blueprint.

### 2. JSON as the carrier

OpenAPI standardized on JSON (and accepts YAML as an equivalent textual
form). This choice was load-bearing. JSON is diff-friendly, greppable,
universally parseable, easy to validate, easy to generate, easy for AI
systems to read and write, and culturally neutral between programming
languages and ecosystems.

Blueprint carries the same choice forward. The canonical serialization
of a Blueprint is JSON. YAML or other textual representations may be used
for authoring, but JSON is the form in which a Blueprint is exchanged
between systems, hashed, and signed.

### 3. A version field, not a version library

Every OpenAPI document declares its version in a top-level field
(`openapi: 3.1.0`, etc.). This is trivial in appearance and profound in
practice. It means tools can correctly interpret any document they read,
backward and forward compatibility are negotiable, and the specification
can evolve without breaking the existing corpus.

Blueprint follows the same pattern with its `blueprint` top-level field.

### 4. Neutral governance

OpenAPI moved from a single-company specification (Swagger, held by
SmartBear) to a neutral foundation (the OpenAPI Initiative under the Linux
Foundation). This move was essential: organizations cannot safely build
their public interfaces on a format controlled by a single vendor that
might later be acquired, shelved, or weaponized against them.

Blueprint commits to the same governance trajectory. The specification is
published under the Apache License, Version 2.0, and the intent is that
the specification will move to a neutral foundation before version 1.0.
See the governance section of the overview document for details.

### 5. A conformance contract between authors and implementers

OpenAPI separates the concerns of authors (people producing documents),
consumers (people writing clients against documents), and implementers
(people building tools that read, write, or render documents). A single
specification binds all three groups to a shared contract.

Blueprint does the same, with the contract extended to cover runtime
behavior rather than just syntactic validity.

### 6. Extensible without forking

OpenAPI reserves a namespace (`x-` prefix) for vendor extensions and
requires tools to tolerate unknown extensions. This is how OpenAPI has
accommodated decades of vendor-specific behavior without fracturing the
format. A document using extensions is still a valid OpenAPI document.

Blueprint adopts a similar philosophy. The specific mechanism is different
and will be defined in the extensions document, but the commitment to
namespaced extensibility without forking is shared.

---

## Where Blueprint departs from OpenAPI

OpenAPI and Blueprint are not the same kind of document. The differences
below are not criticisms of OpenAPI; they are statements of what Blueprint
is trying to be that OpenAPI, by design, is not.

### 1. Descriptive vs. constitutive

OpenAPI describes the surface of a running system. The system itself
exists independently, written in some general-purpose language. The
OpenAPI document records what the system's HTTP interface looks like.
This is descriptive: a true OpenAPI document is one that matches the
real behavior of the server.

A Blueprint is not descriptive in this sense. It is constitutive: the
document *is* the application. A conforming engine loading the Blueprint
produces the application described. There is no separate implementation
the document is describing after the fact.

This is the single deepest difference. It changes what the artifact must
contain, how it must be validated, and what a conforming engine must do.

### 2. Runtime behavior, not just syntax

OpenAPI's validity rules are primarily syntactic: does the document
conform to the OpenAPI schema? A few semantic checks exist (matching
path parameters, consistent response types), but most of the semantic
load is carried by the implementation being described.

Blueprint specifies runtime behavior. Composition has rules. Lifecycle
has rules. Channels have delivery semantics. States have initialization
semantics. Two conforming engines loading the same Blueprint must produce
the same composition, the same scope structure, the same initial state,
and the same observable behavior for every clause covered by the
specification. OpenAPI has no equivalent obligation.

### 3. First-class authorship

OpenAPI documents have a single informational author field (`info.contact`)
and no other authorship structure. In practice, authorship is tracked by
the version control system the document happens to live in.

Blueprint treats authorship as first-class. Marks attach authorship to
individual elements of the document. Signatures make authorship
cryptographically verifiable. Human authors and AI authors are distinguished
without either being privileged. This structure has no direct precedent in
OpenAPI.

### 4. Composition

OpenAPI has schema composition (`$ref`, `allOf`, `oneOf`) and has added
support for overlays in recent versions. This is useful but limited: the
primary composition mechanism is textual reference, not the instantiation
of a reusable unit.

Blueprint is composition-first. The compound is the reusable unit of
design, and composition is how any non-trivial Blueprint is built. The
rules for instantiation, import, and overlay are normative and
deterministic. OpenAPI does not attempt this because it does not need to.

### 5. Durability across execution contexts

An OpenAPI document describes an HTTP API. It is bound to a specific
protocol, a specific interaction style, and an implicit single execution
context (a server handling requests).

A Blueprint describes an application. An application may span client and
server, synchronous and streaming, in-memory and cross-process, browser
and backend and edge. The specification must accommodate all of these
without forcing a particular execution model. The three-category treatment
of channels (in-memory, streaming/binary, cross-process) in the overview
is a direct consequence of this requirement; OpenAPI does not need such a
distinction because its scope is narrower.

### 6. Symmetric human/AI co-authorship

OpenAPI was designed in an era when documents were written by humans and
read by humans and tools. Blueprint is designed for an era in which
documents are regularly written by AI systems, reviewed by humans,
regenerated by AI, annotated by humans, and so on. The symmetry of the
two author kinds is designed into the vocabulary from the start rather
than retrofitted.

---

## What has no precedent in OpenAPI

Three aspects of Blueprint have no real analogue in the OpenAPI tradition
and cannot be evaluated by comparison to it.

### 1. The engine contract

OpenAPI tools are diverse — editors, linters, renderers, gateways — but
none of them *is* the described system. A Blueprint engine is different.
It loads a Blueprint and produces a running application. Multiple
conforming engines must produce equivalent applications. This is a
stronger contract than anything OpenAPI imposes on its tools, and it is
the contract on which the plurality of engines depends. The engine
contract document in the conformance directory exists to state this
obligation plainly.

### 2. Marks and signatures

An OpenAPI document carries no authorship trace beyond version control
metadata. Blueprint makes authorship part of the artifact itself. A
Blueprint released by its authors carries cryptographic proof of who
approved it and what they approved. This is a deliberate departure —
an attempt to close a gap that becomes acute the moment AI systems
participate in authorship at scale.

### 3. Released immutability

OpenAPI documents are versioned but not immutable. A released document
can be edited and republished with the same version. This is acceptable
for OpenAPI because the document describes a separately-running system
and the system is the source of truth.

A Blueprint that has been released is immutable. New work is a new draft
that forks from the previous release. This commitment is what makes a
Blueprint a stable reference — something a regulator, auditor, or a
future implementer can rely on to mean exactly what it meant when it was
released. OpenAPI does not need this commitment because its documents
are not constitutive.

---

## Using the precedent carefully

OpenAPI is not a template to be imitated in every detail. It is a piece
of evidence that the kind of thing Blueprint is trying to be can succeed.
Where OpenAPI's choices serve Blueprint's ambition, Blueprint adopts them.
Where OpenAPI's choices were shaped by its narrower scope, Blueprint
departs from them deliberately.

The rule when designing normative parts of the Blueprint specification:

1. If OpenAPI has solved a problem and the solution generalizes, take the
   solution. Do not reinvent it.
2. If OpenAPI has solved a problem in a way that only works because
   OpenAPI is descriptive and single-protocol, do not copy the mechanism —
   copy the intent, and design a mechanism suitable for a constitutive
   multi-context artifact.
3. If OpenAPI has no equivalent, the problem is genuinely new. Solve it
   on Blueprint's own terms and document that it is new.

---

## Related reading

- [ambition.md](ambition.md) — the ambition this document serves
- [co-authorship.md](co-authorship.md) — the authorship model that has no
  precedent in OpenAPI
- [non-goals.md](non-goals.md) — what Blueprint explicitly is not
- [prior-art.md](prior-art.md) — broader prior art, including OpenAPI and
  other systems
