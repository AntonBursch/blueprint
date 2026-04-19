# Determinism

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document fixes what Blueprint's determinism guarantees cover
> and what they do not. Two tiers are distinguished: *structural*
> determinism, which is required; and *execution* determinism, which
> is not required in general but is guaranteed on replay given a
> recorded input trace. A closed list of non-determinism sources is
> enumerated, making the set of tolerable runtime variations finite
> and auditable.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [composition.md](composition.md) — structural determinism of the instance tree
- [lifecycle.md](lifecycle.md) — snapshot/restore invariant
- [channel-delivery.md](channel-delivery.md) — ordering and fan-in rules
- [../vocabulary/state.md](../vocabulary/state.md) — state determinism claim

## 1. Two tiers

Blueprint's determinism is split into two tiers.

- **Structural determinism** (REQUIRED). Given the same Blueprint
  canonical form and the same imported artifacts, every conforming
  engine produces the same instance tree, the same wiring graph, the
  same parameter bindings, and the same initial state values.
- **Execution determinism** (NOT REQUIRED in general). The exact
  interleaving of concurrent operations and the exact wall-clock
  timing of external inputs are engine-dependent and MAY vary across
  executions. A scoped subset of execution determinism — **replay
  equivalence** — is required (§4).

### 1.1 Why the split

An application that talks to external systems cannot be fully
deterministic at runtime. Timing, ordering of concurrent external
arrivals, and clock skew are all non-deterministic in principle. If
the specification required them to be eliminated, conforming would
require simulating the outside world.

At the same time, portability depends on structure being fixed: two
engines that produce different instance trees for the same Blueprint
cannot be said to run the same application.

The split puts every guarantee on one side or the other. No rule in
Blueprint depends on execution being deterministic in the
unconstrained sense.

## 2. Observable surface

"Deterministic" in this document is defined relative to an
**observable surface**. An engine's observable surface is:

- every exposed state of every live scope, at every moment it is
  projected ([../vocabulary/compound.md](../vocabulary/compound.md)
  §2.3);
- every value emitted on every output channel of every live scope
  ([../vocabulary/channel.md](../vocabulary/channel.md));
- every diagnostic surfaced through the observability surface
  ([authorship.md](authorship.md) §4).

Internal representations, scheduling queues, memory layouts, and
other non-projected engine-internal concerns are **not** part of the
observable surface.

**R-1.** Two executions are *observably equivalent* when they
produce identical sequences on every exposed state and every output
channel and identical diagnostics up to permitted reorderings among
diagnostics from independent sources.

## 3. Structural determinism

**R-2.** Given the same Blueprint canonical form
([../canonical-form.md](../canonical-form.md)) and the same resolved
imports, a conforming engine MUST produce the same instance tree on
every load. The instance tree includes:

- the set of instantiated compounds;
- the parent-child relationships among instances;
- the local name at each instantiation site;
- the instance addresses assigned per
  [composition.md](composition.md) §7;
- the parameter values bound at each site;
- the initial state values per
  [../vocabulary/state.md](../vocabulary/state.md) §3.

**R-3.** Load-time diagnostics produced by composition
([composition.md](composition.md) §8) MUST be deterministic given
the same input. Two engines loading the same Blueprint and imports
MUST report the same load failures in the same form.

## 4. Replay equivalence

Execution determinism is required only under replay conditions.

### 4.1 Input trace

An **input trace** is a record of every value that arrived at every
external input of the Blueprint during an execution, along with its
relative ordering with respect to every other value in the trace.
Relative ordering is expressed as a total order among values on any
single channel, and a partial order across channels that records
which cross-channel ordering was observable at the original
execution's fan-in points
([channel-delivery.md](channel-delivery.md) §4).

### 4.2 The replay claim

**R-4.** Given the same Blueprint, the same imports, the same
initial state (loaded fresh per §3 or restored from a snapshot per
[lifecycle.md](lifecycle.md) §6.4), and the same input trace, a
conforming engine MUST produce an execution that is observably
equivalent (§R-1) to the original.

This is **replay equivalence**. It is the property that makes
Blueprint's snapshots durable across engines and time.

### 4.3 What replay equivalence implies

- Two engines given the same input trace produce the same exposed
  state sequences and the same output sequences, despite differences
  in internal scheduling.
- A recorded execution can be reproduced for audit.
- A snapshot plus subsequent input trace can be replayed from any
  point.

### 4.4 What replay equivalence does not imply

- Identical wall-clock timing. An engine MAY produce the same output
  sequence faster or slower than another.
- Identical internal representations. Two engines MAY hold state in
  different concrete forms provided projections agree.
- Identical interleaving of independent diagnostics. Diagnostics
  from independent scopes MAY appear in different relative orders
  between runs; diagnostics from a single scope are ordered.

## 5. Closed list of non-determinism sources

A conforming engine's observable surface MUST be deterministic except
for variation arising from exactly the following sources:

1. **Wall-clock time.** Any value derived from the engine's clock
   (timestamps, durations, timeout expirations) MAY vary between
   executions.
2. **External input arrival order.** The relative order of value
   arrivals on independent external inputs MAY vary between
   executions. At fan-in points within the Blueprint, this variation
   propagates per [channel-delivery.md](channel-delivery.md) §4.2.
3. **`asynchronous-unordered` delivery.** Messages on channels
   declared `asynchronous-unordered` MAY be delivered in different
   orders, duplicated, or dropped across executions per
   [../vocabulary/channel.md](../vocabulary/channel.md) §6.
4. **Concurrent intra-scope mutations not bounded by channel
   ordering.** When a scope processes events from independent
   producers whose order is not constrained by a common channel, the
   engine's scheduler determines the interleaving of resulting state
   mutations. Per
   [../vocabulary/state.md](../vocabulary/state.md) §4.2 each
   mutation is atomic; the order among them MAY vary.

**R-5.** Engines MUST NOT introduce non-determinism from any source
outside this list. Every observable variation MUST be attributable to
one of the four enumerated sources.

**R-6.** Engine-provided primitives MAY introduce non-determinism
only through the four enumerated sources. A primitive that draws from
a pseudo-random generator, for example, MUST obtain its seed from
external input (source 2) or the clock (source 1); it MUST NOT
introduce a fifth source.

### 5.1 Closure

The list in §5 is closed in 0.1.0-draft. A subsequent revision MAY
extend it; no extension is permitted in 0.1.0-draft.

## 6. Opt-in bit-exact execution

A small class of applications requires bit-exact reproducibility —
simulation engines, cryptographic protocols, scientific pipelines —
and cannot tolerate source (1) or source (2) variation. Blueprint
0.1.0-draft does not define a declaration by which an author
requests bit-exact execution. Authors who need it today rely on
primitive compounds that encapsulate the required guarantees
(synthesized-clock primitives, input-buffering primitives).

**R-7.** Engines MUST NOT silently provide bit-exact execution when
a Blueprint does not request it. Any such guarantee that does not
flow from an explicit compound declaration is an engine-specific
extension and MUST be labeled as such.

A future revision MAY introduce a declarative mechanism for
bit-exact requirements. Its design is out of scope for this
document.

## 7. Interaction with determinism claims elsewhere in the spec

Several documents in this specification commit to determinism. This
document reconciles them.

- [../overview.md](../overview.md) §5.2 claims that composition is
  deterministic — reconciled as structural determinism, §3.
- [../vocabulary/state.md](../vocabulary/state.md) §4.3 claims that
  given the same compound declaration, parameter bindings, initial
  state, and input message sequence, a compound produces the same
  state and output sequences — this is replay equivalence
  instantiated at the scope level, §4.
- [composition.md](composition.md) R-1 commits to instance-tree
  determinism — this is §3.
- [channel-delivery.md](channel-delivery.md) R-9 commits to
  deterministic fan-in interleaving given the input trace — this is
  §4 applied to channels.

All four are instances of this document's two tiers.

## 8. Normative requirements summary

- **R-1** Observable equivalence definition.
- **R-2** Structural determinism of the instance tree.
- **R-3** Load-time diagnostics deterministic.
- **R-4** Replay equivalence given input trace.
- **R-5** No non-determinism outside the closed list.
- **R-6** Primitives use enumerated sources only.
- **R-7** No silent bit-exact provision by engines.

## 9. Open questions

- Whether a declarative bit-exact mode belongs in a future revision
  (§6), and if so, how it interacts with sources (1) and (2).
- Whether the cross-channel ordering record in an input trace
  (§4.1) needs a normative serialization form. 0.1.0-draft leaves it
  to engine-defined snapshot formats.

---

*End of determinism.md.*
