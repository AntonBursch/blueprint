# Ambition

**Status:** Draft. Informative, not part of the normative specification.

---

## What this document is for

Every normative decision in the Blueprint specification is measured against
the ambition described here. When a design choice is unclear, the question is
not "what is elegant" or "what is popular," but: **does this serve the
ambition?** This document exists so that question always has an answer to
check against.

The ambition is best stated in three layers, from the deepest to the most
immediate. All three are the same project at different distances. The deep
layer is the *why*. The middle layer is the *what*. The immediate layer is
the *how*.

---

## 1. The deep ambition

**Blueprint exists to make software authorship universal.**

Not "democratize coding." That phrase has been worn thin by decades of
low-code tools that promised it and delivered a ceiling. The ambition is
stronger: the act of turning an intent into running software should be
available to anyone who can clearly describe what they want — a domain
expert, a manager, an operator on a factory floor, a teacher, a student —
and the resulting artifact should be as legitimate, durable, and extensible
as anything a professional would produce.

The gap that exists today is not an interface problem. It is a translation
problem. Human intent is high-dimensional, messy, and expressed in natural
language. Running software is low-dimensional, precise, and expressed in
a formal grammar. The translation between the two has historically been
done by a scarce labor force at enormous cost and latency. Every attempt
to bypass that labor force has produced tools that work for the easy cases
and collapse when the work gets real.

What has changed is that machines can now do a meaningful part of the
translation. Not all of it. Not perfectly. But the direction is clear and
the rate of improvement is real.

Blueprint exists because the translation work itself needs a target. A
language model producing code today is producing artifacts into an
ecosystem built for humans — frameworks, implicit conventions, undocumented
folklore, a thousand ways to do each thing. The broader shift in authorship
does not finish until there is a **formal substrate designed for humans
and machines to co-produce into**:

- a vocabulary small enough to learn,
- expressive enough not to hit a ceiling,
- honest enough to compose at scale,
- neutral enough not to be owned by any one vendor,
- durable enough to outlive the tools that produced it.

Blueprint aims to be that substrate.

---

## 2. The middle ambition

Sitting on top of the deep ambition is a more concrete one:

> **Blueprint aims to be to applications what HTML is to documents.**

HTML succeeded not because it was elegant. It succeeded because it was:

| HTML property | Why it mattered |
|---|---|
| A finite, learnable vocabulary | `<p>`, `<a>`, `<div>`, the rest — a small set anyone could grasp |
| Authored by many hands | Written by professionals in text editors, by non-professionals in WYSIWYG tools, now by AI — all producing the same artifact |
| Rendered by many engines | Netscape, Internet Explorer, Firefox, Chrome, Safari, command-line fetchers, screen readers, search crawlers, language models |
| A specification, not a product | No vendor owns it. Implementations compete; the artifact survives |
| Extensible without forking | New tags added carefully; decades-old pages still render |
| Agnostic about intent | Marketing, science, government, recipes, fiction — HTML does not care what the document is about |

Applications never received this treatment. Each generation tried:
enterprise Java, web frameworks, mobile frameworks, low-code platforms.
Each ended up as a vendor's product or a developer's framework. The
*artifact* never became first-class the way an HTML document did. You
cannot hand someone "the app" the way you can hand them "the page."

### Why now

The preconditions that make this possible did not exist in 1995:

- Software could not be distributed durably in a platform-independent form.
- It could not be authored collaboratively with machines.
- It could not span browsers, servers, and edge devices with a single model.
- JSON had not become a universal lingua franca.
- A generation of engineers had not yet internalized declarative composition.

All of these are now true.

### The middle ambition, stated

> A portable, durable, vendor-neutral artifact format for applications —
> readable and writable by humans, by AI systems, and by every kind of
> implementation — that captures what an application *is* across behavior,
> state, boundary, distribution, time, and authorship.

If Blueprint works, an application becomes a document: something you can
diff, version, fork, port, share, and hand to any of several conforming
runtimes. Today an application is an entangled pile of repositories, deploy
pipelines, vendor accounts, and tribal knowledge. Blueprint aims to close
that gap.

---

## 3. The immediate ambition

Blueprint will not achieve its long ambitions unless it is useful now.
The immediate ambition is therefore concrete:

> **Blueprint aims to be a credible, neutral substrate for the applications
> being built today — by software teams whose applications are increasingly
> co-authored by humans and AI systems, and by organizations that cannot
> afford to have their working knowledge locked inside any one vendor's
> proprietary format.**

The present moment matters. Work that is co-authored by humans and AI
systems is accelerating, and every proprietary framework that becomes the
default in that work raises the cost of producing portable artifacts later.
A neutral, public specification available now is worth more than a better
one available later. This is the only layer of the ambition that is
time-bounded, and it is the layer that determines whether the other two
are reached at all.

---

## The three ambitions, compressed

| Layer | Ambition | Why it matters |
|---|---|---|
| **Deep** | Make software authorship universal by giving humans and machines a shared substrate to compose into | Why Blueprint should exist at all |
| **Middle** | Be to applications what HTML is to documents — portable, vendor-neutral, multi-implementation | The shape of the win |
| **Immediate** | A credible, neutral substrate usable now, before proprietary defaults foreclose the alternative | The path that actually gets there |

---

## What Blueprint is not

Clarifying the ambition also means being clear about what it rules out.

- **Not another low-code tool.** Low-code platforms optimize for producing
  one vendor's artifact in one vendor's runtime. Blueprint optimizes for
  producing a neutral artifact runnable by many.
- **Not another visual programming language.** Visual is one authoring
  surface among several. The artifact is the point; the visual form is a
  rendering of it.
- **Not another AI coding assistant.** Coding assistants produce code for
  human-authored ecosystems. Blueprint produces artifacts into an ecosystem
  designed for co-authorship.
- **Not a replacement for software engineers.** Engineers remain the primary
  authors of the vocabulary, the implementations, and the sophisticated
  compounds. The goal is to make their work compose and scale beyond them,
  not to do without them.

A more complete list is in [non-goals.md](non-goals.md).

---

## How the design reflects the ambition

The normative choices in the specification are not independent. Each is
made because the ambition above demands it.

| Design choice | Ambition served |
|---|---|
| The specification is the product, not any particular implementation | HTML-like durability; vendor neutrality |
| A small, fixed vocabulary | Learnability by humans and machines |
| Multiple conforming engines | Plural implementations; no single runtime owns the artifact |
| Scope as a first-class structural primitive | Keeps ambient concerns out of the artifact |
| Composition by named, reusable compounds | Scale without ceiling |
| Declarative composition with imperative units inside | Composability without loss of host-language expressiveness |
| Authorship by humans and machines treated symmetrically | The reason the present moment matters |
| JSON as the carrier | Diffable, portable, shareable, universally implementable |
| Separation of artifact from connection, adapter, and store concerns | Agnostic about intent — Blueprint does not care what kind of application is being expressed |

Every choice is load-bearing. Remove any of them and the ambition weakens.

---

## Using this document

When the design conversation zooms in — to a vocabulary term, to a runtime
rule, to a conformance clause — the risk is losing sight of why any of it
is being built. This document is the return path. Every proposed decision
should be checked against the three-layered ambition:

- Does it move toward making authorship universal?
- Does it keep the artifact portable, durable, and vendor-neutral?
- Does it help Blueprint be a credible neutral substrate for the work being
  done now?

A decision that fails all three is almost certainly wrong. A decision that
clearly serves one and is neutral on the others is almost certainly right.
A decision that serves one at the cost of another deserves a deliberate
debate.

---

## Related reading

- [overview.md](../overview.md) — the specification at a glance
- [openapi-precedent.md](openapi-precedent.md) — what Blueprint takes from
  the OpenAPI precedent and where it departs
- [co-authorship.md](co-authorship.md) — the theory of humans and AI as
  peer authors
- [non-goals.md](non-goals.md) — what Blueprint explicitly is not
- [prior-art.md](prior-art.md) — earlier efforts at this ambition and why
  they fell short
