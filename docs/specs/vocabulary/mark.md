# Mark

**Status:** Draft — placeholder. Content not yet written.

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

## Purpose

Defines marks: authorship traces attached to elements of a Blueprint. Every
substantive change to a Blueprint SHOULD be accompanied by a mark identifying
the author (human or AI), the action, and the context. Marks are first-class
and distinguish human authorship from AI authorship without privileging either.
The term derives from the printer's mark used in bookbinding to identify
the shop responsible for a work.

## Dependencies

- [compound](compound.md)
- [channel](channel.md)
- [state](state.md)

## Planned sections

1. What a mark is
2. Mark structure (author, kind, target, timestamp, parent, message)
3. Author identity: human, AI, tool, organization
4. Author kind is structural, not inferred from the author name
5. Required fields for machine-author identity (model, version, operating context)
6. Distinguishing AI and human authorship without privileging either
7. Mark kinds (create, modify, approve, reject, annotate, release)
8. Attachment to scopes, compounds, channels, states
9. Marks are additive, not overwriting; chains and history
10. Approval workflows
11. Normative requirements
12. Examples
13. Open questions

The normative content of this document must be faithful to the rationale
established in [rationale/co-authorship.md](../rationale/co-authorship.md).

---

*Scaffolding placeholder. See [ROADMAP.md](../ROADMAP.md) for authoring order.*
