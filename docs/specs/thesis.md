# Blueprint Thesis

**Status:** Draft
**Editor:** Anton Bursch

---

## The thesis, in one sentence

**Blueprint is a model of an app that lives inside the app.**

Not in the minds of the people who built it.
Not scattered across source files.
Not hidden in runtime memory only a debugger can reach.
A real, first-class, queryable, durable model — present at design time, at runtime, and at audit time.

## Why this matters now

Software is entering a regime where AI participates in both the **creation** and the **use** of applications. The traditional division — humans write code, humans run software, humans take responsibility — does not survive that regime intact.

Without an explicit model:

- AI's creative acts disappear into hidden glue code, invisible side effects, and runtime memory no human can reconstruct.
- Audit becomes archaeology. Responsibility becomes deniable.
- AI capability and AI accountability decouple — and in regulated domains, the gap becomes a liability the technology field cannot pay for.

Blueprint exists so that the gap closes by **construction**, not by discipline.

## What Blueprint is

A Blueprint is the application's own self-description, recorded as data the substrate runs:

- **Compounds** — the units of authored truth. Each is a self-contained graph + a public interface.
- **Edges** — every cross-compound connection, made explicit. There are no auto-wires, no name conventions, no magic.
- **The registry** — the runtime's self-knowledge: every compound def, every live instance, every edge and its current status.
- **Hosts** — organs that observe and render the model. The model has no opinion about which hosts are present.
- **Sources** — declarative entry points for every external signal: time, network, sensors, agents.

The model is the same artifact at design time, runtime, and audit time. Editors edit it. Engines run it. Auditors read it. AI agents propose changes to it. Humans approve those changes against it.

## What this gives us

1. **Constrained surface for AI.** AI agents that participate in apps can only touch the model. They cannot reach around it into hidden imperative code. The substrate's vocabulary is the AI's vocabulary; the substrate's invariants are the AI's invariants.

2. **Auditability by construction.** Every connection in the running app is declared in the model. There is no "what is this widget actually wired to" question that requires reading source. The wiring *is* the source.

3. **Reviewable change.** Model diffs (`+edge`, `-edge`, `+compound`, `propBinding changed`) are first-class artifacts. AI proposes; humans approve; the registry applies. No AI change reaches production without a structured, signed delta a human read.

4. **Human responsibility, by name.** Compounds carry signatures. Compositions carry signatures. The chain of responsibility is recorded in the artifact, not remembered in heads.

5. **Durability across hosts and time.** The model is the long-lived asset. React, Compose, Avalonia, Flutter — all render the same model. New hosts arrive; the model persists. New AI agents arrive; the model persists.

## What this is not

- **Not low-code.** Blueprint does not hide programming behind drag-and-drop. It promotes composition to a first-class authored act, distinct from coding.
- **Not AI-skeptical.** Blueprint does not constrain AI's capability. It constrains AI's accountability surface so that capability can be unleashed responsibly.
- **Not a framework.** Blueprint is a specification. Conforming engines are interchangeable. The artifact is portable.
- **Not centralized.** No vendor owns the spec. No service brokers the model. The artifact is yours.

## What follows from the thesis

Every other decision in this specification — the code/composition boundary, signatures, headless compounds, sources, propBindings, multi-spawn, dormant edges, the registry-as-truth — is a mechanical consequence of the thesis. If a decision violates the thesis, the decision is wrong, regardless of how convenient it is.

The substrate is small on purpose. The thesis is the asset. The mechanics serve it.

---

> "Once we introduce AI into apps and app creation, we need a model of the app — to keep the integration of AI both safe and efficient, to limit AI in the ways it needs to be limited so it can otherwise be unleashed, and to make sure that whatever an AI does, from creation to use of apps, is auditable for humans so humans can take responsibility for the apps."
> — Anton Bursch
