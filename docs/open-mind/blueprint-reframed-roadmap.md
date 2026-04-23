# Blueprint Reframed Roadmap

## Purpose

This roadmap defines how Blueprint Reframed will be delivered in a focused sequence:

- web first
- new applications first
- deterministic code translation as the trust anchor
- graph and code debugging in parallel

The goal is to prove core value quickly, then expand safely.

## Strategic Scope

### In Scope First

- TypeScript + React/Next.js reference stack
- Greenfield applications with clean composition boundaries
- Code <-> graph round-trip for supported constructs
- Graph-native LLM composition with deterministic code emission
- Native dev-server preview and hot reload
- Native debugger + graph projection

### Deliberately Deferred

- Full legacy codebase migration as a default path
- Multi-language parity on day one
- Owning or replacing framework runtimes
- Broad third-party code rewriting

## Core Success Criteria

Blueprint Reframed is successful only if both pillars work in production workflows:

1. Round-trip reliability
- Developers can move between code and graph without semantic drift for supported constructs.

2. Parallel debugging
- Developers can debug running code normally and see synchronized graph execution state.

If either fails, adoption fails.

## Product Phases

## Phase 0: Foundation Lock

### Objective

Lock contracts, semantics, and trust boundaries before scaling implementation.

### Deliverables

- Canonical graph schema and identity model
- Deterministic compiler contract
- Source provenance contract
- Dependency ownership policy:
  - workspace code: editable
  - third-party dependencies: read-only node surfaces
- Debug mapping contract (code location <-> graph node)

### Exit Gate

- Architecture contracts approved and stable enough for implementation

## Phase 1: Web Greenfield MVP

### Objective

Ship end-to-end value for new web applications.

### Deliverables

- TypeScript importer for supported constructs
- Graph composition engine
- Deterministic TypeScript emitter
- Validation pass with fail-fast diagnostics
- LLM graph composition actions (constrained)
- CLI or extension command pipeline for round-trip

### Supported Construct Set (MVP)

- top-level variables
- top-level functions
- top-level classes
- imports/exports
- explicit unsupported-node diagnostics for everything else

### Exit Gate

- Developers can build a real small app in code + graph loop
- deterministic output stability demonstrated

## Phase 2: Native Preview and Debug Projection

### Objective

Make Blueprint feel native in day-to-day engineering workflows.

### Deliverables

- Stable generated file locations in workspace
- Framework dev-server integration for hot reload
- Source/node mapping index
- Code-to-graph navigation
- Graph-to-code navigation
- Graph-highlight on debugger pause
- Node breakpoint to code breakpoint translation (initial)

### Exit Gate

- Engineer can iterate while seeing app as code, graph, and running preview in one loop

## Phase 3: Team Hardening and Governance

### Objective

Make the workflow safe for teams and production pipelines.

### Deliverables

- semantic diff view for graph changes
- approval and audit trail for LLM-applied graph edits
- stronger validation suite:
  - duplicate symbol protection
  - reserved-name and ownership checks
  - unsupported construct guardrails
- compiler version pinning for reproducibility

### Exit Gate

- Team-level trust in repeatability and reviewability

## Phase 4: Ecosystem and Blueprint-Friendly Libraries

### Objective

Enable composable external capabilities without runtime lock-in.

### Deliverables

- read-only dependency node ingestion from type surfaces
- package-based node catalogs
- metadata for capability, ownership, and policy
- clean imports/invocation-only code emission for dependency nodes

### Exit Gate

- Developers can compose apps with internal + external libraries safely

## Phase 5: Legacy and Multi-Language Expansion

### Objective

Expand beyond greenfield web once core model is proven.

### Deliverables

- incremental adoption strategy for existing codebases
- module-by-module onboarding flows
- fallback passthrough node handling for unsupported legacy segments
- first non-TS language adapter pair (importer + emitter)
- first non-web platform adapter path

### Exit Gate

- Existing projects can adopt Blueprint incrementally without rewrite mandates

## Metrics by Phase

### Adoption Metrics

- time from project start to first useful app composition flow
- weekly active engineers using code+graph loop
- percentage of graph edits accepted without manual rollback

### Trust Metrics

- deterministic compile reproducibility rate
- number of semantic round-trip mismatches per release
- debugger mapping accuracy at breakpoint/pause events

### Productivity Metrics

- median time to implement cross-cutting composition changes
- review size reduction for composition-level changes
- median debug resolution time for composition defects

## Non-Negotiable Principles

- Blueprint does not replace native runtimes.
- The LLM plans composition; deterministic compilers emit final code.
- Workspace code is editable; third-party dependency internals are read-only.
- Graph and code must remain synchronized and traceable.
- Unsupported cases must fail visibly, never silently.

## Decision Rules

- Do not expand language/platform scope before proving web greenfield reliability.
- Do not prioritize legacy migration over debugger and round-trip trust.
- Do not hide uncertainty: emit explicit diagnostics for unsupported constructs.

## Immediate Next Execution Steps

1. Freeze MVP construct support list and unsupported diagnostics list.
2. Implement deterministic source mapping index in emitted artifacts.
3. Add debugger pause -> graph highlight path in the extension surface.
4. Add node breakpoint -> code breakpoint mapping for supported nodes.
5. Run pilot projects on new TypeScript web apps and collect metrics.

## Summary

Blueprint Reframed should be built as a disciplined sequence:

- prove value quickly in web greenfield workflows
- harden trust through deterministic round-trip and parallel debugging
- scale outward to ecosystem libraries, legacy onboarding, and multi-language platforms

This preserves momentum while protecting quality and trust.
