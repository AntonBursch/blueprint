# Channel

**Status:** Draft — placeholder. Content not yet written.

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

## Purpose

Defines channels as Blueprint's primitive for communication between compounds.
A channel is a typed, named pathway for messages, with a direction, a schema,
and delivery semantics. Channels are how compounds exchange information
without sharing state.

## Dependencies

- [scope](scope.md)
- [schema](schema.md)

## Planned sections

1. What a channel is
2. Channel identity and scope-binding
3. Payload schema and the logical-shape rule
4. Runtime representation categories (in-memory, streaming/binary, cross-process)
5. Directionality (in, out, bidirectional)
6. Connection and wiring
7. Delivery semantics (best-effort, at-least-once, exactly-once declarations)
8. Ordering guarantees
9. Backpressure model
10. Channel vs. state: when to use which
11. Normative requirements
12. Examples
13. Open questions

---

*Scaffolding placeholder. See [ROADMAP.md](../ROADMAP.md) for authoring order.*
