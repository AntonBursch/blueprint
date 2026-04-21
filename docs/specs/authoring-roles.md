# Authoring Roles

**Status:** Draft stub
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

> This specification defines the roles that author Blueprints and their compounds, the boundary between code and composition, and the constraints a conforming authoring tool MUST enforce to preserve Blueprint's audit guarantees.

---

## 1. Purpose

Blueprint is built on the premise that an application is **legible by construction**: anyone inspecting a Blueprint can see what compounds it uses and how they are wired, and every compound it references is an artifact that a human engineer has reviewed and approved.

This property depends on a structural boundary between two activities:

- **Writing code** — producing the imperative implementation inside a compound.
- **Authoring a composition** — wiring reviewed compounds into an application.

Collapsing the boundary — for example, by letting AI emit arbitrary code into an app on behalf of a non-engineer — destroys the audit guarantee. Code generated faster than it can be reviewed is, operationally, unreviewed. This specification defines how the boundary is maintained.

The rationale for this principle, the organizational and regulatory motivations behind it, and the contrast with code-generation platforms are discussed in [../rationale/authoring-roles.md](rationale/authoring-roles.md) (to be written).

## 2. Roles

*(To be written. Expected roles: engineer, composer, reviewer, AI assistant. Each role is defined by what it MAY and MUST NOT produce into a Blueprint, and by the accountability attached to its marks.)*

## 3. The code/composition boundary

*(To be written. Defines precisely what counts as "code" for the purposes of this specification — imperative implementations inside compounds, processors, connections, and widgets — and what counts as "composition" — the references, wirings, and configurations that form a Blueprint's compound graph. Addresses edge cases: configuration values, expression languages, templates, schemas.)*

## 4. AI assistance

*(To be written. Specifies what AI tooling MAY and MUST NOT do in each authoring role. Composition assistance is permitted. Code generation on behalf of a non-engineer is prohibited. Code generation used by an engineer is permitted only when the engineer reviews and signs the resulting artifact.)*

## 5. Provenance and signatures

*(To be written. Defines how marks and signatures on compounds carry the engineer's accountability forward, how reviewers are recorded, and how an engine or tool validates that a Blueprint's compounds all trace to approved authors.)*

## 6. Conforming authoring tools

*(To be written. Defines the constraints a conforming authoring tool MUST enforce to preserve the boundary: what UI affordances MUST NOT be offered to non-engineer roles, how AI suggestions MUST be presented for review, and how to surface missing provenance to users.)*

---

## Status

This document is a stub. It will be filled in after the adapter, engine, extension, and compound contracts are drafted, because those specifications establish the concrete surfaces this document constrains. The principle it formalizes is stable and referenced from the [overview](overview.md) § 1.4.
