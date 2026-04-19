# Scope

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the **scope**: the runtime boundary introduced
> by a compound instantiation. Scope is the dynamic companion of the
> compound — a compound declares structure; a scope is the place that
> structure lives while running. This document fixes what a scope is,
> how scopes nest, how names resolve within them, how lifecycles are
> bounded, and how capabilities propagate.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [compound.md](compound.md) — the declaration whose instantiation creates a scope
- [state.md](state.md) — state that lives inside a scope
- [channel.md](channel.md) — channels are addressable only within a scope
- [../overview.md](../overview.md) — the surrounding specification

---

## 1. What a scope is

A **scope** is the runtime boundary of a single compound instantiation.

Every scope is created when a compound is instantiated and destroyed
when that instantiation ends. Every runtime scope exists because of
exactly one compound instantiation; every compound instantiation
introduces exactly one scope. The compound-to-scope correspondence is
1:1 and is normatively fixed in [compound.md](compound.md) §1.1 and §9.

A scope holds:

- the **state** declared by the compound, owned by this instance;
- the **nested compound instances** declared by the compound's
  implementation, each in its own child scope;
- the **internal channels** declared by the compound's implementation,
  addressable only within this scope;
- the **bindings** of the compound's interface members — parameters,
  inputs, outputs, exposed states — to their values, ports, or
  projections in this instance;
- the **capabilities** available to this instance (see §5).

A scope has an **identity** as long as it exists. This identity is the
instance's place in the Blueprint's runtime tree. The normative format
of instance addresses is defined in
[../semantics/composition.md](../semantics/composition.md) (Round 5,
forthcoming); this document requires only that the format be stable
for the scope's lifetime.

### 1.1 Scope is a runtime concept

Scope is never directly authored. An author writes a compound
declaration and, separately, an instantiation of that compound; the
engine creates the scope as a consequence. No Blueprint document
contains the word "scope" as a structural element.

A Blueprint author reasoning about a program reasons in terms of
compounds and their instantiations. Scope becomes visible when
reasoning about behavior over time — lifetimes, name lookup, capability
propagation, state persistence.

---

## 2. Scope hierarchy

Scopes nest. The **root scope** is the scope of the entry compound's
instantiation, created when the Blueprint is loaded. Every other scope
in a running Blueprint is a **child scope** of exactly one other scope:
the scope whose compound implementation instantiated it.

```
root scope       (entry compound)
├── child scope  (nested instance A)
│   └── grandchild scope  (nested instance inside A)
└── child scope  (nested instance B)
```

The tree of scopes at any moment is the runtime shape of the
Blueprint. The tree's shape is determined by which compounds have been
instantiated, not by the order of declaration in the document.

A scope's **parent** is the scope that instantiated its compound. A
scope's **children** are the scopes of the compound's nested instances.
A scope's **ancestors** are its parent, its parent's parent, and so on
up to the root. A scope's **descendants** are its children, their
children, and so on.

---

## 3. Lifecycle

A scope has a defined lifecycle.

### 3.1 Begin

A scope **begins** when its compound is instantiated. At the moment of
beginning:

- all parameters declared on the compound's interface are bound to
  values supplied by the instantiator;
- all internal state fields are initialized according to their
  declarations (see [state.md](state.md));
- all internal channels are ready to receive or emit messages
  (see [channel.md](channel.md));
- all nested compounds are instantiated in turn, each creating a child
  scope, recursively.

A scope MUST NOT be observed or addressed until it has begun.

### 3.2 Run

While a scope exists, values may arrive on its inputs, its state may
change, it may emit values on its outputs, and its exposed states
reflect current values. The scope remains addressable by the engine
and, through interface members, by its parent scope.

A scope's existence is independent of external activity. A scope with
no current traffic is still a live scope; absence of messages is not
absence of scope.

### 3.3 End

A scope **ends** when its compound instance ends. A compound instance
ends when:

- its parent scope ends (by any of these rules, recursively); or
- the compound itself declares a terminal condition that the engine
  recognizes (out of scope for 0.1.0-draft); or
- the Blueprint is unloaded or the engine is shutting down.

On ending:

- all descendant scopes MUST end first, in an order that respects
  parent-child relationships;
- state is either released or persisted depending on the engine's
  snapshot semantics (see [../semantics/lifecycle.md](../semantics/lifecycle.md),
  Round 5, forthcoming);
- all channels rooted in this scope MUST cease delivering and receiving
  messages;
- the scope's identity MUST NOT be reused for a new scope within the
  same running Blueprint.

Scope lifecycle is strictly nested. A child scope MUST NOT outlive its
parent scope within a single engine instance. Durable continuation of
a compound instance across engine restarts is a separate concern
handled by snapshot and restore, not by scope lifetime.

---

## 4. Name resolution

Names referenced inside a compound's implementation — references to
nested compounds, to channels, to state fields, to interface members —
resolve within the compound's scope.

The normative rule:

> A name referenced within a scope MUST resolve to the nearest
> enclosing declaration of that name.

"Nearest enclosing" means: first check the compound's own declarations
(its state fields, its internal channel names, its nested instance
local names); if not found, check the compound's interface members
(parameters, inputs, outputs, exposed states); if not found, check the
imported compound aliases at the top of the Blueprint.

A name that cannot be resolved by these rules is an error. An engine
MUST reject a Blueprint that contains an unresolvable name.

Name resolution MUST NOT traverse into a child scope. A parent cannot
name a child's internal channels or state; a sibling cannot name
another sibling's internals. This is a direct consequence of
encapsulation ([compound.md](compound.md) §3).

### 4.1 No leakage from parent

Name resolution does not transparently escape a scope into its parent.
A compound's implementation references interface members (parameters,
inputs, outputs, exposed states) through the interface, not by
reaching into the parent scope. This keeps compounds reusable across
contexts: the same compound declaration runs identically in every
scope that instantiates it, regardless of what the parent scope
contains.

### 4.2 Import aliases

Names declared as import aliases at the top of a Blueprint are
addressable from every scope in that Blueprint as qualified compound
references (see [compound.md](compound.md) §6.2). This is the one
resolution path that crosses scope boundaries freely. Imports are
declarations, not runtime objects; they do not introduce scopes.

---

## 5. Capabilities

A **capability** is a permission, a credential, a connection handle,
or any other authority-bearing value that a scope may hold and that a
compound may need. Capabilities include (illustrative, not
exhaustive):

- the ability to connect to a named external system;
- the ability to read or modify a particular piece of persistent
  storage;
- the authority to sign marks on behalf of a specific author;
- the authority to admit new authors into a signing ring;
- any author identity signed into the scope by a mark.

### 5.1 Propagation

Capabilities flow from parent scope to child scope by default.
A child scope inherits every capability held by its parent unless the
parent explicitly restricts propagation when instantiating the child.

A child scope MUST NOT hold a capability that its parent does not
hold. A scope MUST NOT acquire new capabilities by reference alone;
capabilities are granted only by the scope that holds them.

### 5.2 Restriction

A compound instantiating a child MAY restrict the set of capabilities
the child scope receives. Restriction is explicit and declarative; the
instantiation records which capabilities are withheld. An engine MUST
honor a declared restriction: the child scope MUST NOT exercise a
withheld capability regardless of what the compound declaration
requests.

### 5.3 Revocation

A parent scope MAY revoke a capability previously granted to a child.
Revocation takes effect before any further use of the capability by
the child. The detailed semantics of revocation, including how
in-flight uses are handled, are defined in
[../semantics/scope-propagation.md](../semantics/scope-propagation.md)
(Round 5, forthcoming).

### 5.4 No synthesis

A scope MUST NOT synthesize a capability it was not granted. There is
no runtime operation a compound can perform to acquire a capability
that its scope does not already hold. Capabilities are not values that
can be computed or reconstructed from other values; they are granted,
propagated, restricted, or revoked, and nothing else.

This restriction is the guarantee that reasoning about a compound's
authority can be performed locally: what the scope holds is the
complete enumeration of what the instance can do.

---

## 6. Isolation

The combined effect of encapsulation (in [compound.md](compound.md) §3)
and scope lifecycle and resolution rules (this document §§3–4) is
**isolation**:

- Each scope's state is inaccessible from outside except through
  exposed state (§4, and [compound.md](compound.md) §2.3).
- Each scope's internal channels are addressable only from within it.
- Each scope's lifecycle is bounded by its parent's.
- Each scope's capabilities are bounded by its parent's.

Isolation is what makes a compound instance independently
comprehensible. A reviewer or agent analyzing a compound need look
only at the compound's declaration and its interface; the behavior of
the scope is a function of that declaration and the values arriving
on its interface, nothing else.

---

## 7. Normative requirements

A conforming engine:

1. MUST create a scope when, and only when, a compound is
   instantiated, with the compound-to-scope correspondence 1:1.
2. MUST bind all interface parameters of an instantiation before the
   scope begins.
3. MUST initialize all declared state fields before the scope begins.
4. MUST NOT permit a name in a compound's implementation to resolve
   to a declaration outside that compound's scope, its interface, or
   its Blueprint's imports.
5. MUST enforce scope nesting: a child scope MUST NOT outlive its
   parent scope within a single engine instance.
6. MUST end all descendant scopes before ending a scope.
7. MUST NOT reuse an ended scope's identity for a new scope within
   the same running Blueprint.
8. MUST propagate capabilities from parent to child by default, and
   MUST honor every explicit restriction declared at instantiation.
9. MUST NOT permit a scope to hold or exercise a capability not held
   by its parent (except the root scope, whose capabilities are
   supplied by the engine and its host).
10. MUST NOT permit a scope to synthesize a capability from ambient
    values.
11. MUST reject a Blueprint containing an unresolvable name.

---

## 8. Examples

### 8.1 A scope tree for a small Blueprint

For the Blueprint skeleton in [compound.md](compound.md) §11.4:

```
root scope (App)
└── child scope (panel: CounterPanel, step = 1)
    ├── grandchild scope (counter: Counter, step = 1)
    └── grandchild scope (display: primitive text)
```

Four scopes, one per instantiation. Each scope has its own state,
internal channels, and capability set. The `counter` scope's internal
state (`value`) is inaccessible from the `panel` scope except through
`counter`'s exposed state.

### 8.2 A restricted capability

```
root scope              (capabilities: [db.write, db.read, net.out])
└── child scope: report (capabilities: [db.read, net.out])
    └── grandchild scope: formatter (capabilities: [net.out])
```

The root scope holds three capabilities. Its `report` child was
instantiated with `db.write` withheld; its `formatter` grandchild was
instantiated with `db.read` withheld. A compound running in
`formatter`'s scope cannot exercise `db.read` or `db.write` no matter
what its declaration requests.

### 8.3 Name resolution

Inside a compound's implementation, a reference to `config.timeout`
resolves as follows:

1. Is `config` a local internal name (state field, channel name,
   nested instance local name)? If yes, look up `timeout` on that
   object.
2. Otherwise, is `config` an interface member (parameter, input,
   output, exposed state)? If yes, look up `timeout` on that member's
   value or schema.
3. Otherwise, is `config` an import alias? If yes, look up `timeout`
   as a compound in the imported Blueprint.
4. Otherwise, the reference is unresolvable and the Blueprint is
   invalid.

This is the complete algorithm. No other lookup path exists.

---

## 9. Open questions

1. **Terminal conditions for compound instances.** A compound MAY
   declare that it has completed and no further input is expected.
   The shape of such a declaration is not fixed in 0.1.0-draft.
2. **Scope-aware observability.** Engines MAY expose per-scope
   metrics, logs, or trace spans. The normative observability surface
   is deferred to the engine contract.
3. **Capability grant beyond parent.** Engines MAY delegate to external
   authorization systems that grant capabilities a parent scope does
   not itself hold. The correctness of such delegation is out of scope
   for 0.1.0-draft; the rule in §5.1 treats the parent as the source
   of authority.

---

## 10. Document status

This document is version `0.1.0-draft` of the scope vocabulary
document. It is normative within the matching Blueprint specification
version.

---

*Authored by Anton Bursch. Apache License 2.0.*
