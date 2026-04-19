# State

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines **state**: the evolving data a compound
> carries over time. State is scoped, typed, and serializable. This
> document fixes what a state field is, how it is declared and
> initialized, how it is mutated, how it relates to exposed state
> (the interface projection defined in compound.md), and the
> serialization guarantee that makes Blueprint snapshots portable
> across engines.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [compound.md](compound.md) — state is declared inside a composed compound
- [scope.md](scope.md) — state lives in a scope and its lifecycle is the scope's
- [schema.md](schema.md) — every state field has a declared schema
- [channel.md](channel.md) — the channel / state distinction
- [../overview.md](../overview.md) — the surrounding specification

---

## 1. What a state is

A **state** is a named, typed, serializable field owned by a single
compound instance and mutable by that instance over time.

A state field has:

- a **name**, unique within the compound that declares it;
- a **schema** ([schema.md](schema.md)) describing the logical shape
  of the field's value;
- an **initial value** specification — either an explicit value, a
  default derived from the schema, or a computation over the
  compound's parameters (§3);
- a **scope binding**: every state field lives within exactly one
  scope — the scope of the compound instance that declares it.

State is what a compound **remembers between moments of behavior**:
counts, selections, positions, accumulated samples, session data,
caches, anything whose value the compound must carry forward.

### 1.1 State is private by default

A state field is addressable only within the scope that declares it.
A parent scope MUST NOT read or mutate a child's state field directly.
Cross-scope exposure of state is explicit and goes through the
compound's interface:

- **observing** state from outside the scope is through an **exposed
  state** member on the interface ([compound.md](compound.md) §2.3);
- **setting** state from outside is through an **input** on the
  interface whose reception triggers a mutation in the compound's
  implementation.

There is no other path. A compound's state is not a shared variable,
a global, or an ambient bag. It is owned by the instance and bounded
by the scope.

### 1.2 State is serializable

Every state field MUST be serializable to JSON in a form that
validates against its schema. Serialization is what makes snapshot
and restore possible ([../overview.md](../overview.md) §6.3). A
compound whose state is not representable as JSON violates this
document; such a compound is non-conforming.

Binary values in state fields follow the same rules as binary values
on channels: the schema describes the logical shape, the engine
preserves the bytes, and JSON Schema's `contentMediaType` /
`contentEncoding` carry the format information (see
[schema.md](schema.md) §6).

---

## 2. Declaration

A composed compound ([compound.md](compound.md) §4) declares its state
as a named map of fields. Each field declaration provides a name, a
schema, and initialization information.

Illustrative shape:

```json
{
  "state": {
    "count":      { "schema": { "type": "integer" }, "initial": 0 },
    "selection":  { "schema": { "type": "string", "enum": ["a","b","c"] }, "initial": "a" },
    "samples":    { "schema": { "type": "array", "items": { "type": "number" } }, "initial": [] }
  }
}
```

Names MUST be unique within a single compound's state declaration.
Names MUST follow the same identifier rules as other in-document
names ([compound.md](compound.md) §7.1).

The exact syntactic form is finalized in the Blueprint JSON Schema in
Round 4.

---

## 3. Initialization

Every state field has an initial value. Initialization runs before
the compound's scope begins ([scope.md](scope.md) §3.1). The
initialization sources, in order of precedence:

1. an **explicit `initial` value** declared on the field — used as-is;
2. a **computed initial** expressed in terms of the compound's
   parameters (the expression language and its precise form are
   finalized in Round 5 semantics);
3. a **default implied by the schema** — a value satisfying the
   schema's `default` keyword;
4. if none of the above are present, a **schema-derived zero value**
   (`null`, `""`, `0`, `false`, `[]`, `{}` as appropriate) where the
   schema permits it. The engine MUST reject the Blueprint if no
   valid initial value can be derived.

An initial value MUST validate against the field's schema. An engine
MUST reject any initial value that does not validate.

Initialization is deterministic. Given the same compound declaration
and the same parameter bindings, initialization MUST produce the same
initial state across engines of the same specification version.

### 3.1 Initialization ordering within a scope

When a compound's scope begins ([scope.md](scope.md) §3.1), every
state field in that scope is initialized before any nested compound
is instantiated and before any input begins delivering values. A
compound's implementation MUST observe its own state as fully
initialized before it observes any input.

---

## 4. Mutation

State is mutated by the compound that owns it. Mutation happens in
response to:

- an input value arriving on one of the compound's interface inputs;
- a value arriving on an internal channel the compound has wired to
  its own state;
- a primitive-provided or engine-provided update (for primitive
  compounds and engine-extended compounds);
- an initialization computation completing (§3).

A state field MUST NOT be mutated from outside its owning scope by
any direct path. A parent that needs to influence a child's state
MUST do so through the child's declared interface.

### 4.1 Mutation granularity

A state field MAY be mutated at any JSON-addressable location within
its value, not only at the root. For example, a state field whose
schema is `{ "type": "object", "properties": {...} }` MAY have
individual properties updated without the whole object being
replaced. The declarative form for such partial updates is finalized
in Round 5 semantics.

### 4.2 Atomicity

Mutation at the level of a single delivered message is **atomic**:
between successive messages, the compound's state is coherent and
observable by exposed-state consumers. A single message MUST NOT
leave the compound's state in a half-updated, externally-observable
intermediate form.

Transactional mutation across multiple state fields within a single
compound in response to a single message is RECOMMENDED. The
specific transactional model is engine-defined for 0.1.0-draft.

### 4.3 Determinism

Given the same compound declaration, the same parameter bindings,
the same initial state, and the same sequence of input messages,
a compound MUST produce the same sequence of state values and the
same sequence of output messages across engines of the same
specification version. This is the composition-wide determinism
claim of [../overview.md](../overview.md) §5.2 applied to state.

---

## 5. Observation

A compound observes its own state directly (no other entity can).
Observation from outside the compound's scope goes through an
**exposed state** member on the compound's interface
([compound.md](compound.md) §2.3).

An exposed state member projects one or more internal state fields
(or a derived value over them) into a read-only value addressable by
the enclosing scope. The projection:

- has its own schema, declared on the interface;
- MUST validate against that schema on every change that would affect
  its value;
- is surfaced through an observation mechanism defined by the engine
  (a current-value read, a change callback, a signal, a reactive
  notification — the specification does not fix which).

An exposed state is not a channel. A consumer reading an exposed
state sees "the current value of X", not "the ordered sequence of
values X has taken". If the consumer needs the sequence, the
compound should declare an output channel in addition to the exposed
state.

---

## 6. Snapshot and restore

A conforming engine MUST be able to:

1. **Snapshot** the complete state of a running Blueprint at any
   safe point as a JSON document;
2. **Restore** the Blueprint from that snapshot, on any conforming
   engine of the same specification version, to a state
   behaviorally equivalent to the state at which the snapshot was
   taken.

The snapshot includes, at minimum:

- the identity of the Blueprint (its `id` and `blueprint` version);
- the parameter values of every live instance;
- the current value of every live state field;
- enough instance-graph information to reconstruct the scope tree.

The canonical snapshot format is finalized in
[../semantics/lifecycle.md](../semantics/lifecycle.md) (Round 5,
forthcoming). For 0.1.0-draft, the specification requires only that:

- the format is JSON;
- the serialization round-trips (a snapshot taken on engine A and
  restored on engine B produces behaviorally equivalent execution);
- the schema of every serialized state field validates the serialized
  form.

Snapshot portability is the basis of Blueprint's durability claim: a
Blueprint's running state is not trapped in any one engine's memory.

### 6.1 Snapshot is not canonical form

A snapshot captures the *running state* of a Blueprint. The
**canonical form** defined in [canonical-form.md](canonical-form.md)
(forthcoming in this round) is a canonical serialization of the
Blueprint *document itself*, separate from any running state.
Signatures sign over canonical form, not over snapshots.

---

## 7. State versus channel

Restated from [channel.md](channel.md) §8. The choice between state
and channel is about the mental model of the interaction:

| Interaction | Use |
| --- | --- |
| "Events happened in this order": clicks, messages, samples, notifications | **Channel** (stream) |
| "This thing currently has this value": selection, theme, cursor, counter | **State** (exposed, if externally visible) |

Authors MAY use both for different parts of the same compound's
surface. The two are complementary, not mutually exclusive.

---

## 8. Normative requirements

A conforming Blueprint document:

1. MUST declare every state field with a name and a schema.
2. MUST give every state field a derivable initial value (explicit,
   computed, schema-default, or schema-derived zero).
3. MUST NOT reference a state field outside the compound that
   declared it except through the compound's interface.

A conforming engine:

4. MUST validate every state field's initial value against its
   schema; MUST reject Blueprints whose initial values do not
   validate.
5. MUST initialize every state field in a compound's scope before
   the scope begins observing inputs.
6. MUST NOT permit mutation of a compound's state field from outside
   the compound's scope except through the compound's declared
   interface.
7. MUST preserve atomicity of state change within a single message:
   no external observer SHALL see a partially-updated state mid-
   message.
8. MUST produce deterministic state and output sequences given
   identical compound declarations, parameter bindings, initial
   states, and input sequences.
9. MUST be able to snapshot the complete state of a running
   Blueprint as a JSON document whose schema validates against the
   declared schemas of all live state fields.
10. MUST be able to restore a Blueprint from a snapshot produced by
    any conforming engine of the same specification version, and
    the restored Blueprint MUST be behaviorally equivalent to the
    state at snapshot.

A conforming authoring tool:

11. MUST NOT produce state declarations whose initial values do not
    validate against their schemas.

---

## 9. Examples

### 9.1 A simple counter state

```json
{
  "state": {
    "value": { "schema": { "type": "integer", "minimum": 0 }, "initial": 0 }
  }
}
```

The compound carries a non-negative integer across messages. A
snapshot of a running instance will include the field's current value.

### 9.2 A structured session state

```json
{
  "state": {
    "session": {
      "schema": {
        "type": "object",
        "properties": {
          "user_id":     { "type": "string" },
          "started_at":  { "type": "string", "format": "date-time" },
          "last_seen":   { "type": "string", "format": "date-time" },
          "preferences": { "type": "object" }
        },
        "required": ["user_id", "started_at"]
      },
      "initial": { "user_id": "$params.user_id", "started_at": "$now", "last_seen": "$now", "preferences": {} }
    }
  }
}
```

A richer state with computed initialization referring to the
compound's parameters and to an engine-provided `$now`. The exact
syntax of computed initialization is finalized in Round 5.

### 9.3 State plus exposed state

A compound that counts and exposes the count:

```json
{
  "interface": {
    "exposed_state": {
      "count": { "schema": { "type": "integer", "minimum": 0 } }
    }
  },
  "state": {
    "value": { "schema": { "type": "integer", "minimum": 0 }, "initial": 0 }
  }
}
```

An internal state field `value` is projected to the exposed state
`count` by a wiring in the compound's implementation. Consumers of
this compound read `count` through the interface; they never touch
`value`.

---

## 10. Open questions

1. **Large-state streaming snapshot.** A compound's state may be too
   large for a single JSON snapshot. Chunked or streaming snapshots
   are plausible; deferred.
2. **Time-travel and mutation history.** Authoring tools may want to
   reconstruct the sequence of state values a compound has taken.
   The spec does not require this, but an optional mark-driven
   history mode is plausible. Deferred.
3. **Schema evolution of long-lived state.** When a Blueprint's
   version changes but running instances carry older state, the
   engine needs a migration policy. Deferred to
   [../versioning.md](../versioning.md) (Round 6).
4. **Transactional mutation across compounds.** The specification
   provides per-compound, per-message atomicity. Cross-compound
   transactions are deferred.
5. **Partial-update declarative syntax.** The form in which authors
   express "update this property of this state field" is finalized
   in Round 5 semantics.

---

## 11. Document status

This document is version `0.1.0-draft` of the state vocabulary
document. It is normative within the matching Blueprint specification
version.

---

*Authored by Anton Bursch. Apache License 2.0.*
