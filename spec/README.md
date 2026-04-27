# Blueprint Spec — IR-First Rebuild

**Status:** rebuild in progress (April 2026). This tree replaces `specs/` once it stabilizes.

## Why a new tree

The original `specs/` was authored before Blueprint's IR was crystallized. The
`blueprint-sandbox/` lab — particularly the `14-compiler/` series (#19–#25) —
discovered that Blueprint is fundamentally an **IR + tick-driven runtime**
that compiles to many surfaces, not a graph-on-top-of-React dashboard tool.

This tree is the spec catching up to where the lab has already been.

## Layout

```
spec/
  ir/                  # IR-kind taxonomy + semantics — the keystone
  runtime/             # tick model, scheduling, lifecycle
  compile-target/      # printer contract + conformance suite
  targets/             # per-target manifests (subset matrix as data)
  engine/              # engine contract (rewritten against IR)
  adapter/             # adapter contract (rewritten against IR)
  store/               # 4-store contract (App/Compound/Connection/Skill)
  catalog/             # extensions, connections, skills, compounds
```

## Authoring order (and rationale)

1. `ir/` — every other doc references it.
2. `compile-target/` — codifies what the #14 demos already do.
3. `runtime/` — tick semantics; the one piece the compiler series doesn't fully exercise.
4. Then engine / adapter / store / catalog rewrites against the new IR.

## Relationship to legacy `../specs/`

The legacy `specs/` tree is preserved unchanged. It will be archived to
`blueprint-archives/` in a single commit once `spec/` stands on its own
and sandbox demos cite the new tree.

Do not edit legacy specs. Do not import from them. Treat them as read-only
historical context.
