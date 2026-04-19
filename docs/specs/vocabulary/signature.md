# Signature

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the **signature**: a cryptographic proof of
> authorship and integrity over a Blueprint or a specific mark. A
> signature commits a signer, identified by a public key, to the
> canonical bytes of the signed-over value. Signatures turn Blueprint
> marks from claims into verifiable statements. Blueprint treats human
> and machine signatures symmetrically: a signature from a model's
> signing key is the same kind of evidence as a signature from a
> human's signing key, distinguished not by cryptographic strength but
> by what the accompanying mark's `author.kind` declares.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [overview.md](../overview.md) — §4.9 introduces signatures
- [rationale/co-authorship.md](../rationale/co-authorship.md) — symmetric human/machine signing
- [canonical-form.md](../canonical-form.md) — signatures sign over canonical form
- [mark.md](mark.md) — marks are the primary signing target

## 1. What a signature is

A signature is a JSON object that binds:

- a **signer** (identified by a public key), to
- **canonical bytes** of a specific value (the whole Blueprint, or a
  specific mark), by means of
- a **cryptographic algorithm** that an independent verifier can apply
  to the same bytes and obtain the same verdict.

Signatures are stored in the Blueprint document alongside the marks
and compounds they cover. A Blueprint with no signatures is legal but
unverifiable. A released Blueprint (§10) MUST carry at least one
signature.

## 2. Signature structure

A signature is a JSON object with the following members.

| Field | Requirement | Description |
| --- | --- | --- |
| `id` | MUST | Stable identifier for this signature (§2.1) |
| `mark` | MUST | What this signature covers (§4) |
| `algorithm` | MUST | Signature algorithm identifier (§3) |
| `public_key` | MUST | Signer's public key (§2.2) |
| `signature` | MUST | Signature bytes, base64url-encoded (§2.2) |
| `key_binding` | SHOULD | Reference binding the key to an identity (§8) |

Signatures reside in the top-level `signatures` array of the Blueprint
document, or inline on a mark via the mark's `signature` field
(see [mark.md](mark.md) §8). Inline signatures are equivalent in
meaning to top-level signatures whose `mark` field names the
containing mark's `id`; engines MAY normalize between the two forms,
but MUST preserve whichever form the authoring tool emitted.

### 2.1 `id`

`id` is a string that uniquely identifies the signature within its
Blueprint. Engines MUST accept either a UUID or a `sha256:<hex>`
content-addressed identifier (parallel to marks; see
[mark.md](mark.md) §2.1). The content-addressed form is computed over
the canonical form of the signature object with its `id` and
`signature` fields removed.

### 2.2 `public_key` and `signature` encoding

**R-1.** A signature's `signature` field MUST be base64url-encoded
([RFC 4648 §5](https://www.rfc-editor.org/rfc/rfc4648#section-5)),
without padding.

**R-2.** A signature's `public_key` field MUST be a string in a
format determined by the `algorithm` (§3). For the algorithms defined
in this document, `public_key` is a base64url-encoded byte sequence of
the raw public-key bytes as defined by the algorithm's standard.

## 3. Algorithms

Signature algorithms are partitioned into tiers.

| Tier | Identifier | Algorithm |
| --- | --- | --- |
| MUST | `Ed25519` | Ed25519 per [RFC 8032](https://www.rfc-editor.org/rfc/rfc8032) |
| MAY | `ECDSA-P-256-SHA-256` | ECDSA over NIST P-256 (secp256r1) with SHA-256, per [FIPS 186-5](https://doi.org/10.6028/NIST.FIPS.186-5) |
| MAY | `RSA-PSS-SHA-256` | RSA-PSS with SHA-256, MGF1-SHA-256, salt length equal to hash length, 2048-bit minimum modulus, per [RFC 8017](https://www.rfc-editor.org/rfc/rfc8017) |

**R-3.** Conforming engines MUST support `Ed25519`. An engine that
cannot verify `Ed25519` signatures does not conform to this
specification.

**R-4.** Conforming engines MAY support `ECDSA-P-256-SHA-256` and
`RSA-PSS-SHA-256`. An engine that supports an optional algorithm MUST
implement it per its referenced standard without modification.

**R-5.** An engine that encounters a signature in an algorithm it does
not support MUST reject the signature with a clear diagnostic
identifying the algorithm and the signature's `id`. Engines MUST NOT
silently skip unsupported signatures.

**R-6.** Extensions MAY define additional algorithm identifiers under
the `x-` or `ext.` namespace. Extension algorithms are advisory: an
engine that does not recognize the algorithm MUST reject per R-5.

### 3.1 Why Ed25519 as the baseline

Ed25519 is small, fast, side-channel resistant in straightforward
implementations, and widely available across language ecosystems. It
is symmetric in cost and difficulty across human-operated signing
tools and machine-operated signing services. Making it the MUST tier
ensures every conforming engine can verify every signature authored
by a tool that chose the safe default.

### 3.2 Why ECDSA and RSA are optional

ECDSA P-256 is required by many enterprise PKI deployments, HSMs, and
certificate chains; its inclusion enables Blueprints to carry
signatures anchored in those infrastructures. RSA-PSS is retained for
compatibility with older signing infrastructures where migration to
elliptic curves has not completed. Neither is required for a
conforming engine because neither is required for the core claim —
that authorship is verifiable.

## 4. Signed-over content

A signature commits its signer to a specific byte sequence. The byte
sequence is produced by canonical form ([canonical-form.md](../canonical-form.md))
applied to a specific target. Two targets are defined in 0.1.0.

### 4.1 Whole-Blueprint signature

When `signature.mark` is the literal string `"blueprint"`, the
signature is over the canonical form of the entire Blueprint document
with every `signature` field at every level removed (per
[canonical-form.md](../canonical-form.md) §4.1).

### 4.2 Mark signature

When `signature.mark` is the `id` of a mark in the Blueprint's `marks`
array, the signature is over the canonical form of that mark object
with its own `signature` field removed (per
[canonical-form.md](../canonical-form.md) §4.2).

### 4.3 Compound signature (reserved)

Per-compound signatures are reserved for a future revision. Canonical
form already defines the target ([canonical-form.md](../canonical-form.md) §3.3);
this document does not yet define what a compound signature means or
when it is required.

**R-7.** A signature whose `mark` field refers to an element other
than those permitted by §4.1 or §4.2 MUST be rejected.

## 5. Verification procedure

Given a Blueprint document `D` and a signature `S`:

1. Identify the target per `S.mark`:
   - If `S.mark` is `"blueprint"`, the target is `D` (whole-document case).
   - Otherwise, locate the mark `M` in `D.marks` whose `id` equals
     `S.mark`. If no such mark exists, verification fails.
2. Apply the appropriate signature-field exclusion rule from
   [canonical-form.md](../canonical-form.md) §4 to obtain the scope's
   signable value.
3. Canonicalize the signable value per RFC 8785.
4. Verify `S.signature` against the canonical bytes using
   `S.algorithm` and `S.public_key`. If verification fails, reject.
5. If `S.key_binding` is present, verify that `S.public_key`
   corresponds to the claimed identity per the binding's scheme (§8).
   If the scheme is unknown to the verifier, the verifier MUST report
   the binding as unverified but MUST NOT reject the signature on that
   ground alone in 0.1.0.

**R-8.** Engines MUST implement steps 1–4 of §5 exactly as specified.
Engines MAY implement step 5 subject to the schemes they support.

**R-9.** A signature that passes steps 1–4 is *cryptographically
valid*. A signature that additionally passes step 5 is *identity-bound*.
Tools SHOULD present the two statuses distinctly; they are not the
same guarantee.

## 6. Symmetric implementability

Every algorithm defined in §3 is implementable symmetrically by human
and machine operators:

- A human uses a local signing tool that holds the private key in a
  file, hardware token, or HSM.
- A machine uses a signing service that holds its private key in a
  managed key vault or HSM.

The signature bytes produced are indistinguishable in form and in
strength. The `author.kind` field on the accompanying mark
([mark.md](mark.md) §3) is what tells a reader whether the signer was
a human or a machine.

**R-10.** Engines MUST treat signatures from `kind: machine` authors
as equivalent in cryptographic force to signatures from `kind: human`
authors. Policy layers MAY weight them differently; the specification
does not.

## 7. Machine signing keys

Machine signing keys are first-class. A model, a tool, or a service
MAY hold its own signing key and sign marks under `author.kind:
machine`. The operator sub-object on the mark identifies the human
or organization on whose behalf the machine acts.

**R-11.** Engines MUST NOT require machine signatures to be
countersigned by a human. Co-signing is permitted (via a second
signature covering the same mark) but is a policy choice, not a
specification requirement.

**R-12.** When a machine's signing key is rotated or revoked, the old
key's signatures remain valid against their historical signed-over
bytes. Rotation and revocation are described in §8 and §9.

## 8. Key binding and identity

A public key is a cryptographic identifier; it is not, by itself, an
identity. `key_binding` provides a reference that binds a public key
to an identity in some identity system.

Permitted forms of `key_binding` include, but are not limited to:

- a URI pointing to a key server or verifiable credential
- a [DID](https://www.w3.org/TR/did-core/) (decentralized identifier)
- an X.509 certificate or certificate-chain reference
- a reference to a transparency log entry (e.g. sigstore / fulcio)

This specification does NOT prescribe an identity system. It commits
only that:

- `key_binding`, when present, is a string conforming to a registered
  identity scheme;
- engines that recognize the scheme SHOULD verify the binding per §5
  step 5;
- engines that do not recognize the scheme report the signature as
  cryptographically valid but not identity-bound.

**R-13.** Engines MUST NOT interpret `key_binding` semantically beyond
parsing its scheme. The choice of identity system is a deployment
concern, not a Blueprint concern (see
[rationale/non-goals.md](../rationale/non-goals.md)).

## 9. Revocation

Revocation has two layers, complementary:

### 9.1 Revocation as mark

A signature is revoked by a mark with `action: revoke` whose target is
the signature's mark id (see [mark.md](mark.md) §7). The signature
remains in the Blueprint for historical record. Tools MUST treat the
revoked signature as non-authoritative from the revocation's array
position forward.

**R-14.** When presenting a signature's status, tools MUST check the
mark array for a revocation targeting the signature's `mark` and, if
present, report the signature as revoked.

### 9.2 Key revocation

A `key_binding` scheme MAY provide its own revocation mechanism
(OCSP, CRL, certificate-transparency log, DID resolution). Engines
that recognize the scheme SHOULD honor its revocation signal.

**R-15.** Engines that do not support the scheme MUST NOT treat the
signature as revoked on account of the scheme being unknown.

### 9.3 Effect of revocation on released Blueprints

A signed release (§10) that is subsequently revoked remains in the
document. The release's mark is revoked via §9.1; the Blueprint
transitions back to draft via a new mark
([draft.md](draft.md), forthcoming).

## 10. Release signatures

Per [overview.md](../overview.md) §4.9 and §7.4, a released Blueprint
MUST carry a signed `release` mark. The signature over the release
mark (per §4.2) is what makes the release claim verifiable.

**R-16.** A Blueprint whose status is `released` MUST contain at least
one signature whose covered mark is of action `release`, and that
signature MUST be cryptographically valid per §5 steps 1–4.

**R-17.** After a release mark is signed, the Blueprint's
signed-over-content (the canonical form of the document at release
time minus signature fields) is committed. Any subsequent change
produces a new Blueprint with a new `id` or version, not a silent
edit to the released artifact.

## 11. Examples

### 11.1 Ed25519 signature on a mark

```json
{
  "id": "sha256:11aa22...",
  "mark": "sha256:d4e5f6...",
  "algorithm": "Ed25519",
  "public_key": "MCowBQYDK2VwAyEA…",
  "signature": "x0jKp9b7G…",
  "key_binding": "did:key:z6Mk…"
}
```

### 11.2 Whole-Blueprint release signature

```json
{
  "id": "sha256:33bb44...",
  "mark": "blueprint",
  "algorithm": "Ed25519",
  "public_key": "MCowBQYDK2VwAyEA…",
  "signature": "Qm9rR7L…",
  "key_binding": "https://keys.example.com/anton-bursch"
}
```

### 11.3 ECDSA P-256 signature

```json
{
  "id": "sha256:55cc66...",
  "mark": "sha256:cafe22...",
  "algorithm": "ECDSA-P-256-SHA-256",
  "public_key": "BPjA3P…",
  "signature": "MEQCIE…",
  "key_binding": "x509:spki-sha256:f3a9…"
}
```

## 12. Normative requirements (summary)

1. `signature` is base64url-encoded without padding (R-1).
2. `public_key` format is determined by the algorithm (R-2).
3. Engines MUST support `Ed25519` (R-3).
4. Optional algorithms implemented per their standards, unmodified (R-4).
5. Unsupported algorithms are rejected with a clear diagnostic (R-5).
6. Extension algorithms follow the same rejection rule (R-6).
7. Signatures target `"blueprint"` or a mark `id` only (R-7).
8. Verification steps 1–4 are mandatory; step 5 is optional (R-8).
9. Cryptographic validity and identity binding are distinct (R-9).
10. Human and machine signatures have equal cryptographic force (R-10).
11. Machine signatures do not require human counter-signature (R-11).
12. Key rotation does not invalidate historical signatures (R-12).
13. `key_binding` is syntactic; identity schemes are not prescribed (R-13).
14. Revocation-as-mark is honored when presenting signatures (R-14).
15. Unknown key-binding schemes do not imply revocation (R-15).
16. Released Blueprints carry a valid signed release mark (R-16).
17. Release commits signed-over bytes; subsequent changes create new Blueprints (R-17).

## 13. Open questions (deferred)

- **Compound signatures.** Per §4.3, per-compound signing is reserved
  but undefined in 0.1.0.
- **Multi-signature release policies.** Requiring N-of-M signatures
  for release (human + machine, or two humans) is plausible policy
  but not yet specified.
- **Timestamp authorities.** Binding a signature to a trusted
  timestamp (RFC 3161) is not addressed; mark timestamps are
  descriptive only.
- **Post-quantum algorithms.** Addition of post-quantum signature
  algorithms (e.g. ML-DSA) is expected in a later revision.
- **Key-binding registry.** The mechanism for registering and
  discovering `key_binding` schemes is out of scope for 0.1.0 and
  will likely be folded into the extension contract.
