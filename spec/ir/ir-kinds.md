# Blueprint IR — Kind Taxonomy

**Status:** keystone draft, April 26, 2026.
**Source of truth for:** every other spec in `blueprint/spec/`, every implementation in `blueprint-sdk/sdk/ir/`, every printer in `blueprint-sdk/sdk/compilers/`, every conformance fixture.

---

## 1. What the IR is

A **Blueprint program is a graph**. The IR is the typed, normal-form
representation of that graph — the artifact every engine executes, every
compiler emits source from, every store persists, and every diagnostic walks.

The IR is:

- **Pure data.** No closures, no host bindings, no execution state. JSON-serializable end-to-end.
- **Structural.** Every node carries an explicit `kind` discriminator. Shape per kind is fixed.
- **Closed.** Adding a kind requires a spec change. Authoring a `kind` value not in this document is invalid IR.
- **Typed at the boundary.** Types appear on state declarations, parameters, returns, and literals. They do not appear on every expression.
- **Stratified.** Kinds belong to one of five tiers (Type, Expression, Statement, Resource, Compound). Mixing tiers is a structural error.

The IR is not a syntax tree of any source language. It is the program; the
source languages are projections.

---

## 2. The five tiers

```
Type         — describes shape of values            (Number, String, Array, Record, Ref)
Expression   — produces a value, no side effects    (Ref, Literal, BinaryOp, Compare, ...)
Statement    — performs an effect in a method body  (PropertyWrite, If, ForEach, Return, ...)
Resource     — long-lived runtime object            (Mount, StartTimer, Subscribe, ...)
Compound     — top-level program unit               (Compound, Method, State, Channel, ...)
```

A kind belongs to exactly one tier. The tier determines where it may appear
structurally:

- **Expression** kinds appear inside other expressions or as `value` of a Statement / Resource.
- **Statement** kinds appear in method bodies, `then` / `else` branches, loop bodies.
- **Resource** kinds appear in compound `lifecycle` blocks (`onMount`, `onTick`, `onUnmount`).
- **Compound** kinds appear at program root; they are not nested inside each other directly — references are by ID.

---

## 3. Canonical kinds

The kinds below are normative. Aliases observed in the sandbox demos are
listed under each kind; printer authors must accept canonical names only.
The reconcile pass that lifts demo IR into spec IR is responsible for
rewriting aliases.

### 3.1 Type tier

| Kind | Shape | Notes |
|---|---|---|
| `Number` | `{ kind: 'Number' }` | Numeric scalar. Targets choose representation (i32 / f64 / etc.). |
| `String` | `{ kind: 'String' }` | UTF-8 string. |
| `Boolean` | `{ kind: 'Boolean' }` | |
| `Void` | `{ kind: 'Void' }` | Method return type only. |
| `Array` | `{ kind: 'Array', element: Type }` | Homogeneous sequence. |
| `Record` | `{ kind: 'Record', fields: { key, type }[] }` | Structural product. Order matters for layout-sensitive targets (HLS, Verilog). |
| `Ref` | `{ kind: 'Ref', name: string }` | Named type reference (compound, record, sum). |

Future tiers reserved: `Sum`, `Tuple`, `Stream<T>`, `Optional<T>`. Not yet
required by any demo; do not introduce until a vertical demands it.

### 3.2 Expression tier

| Kind | Shape | Aliases observed | Notes |
|---|---|---|---|
| `Literal` | `{ kind: 'Literal', value, type: Type }` | `Lit`, `NumberLiteral` | Type required so printers can disambiguate `0` vs `0.0` (Kotlin lesson). |
| `Ref` | `{ kind: 'Ref', name: string }` | — | Read of a local / parameter / loop var. (Distinct from the *Type* `Ref` — same name, different tier; tier disambiguates.) |
| `Prev` | `{ kind: 'Prev', name: string }` | — | Read previous-tick value of a stateful field. Tick-runtime kind. |
| `PropertyRead` | `{ kind: 'PropertyRead', path: string[] }` | — | Read from compound state. `path` is a dotted access, e.g. `['vehicles']` or `['config','threshold']`. |
| `MemberRead` | `{ kind: 'MemberRead', target: Expr, key: string }` | — | Read field of an arbitrary expression (loop var, parameter). |
| `IndexedRef` | `{ kind: 'IndexedRef', base: Expr, index: Expr }` | — | Indexed array read. Required for HLS bounded loops. |
| `BinaryOp` | `{ kind: 'BinaryOp', op, left: Expr, right: Expr }` | — | `op ∈ { '+', '-', '*', '/', '%' }`. |
| `UnaryOp` | `{ kind: 'UnaryOp', op, operand: Expr }` | — | `op ∈ { '-', '!' }`. |
| `Compare` | `{ kind: 'Compare', op, left: Expr, right: Expr }` | — | `op ∈ { '==', '!=', '<', '<=', '>', '>=' }`. |
| `LogicalAnd` | `{ kind: 'LogicalAnd', left, right }` | — | Short-circuit. |
| `LogicalOr` | `{ kind: 'LogicalOr', left, right }` | — | Short-circuit. |
| `LogicalNot` | `{ kind: 'LogicalNot', arg }` | — | |
| `Ternary` | `{ kind: 'Ternary', cond: Expr, then: Expr, else: Expr }` | `Conditional` | Expression form. Distinct from `If` statement. |
| `ObjectLiteral` | `{ kind: 'ObjectLiteral', fields: { key, value: Expr }[] }` | — | Constructs a Record value. |
| `StringConcat` | `{ kind: 'StringConcat', parts: (string \| Expr)[] }` | — | Template-literal-style. |
| `RegexTest` | `{ kind: 'RegexTest', pattern: string, flags: string, value: Expr }` | — | |
| `BuiltinCall` | `{ kind: 'BuiltinCall', name: string, args: Expr[] }` | — | Reserved for spec-defined helpers (e.g. `len`, `abs`). Names are normative. |

### 3.3 Statement tier

| Kind | Shape | Aliases observed | Notes |
|---|---|---|---|
| `LetMutable` | `{ kind: 'LetMutable', name: string, type?: Type, init?: Expr }` | `LocalDecl` (immutable variant — *see issue below*) | Mutable local. |
| `LocalWrite` | `{ kind: 'LocalWrite', name: string, value: Expr }` | — | Reassign a `LetMutable`. |
| `PropertyWrite` | `{ kind: 'PropertyWrite', path: string[], value: Expr }` | — | Write to compound state. |
| `StatefulWrite` | `{ kind: 'StatefulWrite', name: string, value: Expr }` | — | Tick-runtime: produce next-tick value. Distinct from `PropertyWrite` (which is immediate). |
| `If` | `{ kind: 'If', cond: Expr, then: Stmt[], else: Stmt[] }` | — | Statement form. |
| `Return` | `{ kind: 'Return', value: Expr }` | — | |
| `ForEach` | `{ kind: 'ForEach', source: Expr, paramName, paramType: Type, body: Stmt[] }` | — | Unbounded array iteration. |
| `BoundedForEach` | `{ kind: 'BoundedForEach', count: number, as: string, body: Stmt[] }` | — | Compile-time bounded; required for HLS pipelining and FPGA/VHDL targets. |
| `ArrayPush` | `{ kind: 'ArrayPush', target: string, element: Expr }` | — | Append to an array-typed state field. |
| `Filter` | `{ kind: 'Filter', source: Expr, paramName, paramType, predicate: Expr }` | — | Pure expression in usage; tier-statement when used standalone. *See issue below.* |
| `Map` | `{ kind: 'Map', source: Expr, paramName, paramType, expr: Expr }` | — | SIMT-friendly; key kind for GPU/FPGA targets. |
| `GroupBy` | `{ kind: 'GroupBy', source: Expr, keyFn: Expr }` | — | Aggregate into `{ key, count }[]`. |
| `Stream` | `{ kind: 'Stream', source, window: number, body: Stmt[] }` | — | Bounded dataflow region. HLS / streaming targets. |

### 3.4 Resource tier

Resources live in a compound's lifecycle blocks (`onMount`, `onTick`,
`onUnmount`). They represent long-lived bindings to runtime systems.

| Kind | Shape | Notes |
|---|---|---|
| `Mount` | `{ kind: 'Mount', childType: string, args?: Expr[], byProp?: string }` | Spawn child compound. `byProp` keys the instance for later `UnmountBy`. |
| `UnmountBy` | `{ kind: 'UnmountBy', childType: string, byProp: string, value: Expr }` | Tear down child(ren) matching `byProp == value`. |
| `StartTimer` | `{ kind: 'StartTimer', intervalMs: number, method: string }` | Periodic invocation of a method on this compound. |
| `Subscribe` | `{ kind: 'Subscribe', channel: string, method: string, pass: string[] }` | Channel listener. `pass` names the args to forward. |
| `Publish` | `{ kind: 'Publish', channel: string, args: Expr[] }` | Emit on channel. Statement-tier when used in method body; resource-tier when fired on lifecycle hook. *See issue below.* |

### 3.5 Compound tier

| Kind | Shape | Notes |
|---|---|---|
| `Compound` | `{ kind: 'Compound', id, state: StateField[], methods: Method[], lifecycle?: { onMount?, onTick?, onUnmount? } }` | The unit of composition. |
| `Method` | `{ kind: 'Method', name, params: Param[], returns: Type, body: Stmt[] }` | Named procedure on a compound. |
| `Channel` | `{ kind: 'Channel', name, args: Param[] }` | Declared message channel. |
| `Module` | `{ kind: 'Module', compounds: Compound[], channels: Channel[] }` | Program root. |

State fields and parameters are records, not kinds:
- `StateField = { id: string, type: Type, init?: Expr }`
- `Param = { name: string, type: Type }`

---

## 4. Structural invariants

These hold for any valid IR module. The reconcile pass and any
pre-printer validator must enforce them.

1. **Tier discipline.** Expression kinds may not appear where Statements are required, and vice versa. (e.g. `Ternary` is an Expression; `If` is a Statement. Compilers that synthesize one from the other do so explicitly.)
2. **Closed kind set.** Any node with a `kind` not listed in §3 is invalid.
3. **Path well-formedness.** `PropertyRead.path` and `PropertyWrite.path` resolve through declared state field types. The validator walks the type chain.
4. **Channel referential integrity.** Every `channel` named by `Subscribe` / `Publish` / `Channel` resolves to a `Channel` declaration in the enclosing module.
5. **Compound referential integrity.** Every `childType` named by `Mount` / `UnmountBy` resolves to a `Compound` declaration in the enclosing module.
6. **Method referential integrity.** Every `method` named by `StartTimer` / `Subscribe` resolves to a `Method` on the same compound.
7. **`Prev` is tick-only.** A `Prev` expression is valid only inside a method invoked from `onTick` (transitively). The validator computes call closures.
8. **Bounded loop constants.** `BoundedForEach.count` is a literal integer at IR-construction time, not an expression. This is what makes FPGA/HLS pipelining tractable.
9. **No expression side effects.** Expression-tier kinds do not write state. `Filter` / `Map` / `GroupBy` are pure transformations. Anything that mutates is a Statement.

---

## 5. Open issues to resolve before this is normative

These are deliberate calls flagged for review. None should be silently
"decided" by the implementation — each is a spec-level question.

### 5.1 `Filter` / `Map` / `GroupBy` — Expression or Statement?

In the sandbox demos these appear as the `value` of a `Return` Statement,
which is the **Expression** position. But `Stream` (which composes the same
kinds) is invoked as a top-level body block — **Statement** position.

**Proposal:** they are **Expressions** that produce a sequence value.
`Stream` is a Statement that *contains* an Expression-tier pipeline. This
matches the SIMT compile model (one Expression → one kernel launch).

### 5.2 `Publish` — Statement, Resource, or both?

Demos use it inside method bodies (Statement) and inside `onTick`
(Resource). The shape is identical.

**Proposal:** classify as **Statement**. A "fire on every tick" usage is
just a Statement inside the `onTick` method body, not a separate Resource.
This keeps the Resource tier strictly about *long-lived bindings* (Mount,
Subscribe, Timer) that have lifetime semantics.

### 5.3 `LetMutable` vs immutable `LocalDecl`

Demos used `LocalDecl` for immutable bindings and `LetMutable` for mutable
ones. The current spec collapses both into `LetMutable`, but loses the
mutability contract.

**Proposal:** add `LetConst` (immutable, init required) as a separate
Statement kind. Targets that distinguish (Rust, Kotlin `val` vs `var`,
Swift `let` vs `var`) get the information; targets that don't (Python,
MATLAB) erase it. This is information-preserving, which is the right
default for an audit-grade IR.

### 5.4 `Prev` and the tick model

`Prev` is the only kind that requires runtime context (which tick are we
in?) outside of state-read kinds. It is the seam between the IR and the
**runtime spec** (`spec/runtime/tick-model.md`, not yet written).

**Proposal:** keep `Prev` in this taxonomy as the contract surface, but
defer the formal semantics to the runtime spec. The runtime spec must
answer: at tick-0, what does `Prev` return? Is there a writeable initial
value, or is `Prev` undefined on first tick? Is `Prev` snapshotted at tick
boundaries or at field-write time?

### 5.5 Kind explosion vs kind generality

The compiler series has accumulated kinds opportunistically. Some pairs
are arguably sub-cases of one general kind:

- `BoundedForEach` could be `ForEach` with a literal-typed source.
- `IndexedRef` could be `MemberRead` with a numeric key.
- `Stream` could be a `Compound` lifecycle hook over a streaming source.

**Proposal:** **keep them split.** The IR is the contract surface; printer
authors and auditors read it. Specialization at the kind level is a
*declaration of intent* that survives into target code. Collapsing kinds
to gain "elegance" punishes downstream tools — every printer has to
re-derive the case analysis. The current verbosity is a feature.

### 5.6 Reconcile against `GraphRuntime` node taxonomy

The pre-IR-first runtime spec (legacy `specs/`) has a separate node-kind
taxonomy used inside `GraphRuntime` (~74 processors). Some overlap:

- `Map` (IR) ≈ map processor
- `Filter` (IR) ≈ filter processor
- `Compound` (IR) ≈ compound node
- `Subscribe` / `Publish` (IR) ≈ channel ports

**Proposal:** the **IR taxonomy supersedes** the processor catalog.
Processors that survive become spec-defined `BuiltinCall`s or compile to
sequences of canonical kinds. The processor catalog is a legacy artifact
to be retired during the runtime spec rewrite. *Do not* attempt forward
compatibility — it would constrain the IR to dashboard-era assumptions.

---

## 6. What this document does not yet cover

Listed so the gaps are explicit, not silently absent:

- **Generics / type parameters.** No demo uses them yet.
- **Sum types / tagged unions.** Implied by `Subscribe` semantics (channels
  carry typed payloads) but not formalized.
- **Effects / capabilities.** What permissions does a method need? Critical
  for the auditability story — defer to a dedicated spec doc.
- **Module imports.** Currently every program is a single `Module`. Real
  programs will compose. Defer.
- **Annotations / metadata.** `_path` tags on demo IR were ad-hoc. The IR
  needs a normalized `meta` slot per node for diagnostic/source-map use.
  Add as `meta?: Record<string, unknown>` on every kind.

---

## 7. Versioning

This document is **v0.1**. The IR is unstable until v1.0. v1.0 ships when:

1. All §5 open issues are resolved.
2. The full sandbox `14-compiler/` series compiles from IR conforming to this taxonomy (i.e. demos lift cleanly into `blueprint-sdk/sdk/ir/` types).
3. The conformance suite passes for at least three printers (one general, one vertical, one audit-target).
4. `spec/compile-target/` and `spec/runtime/` are both at draft-quality.

After v1.0, kind additions require a minor version bump; kind removals or
shape changes require a major version bump and a migration document.
