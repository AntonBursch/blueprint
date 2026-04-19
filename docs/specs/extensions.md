# Extensions

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](overview.md), version 0.1.0-draft.

> This document fixes the extension mechanism that has appeared as
> `x-*` and `ext.*` keys throughout the specification. It defines
> where extensions may appear, what engines MUST and MUST NOT do with
> them, how they interact with canonical form and signatures, and how
> the two prefixes differ. The goal is a single consistent rule for
> extension handling that every engine and every authoring tool can
> follow without consulting a registry.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [canonical-form.md](canonical-form.md) — what hashes with the document
- [vocabulary/signature.md](vocabulary/signature.md) — what signatures cover
- [vocabulary/mark.md](vocabulary/mark.md) — marks also accept extensions
- [schema/README.md](schema/README.md) — the schema bundle opens every closed object to `patternProperties: { "^(x-|ext\\.)": true }`
- [semantics/composition.md](semantics/composition.md) — import resolution
- [versioning.md](versioning.md) — how the standard namespace evolves alongside the extension namespace

## 1. The two prefixes

An **extension key** is an object key that begins with one of two prefixes:

- **`x-`** — the **unregistered** prefix. Any author may use any
  `x-` key; collisions are the author's problem. `x-` is the
  lightweight choice for experiments, engine hints, and tool-local
  notes.
- **`ext.`** — the **reverse-DNS** prefix. Keys beginning with
  `ext.` SHOULD encode a reverse-DNS ownership claim:
  `ext.<reverse-dns-name>.<identifier>`. For example
  `ext.com.coned.weather-refresh` or `ext.ai.c3.type-binding`.
  `ext.` signals that the author owns the namespace and intends the
  extension to be stable across tools.

**R-1.** An engine MUST treat `x-` and `ext.` identically at the
level of load, canonical form, and signature coverage. The prefixes
differ only in the social claim they carry; this document normatively
defines no other distinction.

**R-2.** Extension keys are case-sensitive. `x-foo` and `X-Foo` are
distinct keys; engines MUST NOT normalize case.

**R-3.** The reverse-DNS convention for `ext.` is RECOMMENDED, not
REQUIRED. An engine MUST NOT refuse a Blueprint because an `ext.`
key fails to look like a reverse-DNS name. 0.1.0-draft defines no
central registry; collision avoidance is social.

### 1.1 Why two prefixes

`x-` comes from the JSON-Schema community convention for vendor
extensions. `ext.` comes from the reverse-DNS convention used in
Android manifests, OpenAPI vendor extensions, and XML namespaces.
Both have entrenched communities. Supporting both lets authors pick
the convention that matches their ecosystem; defining them as
identical at the engine level prevents a forked handling rule.

## 2. Where extensions may appear

**R-4.** Extension keys MAY appear on any object in a Blueprint
document. Every schema in this specification opens its closed
objects to `patternProperties: { "^(x-|ext\\.)": true }` for this
reason
([schema/README.md](schema/README.md)).

**R-5.** Extension keys MUST NOT replace required standard fields.
An extension key that shadows a required field (for example by
carrying the effective data the required field was meant to carry)
does not satisfy that requirement.

**R-6.** Extension keys MUST NOT change the meaning of a standard
field. An engine that recognizes an extension MAY attach additional
behavior to an object, but the standard fields on that object keep
their standard meaning.

**R-7.** Arrays are not objects. Extension keys belong on objects;
they cannot be inserted into arrays. An extension that needs
positional behavior MUST wrap its data in an object.

## 3. Standard namespace invariant

**R-8.** The names of standard fields in this specification MUST NOT
begin with `x-` or `ext.`. Every standard field name begins with a
letter (excluding `x` followed by `-`) and does not contain a period
before the first identifier character.

Corollary: a future minor or major version MAY add new standard
fields, and those additions MUST NOT collide with any existing
extension key used in the wild. `ext.com.example.feature` is
permanently safe from being stomped by a standard `feature` field;
`x-feature` is permanently safe from a standard `feature` field
(they begin differently).

## 4. Engine obligations for extensions

### 4.1 Unknown extensions

An **unknown extension** is an extension key the engine does not
recognize.

**R-9.** Engines MUST preserve unknown extensions round-trip. A
Blueprint that is loaded, snapshotted, restored, and saved MUST
emerge with its unknown extensions intact, at the same objects, with
the same values.

**R-10.** Engines MUST NOT alter behavior based on unknown
extensions. An extension the engine does not recognize is opaque
data; the engine does nothing with it.

**R-11.** Engines MUST NOT refuse a Blueprint solely because it
carries unknown extensions. An extension, by definition, is
permitted content.

### 4.2 Recognized extensions

A **recognized extension** is an extension key the engine documents
and acts upon.

**R-12.** An engine MAY recognize specific extension keys and attach
engine-defined behavior to them. The recognized behavior MUST be
documented in the engine's documentation, including:

- the exact key (including prefix);
- the shape of the extension's value;
- the behavior the engine attaches;
- any scope restrictions (e.g., "only on channel declarations").

**R-13.** A recognized extension's behavior MUST NOT violate any
normative requirement in this specification. An engine MUST NOT use
an extension to:

- silently downgrade a delivery mode
  ([semantics/channel-delivery.md](semantics/channel-delivery.md) R-4);
- bypass signature verification
  ([semantics/authorship.md](semantics/authorship.md) R-2);
- introduce a non-determinism source outside the closed list
  ([semantics/determinism.md](semantics/determinism.md) R-5);
- alter an instance address
  ([semantics/composition.md](semantics/composition.md) R-21);
- or otherwise modify standard semantics.

If an engine's extension behavior would violate a requirement, the
engine MUST NOT claim conformance while exercising that extension.

**R-14.** Two engines that both recognize the same extension key
MUST agree on its semantics. If they cannot agree, they MUST use
distinct keys (practically: distinct reverse-DNS namespaces).
Extensions are a coordination mechanism for willing parties; they
are not a mechanism for redefining shared keys.

### 4.3 Required recognition

A recognized extension is by default **optional**: the engine
attaches its behavior when the extension is present and does nothing
when it is absent. An engine MAY declare an extension **required**
for its deployment, meaning it refuses to load Blueprints that omit
the extension.

**R-15.** An engine that treats a recognized extension as required
MUST document the requirement explicitly. A Blueprint that meets
this specification but omits the engine's required extension is not
non-conforming to this specification; the engine's requirement is a
deployment constraint, not a specification constraint.

## 5. Canonical form and signatures

**R-16.** Extensions are part of the canonical form of a Blueprint
([canonical-form.md](canonical-form.md)). They participate in
deterministic key ordering, they hash with the document, and they
are covered by any signature over the document or any relevant
sub-object.

**R-17.** An engine or tool MUST NOT strip extensions before hashing
or signing. An extension the author included is, by inclusion,
intended to be part of the signed artifact.

**R-18.** An engine or tool MUST NOT add extensions to a Blueprint
it did not author. Engines process Blueprints; authoring tools
produce them. An engine that added an extension to a signed
Blueprint would invalidate the signature — and more fundamentally,
would be performing an authoring act outside its role.

## 6. Extensions in marks and signatures

Marks and signatures are objects; per R-4, extension keys MAY appear
on them. The mark and signature schemas explicitly allow
`patternProperties: { "^(x-|ext\\.)": true }`.

**R-19.** Unknown extension keys on a mark MUST be preserved
round-trip (R-9) and MUST NOT alter the engine's interpretation of
the mark (R-10). The mark's standard fields (`id`, `target`,
`action`, `author`, `timestamp`, and optional `parents`, `message`)
retain their standard meaning per
[vocabulary/mark.md](vocabulary/mark.md).

**R-20.** Unknown extension keys on a signature MUST be preserved
round-trip (R-9) and MUST NOT alter the verification procedure. The
signature's standard fields (`id`, `mark`, `algorithm`,
`public_key`, `signature`, optional `key_binding`) retain their
standard meaning per [vocabulary/signature.md](vocabulary/signature.md).

**R-21.** Extensions on a mark or a signature ARE covered by the
signature (R-16 applies recursively). An author who adds an
extension to a mark has committed to that extension's value as part
of what the signature attests.

## 7. Extensions in imports

An imported Blueprint may carry extensions the importing engine
does not recognize, including on the imported document's root, on
its compounds, or on its marks and signatures.

**R-22.** An engine MUST NOT refuse an import solely because the
imported Blueprint carries unknown extensions. Imports resolve per
[semantics/composition.md](semantics/composition.md); unknown
extensions in the imported document remain opaque and preserved per
R-9 through R-11.

**R-23.** If an engine recognizes an extension in its own execution
context, it MAY act on that extension when it appears in an
imported document, subject to R-13 (cannot violate standard
semantics). Whether the engine does so is an engine-policy decision
that MUST be documented.

## 8. Interaction with versioning

Extensions are orthogonal to the specification version
([versioning.md](versioning.md)). A Blueprint targeting
`0.1.0-draft` and a Blueprint targeting `0.2.0` MAY both carry the
same extension key with the same meaning.

**R-24.** A new specification version MUST NOT introduce a standard
field name that begins with `x-` or `ext.` (R-8). This guarantees
that an extension in use today cannot be stomped by a future
standard field.

**R-25.** A new specification version MAY standardize the semantics
of a previously-extension-space feature by introducing a new
standard field. The extension key and the standard field are
distinct names; the new standard field does NOT retroactively
consume the old extension. Authors MAY migrate; engines MUST treat
them as independent.

## 9. Diagnostics

**R-26.** An engine that refuses to load a Blueprint for reasons
related to extensions (for example, an unmet required recognized
extension per R-15) MUST produce a structured diagnostic
identifying:

- the refused extension key;
- the reason (missing required extension, violated engine-defined
  constraint on a recognized extension, etc.);
- the relevant engine documentation reference.

Engines MUST NOT refuse for reasons contrary to R-11 (i.e.,
"unknown extension" is never a valid refusal reason).

## 10. Normative requirements summary

- **R-1** `x-` and `ext.` identical at engine level.
- **R-2** Extension keys are case-sensitive.
- **R-3** Reverse-DNS in `ext.` is RECOMMENDED, not REQUIRED.
- **R-4** Extensions MAY appear on any object.
- **R-5** Extensions MUST NOT replace required standard fields.
- **R-6** Extensions MUST NOT change the meaning of standard fields.
- **R-7** Extensions on arrays are disallowed (arrays aren't objects).
- **R-8** Standard field names MUST NOT begin with `x-` or `ext.`.
- **R-9** Engines preserve unknown extensions round-trip.
- **R-10** Engines MUST NOT alter behavior on unknown extensions.
- **R-11** Engines MUST NOT refuse for unknown extensions alone.
- **R-12** Recognized extensions MUST be documented in engine docs.
- **R-13** Recognized behavior MUST NOT violate standard requirements.
- **R-14** Shared keys require agreed semantics, or distinct keys.
- **R-15** Required-extension deployment constraint is engine-scoped.
- **R-16** Extensions participate in canonical form and signatures.
- **R-17** No stripping extensions before hashing or signing.
- **R-18** Engines MUST NOT add extensions to Blueprints they didn't author.
- **R-19** Unknown extensions on marks are preserved and opaque.
- **R-20** Unknown extensions on signatures are preserved and opaque.
- **R-21** Extensions on marks and signatures ARE signature-covered.
- **R-22** Imports with unknown extensions MUST NOT be refused.
- **R-23** Recognized-extension handling in imports is engine-policy.
- **R-24** New standard fields never collide with extension prefixes.
- **R-25** Standardizing an extension-space feature uses a new name.
- **R-26** Extension-related refusal produces structured diagnostic.

## 11. Open questions

- Whether a central registry for `ext.` keys should exist in a later
  revision, and if so, whether registration becomes mandatory for
  `ext.` (it is not in 0.1.0-draft).
- Whether "required extensions" (R-15) deserve a declaration shape
  in the Blueprint itself (e.g., an `extensions_required` field that
  authors write and engines consume). 0.1.0-draft treats requirement
  as an engine-deployment concern only.
- Whether deprecation cycles for extensions (an author marking an
  extension as deprecated within a Blueprint) belong in a later
  revision. 0.1.0-draft does not define this.

---

*End of extensions.md.*
