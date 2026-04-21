# 01 — Premise: An App is a Function

> **TL;DR** — Every app, no matter what it looks like from the outside, is a **function**: inputs come in, something happens, outputs go out. A React component, a REST handler, a scheduled job, a game, a robot control loop, and a whole SaaS product are all functions. They look different because their **edges** (input sources and output sinks) are different. The middle is always the same. This is the single premise the whole Blueprint platform rests on.

---

## The premise

> **An app is a function.**

Inputs come in. Something happens. Outputs go out.

Sometimes the input is a mouse click. Sometimes it is a clock tick. Sometimes it is a packet on a network. Sometimes it is a sensor reading. Sometimes it is a human pressing "Buy."

Sometimes the output is a pixel on a screen. Sometimes it is a row in a database. Sometimes it is a packet on a network. Sometimes it is a motor firing. Sometimes it is `void` — nothing at all, just a side effect the world absorbs.

The shape of the thing — inputs leading to outputs through some transformation — is the same regardless of what the inputs and outputs actually are.

---

## Every kind of app is a function

Let's walk this through specific examples, because the premise is slippery when it is stated in the abstract. Each of the following is, structurally, the same shape.

### A React component

```
props + lifecycle events  →  the component  →  DOM
```

The component is a function. Its inputs are its props and the lifecycle events that fire on it (mount, update, unmount, events from children). Its outputs are the DOM it describes and the side effects it emits. React calls them "function components" for exactly this reason.

### A REST handler

```
HTTP request  →  the handler  →  HTTP response
```

Obviously a function. Pure request-in, response-out shape. A handler that writes to a database just has a side effect on the way to producing the response.

### A game entity's update

```
state + input events + dt  →  the update  →  new state + events to emit
```

Every frame, the entity's update function fires. In, through, out.

### A scheduled job

```
clock tick (a trigger input)  →  the job  →  whatever side effects the job causes
```

The input is just "the clock says it is time." The output might be rows written to a database, emails sent, files moved. Still a function.

### A whole game

```
controller input stream + time  →  the game  →  pixel stream + audio stream + savefile writes
```

A game at 60fps is a function being called 60 times a second. The input is the player's intentions; the output is the audiovisual world.

### A whole SaaS product

```
user intentions (clicks, typing, purchases, requests) + time  →  the product  →  pixels + stored data + emails + API responses
```

Zoom out far enough and even an entire SaaS business is a function. Users express intentions; the product turns those into outcomes.

### An agent

```
user request + available tools + world state  →  the agent  →  answer + tool invocations + updated memory
```

A function.

### A robot control loop

```
sensor readings + commands  →  the control loop  →  actuator commands + telemetry
```

A function. The physical world is on both sides of it.

---

## Why the shape is always the same

Because **function-shape is just what "doing something in response to something" looks like formally.**

- Something had to happen for the app to act (an input).
- The app did something (the transformation).
- Something came out as a result (an output).

Any system you can build will fit that shape. You cannot do anything in response to nothing. You cannot do nothing once something has happened (doing nothing is still a response). You cannot produce output from nothing. Every system that acts is a function from its inputs to its effects on the world.

This is not a clever framing. It is the definition of a system that does things.

---

## Why apps don't *look* the same

Because the **edges** differ, dramatically.

The middle of every app is "inputs → transformation → outputs." But from the outside, the visible thing is the edges — what kind of input arrives, how often, where it comes from, what kind of output it produces, where the output goes.

- A web app looks like "a thing that answers HTTP requests."
- A background job looks like "a thing that runs on a schedule."
- A UI looks like "a thing you click on that changes what you see."
- A game looks like "a thing where the player pushes buttons and a world moves."
- An agent looks like "a thing you talk to."

These descriptions are not wrong. But they describe the *edges*, not the thing. From the inside, each of them is a function. The differences between them are differences in **where inputs come from and where outputs go**, not differences in what kind of thing the app fundamentally is.

This is why most programming tools feel specialized. A web framework is good at one set of edges (HTTP in/out). A game engine is good at a different set (input devices in, frame buffer out). A data pipeline tool is good at another (scheduled trigger in, database writes out). Each tool bakes in assumptions about the edges of the app. Then the user's freedom of composition gets constrained by those assumptions.

Blueprint takes the opposite approach: **make the edge a separate concern**, handled by a separate, pluggable piece. The same composed function can be wrapped in different edge configurations and become different shapes of app. See [08-adapters.md](08-adapters.md) for how this actually works.

---

## What this means for Blueprint

Four consequences fall out of the premise directly. Each is developed more in later documents.

### 1. Composing an app is composing a function

If an app is a function, and functions are composed of smaller functions, then building an app is the activity of composing smaller functions into a larger one. That is what the platform must enable.

This is not a metaphor or an aspiration. It is a literal description of what is happening whenever an app is being built.

### 2. The medium can be visual

Function composition has a natural visual representation: a **graph**. Nodes are functions; edges are "this function's output becomes that function's input." A graph *is* a composition. That is why visual programming keeps resurfacing in domain after domain (shaders, audio, scenes, dataflow, automation) — because it is a picture of the thing, not a diagram of the thing.

See [02-composition-by-graph.md](02-composition-by-graph.md).

### 3. Terminology must not privilege one kind of app

If an app is a function regardless of its edges, then the core platform vocabulary must not privilege any one edge type. "App" is the right top-level term. "Dashboard," "workflow," "job," "page," "endpoint," "entity," "agent" are all specific edge configurations — they belong in the shape of particular adapters, not in the core.

### 4. Success is measured on the whole space of apps

If Blueprint can let a person compose only one shape of app (only dashboards, only workflows, only pipelines), it has failed the premise. The measure of success is "can a person compose any app they can imagine, across the full space of possible edges, by composing smaller pieces they can see?"

See [11-the-test.md](11-the-test.md).

---

## The uncomfortable corollary

If an app is a function, then **building an app is ultimately just "write the function."**

That is what professional programmers do every day, disguised as a hundred different activities (writing components, designing handlers, structuring workflows, modeling entities, wiring pipelines, assembling agents). All of that is the activity of writing the function.

Blueprint's job is to make that activity **visible and composable without syntax**, so that people who cannot write code (or can but don't want to, for this particular app) can do it too.

It is the same activity a programmer does. It is just being expressed through a different medium.

This is the line that the rest of the platform is measured against. Everything we build must serve that line.

---

## Why this framing matters now

The premise is not new. What makes it urgent is the moment:

- AI is now capable of generating enormous quantities of software. The question shaping the next decade of app development is not *whether* AI helps produce apps, but **what the artifact is that a human ends up accountable for**.
- If that artifact is a wall of generated code nobody fully reads, the accountability is nominal. If that artifact is a composed function a human can see, reason about, and change, the accountability is real.

A graph of composed functions is an artifact a person can hold responsibility for. They can inspect it, approve it, refuse it, hand it back. An AI can propose it, refine it, explain it — and a human can stay the one who decides what the app is.

That is the deeper reason the premise matters. "An app is a function" is both the cleanest way to think about software *and* the shape of artifact that keeps a human meaningfully in the loop as AI participates more directly in app creation. The pillars documented in the rest of this folder are how a platform can make that kind of artifact the normal thing people produce.

---

## Summary

- An app is a function.
- Inputs → transformation → outputs. Always.
- The middle of every app is the same; the edges differ.
- Composing an app is composing a function.
- The medium for composing visually is the graph.
- The platform must serve the composition of functions, not the construction of any particular app shape.

From here, we can start talking about the medium (graphs), the rules (grammars, invariants), and the pillars that make the composition actually run (nodes, compounds, engines, adapters, stores).
