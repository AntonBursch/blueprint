# Channel delivery

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document fixes the runtime semantics of channels. It specifies
> the default delivery mode, the intra-engine restriction on
> synchronous channels, the backpressure observability requirement,
> the deterministic ordering rule for fan-in, and the error-surface
> rule for schema failures at channel boundaries. It is the
> elaboration of [channel.md](../vocabulary/channel.md) §§6–7 into
> engine-normative form.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [../vocabulary/channel.md](../vocabulary/channel.md) — channel concept and delivery modes
- [../vocabulary/schema.md](../vocabulary/schema.md) — payload validation
- [composition.md](composition.md) — wiring comes from composition
- [lifecycle.md](lifecycle.md) — channel behavior at scope end
- [determinism.md](determinism.md) — what guarantees this document's rules support

## 1. Delivery modes

Three delivery modes are defined in
[../vocabulary/channel.md](../vocabulary/channel.md) §6:

- `synchronous`
- `asynchronous-ordered`
- `asynchronous-unordered`

No additional modes are defined in 0.1.0-draft. Extension modes MUST
use the `x-` or `ext.` namespace on the channel declaration.

### 1.1 Default

**R-1.** A channel whose delivery mode is unspecified MUST be treated
as `asynchronous-ordered`.

### 1.2 Semantics recap (normative)

- `synchronous` — the producer's emission and the consumer's receipt
  are a single coordinated action. The producer does not proceed
  until the consumer has received (or explicitly rejected) the value.
- `asynchronous-ordered` — messages delivered in the order emitted
  from a given producer, without duplication, without coordination
  between emission and receipt.
- `asynchronous-unordered` — messages delivered, possibly out of
  order, possibly duplicated, possibly dropped under load.

**R-2.** An engine that cannot meet a declared delivery mode for a
given deployment MUST refuse to run the Blueprint and MUST report the
unsupported requirement.

## 2. Synchronous channels are intra-engine only

A synchronous channel requires lockstep between producer and
consumer. That guarantee is not implementable across process or host
boundaries without prohibitive cost and without abandoning the
producer's independence.

**R-3.** A synchronous channel's producer scope and consumer scope
MUST reside in the same engine instance at runtime. An engine
deploying a Blueprint across multiple engine instances MUST refuse to
place a synchronous channel's endpoints in different instances and
MUST report the failure.

**R-4.** If a deployment tool lacks the information to place
synchronous-channel endpoints in the same instance, it MUST either
refuse the deployment or downgrade the channel to
`asynchronous-ordered` only with explicit author consent expressed
via an authoring-tool modification mark. An engine MUST NOT silently
downgrade a synchronous channel at deploy time.

## 3. Backpressure is observable

When a consumer cannot accept values as fast as a producer emits
them, the channel is under **backpressure**. The default per-mode
policies are named in
[../vocabulary/channel.md](../vocabulary/channel.md) §6.2. This
document adds the observability requirement.

**R-5.** Every backpressure event — a producer blocked, a value
dropped, a buffer exceeded — MUST be surfaced as a structured
diagnostic through the engine's observability surface
([authorship.md](authorship.md) §4). Diagnostics MUST identify:

- the channel, by the scope-qualified name used at its declaration
  site;
- the instance addresses of the producer and consumer endpoints;
- the event kind (block, drop, buffer-limit);
- the count of affected messages, when known.

**R-6.** Buffer sizes on `asynchronous-ordered` channels are an
operator concern, not an author concern. An engine MUST NOT require
authors to declare buffer sizes in a Blueprint. An engine MAY accept
buffer-size hints via engine-extension namespace fields; authors MAY
omit them.

## 4. Ordering

The ordering guarantees of each delivery mode are stated normatively
below.

### 4.1 Per-producer ordering

**R-7.** On a channel of mode `synchronous` or
`asynchronous-ordered`, the sequence of values received by a given
consumer MUST be consistent with the sequence emitted by each
individual producer that feeds that consumer. For a single producer,
message order is preserved end-to-end.

### 4.2 Fan-in interleaving

When a channel is fan-in (multiple producers to a single consumer),
the consumer observes messages from different producers in some
interleaving. This interleaving is determined by the relative arrival
order of the producers' values at the engine's fan-in point, which
is a function of the engine's scheduling and is not, in general,
fixed by this specification.

**R-8.** The interleaving across producers MUST preserve each
individual producer's order (R-7). An engine MUST NOT reorder a
single producer's output to accommodate another's.

**R-9.** Given the same **input trace** — an ordered record of the
values arriving at every producer endpoint with their relative
timing preserved — an engine MUST produce the same interleaving on
every execution. This is the per-replay determinism of fan-in and
supports [determinism.md](determinism.md) R-2.

Note: R-9 is a replay property. Two independent runs of the same
Blueprint with different external input arrival orders are permitted
to produce different fan-in interleavings. Determinism
[determinism.md](determinism.md) §3 enumerates the sources of such
variation.

### 4.3 No cross-channel ordering

This specification does not guarantee order across distinct
channels. A value emitted on channel `A` before a value emitted on
channel `B` by the same producer MAY be observed after `B`'s value
by a common downstream consumer.

**R-10.** Engines MUST NOT be required to preserve cross-channel
ordering. Authors who need cross-channel ordering MUST encode it
explicitly (for example, via a correlating compound or a single
merged channel with a discriminator in its schema).

## 5. Disconnection, end-of-life, and in-flight

A channel's endpoints are tied to scopes
([../vocabulary/channel.md](../vocabulary/channel.md) §3). When a
scope ends ([lifecycle.md](lifecycle.md) §4), its endpoints close.

### 5.1 Producer end

**R-11.** When a producer endpoint's scope ends, the engine MUST NOT
accept further messages from that endpoint. In-flight messages
already accepted from the producer remain subject to delivery per the
channel's mode.

### 5.2 Consumer end

**R-12.** When a consumer endpoint's scope ends, the engine MUST NOT
deliver further messages to that endpoint. Messages not yet
delivered MAY be dropped; engines MUST surface the drop as a
diagnostic per §3 (R-5).

### 5.3 In-flight at snapshot

For snapshot/restore ([lifecycle.md](lifecycle.md) §6), the engine's
record of in-flight messages on channels whose delivery mode
guarantees delivery (`synchronous`, `asynchronous-ordered`) MUST be
part of the snapshot. Messages on `asynchronous-unordered` channels
MAY be dropped at snapshot boundaries.

**R-13.** A restored Blueprint MUST receive the same sequence of
in-flight messages on `synchronous` and `asynchronous-ordered`
channels as the original engine was committed to delivering at the
moment the snapshot was taken.

## 6. Schema validation at boundaries

Every value placed on a channel MUST validate against the channel's
declared schema ([../vocabulary/schema.md](../vocabulary/schema.md)
§5.1). Validation happens at the channel boundary.

### 6.1 Failure handling

**R-14.** A value that fails validation at a channel boundary MUST
NOT be delivered to the consumer. The engine MUST:

1. reject the value;
2. surface a structured diagnostic per §3 (R-5) identifying the
   channel, the endpoints, and the violated schema constraint;
3. continue operating. Engines MUST NOT silently coerce, truncate, or
   substitute a rejected value.

### 6.2 No implicit error channel

This specification does not create an implicit error channel for
validation failures. An author who requires the producer to be
notified of a rejection MUST declare an explicit reverse channel or
use a primitive compound that implements the pattern.

**R-15.** Engines MUST NOT synthesize a channel not declared in the
Blueprint to carry validation-failure notifications.

### 6.3 Validation elision

Per [../vocabulary/channel.md](../vocabulary/channel.md) §4.3 of the
overview ([../overview.md](../overview.md) §4.3, in-memory channel
paragraph), an engine MAY elide validation on in-memory channels
when the producer's type system already guarantees conformance. This
elision is engine-internal; the observable behavior MUST be
indistinguishable from validating at the boundary.

**R-16.** An engine that elides validation MUST ensure the producer
cannot emit a value that violates the channel's schema. If that
guarantee cannot be proved, the engine MUST validate at the
boundary.

## 7. Cross-process channels

Channels spanning engine instances require a wire encoding. JSON is
the default ([../overview.md](../overview.md) §4.3). Engine pairs MAY
negotiate more compact encodings provided round-trip validation
holds.

**R-17.** An encoding negotiated between engines MUST preserve
schema validity: a value that validates before encoding MUST
validate after decoding. Engines MUST reject any encoded payload
whose decoded form does not validate.

**R-18.** Cross-process delivery MUST honor the channel's declared
delivery mode (R-2) across the network boundary, or the engines MUST
refuse to establish the cross-process channel.

## 8. Normative requirements summary

- **R-1** Default delivery mode is `asynchronous-ordered`.
- **R-2** Engines unable to meet the mode refuse the Blueprint.
- **R-3** Synchronous channels are intra-engine.
- **R-4** No silent downgrade at deploy time.
- **R-5** Backpressure events are diagnosed.
- **R-6** Buffer size is operator-configurable, not authored.
- **R-7** Per-producer order preserved.
- **R-8** Fan-in interleaving preserves per-producer order.
- **R-9** Interleaving deterministic on replay.
- **R-10** No cross-channel ordering guarantee.
- **R-11** Closed producer endpoint accepts no new messages.
- **R-12** Closed consumer endpoint receives no more; drops diagnosed.
- **R-13** In-flight delivery survives snapshot for reliable modes.
- **R-14** Validation failures reject, diagnose, continue.
- **R-15** No implicit error channel synthesized by engines.
- **R-16** Validation elision requires proven type safety.
- **R-17** Wire encoding preserves validation.
- **R-18** Cross-process honors mode or refuses to establish.

## 9. Open questions

- Whether mode extensions beyond the three should be formalized in a
  later revision (e.g., at-most-once, exactly-once as distinct
  declared modes). 0.1.0-draft keeps the three.
- Whether a standard diagnostic schema should be specified, or
  whether this document's "structured machine-readable" requirement
  is sufficient. 0.1.0-draft specifies neither a common schema nor a
  transport.

---

*End of channel-delivery.md.*
