# Co-authorship

**Status:** Draft. Informative, not part of the normative specification.

---

## Why this document exists

Two questions sit at the heart of Blueprint and can only be answered
together:

1. Who is authoring the software we are building?
2. What should the software artifact say about that?

For the decades in which software was written almost exclusively by human
engineers in text, the answer to the second question was: *nothing in
particular*. Authorship was tracked by the version control system the
source code happened to live in. A commit bore the name of a human; a pull
request was reviewed by other humans; a release was cut by a human. The
artifact itself — the compiled binary, the deployed service, the
distributed package — carried no authorship trace beyond what the
toolchain happened to embed.

That arrangement worked because the answer to the first question was
stable. Software was authored by humans. The version control system's
authorship record was close enough to the truth to be useful.

The arrangement has stopped working. Today, a substantial fraction of the
code committed to production systems is produced by language models,
sometimes unedited, sometimes extensively revised by humans, sometimes in
conversation with humans who neither wrote the prose nor fully read the
result. The version control system cannot tell the difference. The
committer field says "Alice" whether Alice typed every character, accepted
a generated suggestion, or directed a model to produce the change and
reviewed the diff afterward.

This is not a small problem. It is a gap in the record of who is
responsible for what, at the exact moment in history when that record
matters most. Blueprint takes the position that the gap must be closed in
the artifact itself, and that the mechanism for closing it must work for
humans and machines symmetrically — not by privileging one over the
other, not by pretending one does not exist, not by making one subordinate
to the other.

This document explains what that symmetry means, why Blueprint commits to
it, and what it implies for the normative specification of marks and
signatures.

---

## The claim

The core claim has three parts:

1. **Humans and AI systems are both authors.** Not metaphorically; not
   provisionally; not pending further theory. In the lived practice of
   software production in 2026 and onward, both kinds of authors
   regularly originate, modify, and approve material that ends up in
   running software. A specification that refuses to treat both as
   authors is a specification that refuses to describe reality.

2. **Their authorship should be distinguishable.** A Blueprint must make
   it possible to know, for any element, whether a human or a machine
   originated the change, modified it, or approved it — and which human,
   which machine, and in what context. This is not a judgment about which
   kind of author is better; it is a commitment to accuracy.

3. **Neither kind of authorship should be privileged in the
   specification.** The vocabulary for identifying a human author and the
   vocabulary for identifying a machine author are symmetric. Marks
   produced by a machine are not subordinate to marks produced by a
   human. Signatures from a machine are not weaker than signatures from
   a human. Whether an organization wants to require human approval for
   certain actions is a policy decision for that organization, expressed
   in its processes, not in the specification.

These three parts have to travel together. Removing any one of them
produces a specification that either denies reality, blurs it, or takes a
political position on what kind of author ought to be central. Blueprint
takes none of those positions.

---

## Why symmetry is the right commitment

The alternative commitments are each tempting and each wrong.

### The "human-only author" position

*Only humans are authors; AI output is a suggestion that a human
accepts.* This is the default position inherited from the version control
systems Blueprint is built around. It is wrong for two reasons.

First, it does not match the evidence. A human who has accepted thousands
of lines of generated code by scrolling through a diff in a hurry is not
in the same epistemic position as a human who wrote those thousands of
lines from scratch. Pretending both are "the author" collapses a
distinction that is actually present and actually matters.

Second, it creates incentives to suppress the record of machine
authorship. If only humans can be authors, then every machine
contribution must be laundered through a human signature, and the truth
of who produced what becomes unrecoverable. The record becomes less
honest over time, not more.

### The "AI-only author" position

*All software is now AI-authored; human involvement is quality
assurance.* This is the mirror image of the above, popular in certain
strains of contemporary rhetoric and equally wrong. It denies the
substantive contributions of engineers who design, review, correct, and
ship software in collaboration with models. It also creates the same
problem in reverse: the record of human authorship becomes unrecoverable.

### The "primary and secondary author" position

*Humans are primary authors; AI systems are tools that produce suggestions
attributed to the human operating them.* This is the position implicit in
most AI coding assistants today. It is convenient, but it degrades
gracefully in neither direction. When the AI has produced most of the
work, attributing all of it to the human is inaccurate. When the human
has transformed the AI's output so thoroughly that the AI's contribution
is structural advice only, attributing any of it to the AI is also
inaccurate. The "primary/secondary" framing picks a universal answer
where no universal answer exists.

### The symmetric position

Blueprint's position: **both kinds of author are first-class and
distinguishable.** The specification provides the primitives for
recording who did what. Organizations and individuals use those
primitives as they see fit. A mark produced by a language model naming
the model as author is a first-class mark, as valid and as verifiable as
a mark produced by a human. A mark produced by a human naming the human
as author is also first-class, equally valid and equally verifiable.
Neither is subordinate.

The specification does not, for example, say that a human signature can
override a machine signature. It does not say that a machine mark
"counts as" a human mark if a human later approved the work. It does
not say that a machine approval is weaker than a human approval. All of
those are policy decisions. The specification carries the facts;
organizations carry the policies.

---

## What this means for marks

Marks are the authorship primitive at the artifact level. The theory
above has concrete consequences for how marks are specified.

- **Every mark identifies its author.** Authorship is not optional.
- **The author identifier is rich enough to distinguish humans, machines,
  tools, and organizations.** A mark names one specific entity, not a
  vague "system" or "anonymous."
- **The kind of author is recorded structurally, not inferred from the
  name.** An engine or auditing tool must be able to answer "was this
  mark produced by a human or by a machine?" without parsing names.
- **The model, version, and context of a machine author are part of
  the record.** Attributing a mark to "a language model" is insufficient;
  attributing it to a specific model identity at a specific version
  operating under specific instructions is the standard the specification
  aims for. The exact set of required fields is fixed in the mark
  vocabulary document.
- **Marks are additive, not overwriting.** A later mark does not
  obliterate an earlier one. A long history of authorship can therefore
  be preserved, not collapsed.

The normative details will be fixed in [vocabulary/mark.md](../vocabulary/mark.md).
The rationale above is what that document must be faithful to.

---

## What this means for signatures

Signatures lift marks from claims to proofs. The theory has consequences
for signatures too.

- **A signature is always over canonical form.** Two parties verifying
  the same signature must compute the same bytes. Canonical serialization
  is therefore a prerequisite for the signature mechanism.
- **The signing algorithm must be implementable symmetrically by humans
  and machines.** Any cryptographic algorithm that a human can invoke
  through a signing device, a machine can also invoke. The specification
  does not permit schemes that presuppose a human operator (e.g.,
  biometric signatures that cannot be issued by a machine on behalf of
  itself).
- **Machine signing keys are real keys.** A language model operating on
  behalf of itself, with a published identity and a signing key, signs
  marks as a first-class author. The signature is no weaker and no
  stronger than a signature from any other author. What an organization
  trusts is a policy question.
- **Revocation is a first-class concern.** Keys can be rotated, and
  authorship made with a rotated key can be revoked by the original
  author. This applies equally to human and machine keys.

Normative details will be fixed in [vocabulary/signature.md](../vocabulary/signature.md).

---

## Risks the symmetric position accepts

Blueprint's position is not without risk. A public specification has to
be honest about them.

### Risk: machine authors without accountability

A machine can be named as an author without any natural person or
organization accepting responsibility for its actions. A signature from
a machine proves *that* the machine signed, not that anyone is
responsible for what was signed.

Blueprint's answer is that this is already true of any cryptographic
signature and is an organizational problem, not a specification problem.
The specification provides the mechanism; organizations provide the
policy that, for example, requires every machine author to be associated
with a named human or organization that accepts responsibility. The
specification does not mandate such a policy because specifications that
mandate policy lose portability.

### Risk: forged authorship

A bad actor with access to a signing key can produce signatures that
falsely attribute work. This is true of every signature scheme ever
designed. Blueprint does not claim to prevent this. It claims to make
forgery detectable when keys are rotated or compromised, and to preserve
the full history of signed material so that the extent of any forgery
can be determined.

### Risk: overclaiming of AI authorship

A human could attach a machine author to work the machine did not
actually do, either to dilute their own responsibility or to lend
spurious authority to a change. The specification cannot prevent this;
as above, it is a policy question. What the specification *does* provide
is a complete record that an organization's review processes can
interrogate.

### Risk: underclaiming of AI authorship

The reverse failure mode: a human uses AI heavily but attaches only a
human author, because the tooling makes it easy or the incentives make
it expedient. The specification cannot prevent this either. It can make
the honest option equally easy by providing symmetric primitives.
Authoring tools that take the specification seriously will produce
honest records by default.

---

## What this is not

This document takes a position on authorship. It does not take positions
on:

- **Whether AI systems are moral patients or agents.** Blueprint does
  not require a view on this.
- **Whether AI-authored software is better or worse than
  human-authored software.** Blueprint does not require a view on this.
- **Whether AI systems should be allowed to author software at all.**
  That is an organizational and societal question, not a specification
  question.
- **Whether a particular AI system is trustworthy.** Trust is policy.

Blueprint commits only to: *when software is co-authored, the record
should reflect it accurately, symmetrically, and verifiably.* Everything
else is left to the people using the specification.

---

## Related reading

- [ambition.md](ambition.md) — the ambition that makes co-authorship
  a requirement rather than an amenity
- [non-goals.md](non-goals.md) — in particular, "Blueprint is not an AI
  safety specification" and "Blueprint is not an identity or authorization
  system"
- [vocabulary/mark.md](../vocabulary/mark.md) — where the normative
  structure of authorship records is fixed
- [vocabulary/signature.md](../vocabulary/signature.md) — where the
  cryptographic side of authorship is fixed
