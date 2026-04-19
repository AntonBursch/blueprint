# Scope propagation

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document formalizes how names resolve inside a running
> Blueprint and how capabilities flow between parent and child scopes.
> The rules are stated informally in
> [scope.md](../vocabulary/scope.md) §§4–5; here they become
> engine-normative.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [../vocabulary/scope.md](../vocabulary/scope.md) — the scope concept and its rules
- [../vocabulary/compound.md](../vocabulary/compound.md) — encapsulation invariant (§3)
- [../vocabulary/channel.md](../vocabulary/channel.md) — channel scope binding
- [composition.md](composition.md) — name resolution during load (§4)
- [lifecycle.md](lifecycle.md) — scope begin/end

## 1. Two kinds of propagation

Two things propagate across scope boundaries, and nothing else:

1. **Names**, at load time, per the composition rules in
   [composition.md](composition.md) §4. Name resolution is a static,
   load-time concern; it never traverses into a child scope and never
   reaches beyond an immediate parent.
2. **Capabilities**, at scope-begin time, per §3 below. Capabilities
   flow downward by default; they MAY be restricted by an enclosing
   compound's declaration.

Anything else a compound needs from its surroundings is expressed
explicitly: through its interface, through imports, or through
wiring.

## 2. Name resolution at runtime

At runtime, references by name are the ones already resolved at
load by [composition.md](composition.md) §4. No name resolution
occurs at runtime in 0.1.0-draft.

**R-1.** Once the load sequence completes
([composition.md](composition.md) §6), all compound-name references,
import references, and parameter-name references are fully resolved.
Engines MUST NOT perform late name resolution during the run phase.

**R-2.** A name that failed to resolve at load is a load failure,
per [composition.md](composition.md) R-12. It is never deferred to
runtime.

### 2.1 Why no late resolution

Late resolution would require either global lookup (violating
encapsulation, [../vocabulary/compound.md](../vocabulary/compound.md)
§3) or dynamic reflection (adding a second language to the
specification). Blueprint 0.1.0-draft takes neither. A compound that
needs behavior to vary at runtime does so through channel inputs and
parameter selection, not through late name resolution.

## 3. Capability propagation

A **capability** is a permission, credential, connection handle, or
other authority-bearing value held by a scope. Examples enumerated
in [scope.md](../vocabulary/scope.md) §5 include connection handles,
storage permissions, and signing authority.

### 3.1 Default flow

Capabilities flow **from parent to child** on scope begin. When a
child scope begins, it inherits every capability its parent scope
holds at that moment, except as restricted by §3.2.

**R-3.** Unless a compound's declaration says otherwise, every child
scope begins with the full capability set of its parent at the
moment of begin.

**R-4.** Capability inheritance is a snapshot-at-begin operation.
Capabilities acquired by a parent *after* a child scope has begun
are not automatically propagated to that child. If the child needs
them, it receives them through an input channel or a new scope.

### 3.2 Restriction

An enclosing compound MAY restrict the capabilities a child scope
inherits. The restriction mechanism is declared on the compound's
implementation at the instantiation site; the exact syntactic form
is finalized in the Blueprint JSON Schema evolution of this round.

A restriction MUST NOT grant capabilities the parent does not hold.
Restrictions may only narrow, never widen. This preserves the
invariant that a scope's capabilities are a subset of its ancestor
chain's capabilities.

**R-5.** Engines MUST enforce capability restrictions at the point
of inheritance: a restricted capability MUST be absent from the
child scope's capability set. Attempts by the child to exercise the
capability MUST fail.

**R-6.** An engine MUST reject a Blueprint whose restriction
declaration attempts to grant a capability not held by the enclosing
scope.

### 3.3 Capabilities do not flow upward

A child scope MUST NOT expose its capabilities to its parent or to a
sibling except through an interface member that the child's compound
declared. A compound that needs to confer a capability on another
MUST emit it through a channel whose schema describes it as data,
and the receiving scope MUST explicitly accept it.

**R-7.** Engines MUST NOT provide any mechanism by which a child's
capabilities are automatically observable or usable from outside its
scope.

### 3.4 Capabilities and snapshots

Capabilities are engine concerns. They are not part of a snapshot
([lifecycle.md](lifecycle.md) §6). On restore, the engine acquires
capabilities anew, applies the same restrictions declared in the
Blueprint, and presents them to each scope as if the scope were
beginning for the first time — except that the scope's state comes
from the snapshot, not from initialization.

**R-8.** Engines MUST NOT include raw capability values (keys,
tokens, handles) in snapshots. A capability's declared shape MAY be
referenced by name; the value is reacquired.

## 4. Interaction with imports

An imported compound's implementation resolves names against its own
declarations ([composition.md](composition.md) R-5). Capability
propagation, however, follows the runtime scope chain, not the
document chain: an imported compound instance is a child of the
scope that instantiated it, not of the Blueprint that declared it.

**R-9.** An imported compound's instance MUST inherit capabilities
from its instantiating scope, not from the imported Blueprint.
Imports carry declarations, not runtime capabilities.

## 5. Diagnostics

Every enforcement failure in this document — a violation of R-5, R-6,
R-7, or R-8 — MUST be reported as a structured diagnostic that
identifies:

- the scope involved, by its instance address
  ([composition.md](composition.md) §7);
- the capability in question, by the identifier used in its
  declaration;
- the rule violated, by requirement number where applicable.

**R-10.** Engines MUST produce such diagnostics and MUST NOT mask a
capability failure by degrading silently to a different behavior.

## 6. Normative requirements summary

- **R-1** No late name resolution at runtime.
- **R-2** Unresolved names fail at load.
- **R-3** Default capability inheritance is full.
- **R-4** Inheritance is a snapshot-at-begin operation.
- **R-5** Restrictions are enforced at the inheritance point.
- **R-6** Restrictions cannot widen; engines reject attempts.
- **R-7** No upward capability flow.
- **R-8** Snapshots do not carry raw capability values.
- **R-9** Imported instances inherit from their instantiating scope.
- **R-10** Capability failures are diagnosed, not silently degraded.

## 7. Open questions

- The exact declared form of capability restrictions is not fixed in
  0.1.0-draft. The present document fixes the *semantics* of
  restriction; the *syntax* is finalized in a Blueprint JSON Schema
  revision.
- Whether scope-level audit logging of capability use belongs in
  [authorship.md](authorship.md) or as a separate observability
  specification.

---

*End of scope-propagation.md.*
