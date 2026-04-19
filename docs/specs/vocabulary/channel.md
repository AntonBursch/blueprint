# Channel

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the **channel**: Blueprint's primitive for
> communication between compounds. A channel is a typed, named
> pathway for values that flow between compound instances over time.
> Channels are the only mechanism by which compounds communicate
> across encapsulation boundaries.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [compound.md](compound.md) — compounds declare channels on their interfaces and in their implementations
- [scope.md](scope.md) — channels are addressable only within a scope
- [schema.md](schema.md) — every channel has a declared payload schema
- [state.md](state.md) — the distinction between channel (stream) and exposed state (projection)
- [../overview.md](../overview.md) — the surrounding specification

---

## 1. What a channel is

A **channel** is a named pathway for values that flow between
compound instances over time.

A channel has:

- a **name**, unique within the scope where it is declared;
- a **schema** ([schema.md](schema.md)) describing the logical shape
  of every value the channel carries;
- a **direction**, relative to the compound declaring it:
  **input**, **output**, or **bidirectional** (§4);
- a **scope binding**: every channel lives within exactly one scope
  and is addressable only within that scope (§3);
- optional **delivery semantics**, specifying ordering, duplication,
  and reliability expectations (§6).

Values placed on a channel MUST validate against the channel's schema
at the boundaries where validation is REQUIRED (see
[schema.md](schema.md) §5.1).

### 1.1 Channels are not function calls

A channel is not a function call. A value placed on a channel does not
return a response. Request/response patterns are expressed as a pair
of channels (one in each direction) with matching correlation metadata
on their schemas, or as a primitive compound that implements
request/response internally. The channel itself carries values one
way.

### 1.2 Channels are streams, not variables

A channel's value is a *sequence of values over time*, not a single
current value. Consumers receive values as they are emitted. A channel
has no "current value" in the sense that a variable does; if that is
what an author needs, they should declare an **exposed state**
([compound.md](compound.md) §2.3), not a channel.

The distinction:

| Construct | Nature | Observable as |
| --- | --- | --- |
| Channel | Ordered sequence of values over time | A stream |
| Exposed state | A named value whose contents change over time | A current value |

Both are valid. Authors and tools choose based on which mental model
fits the interaction.

---

## 2. Channel declaration sites

Channels appear in two places in a Blueprint:

### 2.1 Interface channels

An interface **input** or **output** of a compound ([compound.md](compound.md)
§2) is a channel on the compound's interface. The compound itself
observes or emits on the channel; the enclosing scope connects the
channel to an internal channel or another compound's interface.

### 2.2 Internal channels

A composed compound ([compound.md](compound.md) §4) may declare
**internal channels** inside its implementation, connecting its
nested instances to one another and, where authorized by the
compound's interface, to the compound's own interface. Internal
channels are private to the compound's scope.

A channel's declaration is always one of these two things. There are
no free-floating channels and no channels that span scopes without
going through at least one compound's interface.

---

## 3. Scope binding

Every channel is bound to exactly one scope. That scope is:

- the scope of the compound whose interface declares the channel
  (for interface channels); or
- the scope of the composed compound whose implementation declares
  the channel (for internal channels).

A channel MUST NOT be addressed by name from outside its binding
scope. A channel MAY be addressed indirectly by a parent scope that
has been wired to one end of the channel through an interface member,
but the reference is through the interface member, not through the
channel itself.

This rule enforces encapsulation ([compound.md](compound.md) §3):
channels are not ambient resources that any compound can discover.

---

## 4. Direction

Channel direction is declared **relative to the compound that declared
the channel on its interface**:

- An **input** channel receives values from outside; the compound
  observes them.
- An **output** channel emits values to outside; the compound
  produces them.
- A **bidirectional** channel is both an input and an output on the
  same name. Bidirectional channels are RECOMMENDED only for
  request/response patterns where correlation is required.

Internal channels are not described by these terms — they are simply
connections between specified producer and consumer ports on nested
instances, with the direction determined by which end is the producer
and which is the consumer.

### 4.1 Events as outputs

Blueprint has no separate "event" channel. An event is a convention:
an output channel whose schema describes a discrete notification
(typically a type tag plus a payload) and whose delivery semantics are
suited to discrete occurrences. Authors and tools MAY use the word
*event* for such channels; normatively, an event is an output channel.

This matches the commitment made in [compound.md](compound.md) §2.2.

---

## 5. Wiring

A compound's implementation declaratively **wires** channels: it says
that an endpoint on one nested instance (or on the enclosing
interface) is connected to an endpoint on another nested instance (or
on the enclosing interface).

A wiring is valid when:

1. the two endpoints' schemas are compatible (consumer accepts every
   value the producer may emit);
2. the two endpoints' directions are compatible (a producer connects
   to a consumer);
3. the two endpoints are visible in the same scope (the enclosing
   compound's scope); and
4. encapsulation is respected (the enclosing compound is not reading
   into or writing into a nested instance's internals beyond the
   nested instance's declared interface).

Engines MUST reject Blueprints whose wirings violate any of the four
conditions.

The syntactic form of wiring declarations is finalized in the
Blueprint JSON Schema in Round 4. The examples in
[compound.md](compound.md) §11 illustrate an indicative form.

### 5.1 Fan-out and fan-in

A single producer endpoint MAY be wired to multiple consumer endpoints
(fan-out): every consumer receives every value.

A single consumer endpoint MAY be wired from multiple producer
endpoints (fan-in): the consumer receives values from every producer,
subject to the delivery semantics of the channel (§6). The ordering
of messages from different producers at a fan-in point is determined
by the channel's declared delivery semantics; by default, a fan-in
MUST preserve each individual producer's ordering but MAY interleave
across producers.

Fan-in MUST NOT combine producers whose schemas are incompatible with
the consumer's schema.

---

## 6. Delivery semantics

A channel's **delivery semantics** describe how the engine delivers
messages on the channel. Declared delivery semantics are advisory
requirements on the engine; they MUST be honored by any conforming
engine that can meet them, and engines that cannot MUST refuse to
run the Blueprint and MUST report the unsupported requirement.

Blueprint 0.1.0-draft defines three delivery modes:

- **`synchronous`** — the producer's emission and the consumer's
  receipt are a single coordinated action. The producer does not
  proceed until the consumer has received (or rejected) the value.
  Suitable for same-scope, same-process fine-grained control flow.
- **`asynchronous-ordered`** — messages are delivered in the order
  they were emitted, without duplication, without coordination
  between emission and receipt. The default for most channels.
- **`asynchronous-unordered`** — messages are delivered, possibly out
  of order, possibly duplicated, possibly dropped under load.
  Suitable for high-volume telemetry and streams where strict
  ordering is not required.

A channel MUST declare its delivery mode. If undeclared, the engine
MUST treat the channel as `asynchronous-ordered`.

### 6.1 Reliability

Blueprint 0.1.0-draft does not normatively define separate reliability
modes (at-most-once, at-least-once, exactly-once). The three delivery
modes above capture the coarse-grained requirement. More precise
reliability modeling is deferred to a future version; tools that need
it today SHOULD use schema metadata or primitive compounds that
implement specific reliability guarantees.

### 6.2 Backpressure

When a consumer cannot accept values as fast as a producer emits
them, the channel is under **backpressure**. Blueprint does not
prescribe a single backpressure strategy. Each delivery mode suggests
a default:

- **`synchronous`** — the producer blocks until the consumer
  receives.
- **`asynchronous-ordered`** — the engine SHOULD buffer up to a
  configured limit, then block the producer when the buffer is full.
- **`asynchronous-unordered`** — the engine MAY drop values when the
  consumer cannot keep up, and SHOULD surface the drop as a
  diagnostic.

Channels MAY declare explicit backpressure behavior (buffer size,
drop policy, block policy); the declaration syntax is fixed in
Round 4 and Round 5.

### 6.3 Ordering

Within a single producer-consumer pair on an `asynchronous-ordered`
channel, the engine MUST deliver values in the order the producer
emitted them, without gaps and without reordering. Across multiple
producers on a fan-in, the engine MUST preserve each producer's
ordering individually (see §5.1) but MAY interleave.

---

## 7. Runtime representation categories

A channel's runtime representation depends on where its endpoints
live. Blueprint recognizes three categories, introduced in
[../overview.md](../overview.md) §4.3:

### 7.1 In-memory channels

Both endpoints are in the same engine process. The payload passes as
the engine's native runtime value. Validation at channel boundaries
is REQUIRED for correctness in principle but MAY be elided when the
producer's static type system already guarantees conformance to the
schema (see [schema.md](schema.md) §5.1).

### 7.2 Streaming and binary channels

The payload is a binary or streamed value — audio, video, tensors,
images, raw buffers. The schema describes the payload's logical shape
using JSON Schema's `contentMediaType`, `contentEncoding`, and
`contentSchema` facilities (see [schema.md](schema.md) §6). The
payload passes in its native binary form at runtime; the engine MUST
preserve the payload's bytes across delivery.

Streaming channels MAY carry continuous payloads (audio streams,
video frames at rate, sensor streams). The schema describes the per-
element shape; the stream itself is a sequence of such elements
delivered at rate. Per-element validation in a streaming channel MAY
be elided after the first element validates, provided the engine
verifies that subsequent elements carry the same schema tag.

### 7.3 Cross-process channels

The endpoints are in different engine processes, hosts, or devices.
Cross-process channels require a wire encoding. JSON is the default;
engine pairs MAY negotiate a more compact encoding (MessagePack,
CBOR, Protobuf, Avro, Arrow) provided the payload round-trips without
loss and continues to validate against the channel's schema.

Cross-process channels REQUIRE validation at the decoding end (see
[schema.md](schema.md) §5.1). Wire encoding is an engine concern, not
a specification concern; the specification constrains only that
round-trip validation is preserved.

---

## 8. Channel versus exposed state

When an interaction is naturally described as "events happened in
this order" — clicks, messages, samples, notifications — an author
should use a channel.

When an interaction is naturally described as "this thing currently
has this value" — selection, theme, zoom level, cursor position — an
author should use an exposed state ([compound.md](compound.md) §2.3).

Both are supported. Both are valid. The choice shapes downstream
reasoning and the shape of consumers; neither is intrinsically
superior.

---

## 9. Normative requirements

A conforming Blueprint document:

1. MUST declare every channel (interface or internal) with a name and
   a schema.
2. MUST declare every channel's direction when it appears on a
   compound's interface.
3. MUST NOT wire channels whose schemas are incompatible at a
   producer-consumer pair.
4. MUST NOT wire channels that violate encapsulation.
5. MUST declare delivery mode where the default (`asynchronous-ordered`)
   is not desired.

A conforming engine:

6. MUST validate values at REQUIRED validation boundaries
   ([schema.md](schema.md) §5.1).
7. MUST preserve per-producer ordering on `asynchronous-ordered`
   channels.
8. MUST preserve payload bytes across binary and streaming channels.
9. MUST preserve round-trip validation across cross-process channels.
10. MUST reject Blueprints whose wirings violate encapsulation,
    direction, or schema compatibility.
11. MUST refuse to run a Blueprint whose declared delivery semantics
    the engine cannot meet, and MUST report the unsupported
    requirement.
12. MUST handle backpressure in a manner consistent with the
    channel's declared delivery mode and any explicit backpressure
    declaration.

A conforming authoring tool:

13. MUST NOT produce wirings that reference channel names not
    declared in the enclosing scope.
14. MUST NOT produce wirings that violate encapsulation or direction.

---

## 10. Examples

### 10.1 A simple output channel (event convention)

```json
{
  "outputs": {
    "clicked": {
      "schema": { "type": "object", "properties": { "x": { "type": "number" }, "y": { "type": "number" } }, "required": ["x", "y"] },
      "delivery": "asynchronous-ordered"
    }
  }
}
```

A discrete output: each emission describes one click event.

### 10.2 A streaming channel

```json
{
  "inputs": {
    "audio": {
      "schema": { "type": "string", "contentMediaType": "audio/pcm", "contentEncoding": "binary" },
      "delivery": "asynchronous-ordered"
    }
  }
}
```

A continuous PCM audio input. Each delivered element is a PCM buffer;
the schema's logical shape is "binary audio PCM".

### 10.3 A bidirectional channel for request/response

```json
{
  "channels": {
    "query": {
      "direction": "bidirectional",
      "schema": { "type": "object", "properties": { "correlation_id": { "type": "string" }, "request": { "...": "..." }, "response": { "...": "..." } }, "required": ["correlation_id"] }
    }
  }
}
```

Bidirectional channels are RECOMMENDED only for request/response
patterns where correlation metadata is part of the schema. The exact
encoding of direction on a bidirectional channel is finalized in
Round 4.

### 10.4 Fan-out

```json
{
  "wiring": [
    { "from": "source.reading", "to": "chart.series" },
    { "from": "source.reading", "to": "alarm.check" },
    { "from": "source.reading", "to": "logger.record" }
  ]
}
```

One producer emission reaches all three consumers. Each consumer
receives every value.

---

## 11. Open questions

1. **Precise reliability modes.** The three delivery modes are
   coarse. A future version MAY add at-most-once, at-least-once, and
   exactly-once as explicit modes.
2. **Cross-engine backpressure.** When producer and consumer are in
   different engine processes, backpressure crosses a wire. The
   semantics of cross-process backpressure propagation are deferred.
3. **Priority channels.** Some channels carry values that must
   preempt others (safety alerts, cancellation). Not in 0.1.0.
4. **Named delivery profiles.** Registered delivery profiles with
   precisely specified guarantees (e.g., "CloudEvents delivery",
   "MQTT QoS 1") would help cross-system interop. Deferred to
   extensions (Round 6).
5. **Bidirectional channel encoding.** The document notes
   bidirectional channels exist but does not fully fix their
   encoding. The JSON Schema in Round 4 will finalize the form.

---

## 12. Document status

This document is version `0.1.0-draft` of the channel vocabulary
document. It is normative within the matching Blueprint specification
version.

---

*Authored by Anton Bursch. Apache License 2.0.*
