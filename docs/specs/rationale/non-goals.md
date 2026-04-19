# Non-goals

**Status:** Draft. Informative, not part of the normative specification.

---

## Why a specification needs an explicit non-goals document

Every successful specification is as much shaped by what it refuses to do
as by what it promises. HTML refused to be a layout language. JSON refused
to be a rich type system. OpenAPI refused to be an implementation framework.
Each of those refusals was costly in the short term — reasonable people
asked for the missing capability — and each was, in hindsight, load-bearing.
The refusal was what kept the artifact small, learnable, portable, and
durable.

Blueprint will be under the same pressure. In every round of spec work, and
in every implementation discussion, someone will ask Blueprint to take on
a concern that sits outside its current scope. Many of those requests will
be reasonable on their own terms. A few of them will be load-bearing in the
same way HTML's refusals were: accepting them would compromise what Blueprint
is trying to be.

This document collects the concerns Blueprint has decided not to absorb. It
is informative: the non-goals below do not impose normative requirements.
But they inform every normative decision the specification makes.

The overview document (§9) lists these in summary form. This document
explains each in depth and adds non-goals that the overview does not name.

---

## 1. Blueprint is not a programming language

Compounds are data. Their implementations are not. A compound's behavior
lives in code written by a human or a machine in some host language —
TypeScript, Python, Kotlin, Rust, Go, C#, Dart, Java — and an engine binds
that code to the compound at runtime. Blueprint does not attempt to define
a language in which that code is expressed.

The refusal matters because:

- Programming languages take decades to mature. Blueprint cannot wait.
- Programming languages divide their communities. Blueprint wants all of
  them at once.
- Every choice a language makes about syntax, types, concurrency,
  exceptions, and memory shapes what can be written in it. Blueprint must
  not impose those choices on the millions of engineers who work in other
  languages.

The decision has a cost: Blueprint cannot directly express behavior, only
reference it. That cost is real and is accepted deliberately. A Blueprint
document that cannot be understood without running its compounds is
working as designed.

## 2. Blueprint is not a rendering technology

Engines decide how to render. A compound declared to show a chart does
not specify pixels, colors, fonts, or a drawing model. The engine chooses
those, and different engines will make different choices.

The refusal matters because the same Blueprint must run on web browsers,
native desktop apps, mobile phones, embedded screens, command-line
environments, and interfaces that do not render at all — servers,
batch systems, AI agents. A specification that prescribes rendering
forecloses most of those contexts.

This is where Blueprint differs most sharply from the low-code tradition,
which typically binds a document format tightly to a rendering engine and
inherits its limits.

## 3. Blueprint is not a persistence model

Engines persist state however they choose. The only obligation is that a
state can be serialized against its schema and restored. An engine may use
SQLite, Postgres, a custom log, a flat file, Redis, an object store, or
nothing at all. Two conforming engines persisting the same Blueprint are
not required to use compatible storage.

The refusal matters because persistence is where most frameworks make their
most opinionated and least portable choices. Baking a persistence model
into the artifact would bind every Blueprint to a class of deployments.

## 4. Blueprint is not a protocol

A Blueprint describes an application; it does not describe a wire protocol.
Channels specify the *logical shape* of their payloads, not how those
payloads are transmitted. For in-memory channels the payload is a native
value; for streaming channels it is a raw buffer; for cross-process channels
the engine chooses the wire encoding.

The refusal matters because protocol choices are contextual: what is right
for a browser talking to a backend is wrong for two embedded devices on
a serial bus, and both are wrong for two processes on the same host
communicating over a ring buffer. Prescribing a protocol would make
Blueprint a bad fit for most of the contexts it needs to serve.

## 5. Blueprint is not a deployment model

Blueprint says nothing about containers, VMs, Kubernetes, serverless,
platforms-as-a-service, or where an engine runs. An engine is free to
interpret a Blueprint within any deployment topology the engine's
operator chooses. A single Blueprint may be split across a browser and
a backend by one engine and collapsed into a single process by another,
as long as both produce the observable behavior the specification requires.

The refusal matters because deployment is the single most volatile concern
in modern software. A specification that baked in a deployment model in
2010 would look absurd in 2026. Blueprint intends to outlive the current
deployment regime.

## 6. Blueprint is not an identity or authorization system

Marks identify authors. Signatures prove authorship. Neither establishes
*trust*: whether a particular author is authorized to make a particular
change, whether a runtime should accept a particular release, whether a
user should be allowed to perform a particular action, all lie outside
the specification.

The refusal matters because trust is organizational and contextual. The
same signed Blueprint is legitimate in one organization and unacceptable
in another. The specification provides the primitives for expressing and
verifying authorship; organizations provide the policy that decides what
to do with that information.

## 7. Blueprint does not replace software engineers

Blueprint is a substrate for authorship, not a mechanism for eliminating
the need for software expertise. Engineers remain the primary authors of
the vocabulary, the implementations, the engines, the authoring tools, and
the difficult compounds. Blueprint aims to make their work compose and
scale beyond them, and to allow non-engineers to participate meaningfully
where they have previously been excluded — not to do without engineers.

The refusal matters because the failure mode of a specification that
promises to replace engineers is to make itself useless to them and thus
to the people who would build everything it depends on. Blueprint's
primary audience is the engineers who will build engines, write
connections, design compounds, and author specifications for their
organizations. Everything else follows from their success.

## 8. Blueprint is not a general-purpose programming environment

A language model given a Blueprint cannot answer arbitrary computational
questions by interpreting it alone. A Blueprint describes the structure
of an application and the composition of its compounds; it does not carry
the implementations of those compounds' behavior, and it does not define
a general-purpose evaluation model.

The refusal matters because Blueprint is not competing with general-purpose
programming languages. It is cooperating with them. A Blueprint plus its
engine and its compound implementations constitutes an application;
Blueprint alone is only the structural part of that application.

## 9. Blueprint is not a universal data format

JSON is a universal data format. Blueprint is not. Blueprint is a format
for applications — structured in particular ways, bearing particular
authorship information, submitting to particular composition and runtime
rules. It is not suitable for arbitrary data, arbitrary documents, or
arbitrary messages.

The refusal matters because specifications that try to be universal end
up serving no particular purpose well. Blueprint is shaped by the
demands of applications: composition, scope, channels, state,
lifecycles, authorship. Those shapes are not right for every kind of
data and would be limiting if forced onto them.

## 10. Blueprint is not an AI safety specification

Blueprint supports AI authorship by making it distinguishable and
verifiable. It does not prescribe how AI systems should be trained,
evaluated, or constrained, nor does it define what makes an AI-authored
Blueprint "safe" for a given use. Those concerns are outside the
specification's scope. Organizations adopting Blueprint will bring their
own AI governance and use Blueprint's authorship primitives as inputs to
that governance.

The refusal matters because AI safety is a live and disputed field.
Tying the specification to any particular safety framework would date
the specification and compromise its neutrality.

## 11. Blueprint does not replace version control, issue tracking, or code review

Marks record authorship on Blueprint elements. They are not a substitute
for a version control system, an issue tracker, a code review tool, or a
project management system. A Blueprint may reference such systems; it
does not attempt to replace them. The authoring tools that produce
Blueprints will typically integrate with those systems, but that
integration is out of scope for the specification.

The refusal matters because the software industry has built decades of
tooling around version control and review, and that tooling is not going
away. Blueprint's job is to be a well-behaved citizen of the workflows
those tools enable, not to replace them.

## 12. Blueprint does not prescribe a user experience for authoring

Blueprint specifies the artifact and the contracts around it. It does not
specify what an authoring tool looks like, how it helps a user, or what
interaction model it uses. A visual node graph, a text editor, a natural
language interface, a conversational AI agent, a form-based configuration
tool, and a specialized DSL compiler are all valid authoring surfaces.
Each produces the same kind of artifact.

The refusal matters because authoring experience is where innovation
should be loud. Prescribing a single authoring model would foreclose most
of the experimentation that Blueprint is trying to enable.

---

## A test for proposed extensions

When a proposed addition to the specification is evaluated, each of the
non-goals above serves as a check. If the addition:

- requires choosing a language, binds to a specific protocol, embeds a
  persistence model, or assumes a deployment topology — it is probably
  outside scope;
- establishes a trust, authorization, or safety policy rather than
  providing a mechanism that enables such a policy — it is probably
  outside scope;
- duplicates capabilities already provided by version control, issue
  tracking, or review systems — it is probably outside scope;
- prescribes a particular authoring interface or tries to replace
  engineers — it is probably outside scope.

An addition that clears these checks is a candidate for inclusion. An
addition that fails one of them is rejected or, at most, accommodated as
an extension rather than a normative feature.

---

## Related reading

- [overview.md §9](../overview.md) — the short form of this list
- [ambition.md](ambition.md) — the ambition these non-goals protect
- [openapi-precedent.md](openapi-precedent.md) — how OpenAPI's refusals
  shaped what it became
- [extensions.md](../extensions.md) — how namespaced extensions allow
  some non-goals to be addressed outside the core specification
