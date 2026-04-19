# Lifecycle

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the runtime lifecycle of a Blueprint and the
> scopes within it: how loading becomes running, how running becomes
> stopped, and how a stopped Blueprint is restored. It elaborates
> [scope.md](../vocabulary/scope.md) §3 into engine-normative rules
> and specifies the snapshot-and-restore invariant that makes a
> Blueprint portable across engines of the same specification version.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [../vocabulary/scope.md](../vocabulary/scope.md) — scope lifecycle fundamentals (§3)
- [../vocabulary/compound.md](../vocabulary/compound.md) — compound-scope correspondence
- [../vocabulary/state.md](../vocabulary/state.md) — serializability and initialization
- [composition.md](composition.md) — the instance tree this document animates
- [channel-delivery.md](channel-delivery.md) — delivery invariants during lifecycle transitions
- [determinism.md](determinism.md) — replay equivalence across restart

## 1. The three phases

A Blueprint's runtime life has three phases.

- **Load.** Defined in [composition.md](composition.md) §6. Produces a
  validated instance tree. The Blueprint is not yet running.
- **Run.** The instance tree is live: scopes exist, state is present,
  channels carry messages.
- **Stop.** The instance tree is torn down. Scopes end, state is either
  released or persisted, channels are closed.

Every run begins with exactly one load. A load does not imply a run: a
tool MAY load a Blueprint for validation or inspection without
entering the run phase.

**R-1.** An engine MUST complete the load sequence
([composition.md](composition.md) §6) successfully before entering the
run phase. A Blueprint that failed to load MUST NOT be executed.

## 2. Scope begin

A scope begins when its compound is instantiated. Begin is a single
atomic transition from "declared" to "live". In order:

1. Parameter values bound at the instantiation site
   ([composition.md](composition.md) §3) are captured in the scope.
2. Every state field declared by the compound is initialized per
   [../vocabulary/state.md](../vocabulary/state.md) §3. Initial values
   MUST validate against their schemas.
3. Internal channels declared by the compound's implementation are
   brought to a state where they can carry messages.
4. Every nested instance declared by the compound's implementation is
   instantiated, recursively, producing a child scope. Child scopes
   begin after the enclosing scope's state and channels are ready.
5. The scope becomes addressable by its instance address
   ([composition.md](composition.md) §7).

**R-2.** An engine MUST complete steps 1–4 before making the scope
observable to other compounds or to external observers.

**R-3.** A scope's state MUST be fully initialized before any input
begins delivering values to the compound
([../vocabulary/state.md](../vocabulary/state.md) §3.1).

**R-4.** A scope MUST NOT be reported as begun until every descendant
scope it transitively contains has also begun. Begin propagates
depth-first through the instance tree.

## 3. Scope run

A live scope holds state, receives inputs, produces outputs, and
exposes state projections per its compound's declared interface. While
a scope is running:

- The scope's state changes exclusively in response to the causes
  enumerated in [../vocabulary/state.md](../vocabulary/state.md) §4.
- The scope's internal channels deliver messages per
  [channel-delivery.md](channel-delivery.md).
- The scope's address (§7 of [composition.md](composition.md)) is
  stable.
- The scope's exposed states reflect the current value of their
  projected internal state fields.

**R-5.** An engine MUST NOT mutate a scope's state, modify its
channels, or change its address while the scope is running, except
through the operations defined in
[../vocabulary/state.md](../vocabulary/state.md) and
[channel-delivery.md](channel-delivery.md).

## 4. Scope end

A scope ends when its compound instance ends. An instance ends when
any of the following occurs:

1. The enclosing scope ends.
2. The Blueprint is unloaded (the root scope ends).
3. The engine is shutting down.
4. An engine-recognized terminal condition is reached (engine-defined
   in 0.1.0-draft; a later revision may fix this normatively).

On end, in order:

1. Every descendant scope ends first, depth-first, leaves up.
2. The scope's channels cease accepting new messages. In-flight
   messages on synchronous channels complete or fail per
   [channel-delivery.md](channel-delivery.md) §5.
3. The scope's state is either released (if no snapshot is requested)
   or captured for persistence (§6).
4. The scope's identity (instance address) is retired. An engine MUST
   NOT reuse it in the same running Blueprint
   ([composition.md](composition.md) R-21).

**R-6.** An engine MUST end descendant scopes before ending their
parent. Parents never outlive their children's end sequence.

**R-7.** An engine MUST NOT reuse an instance address after the scope
it identified has ended, within the same running Blueprint.

**R-8.** A scope end MUST be observable: tools consuming the engine's
observability surface
([authorship.md](authorship.md) §4) MUST receive a scope-end event
that identifies the scope by its address.

## 5. Stop

Stopping a Blueprint is ending the root scope. Stop rules follow §4
recursively. A clean stop leaves the engine in a state where the same
Blueprint can be loaded again and where any snapshot produced at stop
is usable by §6.

**R-9.** An engine MUST provide a deterministic stop operation: when
called, it ends the root scope per §4 and does not leave orphan
scopes, dangling channels, or partially-released state.

## 6. Snapshot and restore

A **snapshot** is a serialized capture of a running Blueprint's state
at a safe point. A **restore** is the inverse: reconstructing a
running Blueprint from a snapshot.

### 6.1 What a snapshot captures

A snapshot of a running Blueprint captures:

- the canonical form of the loaded Blueprint document
  ([../canonical-form.md](../canonical-form.md) §3.1), sufficient to
  reproduce the instance tree;
- for every live scope, its instance address and its state (every
  declared state field, serialized per
  [../vocabulary/state.md](../vocabulary/state.md) §1.2);
- for every live scope, its parameter bindings as resolved at the
  time its scope began;
- the engine's record of in-flight channel messages that are
  guaranteed to be delivered, per
  [channel-delivery.md](channel-delivery.md) §5;
- the Blueprint's specification version.

A snapshot does NOT capture:

- engine-internal scheduling queues, threads, or fibers;
- runtime-only representations that do not project back to declared
  state;
- external resources held by compounds (those are the engine's
  responsibility to acquire anew on restore).

### 6.2 The snapshot invariant

**R-10.** A snapshot produced by any conforming engine MUST be
restorable by any conforming engine of the same specification version
and with compatible primitives and extensions. Restoring a snapshot
MUST produce a running Blueprint that is indistinguishable, through
exposed state and outputs, from the Blueprint at the moment the
snapshot was taken, modulo ongoing external input.

This is the **portable-durability property**. It is the property that
makes Blueprint portable in time: a snapshot taken today can be
restored years from now on a different engine.

### 6.3 Safe points

A snapshot MAY be taken only at a **safe point**: an instant when
every scope's state is coherent per
[../vocabulary/state.md](../vocabulary/state.md) §4.2 (no
half-updated state). Every boundary between successive message
deliveries is a safe point.

**R-11.** Engines MUST NOT take a snapshot while any scope is in the
middle of processing a message. Engines MAY reach a safe point by
waiting for current processing to complete; they MUST NOT interrupt
processing to force one.

### 6.4 Restore sequence

Restoring a snapshot proceeds as follows:

1. Verify the snapshot's Blueprint declaration matches the engine's
   supported specification version.
2. Load the Blueprint per the full load sequence
   ([composition.md](composition.md) §6).
3. For each scope in the snapshot, in the order scopes would begin
   (§2), initialize the scope with the snapshot's captured state
   instead of the default initial values.
4. Replay any in-flight channel messages captured in the snapshot,
   per [channel-delivery.md](channel-delivery.md) §5.

**R-12.** A restored scope's state MUST validate against its
compound's state schemas. If it does not, the restore MUST fail with
a diagnostic; engines MUST NOT proceed with partial restore.

**R-13.** A restored Blueprint MUST carry the same instance addresses
as the snapshot's addresses. Addresses are stable across
snapshot/restore.

## 7. Cross-engine migration

A restore on a different engine of the same specification version is
a migration. The snapshot invariant (R-10) makes migration a
consequence of the restore contract, not a separate operation.

**R-14.** Engines MUST NOT require engine-specific metadata in a
snapshot to restore it. A snapshot that carries engine-specific
extension data MAY carry it in an `x-`- or `ext.`-namespaced field;
receiving engines MUST preserve the field round-trip but MUST NOT
require it for restore.

## 8. Hot reload

A **hot reload** is the replacement of the loaded Blueprint document
without ending the root scope. 0.1.0-draft does not normatively
define hot reload. Engines MAY implement it; those that do MUST
define their invariants in engine documentation. No conformance
requirement depends on hot reload in this version.

## 9. Normative requirements summary

- **R-1** Run requires a successful load.
- **R-2** Begin is atomic: steps 1–4 before observability.
- **R-3** State is fully initialized before inputs flow.
- **R-4** Begin propagates depth-first.
- **R-5** No extra-rule mutation during run.
- **R-6** Children end before parents.
- **R-7** Instance addresses MUST NOT be reused.
- **R-8** Scope end is observable.
- **R-9** Stop is deterministic; no orphans.
- **R-10** Snapshot/restore invariant: portable durability.
- **R-11** Snapshots at safe points only.
- **R-12** Restore validates state; no partial restore.
- **R-13** Addresses stable across snapshot/restore.
- **R-14** Snapshots MUST NOT require engine-specific metadata.

## 10. Open questions

- Whether terminal conditions (§4 item 4) should be normatively fixed
  in a subsequent revision (for example, via a `terminate` core
  primitive) or remain engine-defined.
- Whether hot reload gets a normative invariant. 0.1.0-draft defers.

---

*End of lifecycle.md.*
