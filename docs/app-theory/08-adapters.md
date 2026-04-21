# 08 — Adapters

> **TL;DR** — An **adapter** is what turns the top-level function (the root compound) into a *living app in a specific environment*. The root function on its own is inert; something has to call it, feed it inputs, receive its outputs, and translate between the composition and the world. Adapters answer the question: **how does this function come alive?** A task adapter makes it come alive on a schedule. A server adapter makes it come alive on HTTP requests. A UI adapter makes it come alive on user events and frames. A device adapter makes it come alive on sensor readings. Adapters are **edges**, not integrations — they live on the outermost ports of the app and translate between the graph's opaque values and the specific environment's real-world signals. Adapters are plural, each specialized for one edge kind; a real app typically uses several.

---

## The role of an adapter

Restating invariant 3:

> The app itself is a function; adapters are its edges.

A function on paper is inert. It doesn't do anything on its own. Something has to invoke it — pass it inputs, receive its outputs, decide when and why.

For a Blueprint app (= a composed function), that something is an adapter. Each adapter is an answer to:

> *"How does this function come alive, and for what kind of aliveness?"*

Different adapters embody different forms of aliveness:

- **Scheduled aliveness.** "Wake this function up on a schedule, deliver the clock tick as input, and carry its outputs to where they need to go." That is a task adapter.
- **Request/response aliveness.** "Wake this function up when an HTTP request arrives, deliver the request as input, and return its output as the HTTP response." That is a server adapter.
- **Interactive aliveness.** "Wake this function up on every frame and on every user event, deliver events as inputs, and render its outputs to the screen." That is a UI adapter.
- **Device aliveness.** "Wake this function up when a sensor changes, deliver the reading as input, and drive its outputs to actuators." That is a device adapter.
- **Mount/lifecycle aliveness.** "Wake this function up once when it is embedded (a component in a host app), then re-wake on lifecycle events." That is a component adapter.
- **Message aliveness.** "Wake this function up when a message arrives on a queue or topic, deliver the message as input, and publish its outputs to other topics." That is a messaging adapter.

Each of these is one kind of edge configuration. The same root function can be wrapped in any of them. The function doesn't know which; the adapter handles all the specifics.

---

## Adapters live on edges

An **edge** of the root function is a point where it meets the world. Common edges:

- **Time** — clock ticks, deadlines, schedules, frame rates, timeouts.
- **User actions** — mouse, keyboard, touch, gestures, voice.
- **Network I/O** — HTTP, WebSocket, MQTT, gRPC, TCP, UDP.
- **Persistent storage** — database reads/writes, file I/O, log append.
- **Sensors / actuators** — physical devices, GPIO, industrial protocols.
- **External services** — third-party APIs.
- **Lifecycle** — mount, unmount, start, stop, suspend, resume.
- **Display** — screen pixels, audio output, haptics.

Each of these is some adapter's responsibility. The root graph declares ports that describe its expected edges (it expects an input of this kind, produces an output of that kind); the adapter connects those ports to the real thing.

**Adapters are specialized per edge kind.** A single adapter handles one kind of edge well. A real app typically has several adapters working together.

---

## The adapter's job, concretely

An adapter typically does five things:

### 1. Drive the engine

Decide when the engine should evaluate. Call the engine with the appropriate inputs and `now` value. This is the "invoke the function" part.

### 2. Translate inputs from the world to the graph

Convert whatever arrived from the outside (an HTTP request, a cron tick, a mouse event, a sensor reading) into a shape the graph's input ports accept. This is "pack the arguments."

### 3. Translate outputs from the graph to the world

Take what the graph produced and deliver it to where it needs to go (an HTTP response, a database write, a screen draw, an actuator signal). This is "unpack the return."

### 4. Own lifecycle

Decide when the app starts, when it pauses, when it stops. Restore state from stores if needed. Snapshot state when appropriate. Clean up on shutdown.

### 5. Own policy the engine shouldn't

Retry on failure. Back off. Cache. Deduplicate. Batch. These are all adapter concerns — they are *how the function is called into the world*, which is separate from what the function does.

Some of these concerns belong in the adapter, some could belong in a processor (e.g., an HTTP-request processor might have its own retry logic), some could be grammar-specific (an event grammar might have backpressure rules). The dividing line is: **is this about composition, or about meeting the world?** If composition, it belongs in the graph or engine. If world-meeting, it belongs in an adapter.

---

## A real app has several adapters

This is where most mental models go wrong. It is tempting to think of "the adapter" (singular). Real apps aren't like that.

### Example: a SaaS backend

- **Server adapter** — wakes on HTTP requests.
- **Task adapter** — wakes on cron for scheduled jobs.
- **Store adapter** — bridges to a database for persistence.
- **Queue adapter** — wakes on messages from a job queue.
- **Logging adapter** — sinks telemetry to an observability stack.

Five adapters, one composed function in the middle, each adapter handling one edge kind.

### Example: a desktop UI tool

- **UI adapter** — wakes on user events, renders the composed output.
- **File-watch adapter** — wakes when files change.
- **Settings-store adapter** — persists preferences across restarts.
- **Auto-update adapter** — checks for updates on a schedule.

### Example: a robot

- **Sensor adapter** — wakes on sensor readings.
- **Actuator adapter** — drives motors, lights, servos from graph outputs.
- **Task adapter** — wakes on time (safety checks, heartbeats).
- **Telemetry adapter** — streams data back to a base station.
- **UI adapter** — local operator screen (optional).

### Example: a game

- **Frame-loop adapter** — wakes every frame.
- **Input adapter** — wakes on controller / keyboard / mouse.
- **Audio adapter** — consumes audio outputs.
- **Render adapter** — consumes draw commands.
- **Save adapter** — persists savegames.

Each adapter is a specialist in one edge kind. They cooperate by each targeting different ports on the root function, at different cadences.

---

## Multiple adapters → same graph

The same composed function can be wrapped in different adapter configurations to produce completely different shapes of app.

Consider a single graph that takes a "user" and a "request" as inputs and produces a "rendered page" as output:

- Wrap it in a **server adapter**: it's a web server.
- Wrap it in a **component adapter**: it's embedded in a host app.
- Wrap it in a **task adapter**: it's a nightly report generator.
- Wrap it in a **CLI adapter**: it's a one-shot command-line renderer.

Same composition. Four shapes of app. The only difference is which adapter is driving the function and where its inputs come from and outputs go.

This is the portability story. It is only possible because:

1. The graph does not know what kind of app it is. (Invariant 3.)
2. The engine does not know where inputs come from or go. (Engine stays in its lane.)
3. The adapter does everything environment-specific. (This document.)

Break any of those three and portability collapses. Keep them and Blueprint becomes a composition environment rather than a specific-app-shape builder.

---

## What adapters own

An adapter owns:

1. **Lifecycle.** When the app starts and stops, boot sequence, shutdown, restart, crash recovery.
2. **Scheduling / triggering.** When the engine should run.
3. **Input translation.** World → graph.
4. **Output translation.** Graph → world.
5. **Environment-specific policy.** Retry, backoff, caching, deduping, batching, rate limiting, throttling.
6. **Environment-specific permissions / auth.** For edges that need it (HTTP auth, file permissions, device access).
7. **Snapshot / restore coordination** (with stores). When to persist engine state, when to rehydrate.
8. **Resource management.** Connection pools, sockets, file handles, threads.

That's a lot. Adapters are substantial pieces of software. The fact that the engine and graph are clean and small means adapters carry real weight. That is the right division — the part that changes per environment is in the adapter; the part that composes the app is in the graph.

---

## What adapters must *never* own

Adapters must never:

1. **Decide what the app does.** Adapter code does not compute business logic; it shuttles data between the world and the graph.
2. **Touch graph internals.** An adapter connects to the root function's declared ports only. Reaching into internal nodes or compounds is a violation.
3. **Embed grammar assumptions.** The adapter doesn't know whether the graph is a DAG, an FSM, or anything else. It drives the engine through the engine's contract.
4. **Own processors.** Processors are the graph's building blocks. Adapters wake the graph; they don't substitute for its composition.
5. **Be shared across unrelated edge kinds.** A "web adapter" that also does cron and rendering is doing three jobs and should be three adapters.

### The test

If an adapter is asked to do something and the answer is "add business logic to the adapter to make this work," the work is in the wrong place. Find where in the graph that logic should live and put it there. The adapter's job is always and only: **wake the graph, hand it inputs, take its outputs, meet the world on the edges.**

---

## Adapters and the engine contract

The adapter talks to the engine through a defined contract. That contract covers:

- How to initialize an engine with a graph and initial state.
- How to tick the engine with inputs and a `now` value.
- How to receive outputs and state from the engine.
- How to snapshot and restore engine state.
- How to propagate signals (abort, pause, resume).

The contract is **grammar-agnostic**: the adapter doesn't know whether it's driving a DAG engine or an FSM engine. It calls `tick(inputs, now)` or the equivalent, gets results back, and moves on. The multi-grammar dispatch happens inside the runtime.

See `specs/engine-contract.md` and `specs/adapter-contract.md` for the concrete contracts.

---

## Adapters, connections, and processors

There is a subtle distinction worth making clear:

- **An adapter** lives on an edge of the root function and drives the engine.
- **A connection** is a package that knows how to talk to a specific external service (e.g., the Postgres connection, the Stripe connection). Connections are typically used by processors, not adapters.
- **A processor** is a node's function body. Some processors (e.g., "HTTP request") use connections to do their work; some are pure.

So when a user's graph contains a node that calls an external API, that's typically a **processor using a connection** — not an adapter. The processor is part of the graph's composition. The connection is the reusable "how to talk to service X" package.

The adapter is different. It is what *wakes the whole graph* on some trigger. It doesn't live in the graph; it lives around it.

A useful rule of thumb:

- If it is triggered by something outside the graph and drives the engine, it is an adapter.
- If it is invoked by a node inside the graph to interact with the outside, it is a processor (possibly using a connection).

There are edge cases — some concerns (e.g., WebSocket) can reasonably be implemented as either an adapter or a processor-with-connection, depending on whether the graph should react to every inbound message (adapter) or pull messages when it chooses (processor). That is a design choice, not a platform ambiguity.

---

## Adapter boundaries and observability

Because adapters are the boundary between composition and world, they are natural places for:

- **Metrics.** "How many HTTP requests per second reached the graph?" "How long did the average tick take?"
- **Tracing.** "What outer event triggered this evaluation?"
- **Audit logs.** "What entered the graph from the world, what left to the world, when, and why?"
- **Rate limits.** "Shed load at the edge rather than inside the graph."
- **Health checks.** "Is the adapter correctly receiving from the world?"

Good adapters instrument themselves at these seams. This is separate from any in-graph observability the engine offers (see [07-engines.md](07-engines.md)). The two complement each other: adapter tells you about edges; engine tells you about composition.

---

## Adapter lifecycle patterns

Different adapters have different lifecycles, but some common shapes:

- **Long-running loop** — task adapter, frame-loop adapter, device adapter. Runs until told to stop.
- **Request-scoped** — server adapter. One short engine invocation per inbound request.
- **Event-driven** — UI adapter, queue adapter. Engine runs in response to external events.
- **One-shot** — CLI adapter, build-time adapter. Engine runs once and exits.
- **Mounted** — component adapter. Engine runs while the component is on screen, paused/torn down when not.

Each shape has its own start/stop rituals, error handling, and snapshot/restore semantics. The adapter contract covers the common parts; specific adapters implement the pattern that fits their environment.

---

## Adapters are not "platform integrations"

A common misread: "an adapter is how Blueprint integrates with React / Postgres / Kafka / the DOM / etc." That framing leads astray.

The integration framing treats Blueprint as a thing separate from those environments, that reaches out to connect with them. But **adapters are not a bridge to external platforms; they are the edges of the app itself**. The app *is* the function composed by the user; the adapter is its edge to the specific environment it's going to live in. The platform Blueprint is integrating with is wherever this particular app wants to live.

Rephrased: **the adapter is part of the app's architecture, not part of Blueprint's integration story.** A task adapter is not "Blueprint's cron integration"; it is "how this specific app wakes up on a schedule." That is a small difference in framing with a big difference in how you build things.

---

## When to add a new adapter

Add a new adapter when a new **edge kind** needs to be supported. Not when a new platform integration is desired, not when a new service needs a client — those are usually processors or connections.

Examples that warrant a new adapter:
- A new kind of event source (a new IoT protocol, a new frame-loop system, a new device class).
- A new kind of embedding environment (a new framework host, a new OS).
- A new kind of lifecycle or scheduling model.

Examples that don't warrant a new adapter:
- Another database. Use a store / connection.
- Another API. Use a processor with a connection.
- Another JSON format. Use a processor.

If you aren't sure, ask: "does this change how the graph comes alive?" If yes, adapter. If no, probably a processor or connection.

---

## Summary

- An **adapter** is what turns the top-level function into a running app in a specific environment.
- Adapters live on **edges** — points where the app meets the world.
- A real app has **several adapters** (one per edge kind) cooperating around a single composed function.
- Adapters own: lifecycle, triggering, input/output translation, environment-specific policy.
- Adapters must never own: business logic, graph internals, grammar assumptions, unrelated edge kinds.
- The same graph can run in **different adapter configurations** → different shapes of app.
- Adapters are **not "platform integrations"**; they are the app's edges to its specific environment.

Next: **stores** — durable memory that outlives a single evaluation.
