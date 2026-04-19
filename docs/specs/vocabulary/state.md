# State

**Status:** Draft — placeholder. Content not yet written.

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

## Purpose

Defines state as Blueprint's primitive for values that persist and evolve
within a scope. A state has a schema, an optional initial value, update
semantics, and an observation interface. States are distinct from channels:
a channel carries messages, a state holds a current value.

## Dependencies

- [scope](scope.md)
- [schema](schema.md)

## Planned sections

1. What a state is
2. State identity and scope-binding
3. Typed states and schema binding
4. Initial values and initialization order
5. Update semantics (replace, merge, transactional updates)
6. Observation (read, watch)
7. Persistence and restoration across runtime restarts
8. State vs. channel: when to use which
9. Normative requirements
10. Examples
11. Open questions

---

*Scaffolding placeholder. See [ROADMAP.md](../ROADMAP.md) for authoring order.*
