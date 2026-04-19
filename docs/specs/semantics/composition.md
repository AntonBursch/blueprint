# Composition

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the runtime rules by which a Blueprint is
> composed: how compounds are instantiated, how imports are resolved,
> how overlays produce new compounds, how parameter values flow, how
> names are resolved, how the instance tree is addressed, and the
> ordered sequence of checks an engine performs when it loads a
> Blueprint. Composition is the keystone of Blueprint's runtime
> contract. Everything else in `semantics/` assumes it.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [../vocabulary/compound.md](../vocabulary/compound.md) — compound declarations and the three composition operators
- [../vocabulary/scope.md](../vocabulary/scope.md) — the scope created by each instantiation
- [../vocabulary/schema.md](../vocabulary/schema.md) — validation at boundaries
- [../vocabulary/blueprint.md](../vocabulary/blueprint.md) — the top-level artifact
- [../schema/blueprint.schema.json](../schema/blueprint.schema.json) — well-formedness
- [lifecycle.md](lifecycle.md) — scope begin/end driven by composition
- [channel-delivery.md](channel-delivery.md) — runtime semantics of the wiring composition produces
- [determinism.md](determinism.md) — what composition determinism guarantees
- [authorship.md](authorship.md) — mark and signature handling at load

## 1. What composition is

Composition is the act by which a Blueprint document becomes a running
instance graph. An engine composes a Blueprint by, recursively:

1. selecting the **entry compound**;
2. instantiating it, creating the **root scope**;
3. for each nested instance declared in that compound's implementation,
   instantiating the referenced compound, creating a child scope;
4. stopping when every instance is either a composed compound whose
   nested instances have all been instantiated, or a primitive
   compound.

The result is a finite tree of scopes rooted at the root scope. This
tree is the **instance tree**. Its shape is fixed by the Blueprint and
its imports. Nothing external to those sources determines it.

**R-1.** Given the same Blueprint and the same imported artifacts, an
engine MUST produce the same instance tree on every load. Composition
is structurally deterministic per [determinism.md](determinism.md) §3.

## 2. The three operators

The three composition operators are named normatively in
[../vocabulary/compound.md](../vocabulary/compound.md) §6. This
document fixes their runtime semantics.

### 2.1 Instantiate

Instantiating a compound creates exactly one runtime instance of it
inside the enclosing scope. The instantiation declaration:

- references the instantiated compound by a name resolvable per §4;
- assigns a **local name** to the instance, unique within the
  enclosing compound's implementation;
- supplies a **value** for every parameter declared on the
  instantiated compound's interface, per §3.

The resulting instance is a first-class runtime object bounded by its
own scope ([../vocabulary/scope.md](../vocabulary/scope.md)).

**R-2.** Every instantiation MUST supply a value for every parameter
declared on the instantiated compound's interface. An engine MUST
reject a Blueprint whose instantiation omits a required parameter.

**R-3.** Local names of instances within a single enclosing compound's
implementation MUST be unique. An engine MUST reject duplicates.

### 2.2 Import

Imports are resolved during load (§6). An import is a declaration;
instantiation is a separate act. A Blueprint MAY import a compound and
never instantiate it.

**R-4.** An engine MUST resolve every declared import before it
instantiates any compound. An unresolved import is a load failure per
§6.

**R-5.** An imported compound's interior is resolved against the
imported Blueprint's own declarations, not the importing Blueprint's.
An imported compound's imports are resolved transitively in the
imported Blueprint's namespace.

### 2.3 Overlay

An overlay declaration produces a **new compound** with a new
document-local name. It is not a runtime mutation of the base. At
composition time, an overlay is treated exactly like any other
compound declaration.

**R-6.** An engine MUST treat an overlay compound as an independent
declaration with the base compound as its authored origin (preserved
through marks, per [../vocabulary/mark.md](../vocabulary/mark.md)).
Instantiating the overlay does not affect any instantiation of the
base.

## 3. Parameter binding

A parameter value at an instantiation site is one of:

1. a **JSON literal** — any valid JSON value whose shape validates
   against the parameter's schema; or
2. a **reference** to a parameter declared on the enclosing compound's
   interface, denoted in the syntactic form finalized by the Blueprint
   JSON Schema (Round 4) and this document's Round 5 revision of
   composition syntax.

Nothing else. A parameter value is not an expression, not a template,
not a computation.

### 3.1 No string-embedded logic

This specification does not define, and will not define, a
string-embedded expression language for parameter values. A Blueprint
author who needs logic — arithmetic, conditional selection, string
formatting, lookup, comparison — expresses that logic as a compound
(core primitive, extension primitive, or composed) and instantiates
it, wiring its output to the consuming instance.

This rule is permanent, not a 0.1.0-draft restraint. Its rationale:

- A Blueprint is a composition graph. A parallel expression language
  inside parameter strings would be a second, hidden graph that tools
  would have to parse, agents would have to reason about, and
  validators would have to model. The cost compounds as the expression
  language grows.
- The same principle already applies to mark authors
  ([../vocabulary/mark.md](../vocabulary/mark.md) §Relationship to
  overview §4.8): structural data, never a string carrying semantics.
- Any computational capability is cleanly expressible as a compound.
  The vocabulary already has the machinery: inputs, outputs,
  parameters, schemas.

**R-7.** Parameter values MUST NOT carry expression syntax. An engine
MUST treat a JSON string parameter value as a literal string, not as a
source of logic.

**R-8.** If a future revision of this specification introduces
computation at parameter sites, it MUST do so through a new
vocabulary mechanism (a primitive compound, a new interface kind), not
through string reinterpretation.

### 3.2 Parameter-to-parameter references

A parameter value MAY be a reference to a parameter declared on the
enclosing compound's interface. This is how values flow down the
instance tree without being copied verbatim.

**R-9.** A parameter reference MUST name a parameter on the
compound whose implementation contains this instantiation. Engines
MUST reject references to parameters declared anywhere else.

**R-10.** The referenced parameter's schema MUST be compatible with
the instantiated compound's parameter schema (the referenced schema
accepts every value the consuming schema would accept). Engines MUST
validate compatibility at load.

### 3.3 Parameter binding validates at load

**R-11.** Every parameter value — literal or reference — MUST
validate against the parameter's declared schema at load. An
instantiation whose bound value fails validation is a load failure.

## 4. Name resolution order

When a compound reference is encountered during composition, the
engine resolves it in exactly this order:

1. The Blueprint's top-level `compounds` object (document-local).
2. The qualified import aliases declared in `imports` (qualified
   imported names per
   [../vocabulary/compound.md](../vocabulary/compound.md) §7.2).
3. The core primitives defined by this specification version.

No further lookup is performed. An engine MUST NOT silently fall back
across namespaces or retry a failed resolution in a different
namespace.

**R-12.** A compound reference that fails all three resolution steps
is a composition error. Engines MUST reject the Blueprint with a
diagnostic identifying the unresolved name.

**R-13.** Resolution never traverses into a nested scope or into a
sibling instance's internals. Name resolution is a document-level
concern, not a runtime one. At runtime, compounds reach each other
exclusively through wired channels and exposed state
([../vocabulary/channel.md](../vocabulary/channel.md)).

## 5. Cycles

A compound's implementation that instantiates itself — directly
(compound `A` contains an instance of `A`) or transitively (`A`
instantiates `B` which instantiates `A`) — would produce an infinite
instance tree. This specification forbids it.

**R-14.** An engine MUST compute, at load, the compound-instantiation
graph: a directed graph with a node per compound and an edge from each
composed compound to every compound its implementation instantiates.
If this graph contains a cycle, the Blueprint is malformed. Engines
MUST reject it with a diagnostic identifying the cycle.

**R-15.** Overlay does not produce a cycle. An overlay is a
declaration of a distinct compound (§2.3); its base appears as a
different node in the instantiation graph.

**R-16.** Imports are resolved before the cycle check (§6). Cycles
that pass through imported compounds are detected exactly the same
way as cycles within the host Blueprint.

### 5.1 Recursion, not cycles

The ban on cycles does not forbid recursion at runtime — a compound
MAY, through dynamic channel behavior, produce arbitrarily deep
interaction patterns. What is forbidden is **declared structural
recursion in the instance tree**. Runtime dynamics are governed by
[channel-delivery.md](channel-delivery.md) and
[lifecycle.md](lifecycle.md), not by this section.

## 6. Load sequence

An engine loading a Blueprint MUST perform the following checks in
order. First failure stops the load.

1. **Parse.** Parse the artifact as JSON per
   [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259). Malformed JSON
   is a load failure.
2. **Well-formedness.** Validate the parsed document against
   [blueprint.schema.json](../schema/blueprint.schema.json). A
   well-formedness failure is a load failure.
3. **Import resolution.** Resolve every declared import. For each
   import, fetch the imported artifact, verify its version pin matches
   the canonical form of the fetched artifact per
   [../canonical-form.md](../canonical-form.md), and recursively
   perform steps 1–4 on the imported artifact. An unresolvable or
   mismatched import is a load failure.
4. **Schema validation.** Validate top-level `schemas` and every
   declared schema reference per
   [../vocabulary/schema.md](../vocabulary/schema.md). An invalid
   schema is a load failure.
5. **Signature verification.** If signature verification is enabled
   (engine policy per [authorship.md](authorship.md) §5), verify every
   signature in the Blueprint and every signature in resolved
   imports. Invalid signatures fail the load if policy requires.
6. **Cycle check.** Compute the compound-instantiation graph (§5) and
   check for cycles. A cycle is a load failure.
7. **Composition.** Build the instance tree rooted at `entry`,
   validating parameter bindings per §3.3 and name resolutions per
   §4. Any failure at this step is a load failure.

**R-17.** Engines MUST perform steps 1–7 in this order. Engines MAY
skip step 5 when signature verification is disabled; all other steps
are mandatory.

**R-18.** Load failures MUST produce a diagnostic identifying the
failing step and the specific cause. Engines MUST NOT proceed to any
subsequent step after a failure.

## 7. Instance addresses

Every instance in the instance tree has a stable **instance address**.
The address is a slash-separated path from the root scope, using the
local name assigned at each instantiation site. The root scope's
address is `/`.

### 7.1 Format

An instance address is a JSON string of the form:

```
/<local-name-1>/<local-name-2>/.../<local-name-N>
```

where each `<local-name-k>` is the local name assigned by the
enclosing compound's implementation to the instance at that level.
The empty path (`/`) is the root scope (the entry compound's
instance).

**R-19.** Engines MUST compute instance addresses using local
instantiation names, not compound-type names. Two instances of the
same compound under the same parent have different local names and
therefore different addresses.

### 7.2 Escaping

A local name MAY contain characters that conflict with address
syntax. Instance addresses MUST escape such characters using the
rules of [RFC 6901](https://www.rfc-editor.org/rfc/rfc6901) §3:

- `~` becomes `~0`;
- `/` becomes `~1`;
- these replacements apply to the local name only, not to the
  path-separating `/`.

A local name that does not contain `/` or `~` is used verbatim.

**R-20.** Engines MUST apply RFC 6901 §3 escaping when forming or
parsing instance addresses.

### 7.3 Stability

An instance address is stable for the life of the instance. It does
not change across engine restarts of the same Blueprint at the same
canonical form. It is the identifier used by observability
([authorship.md](authorship.md) §4), snapshot and restore
([lifecycle.md](lifecycle.md) §6), and cross-engine coordination
([channel-delivery.md](channel-delivery.md) §4).

**R-21.** An instance address MUST uniquely identify an instance
within the running Blueprint at the moment the address is captured.
Addresses MUST NOT be reused: if an instance ends and a new
instantiation takes its place (for example, a dynamic list of nested
instances grows back to its previous length), the new instance
receives a new address.

## 8. Composition errors and diagnostics

Every failure mode in this document produces a diagnostic with:

- the step of the load sequence at which the failure occurred (§6);
- the specific rule violated (by requirement number where available);
- the location in the Blueprint (JSON Pointer into the source, or
  instance address for composition-stage failures);
- the name of the compound, import, or instance responsible.

**R-22.** Engines MUST report composition failures with sufficient
information for an author or tool to locate and correct the cause.
Diagnostics are not required to follow a particular format in
0.1.0-draft; they MUST be structured machine-readable data, not
free-form prose alone.

## 9. Examples

### 9.1 A minimal composition

Two compounds, one instantiation. The entry compound `Dashboard`
instantiates a `Counter` as its only nested instance.

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:dashboard",
  "compounds": {
    "Dashboard": {
      "implementation": {
        "instances": {
          "main_counter": { "compound": "Counter", "parameters": { "step": 1 } }
        }
      }
    },
    "Counter": {
      "interface": { "parameters": { "step": { "schema": { "type": "integer" } } } }
    }
  },
  "entry": "Dashboard"
}
```

Instance addresses:

- `/` — root scope (the `Dashboard` instance)
- `/main_counter` — the `Counter` instance

### 9.2 A parameter-to-parameter reference

`Dashboard` accepts a `step` parameter and forwards it to its
`Counter` instance.

```json
{
  "compounds": {
    "Dashboard": {
      "interface": { "parameters": { "step": { "schema": { "type": "integer" } } } },
      "implementation": {
        "instances": {
          "main_counter": {
            "compound": "Counter",
            "parameters": { "step": { "$ref": "#/parameters/step" } }
          }
        }
      }
    }
  }
}
```

The exact reference syntax (`$ref` here is indicative) is locked by
the Blueprint JSON Schema in Round 4 and may be refined in a
subsequent round.

### 9.3 A rejected cycle

```json
{
  "compounds": {
    "A": { "implementation": { "instances": { "b": { "compound": "B" } } } },
    "B": { "implementation": { "instances": { "a": { "compound": "A" } } } }
  },
  "entry": "A"
}
```

Step 6 of the load sequence detects the cycle `A → B → A` and fails
the load with a diagnostic identifying both compounds.

## 10. Normative requirements summary

- **R-1** Instance tree is deterministic given Blueprint and imports.
- **R-2** Every parameter MUST be bound at instantiation.
- **R-3** Local instance names MUST be unique within an enclosing compound.
- **R-4** All imports resolved before any instantiation.
- **R-5** Imported compound interiors resolve in the imported Blueprint's namespace.
- **R-6** Overlay is a distinct compound at composition time.
- **R-7** Parameter values MUST NOT carry expression syntax.
- **R-8** Future computation at parameter sites MUST use a new vocabulary mechanism, not string reinterpretation.
- **R-9** Parameter references MUST name an enclosing-compound parameter.
- **R-10** Referenced parameter schemas MUST be compatible.
- **R-11** Parameter values MUST validate at load.
- **R-12** Unresolvable names fail the load.
- **R-13** Resolution does not traverse scope boundaries.
- **R-14** Instantiation cycles are forbidden.
- **R-15** Overlays do not constitute cycles.
- **R-16** Cycle detection includes imported compounds.
- **R-17** Load sequence order is mandatory (§6).
- **R-18** Load failures stop subsequent steps and produce a diagnostic.
- **R-19** Instance addresses use local names, not type names.
- **R-20** RFC 6901 §3 escaping in instance addresses.
- **R-21** Addresses MUST NOT be reused.
- **R-22** Diagnostics MUST be structured and locatable.

## 11. Open questions

- Whether overlay gets its own mark action (`overlay` vs. `create`
  with base identified in `provenance`). Mark vocabulary is closed at
  six values per [../vocabulary/mark.md](../vocabulary/mark.md); this
  document treats overlay as `create` with base-in-provenance.
- Whether instance address escaping needs to handle characters beyond
  `/` and `~`. 0.1.0-draft adopts RFC 6901 §3 only; a future revision
  may extend.

---

*End of composition.md.*
