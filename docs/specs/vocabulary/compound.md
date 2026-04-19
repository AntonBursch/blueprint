# Compound

**Status:** Draft — placeholder. Content not yet written.

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

## Purpose

Defines compounds as Blueprint's unit of composition. A compound is a named,
reusable declaration that may contain other compounds, scopes, channels,
and states. Every Blueprint document has an entry compound; every richer
design is expressed by compounds instantiating and importing other compounds.

## Dependencies

- [scope](scope.md)
- [schema](schema.md)
- [channel](channel.md)
- [state](state.md)

## Planned sections

1. What a compound is
2. Compound identity, naming, and version pinning
3. Declared surface: parameters, inputs, outputs, exposed states
4. Internal composition (nested compounds, scopes, channels, states)
5. Instantiation
6. Import and overlay
7. Encapsulation and visibility rules
8. Compound vs. scope: structural vs. dynamic
9. Entry compound
10. Normative requirements
11. Examples
12. Open questions

---

*Scaffolding placeholder. See [ROADMAP.md](../ROADMAP.md) for authoring order.*
