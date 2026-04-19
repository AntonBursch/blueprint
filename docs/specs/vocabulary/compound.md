# Compound

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the **compound**: Blueprint's unit of composition.
> Every Blueprint is made of compounds and nothing else. This document
> fixes what a compound is, what its interface looks like, how compounds
> compose, and the invariants a conforming engine and a conforming
> authoring tool MUST preserve.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

Normative cross-references in this document:

- [scope.md](scope.md) — the scope introduced by each compound instantiation
- [channel.md](channel.md) — the pathway by which values cross an interface
- [state.md](state.md) — the evolving data a compound carries over time
- [schema.md](schema.md) — the typed shape of every declared surface member
- [mark.md](mark.md) — authorship trace attached to a compound
- [../overview.md](../overview.md) — the surrounding specification
- [../rationale/ambition.md](../rationale/ambition.md) — why the compound model

This document is normative. The rationale this document is accountable to is
established informatively in [../rationale/ambition.md](../rationale/ambition.md)
and [../rationale/co-authorship.md](../rationale/co-authorship.md).

---

## 1. What a compound is

A **compound** is a named, reusable declaration inside a Blueprint document
that has four things:

- an **interface** — the only surface through which the compound interacts
  with anything outside itself;
- an **implementation** — either a composition of other compounds (a
  *composed compound*) or a reference to a behavior fixed by the
  specification, by the engine, or by a registered extension (a
  *primitive compound*);
- an **identity within the document** — a name, unique within the Blueprint's
  `compounds` object, by which the compound is referenced;
- zero or more **marks** — authorship trace attached to this declaration,
  defined in [mark.md](mark.md).

A compound declaration describes a *kind of thing*. At runtime, a compound
is **instantiated**. Each instantiation creates a new, independent runtime
object whose boundary is a **scope** (see [scope.md](scope.md)).

### 1.1 The compound–scope correspondence

Every compound instantiation introduces exactly one scope. Every scope in a
running Blueprint exists because exactly one compound was instantiated. The
correspondence is 1:1.

This correspondence is a defining property of Blueprint and is the reason
`scope.md` is a thin companion document: a scope is not a separate
authored object. An author creates a compound; the engine creates the
scope when the compound is instantiated.

### 1.2 What a compound is not

A compound is not a function: compounds persist over time, carry state,
and communicate over channels, none of which functions do.

A compound is not a process: a compound does not imply a thread, a
worker, a fiber, or any particular execution substrate. Scheduling is an
engine concern.

A compound is not a component in a UI sense: a compound MAY describe a
unit of UI, but it MAY equally describe a unit of logic, a unit of data
access, a unit of authorization, a durable workflow, an external
connection, or any other coherent whole. The word is deliberately
substrate-neutral.

---

## 2. The declared surface

A compound's **interface** is the complete, declared, typed surface
through which the compound interacts with anything outside its own
scope. An interface has four kinds of member:

| Member            | Direction            | Changes over time? | Purpose                                                  |
| ----------------- | -------------------- | ------------------ | -------------------------------------------------------- |
| **parameter**     | inward, static       | no                 | A configuration value fixed at instantiation.            |
| **input**         | inward, dynamic      | yes                | A channel flowing values into the compound over time.    |
| **output**        | outward, dynamic     | yes                | A channel flowing values out of the compound.            |
| **exposed state** | outward, read-only   | yes                | A named projection of internal state, readable by caller.|

An interface MUST declare each of its members with a name and a schema
(see [schema.md](schema.md)). Names MUST be unique within a single
interface. A compound's interface declaration is normative: an engine
MUST reject any wiring that references a member not declared on the
interface.

### 2.1 Parameters versus inputs

Parameters and inputs are both inward-flowing, but they are categorically
different:

- A **parameter** is part of the *static shape of an instance*. Its value
  is fixed at the moment of instantiation and does not change while the
  instance exists. Changing a parameter value requires a new instance.
- An **input** is part of the *dynamic life of an instance*. Values arrive
  over time and are consumed over time; successive values are expected.

This distinction matters for three reasons:

1. **Determinism.** Parameters are part of the scope's identity; inputs
   are not.
2. **Snapshots.** A snapshot MUST record current parameter values once
   per instance; input traffic is recorded differently (see
   [../semantics/lifecycle.md](../semantics/lifecycle.md) when written).
3. **Agent reasoning.** Agents reasoning about a Blueprint can determine
   the instance graph from parameters alone, independent of runtime
   traffic.

An engine MUST treat parameter values as fixed for the duration of an
instance. An engine MUST NOT silently reinstantiate a compound when a
parameter value appears to change; such a change requires an explicit
new instantiation at the authoring layer.

### 2.2 Outputs and events

Blueprint does not declare a separate "event" interface member. An
*event* is a convention: an output channel whose schema describes a
discrete notification (typically carrying a type tag and a payload) and
whose delivery semantics are suited to discrete occurrences.

Authors and tools MAY use the word "event" in documentation and UI.
Normatively, an event is an output.

> **Note to specification readers.** The Overview
> ([../overview.md](../overview.md) §4.2) enumerates "inputs, outputs,
> and events" when naming a compound's interface. That wording is
> informative and intentionally plain-language. This document is
> normative and collapses the three into the list given in this section.
> A future overview revision will be aligned to this document.

### 2.3 Exposed state

An **exposed state** member is a read-only projection of a compound's
internal state, addressable through the interface by the enclosing
scope. Exposed state is *not* a channel: it has an identity, a schema,
and a current value, but no ordered stream of messages. Consumers of
exposed state MAY observe changes through observation hooks defined by
the engine (see [channel.md](channel.md) §delivery for the distinction
between streamed and observable surfaces).

Exposed state exists because some interactions are more naturally
described as "X's current value is Y" than as "X has emitted Y then Z
then Y". Both forms are supported; authors choose.

### 2.4 What is not on the interface

A compound's interface does not include:

- internal state (always private to the compound's own scope);
- nested compounds (always private to the compound's implementation);
- internal channels (always private);
- the implementation itself (opaque to any consumer).

---

## 3. Encapsulation: the central invariant

The compound model's load-bearing rule:

> A compound MUST NOT observe or influence anything outside its own
> scope except through its declared interface. A compound MUST NOT
> observe or influence anything inside a child compound's scope except
> through the child's declared interface.

Corollaries:

1. No identifier outside a compound's scope may be referenced by name
   inside it except through an interface member or an imported compound
   name (see §6.2).
2. A parent MUST NOT read a child's internal state. A parent MAY read a
   child's exposed state.
3. Two sibling compound instances MUST NOT share state directly. They
   communicate through channels wired by their common parent (see §5).
4. A compound's implementation MAY be replaced by a different
   implementation with the same interface without any externally
   observable change beyond behavior differences the interface permits.

These corollaries are the basis of every property Blueprint claims about
compounds: that they can be independently reviewed, independently
signed, independently swapped, and independently reasoned about. If
encapsulation is weakened, those properties degrade proportionally.

A conforming engine MUST enforce encapsulation. A conforming authoring
tool MUST NOT produce wiring that violates encapsulation.

---

## 4. Composed compounds and their internals

A **composed compound** declares its implementation as a composition of
other compounds. Inside the implementation, an author MAY declare:

- **nested compound instances** — instantiations of other compounds
  (local, imported, or primitive), each with a locally unique name;
- **internal channels** — named pathways that connect nested instances'
  inputs and outputs and, where authorized by the enclosing compound's
  interface, connect nested instances to the enclosing interface;
- **state** — named, typed, serializable fields owned by this compound
  and, by default, private to it (see [state.md](state.md)).

Wiring is **declarative**. The implementation document says *what is
connected to what*; the engine is responsible for making those
connections behave according to the channel's declared delivery
semantics (see [channel.md](channel.md) and the forthcoming
[../semantics/channel-delivery.md](../semantics/channel-delivery.md)).

### 4.1 Leaf compounds

A **leaf compound** is a composed compound with no nested compound
instances. A leaf compound MAY still declare state, internal channels,
and (through primitive references, §5) arbitrary behavior. Leaf
compounds are where logic ultimately lives.

### 4.2 Private by default

All internals are private by default. Making an internal value available
to the outside is an explicit act: declaring an interface member
(parameter, input, output, or exposed state) and wiring an internal
channel or state to it.

A compound's implementation MUST NOT be readable from outside the
compound's scope by any means other than the compound's interface.
Engines that provide introspection or debugging surfaces MUST treat them
as operator-facing capabilities, not authored-compound capabilities.

---

## 5. Primitive compounds

Not every compound has a Blueprint-authored implementation. A
**primitive compound** is a compound whose behavior is fixed by:

- the Blueprint specification itself (a *core primitive*); or
- the engine that loaded the Blueprint (an *engine-provided primitive*);
  or
- a registered extension (an *extension-provided primitive*).

A primitive compound is referenced the same way any other compound is
referenced: by name. The Blueprint document does not distinguish
primitives from composed compounds in its reference syntax. The engine
is responsible for recognizing which references resolve to primitives
and for providing their behavior.

### 5.1 Why primitives are not a separate vocabulary member

Keeping primitives out of the vocabulary keeps the vocabulary small. The
document-level concept is "a compound". Whether a compound is composed,
core-primitive, engine-primitive, or extension-primitive is a
*resolution outcome*, not a *declaration kind*. This choice means:

- Adding a new primitive does not change the specification version.
- A Blueprint that uses a primitive is syntactically identical to a
  Blueprint that composes its own equivalent.
- Swapping a primitive for a composed compound (or vice versa) is
  invisible at the reference site.

### 5.2 Conformance for primitives

An engine claiming conformance to a specification version MUST provide
every core primitive defined for that version. The list of core
primitives is maintained in a registry document
(forthcoming, under `primitives/`), outside this vocabulary document, so
that adding a primitive is an additive ecosystem act rather than a
vocabulary revision.

An engine MAY provide engine-specific primitives. A Blueprint that
references an engine-specific primitive is bound to that engine family
and MUST declare the required engine capability in the forthcoming
capability section.

Extension-provided primitives are resolved through the extension
mechanism defined in [../extensions.md](../extensions.md) (forthcoming,
Round 6). Until extensions are normative, primitive references other
than core are out of scope for this draft.

---

## 6. Composition operators

Blueprint defines three composition operators. All three are already
named in [../overview.md](../overview.md) §5.1; this section fixes their
semantics normatively.

### 6.1 Instantiate

**Instantiation** uses a compound within another compound's
implementation. The result is a nested compound instance inside the
enclosing compound's scope.

An instantiation:

- MUST reference the instantiated compound by name (either a
  document-local name or a qualified imported name);
- MUST provide a value for every parameter declared on the instantiated
  compound's interface;
- MUST be assigned a **local name** by the enclosing compound, unique
  within the enclosing compound's implementation;
- MAY wire the instance's inputs and outputs to channels visible in
  the enclosing compound's scope;
- MAY wire the instance's exposed states to observation surfaces
  visible in the enclosing compound's scope.

An instance exists for the duration of the enclosing compound
instance's scope. When the enclosing scope ends, every nested instance
ends.

Multiple instantiations of the same compound in the same enclosing
scope MUST have distinct local names. The same compound MAY be
instantiated arbitrarily many times across a Blueprint; each
instantiation is an independent instance.

### 6.2 Import

An **import** makes compounds declared in another Blueprint or
published in a registry addressable within the current Blueprint. An
import is a declaration at the top of the document; it does not
instantiate anything. Instantiation (§6.1) is always a separate,
subsequent act.

Every import MUST specify:

- the **source** of the imported Blueprint or registry entry
  (a URI or registry identifier);
- the **version** of the imported artifact, pinned to an exact
  specification-compatible version;
- a **local alias** by which the imported compounds are referenced in
  this Blueprint.

An engine MUST reject a Blueprint whose import cannot be resolved,
cannot be verified, or specifies a version incompatible with the
engine's declared conformance. Import verification includes, at
minimum, that the imported artifact's declared specification version
satisfies the engine's supported range.

### 6.3 Overlay

An **overlay** is a declarative modification of a base compound that
produces a new compound with a new identity. The new compound is used
the same way any other compound is used. Overlays are how a Blueprint
specializes a compound — often an imported one — for local use without
forking its source.

An overlay:

- MUST identify a base compound by name (local or imported);
- MUST NOT change the base compound's declared interface (same
  parameters, same inputs, same outputs, same exposed states, same
  schemas);
- MAY modify: default parameter values, nested instance configuration
  values, metadata fields, and implementation details that do not
  change the interface;
- MUST produce a new compound declaration with a new document-local
  name;
- MUST preserve the base compound's marks and MUST add an overlay mark
  (see [mark.md](mark.md));
- MAY itself be overlaid.

**Overlay does not mutate the base.** A Blueprint that overlays an
imported compound does not change the imported artifact. The base
continues to exist; the overlay sits alongside it under a new local
name. This is what makes overlay useful across ownership boundaries:
an author MAY overlay compounds from Blueprints they do not own.

An overlay's mark of action `modify` (or, by convention, `overlay` once
mark actions are fixed in [mark.md](mark.md)) MUST record the base
compound's identity, the overlay author, and, where signed,
establish attribution.

---

## 7. Identity and naming

Compound identity is layered. This document normatively defines the
first two layers.

### 7.1 Document-local name

A compound's **document-local name** is the key under the Blueprint's
`compounds` object. Document-local names MUST be unique within a single
Blueprint. Document-local names are stable across edits to the
Blueprint: renaming a compound is a distinct authoring act that MUST be
traceable through marks.

A document-local name MUST be a non-empty string. A document-local name
SHOULD be a JSON identifier of the form `[A-Za-z_][A-Za-z0-9_-]*`.
Engines MUST accept conforming identifiers. Engines MAY accept
non-conforming names; if they do, they MUST accept all characters
legal in a JSON string key and MUST treat the name as opaque.

### 7.2 Qualified imported name

A compound imported into another Blueprint is addressable as
`<import-alias>:<compound-name>`, where `<import-alias>` is the alias
declared in the import (§6.2) and `<compound-name>` is the imported
compound's document-local name in its source Blueprint. Qualified
imported names MUST be used whenever referencing an imported compound
from within the importing Blueprint.

### 7.3 Instance address

At runtime, a compound instance has an **instance address** — a path
from the entry compound through the instantiation graph to the
instance. Instance addresses are used for observability, snapshotting,
and cross-process references. This document does not fix the instance
address format; the format is defined normatively in
[../semantics/composition.md](../semantics/composition.md) (Round 5,
forthcoming).

---

## 8. The entry compound

Every Blueprint MUST designate exactly one **entry compound** through
the top-level `entry` field (see [../overview.md](../overview.md)
§3.1). The entry compound:

- is the root of instantiation;
- introduces the Blueprint's **root scope** when the Blueprint is
  loaded;
- MUST NOT declare parameters — nothing exists outside the Blueprint to
  supply them;
- MAY declare inputs and outputs — these become the Blueprint's
  **external surface** when the Blueprint itself is used as a unit
  (for example, when it is embedded in another Blueprint as an imported
  compound, or hosted by a runtime that supplies host values);
- MAY declare exposed states, on the same terms.

An engine MUST reject a Blueprint whose entry compound declares
parameters.

When a Blueprint is used as a compound within another Blueprint (via
import and instantiation), the entry compound's inputs and outputs
become the imported compound's interface. A Blueprint with no entry
inputs or outputs is still valid; it simply has no values flowing
across its boundary.

---

## 9. Relationship to scope

A **scope** is the addressability boundary introduced by the
instantiation of a compound. The compound-to-scope correspondence is
1:1 (§1.1): every compound instantiation introduces exactly one scope,
and every scope in a running Blueprint exists because exactly one
compound was instantiated.

Scope is the authored object's *dynamic companion*. A compound declares
structure; a scope is the place that structure lives while running.

This document is normative for what a compound is and what it declares.
[scope.md](scope.md) is normative for what a scope does at runtime —
name resolution, lifecycle bounds, capability propagation. The boundary
between the two documents is: *if the property is about the declaration,
it belongs here; if the property is about the runtime, it belongs in
`scope.md`.*

---

## 10. Normative requirements

A conforming Blueprint document:

1. MUST declare a non-empty `compounds` object.
2. MUST declare exactly one `entry` value whose name resolves to a
   compound in the `compounds` object.
3. MUST NOT declare parameters on its entry compound.
4. MUST give every compound declaration a name unique within the
   `compounds` object.
5. MUST declare every interface member with a name and a schema.
6. MUST NOT reference a compound by name unless the name resolves to
   a compound declared locally or imported.
7. MUST specify the version of every imported artifact.
8. MUST preserve the base compound's marks on any overlay and MUST
   add a mark recording the overlay.

A conforming engine:

9. MUST reject a Blueprint that violates any of the document
   requirements above.
10. MUST enforce encapsulation (§3): a compound MUST NOT observe or
    influence state, channels, or identifiers outside its declared
    interface.
11. MUST provide every core primitive defined for the specification
    version the engine claims to support.
12. MUST resolve every compound reference to either a composed
    compound, a core primitive, an engine-provided primitive it
    supports, or an extension-provided primitive for a registered and
    accepted extension. If a reference cannot be resolved, the engine
    MUST refuse to load the Blueprint and MUST report the unresolved
    reference.
13. MUST treat parameter values as fixed for the duration of an
    instance and MUST NOT silently reinstantiate a compound when a
    parameter value appears to change.
14. MUST NOT alter the base compound when an overlay is resolved; the
    overlay is a distinct compound.
15. MUST NOT permit authored compounds to bypass encapsulation through
    introspection or debugging surfaces.

A conforming authoring tool:

16. MUST NOT produce wiring that references an undeclared interface
    member.
17. MUST NOT produce wiring that violates encapsulation.
18. MUST preserve compound identity across reads and writes; renaming
    a compound is an explicit authoring act traceable through marks.

---

## 11. Examples

The examples here are illustrative. They use a compact pseudo-JSON for
readability; the normative document shape is defined by the JSON Schema
in Round 4.

### 11.1 A leaf compound

A compound that counts how many times it has been pinged, with a
configurable step size.

```json
{
  "compounds": {
    "Counter": {
      "interface": {
        "parameters": {
          "step": { "schema": { "type": "integer", "default": 1 } }
        },
        "inputs": {
          "ping": { "schema": { "type": "null" } }
        },
        "outputs": {
          "changed": { "schema": { "type": "integer" } }
        },
        "exposed_state": {
          "value": { "schema": { "type": "integer" } }
        }
      },
      "state": {
        "value": { "schema": { "type": "integer" }, "initial": 0 }
      },
      "implementation": "primitive:blueprint.core/accumulator"
    }
  }
}
```

`Counter` has one parameter (`step`), one input, one output, one
exposed state, and one internal state field. Its implementation is a
core primitive, referenced by name.

### 11.2 A composed compound

A compound that pairs a counter with a display, and exposes the current
count.

```json
{
  "compounds": {
    "CounterPanel": {
      "interface": {
        "parameters": {
          "step": { "schema": { "type": "integer", "default": 1 } }
        },
        "inputs": {
          "click": { "schema": { "type": "null" } }
        },
        "exposed_state": {
          "count": { "schema": { "type": "integer" } }
        }
      },
      "implementation": {
        "instances": {
          "counter": {
            "compound": "Counter",
            "parameters": { "step": "$params.step" }
          },
          "display": {
            "compound": "primitive:blueprint.core/text",
            "parameters": { "template": "Clicks: {value}" }
          }
        },
        "wiring": [
          { "from": "$inputs.click",         "to": "counter.ping" },
          { "from": "counter.value",         "to": "display.value" },
          { "from": "counter.value",         "to": "$exposed_state.count" }
        ]
      }
    }
  }
}
```

`CounterPanel` instantiates `Counter` and a text primitive, and wires
them. Its own `click` input is forwarded to the child `counter`'s
`ping` input; its own `count` exposed state projects the child's
`value`. The child's `value` is also routed to the `display`'s
template variable.

The exact wiring syntax shown here is illustrative; the normative syntax
is fixed in Round 4 (schema) and the semantics in Round 5.

### 11.3 An overlaid import

Importing a chart compound from another Blueprint and overlaying it
with local defaults.

```json
{
  "imports": {
    "charts": {
      "source": "urn:blueprint:example:charts",
      "version": "1.4.0"
    }
  },
  "compounds": {
    "BlueLineChart": {
      "overlay": {
        "base": "charts:LineChart",
        "parameters": { "color": { "default": "#2b6cff" }, "grid": { "default": true } }
      }
    }
  }
}
```

`BlueLineChart` is a new compound with the same interface as
`charts:LineChart`, but with different parameter defaults. It can be
instantiated like any other compound. The imported artifact is not
modified; the Blueprint need not own `charts:LineChart` to overlay it.

### 11.4 A Blueprint's entry compound

A small, complete Blueprint skeleton.

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:counter-app",
  "entry": "App",
  "compounds": {
    "App": {
      "interface": {
        "outputs": {
          "clicks": { "schema": { "type": "integer" } }
        }
      },
      "implementation": {
        "instances": {
          "panel": { "compound": "CounterPanel", "parameters": { "step": 1 } }
        },
        "wiring": [
          { "from": "panel.count", "to": "$outputs.clicks" }
        ]
      }
    },
    "CounterPanel": { "...": "as in §11.2" },
    "Counter":      { "...": "as in §11.1" }
  }
}
```

`App` is the entry compound. It declares no parameters (none could be
supplied). Its `clicks` output is the Blueprint's external surface: a
host runtime or an outer Blueprint can read it.

---

## 12. Open questions

These questions are deliberately *not* resolved in
`0.1.0-draft`. They are flagged so readers know they are known and not
overlooked.

1. **Generic (type-parameterized) compounds.** A compound whose
   interface schema is parameterized by a type supplied at
   instantiation. Not in 0.1.0.
2. **Compound polymorphism.** Swapping compound `B` for a different
   compound `B'` at instantiation time when `B'`'s interface is
   compatible with `B`'s. Not in 0.1.0; likely addressed by overlay
   mechanisms and a future "compound interface type" concept.
3. **Dynamic instantiation from input values.** Instantiating
   compounds at runtime based on values arriving on an input channel.
   Likely belongs to a *repeater* primitive rather than to the
   vocabulary. Not in 0.1.0.
4. **Strict naming grammar.** The naming constraint in §7.1 is
   deliberately loose. A stricter grammar MAY be adopted in a future
   draft.
5. **Interface-as-type.** Declaring a compound's interface separately
   from any implementation, so multiple compounds can share an
   interface nominally and overlay / polymorphism can reference the
   shared interface. Not in 0.1.0.
6. **Introspection surfaces.** Engines sometimes expose debugging and
   replay surfaces that look like reading a compound's internals. The
   shape of that operator-facing surface is out of scope for this
   document and deferred to the engine contract.

---

## 13. Document status

This document is version `0.1.0-draft` of the compound vocabulary
document. It is normative within the matching Blueprint specification
version. Its claims will be reviewed at the close of every subsequent
authoring round; fixes land as their own commits and are visible in
the document history.

---

*Authored by Anton Bursch. Apache License 2.0.*
