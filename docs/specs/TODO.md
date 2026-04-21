# Blueprint Specification — TODO

Working plan for Rounds 8–19. Companion to [ROADMAP.md](ROADMAP.md). Internal
planning document, not part of the normative spec.

Status: Active. Last updated 2026-04-19, after Round 7 (conformance) closed.
Branch: `dev`.

---

## The principle that governs every remaining round

**The Blueprint specification defines the contracts at the seams of the system;
everything else — implementation, performance, polish, operational
sophistication — is unspecified and competitive.**

For each round below, the question to answer is not "what should be in the
spec?" but: **"what's the minimum interop contract such that any conforming
implementation interoperates with any other?"** That is the spec; everything
else is implementation detail and out of scope.

This principle should also be written into the spec itself — likely as a
`design-principle.md` document or as an expansion of `overview.md` — at the
appropriate round (probably 8 or alongside the architectural rounds).

---

## The free / paid line — the strategic frame

The reference implementations Anton releases at v1.0 establish the "free tier"
of the Blueprint ecosystem. They prove the contracts are sufficient and define
the floor of what's possible. They are not intended to be the best
implementations on the market.

The free reference web app (TypeScript / Node / React) at v1.0 will be:

- Cross-server, multi-instance per server
- Different UIs for different roles
- Server-to-server orchestration
- Mix of HTTP, WebSocket, in-process channels
- A real distributed app, not a toy

### Free transports (mandatory in the spec)

- In-process / in-memory
- HTTP / JSON (request-response, server-sent events)
- WebSocket / JSON (bidirectional streaming)

### Paid transports (vendor; must conform to the transport contract)

- TCP, UDP, QUIC
- Shared memory, IPC, RDMA, DMA
- Industrial protocols (OPC-UA, Modbus, MQTT-with-broker)
- Hardware buses (CAN, I2C, SPI)
- Vendor-specific high-throughput message buses (Kafka, Pulsar, NATS) with
  enterprise reliability

### The invariant that protects the line

**Transport choice is a deployment concern, never an artifact concern.**

The artifact says "this channel needs ordered delivery, async semantics,
throughput X." The deployment manifest picks the wire transport. An app
authored against the channel spec runs on either tier — you swap transports in
the deployment manifest, not in the artifact. This will be a normative
requirement in Round 12 (channel transports).

This pattern generalizes to every seam:

| Seam | Free implementations | Paid implementations |
|---|---|---|
| Engine | Reference engines (TS/Node/Python) | Optimized engines (.NET, Java, Rust, embedded; multi-core schedulers) |
| Adapter | Reference adapters (web, CLI, basic native) | Premium adapters (native iOS/Android with custom GPU pipelines, AAA UX) |
| Transport | In-process, HTTP, WebSocket | TCP, UDP, RDMA, hardware buses, industrial protocols |
| Store | SQLite, in-memory, basic Postgres | Distributed stores with cross-region replication, sub-ms latency |
| Package | Standard library packages | Domain-specific commercial packages (e.g. quantitative finance) |
| Planner | Naive topology placement | Constraint-solver placement, cost-model optimization, live re-planning |
| Bundler | Reference bundler | Optimizing bundlers (tree shaking, AOT compilation, code splitting) |
| Authoring tool | Reference CLI scaffolder | Agent-driven IDEs with intent-tracing, live preview, fleet UX |
| Deployment platform | Local Docker Compose runner | Multi-cloud control planes with fleet management, RBAC, audit |

Both columns conform to the same contracts. Enterprises mix freely. No vendor
locks the artifact. **Lack of lock-in is the product.**

---

## Round plan — 12 technical rounds + 2 narrative rounds

Each technical round answers exactly one interop-contract question. Most can be
written tighter than Rounds 5 and 7 (the "keystone" rounds); these define
interfaces, not new semantics.

| Round | Seam / Topic | Contract question |
|---|---|---|
| **8** | Naming fix + capability extensions | Rename JSON-field "extensions" to a non-colliding term. Add capability-extension spec matching SDK `extension-contract.md` v3.1.0. May merge into Round 10. |
| **9** | Capabilities | What must a compound declare for the planner to verify safety properties before execution? |
| **10** | Packages | What must a package manifest contain for any engine to load it correctly? |
| **11** | Adapters | What must an adapter implement for any engine to drive it? |
| **12** | Channel transports | What must a transport implement for any engine on either end? Mandatory in-process, HTTP, WebSocket bindings. The transport contract for vendor transports. |
| **13a** | Distribution model | What kinds of distributed shapes does 0.1.0 promise interop for? What's deferred to 0.2? |
| **13b** | Deployment manifests | What must a manifest contain for any orchestrator to read it? |
| **14** | Planner contract | What must a planner consume and produce for its output to be usable by any engine? |
| **15** | Stores | What must a store implement for any engine to use it? |
| **16** | Project file | What must a project file contain for any tool to understand it? |
| **17** | Package registry | What must a registry serve for any client to consume it? |
| **18** | Examples + primers | (Pedagogy. The original "Round 8" before this re-plan.) |
| **19** | Prior art + closer | (Narrative. Where Blueprint sits among IPLD, OCI, SLSA, in-toto, W3C VC, JSON Schema, OpenAPI, Nix, Bazel.) |

---

## Round 8 — Naming fix + capability extensions (next)

### The naming collision

- Public spec `extensions.md` (Round 6) uses "extension" for JSON keys with
  the `x-` or `ext.` prefix. 26 normative requirements.
- SDK `extension-contract.md` v3.1.0 uses "extension" for capability packages —
  the things that ship processors, widgets, themes, templates, skills, and
  connector behavior.
- Same word, two operationally different concepts.

### Decision: Option A

- Rename the JSON-field document to a non-colliding term. Word choice still
  pending — candidates: "annotations", "vendor fields", "specification
  extensions", "metadata fields". Anton to choose at Round 8 memo time.
- The 26 requirements move with the renamed document; their content does not
  change.
- Write a new public-spec document under the name "extensions" that is the
  public counterpart to the SDK's `extension-contract.md`. This may end up
  being Round 10's package spec under a different name; decide at memo time
  whether to combine.

### Cross-references that must update

- `overview.md`
- Schema bundle's `patternProperties` documentation
- Every spec document that cites `extensions.md` for the JSON-field rule
- `REQUIREMENTS.md` — section heading and cross-references

---

## Distribution scope — heterogeneous instances (Anton's case)

Round 13 must support: one Blueprint app spanning multiple machines, multiple
instances per machine, instances that may use **different engines** and
**different package profiles**, each instance doing different things,
orchestrated by the artifact.

Implications already captured for Round 13 memos:

- Per-instance package profiles in the deployment manifest.
- Aggregate conformance claim shape (deployment-level claim composed from
  per-instance claims).
- Cross-engine channel rules: a synchronous channel between compounds placed on
  different engines must be refused at planning time, or upgraded with an
  explicit diagnostic.
- Wire encoding for cross-language schema fidelity (decimal, big-int, dates).
- System-level non-determinism at engine boundaries — make explicit.
- Package version coherence across instances.
- Fault tolerance contract — what an engine MUST do when it loses a peer.

---

## Distributed-pattern coverage — 0.1.0 commitments

Table to be elaborated in Round 13a. Initial taxonomy:

| Pattern | 0.1.0 status |
|---|---|
| Single-instance | Covered |
| Client-server | Covered with R13 |
| Three-tier client / server / edge | Covered with R13 + R14 |
| Cross-language fleet | Covered (per-engine conformance) |
| Heterogeneous-instance orchestration | Covered with R13 + per-instance profiles |
| Federation (multi-org, same artifact) | Partial; trust-chain spec deferred |
| Offline-first (intermittent sync) | **Weak**; durable channels + conflict resolution NOT in 0.1.0 |
| High-cardinality fleet (thousands of devices) | Partial; fleet-aware deployment manifest needed |
| Peer-to-peer | Covered (transport contract) |
| Hybrid cloud / on-prem | Covered with R13 |
| Embedded | Covered (adapter-as-embedded) |

R13a must explicitly enumerate what's deferred to 0.2 so silence does not read
as "Blueprint handles offline-first."

---

## Open decisions deferred but flagged

### Scope as primitive vs. modifier

The from-scratch design said: 4 primitives (compound, channel, state, schema)
+ 3 modifiers (scope, capability, requirement). The spec drifted; scope is
currently a primitive. Changing this after 0.1.0 release is a major-version
bump. Decide before final 0.1.0 release.

### Standard library tier

Should there be a Blueprint Standard Library — a curated set of first-party
packages versioned alongside the spec? Python / Node / Rust precedent says
yes. Strategic call. Decide before final 0.1.0 release.

### Capabilities-as-modifier (Round 9)

Compounds declare required capabilities (network, file system, GPU, unbounded
time, etc.) and granted capabilities to children. Planner can prove safety
properties before execution. Critical for AI authorship safety. Memo at
Round 9.

---

## SDK framing

The `blueprint-sdk` repository is the **reference implementations of every
conforming role** — free, MIT-licensed, sufficient to run real apps. It is
**not** "the implementation of Blueprint." Worth a documentation pass on the
SDK README and architecture document when the public spec round plan
completes, so the relationship between spec and SDK is explicit.

---

## Resume rhythm

Rhythm is unchanged from Rounds 1–7:

1. Memo for the next round → Anton confirms decisions
2. Write to disk
3. Commit on `dev` (this branch) with `spec:` prefix and detailed second
   message
4. Tag `v0.1.0-draft-roundN` annotated; push
5. Move to next round

When 0.1.0-draft is complete (after Round 19), merge `dev` to `main` for the
final draft tag, then resolve the open decisions, then publish 0.1.0 release.
