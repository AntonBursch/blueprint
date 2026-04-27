# Spec authoring index

Tracks which spec docs exist, which are next, and where each fits.

| Doc | Status | Tier | Depends on |
|---|---|---|---|
| [`ir/ir-kinds.md`](./ir/ir-kinds.md) | **v0.1 draft** (2026-04-26) | keystone | — |
| `compile-target/contract.md` | not started | next | `ir-kinds.md` |
| `compile-target/manifest-format.md` | not started | next | `ir-kinds.md` |
| `compile-target/conformance.md` | not started | next | `compile-target/contract.md` |
| `runtime/tick-model.md` | not started | parallel to compile-target | `ir-kinds.md` § 5.4 |
| `runtime/lifecycle.md` | not started | parallel | `ir-kinds.md` § 3.4 |
| `runtime/scheduling.md` | not started | parallel | `tick-model.md` |
| `targets/*.json` | not started | data | `compile-target/manifest-format.md` |
| `engine/contract.md` | not started | downstream | `runtime/*` |
| `adapter/contract.md` | not started | downstream | `runtime/*` |
| `store/contract.md` | not started | downstream (mostly survives from legacy) | `ir-kinds.md` (Compound) |
| `catalog/extensions.md` | not started | downstream | all of the above |

## Authoring rule

Every doc cites the kinds it touches by canonical name from
[`ir/ir-kinds.md`](./ir/ir-kinds.md). If a doc needs a kind that doesn't
exist there yet, **the kind is added to `ir-kinds.md` first**, never
ad-hoc in the consuming doc.
