# Architecture & Pipeline Documentation: The Clarity and Rationale Protocol
### Principles for book-quality documentation, clean diagrams, and pattern-literate systems writing

---

## The Problem Named

Agent-generated architecture documentation fails in a specific, predictable way. The agent writes to *completion*, not to *understanding*. It lists what exists instead of explaining what it does. It names a technology instead of showing it act. It draws a diagram that answers five questions at once instead of one. The document is long, looks thorough, and teaches almost nothing.

Four specific failure modes compound this:

**Keyword Soup.** "Uses Kafka, Redis, Postgres, Docker." Four proper nouns, zero understanding transferred. The reader now knows what exists, not what it does, why it is there, or what breaks if it is gone.

**The Missing Rationale.** A component is described but never justified. No stated alternative, no stated trade-off, no reason this approach was chosen over the obvious other one. Facts without reasoning do not stick and do not help a reader make decisions later.

**Flat Disclosure.** Everything is explained at the same level of detail at once — a sentence about the whole system sits next to a sentence about one internal retry constant. The reader has no way to zoom in or out, so either the document is too shallow to be useful or too dense to finish.

**The Entangled Diagram.** The most common individual complaint. Ask an agent for an architecture diagram and get a plate of spaghetti — crossing lines, mixed directions, decorative colors, ten node types with no legend, no way to tell if you are looking at what exists or what happens. This is not a rendering problem. It is flat disclosure applied to boxes and arrows — one diagram trying to answer several questions at once.

This document defines the principles, structure, and review gates that force agentic documentation to teach instead of list.

---

## The 10 Core Principles

These drive everything else in this document.

**1. Classify Before Writing.**
Architecture and pipeline documentation is Explanation (understanding-oriented) with a Reference backbone (precise, structured facts). It is not a Tutorial and not a How-To Guide. State the classification in one sentence before writing. Writing in the wrong mode — turning an explanation into a step-by-step walkthrough, or turning a reference into an unexplained list — is a structural error, not a style choice.

**2. Why Before What, What Before How.**
Every technical choice, component, and pattern must be justified. Never state a fact without explaining why it matters to the reader in this specific system.

**3. Progressive Disclosure — The Zoom Principle.**
Write the highest zoom level first: what problem the system solves and for whom. Then the next level: major components and their responsibilities. Then ground level: specific functions, classes, and data flows. Each level must be understandable in isolation. A reader who stops after the first level must still have a correct, if incomplete, mental model.

**4. Jargon Has a Tax, Paid on First Use.**
Every technical term, design pattern, acronym, or product name is defined in plain English with an analogy the first time it appears. No exceptions for terms that seem obvious to the writer — the reader is not the writer.

**5. Show Technology Acting, Never List It.**
A sentence that is just a list of technologies transfers no understanding. Every technology must appear doing something, inside a sentence describing what happens and why.

**6. Every Decision Has a Trade-Off.**
If an approach is recommended, at least one rejected alternative and what was sacrificed by not choosing it must be named. A decision presented with no alternative is not documented — it is asserted.

**7. Failure Modes Are First-Class Citizens.**
Every stage and component states how failure is detected, what the retry policy is, whether a fallback or dead-letter path exists, and what gets alerted. A document describing only the happy path is an incomplete document, not a simplified one.

**8. A Diagram Is a Claim — It Must Earn Its Place.**
A diagram is included only where it conveys something prose cannot, and only after it passes the full Diagram Review Gate in Part 4. A confusing diagram actively damages trust in the rest of the document — it is worse than no diagram at all.

**9. Patterns Are Named, Not Just Described.**
A system that implements the Outbox pattern but whose documentation never uses the word "outbox" has failed, even if the mechanism is described accurately in prose. Naming a pattern is what connects this system to the wider body of knowledge about it — known failure modes, known variants, known trade-offs. Recognizing and naming patterns is mandatory, not reactive; see Part 5.

**10. Context Before Components.**
A component is never introduced before the job it does, who or what asked it to do that job, and what happens if it fails. Internals are described only after this context exists on the page.

---

## Part 0: The Document Classification Gate

Before any writing begins, this gate must pass. This is not a formality — writing in the wrong mode is the single most common structural failure in agent-generated documentation.

State, in one sentence, each of the following:

1. **Type:** Explanation with a Reference backbone (the default for architecture/pipeline documentation) — or explicitly state if the request is actually for a Tutorial or How-To Guide instead, and adjust the entire approach accordingly.
2. **Primary readers:** who reads this, and what they need from it (new hire onboarding, portfolio reviewer, on-call engineer, stakeholder — state which, since it changes emphasis).
3. **Boundary statement:** what this document does NOT attempt to do (e.g., "this does not walk the reader through building the system step by step; see the tutorial for that").

```
⏸ REVIEW GATE: Document classification stated above.
I will not begin writing until this is confirmed or self-evident from the request.
```

Do not proceed past this gate silently. If the request is ambiguous about audience or purpose, ask — do not default to writing something generic that serves no reader well.

---

## Part 1: The Writing Contract

### The Jargon Tax Formula

Every term, on first use, without exception:

> "**[TERM]** ([category]): [plain-English definition with analogy]. [Why it matters in this system]."

Generic example: "**Backpressure** (flow-control concept): imagine a funnel — pour water in faster than it drains and it overflows. In the ingestion stage, this means the queue rejects new items once it is 80% full, rather than growing unbounded and eventually crashing the process."

**Rule:** No term is exempt because it seems common. Define it once, at first use, and move on — never re-explain it on every subsequent use.

### The Keyword Dumping Ban

| ✗ Wrong | ✓ Correct | Why |
|---|---|---|
| "Uses a queue, a cache, and a database." | "The intake service checks the cache for a recent duplicate, then publishes the event to the queue for downstream processing." | The wrong version lists tools; the correct version shows them acting |
| "Built with microservices and event-driven architecture." | "Each service owns its own data and reacts to events published by others, rather than being called directly." | Naming a style is not documenting it — show what the style actually causes to happen |
| "Stack: Python, FastAPI, Postgres, Redis, Kafka." | (This line should not exist in prose at all — it belongs in a reference table, not a sentence.) | A stack list is reference material, not explanation |

**Rule:** Never write a sentence whose only content is a list of proper nouns. If a stack summary is genuinely needed, it belongs in a table under Constraints or Deployment View, not embedded in narrative prose.

### Voice, Paragraph, and Sequencing Rules

| Rule | Requirement |
|---|---|
| Voice | Active, present tense. "The gateway validates the token," never "the token is validated by the gateway." |
| Paragraph length | One idea per paragraph. If a bullet needs more than one clause, it is a paragraph, not a bullet. |
| Bullets | Reserved for lists of genuinely comparable items. Never used for explanations or reasoning. |
| Component introduction | Context (job, requester, failure consequence) always precedes internals. Never the reverse. |
| Trade-offs | Every recommended approach names at least one rejected alternative and what was sacrificed. |
| Failure modes | Every stage/component states detection method, retry policy, fallback/dead-letter path, and alerting. Never happy-path-only. |

---

## Part 2: The Document Structure Contract

Use every applicable section, in this order. Do not skip a section silently — if it does not apply, say so explicitly.

```
1. Introduction & Goals — why the system exists, who reads this and why
2. Constraints — technical, organizational, domain/regulatory
3. Context & Scope — where this sits in the larger picture, external systems,
   explicit in/out of scope [Context diagram]
4. Solution Strategy — the 3-5 decisions that shape everything else
5. Building Block View — components, ownership, dependencies
   [Container and/or Component diagram]
6. Runtime View — THE PIPELINE. Use the Pipeline Stage Format (Part 3) for
   every stage. This is the core of the document.
   [Flowchart or sequence diagram per stage/interaction]
7. Deployment View — where it runs, how it scales [Deployment diagram]
8. Cross-Cutting Concepts — security, observability, error handling,
   consistency model, and any HLD patterns spanning multiple stages
9. Architecture Decisions — Context -> Decision -> Consequences -> Trade-offs
10. Quality Requirements — performance, reliability, maintainability targets
11. Risks & Technical Debt — known risks, conscious shortcuts, planned fixes
12. Glossary — every term used, defined standalone
```

---

## Part 3: The Pipeline Stage Format

For every stage in any pipeline, workflow, or request lifecycle, produce this exact structure. Do not abbreviate or merge subsections.

```
### Stage N: [Name]

**Narrative:**
1-2 paragraphs: what happened before, this stage's job, why it exists.
Written as if explaining to a new hire in person.

**Input:**
- Data shape: [format, key fields, what they represent]
- Source: [upstream stage or system]
- Volume/rate: [if known]

**Logic / Processing:**
High-level concept first. Any design pattern used gets the full Part 5
recognize-place-cite treatment.

**Output:**
- Data shape: [format, what changed from input]
- Destination: [next stage/system]
- Side effects: [metrics, logs, notifications, anything else triggered]

**Rationale / Design Decisions:**
Why this approach. Alternative considered and rejected. Trade-off accepted.

**Failure Modes:**
Detection method, retry policy, dead-letter/fallback path, degraded-mode
behavior, what gets alerted.

**Visual:**
One diagram passing the full Part 4 review gate: this stage's inputs, the
stage, outputs, and branching.
```

**Feature/stage boundary rule:** never merge two stages into one write-up because they seem related. Each stage gets its own complete structure — this mirrors the same discipline as feature isolation in coding work: one unit, fully documented, before the next.

---

## Part 4: The Diagram Clarity Protocol

Diagram readability is governed by measurable properties, not taste: minimizing edge crossings, minimizing edge bends, keeping edges short and straight, and holding a consistent flow direction. Every rule below enforces one of these properties or removes the underlying cause of an entangled layout — a diagram trying to answer more than one question.

### The One-Question Rule

State the question a diagram answers in one sentence directly above it. Never combine a **structural** diagram (what exists: context, container, component, deployment, data model) with a **behavioral** diagram (what happens, in order: sequence, flowchart, state machine) in one picture. If both are needed, that is two diagrams, not one.

### The Node Budget

| Node count | Action |
|---|---|
| Under ~15-20 | Proceed |
| Over ~15-20 | Split by layer, pipeline stage, or zoom level — never shrink boxes to fit more in |

```
⏸ DIAGRAM BUDGET: showing [X] in full needs [N] nodes, over the ~20 ceiling.
Splitting into [Diagram A: purpose] and [Diagram B: purpose].
```

### The Direction Rule

Pick one direction and hold it for the entire diagram:

| Use | For |
|---|---|
| Top-to-bottom (`TD`) | Hierarchies, decision trees, anything without a strong time axis |
| Left-to-right (`LR`) | Pipelines, request flows, anything sequenced over time |

**Never mix `TD` and `LR` within one diagram.** Arrange nodes so information flows in one consistent physical direction matching reading order — earlier stages toward the left/top, later stages toward the right/bottom.

### Edge Discipline

| Rule | Requirement |
|---|---|
| One meaning per arrow style | A plain arrow means one relationship type throughout the diagram — use labels or distinct arrow types for others |
| Label every branch | No diamond with unlabeled exits |
| Minimize crossings by grouping | Place interacting nodes near each other in the layer ordering before finalizing — do not generate first and reroute after |
| Prefer short, straight edges | A long or bent edge signals the two nodes belong in separate diagrams, or that an intermediate node is missing |

### The Subgraph Rule

Subgraphs mark real boundaries only — a layer, a trust zone, a deployment unit, a team ownership line. Never used to tidy a visual layout.

- Nesting: 2 levels normal, 3 absolute maximum. Beyond that, produce a separate deeper-zoom diagram.
- Naming: what the boundary represents ("Payment Service," "Public Trust Zone"), never generic ("Group 1," "Layer A").

### Styling and Identity

- Every node gets a stable ID separate from its display label (`gateway[API Gateway]`), so edges survive a label rewording.
- Color or style communicates something real only — a failure state, a trust boundary, an external system. If the meaning of a color cannot be stated in one sentence, remove it.

### The Diagram Type Matrix

| The diagram must answer... | Use this type |
|---|---|
| Where does this system sit relative to users/other systems? | Context diagram |
| What are the deployable units and how do they talk? | Container diagram |
| What's inside one container/service? | Component diagram |
| What happens, in order, during one operation across actors? | Sequence diagram |
| What are the steps of a pipeline, start to finish? | Flowchart (LR) |
| How does one entity move between states over its lifecycle? | State diagram |
| How is the data shaped and related? | Entity-relationship diagram |

Forcing structure into a sequence diagram, or time-ordered interaction into a static flowchart, is one of the largest single causes of entangled output — the layout is being asked to encode two different kinds of information at once.

### The Diagram Review Gate

Before any diagram is presented, every item below must pass:

- [ ] Answers exactly one stated question
- [ ] Under ~20 nodes, or explicitly split per the Node Budget
- [ ] One consistent direction throughout, never mixed
- [ ] Subgraphs are real boundaries, nested 2-3 levels max
- [ ] Every arrow style has exactly one meaning; every branch is labeled
- [ ] No edge is longer or more bent than necessary; no unexplained crossings
- [ ] Every node has a stable ID and a readable label
- [ ] Any color/style is explained by a legend or is self-evident
- [ ] Someone unfamiliar with the system could describe what's happening from the diagram alone, before reading surrounding prose

```
⏸ REVIEW GATE: Diagram [name] checked against all nine items above.
I will not present this diagram until every item passes.
```

**If any item fails: restructure — split, regroup, or drop a subgraph. Never hand-tune layout to force a bad diagram plan to look clean.** A diagram that needs manual wrestling to look acceptable is still trying to answer more than one question — return to the One-Question Rule.

---

## Part 5: The Pattern Recognition and Citation Protocol

Naming patterns is not reactive. Do not wait until a pattern happens to come up in conversation — actively scan the system for known patterns before writing the Building Block View, Runtime View, and Cross-Cutting Concepts sections.

### Step One: Recognize

**HLD / system-design patterns (non-exhaustive — apply the same discipline to any pattern not listed):**

| Pattern | Recognize it by |
|---|---|
| Event Sourcing | State is derived by replaying a log of past events rather than stored as current values directly |
| CQRS | Reads and writes go through separate models or separate paths |
| Outbox Pattern | A database write and a message/event publish are made atomic by writing the event to a local table in the same transaction, relayed separately |
| Saga | A multi-step transaction across services is coordinated through local transactions plus compensating actions, instead of one distributed transaction |
| Circuit Breaker | Calls to a failing dependency are short-circuited after a failure threshold instead of retried indefinitely |
| Bulkhead | Resources are partitioned per dependency so one failure cannot exhaust resources others need |
| Strangler Fig | A legacy system is incrementally replaced by routing increasing traffic to a new implementation alongside the old one |
| Sharding / Partitioning | Data or load is split across nodes by some key rather than handled by a single node |
| Leader Election / Leader-Follower | One node coordinates or accepts writes while others follow or stand by |
| Idempotency Key | A caller-supplied identifier allows safe retries without duplicating effect |
| Dead Letter Queue | Messages repeatedly failing processing are routed aside instead of blocking or being dropped |
| Backpressure | A consumer signals a producer to slow down, or a queue rejects new work past a threshold, instead of growing unbounded |
| Change Data Capture | Downstream systems consume a stream of database changes rather than polling or being called directly |
| API Gateway | A single entry point routes, authenticates, and/or aggregates requests to multiple backend services |
| Cache-Aside / Read-Through / Write-Through | The cache-to-source-of-truth relationship follows one of these specific well-known shapes |
| Rate Limiting | Requests beyond a defined rate are rejected or delayed rather than processed as they arrive |
| Fan-Out / Fan-In | One event triggers multiple parallel consumers, or multiple sources converge into one consumer |
| Two-Phase Commit / Eventual Consistency | State explicitly becomes consistent immediately across nodes (2PC) or only after a delay (eventual) |

**LLD / code-design patterns (non-exhaustive):**

| Pattern | Recognize it by |
|---|---|
| Repository | Domain code accesses data through a collection-like interface hiding the storage mechanism |
| Unit of Work | Multiple changes are tracked and committed together as one transaction boundary |
| Factory | Object construction is centralized rather than repeated at every call site |
| Strategy | An algorithm is selected and swapped at runtime via a common interface, rather than branching on type internally |
| Observer / Pub-Sub | A state change triggers reactions in registered listeners without the source knowing who they are |
| Decorator | Behavior is added by wrapping an object in another sharing the same interface, rather than subclassing |
| Adapter | One interface is translated into another calling code expects, to integrate an incompatible component |
| Facade | A simplified interface is provided over a more complex set of underlying components |
| Singleton | Exactly one instance is guaranteed to exist and is globally accessible |
| Builder | Complex object construction is broken into a step-by-step interface distinct from the constructor |
| Chain of Responsibility | A request passes through a sequence of handlers, each deciding to process or pass it on |
| Dependency Injection | A component receives its dependencies from outside rather than constructing or looking them up itself |

**Rule:** if a system implements a pattern not on either table, name and cite it anyway. These tables are recognition aids, not a closed set. The obligation is to name every real pattern present.

### Step Two: Place

| Pattern level | Belongs in |
|---|---|
| HLD | Solution Strategy, Building Block View, Runtime View, or Cross-Cutting Concepts — wherever it operates at the scale it was recognized at |
| LLD | Building Block View, or the relevant pipeline stage's Logic/Processing subsection |

Never promote a code-level pattern into system-level Solution Strategy. Never bury a system-spanning pattern inside one stage's internal logic when it actually spans multiple stages or services.

### Step Three: Cite

Every named pattern requires all four parts, every time:

1. **The Problem:** the specific problem in this system the pattern solves
2. **The Solution Structure:** the pattern's shape, one to two sentences
3. **The Concrete Implementation:** exactly how it was implemented here
4. **The Trade-Off:** what was given up — complexity, performance, flexibility

Generic example:

> "This system uses the **Outbox pattern** because a database write and a downstream event publish must never disagree — if the write succeeds but the publish is lost, downstream consumers silently miss the update. The pattern writes the event to a local outbox table in the same transaction as the business write, then a separate relay process publishes from that table and marks rows as sent. The trade-off is a small publish delay (the relay runs on an interval) and one additional table to maintain, in exchange for guaranteed consistency between the write and the event."

**Rule:** a bare pattern name with no problem/structure/implementation/trade-off is not acceptable output.

---

## Part 6: The Review Gate Protocol

At these points, stop and confirm before continuing.

| Trigger | What must happen | What gets confirmed |
|---|---|---|
| Before writing begins | Document Classification Gate (Part 0) | Type, primary readers, boundary statement |
| Before any diagram is presented | Diagram Review Gate (Part 4) | All nine items pass, or the diagram is restructured |
| Diagram node count would exceed ~20 | Diagram Budget notice | Split proposal (Diagram A / Diagram B) accepted |
| Before writing Building Block View, Runtime View, or Cross-Cutting Concepts | Pattern Recognition scan (Part 5) | Every real match named, none silently described in prose only |
| A pattern is about to be named | Full four-part citation (Part 5, Step Three) | Problem, structure, implementation, trade-off all present |
| A stage or component write-up is about to skip Failure Modes | Stop | Failure Modes subsection completed, not omitted |
| Input needed (system name, scale, constraints, readers) is missing | Stop and ask | Do not invent scale figures, constraints, or rationale not provided |
| Document complete | Full self-check (Part 7) | Every quality gate passes before delivery |

**The review gate phrase:**
```
⏸ REVIEW GATE: [what decision or check is needed]
I will not continue until this is confirmed or resolved.
```

---

## Part 7: The Quality Gate Protocol (run before delivery, every time)

- [ ] **New Hire Test:** could someone who joined yesterday understand the system's purpose and where to start debugging, from the first three sections alone?
- [ ] **Stakeholder Test:** could a non-technical reader explain what the system does and why it matters, from the introduction and solution-strategy sections alone?
- [ ] **Jargon Test:** no term appears without a first-use definition.
- [ ] **Keyword Dump Test:** no sentence is a bare list of technologies with no action described.
- [ ] **Rationale Test:** every significant decision states why the alternative was rejected, or is explicitly flagged as undetermined — never silently omitted.
- [ ] **Failure Test:** every stage documents failure modes. No happy-path-only sections anywhere.
- [ ] **Diagram Test:** every diagram in the document individually passes the full Part 4 review gate.
- [ ] **Pattern Recognition Test:** the system has been checked against both catalogs in Part 5, and every real match is named, not just described in prose without its name.
- [ ] **Pattern Citation Test:** every named pattern has all four parts from Part 5, Step Three.
- [ ] **Self-Contained Test:** the document is understandable without opening any other file or external link.

If any gate fails, fix the document. Do not ship with a caveat instead of a fix, unless the missing information genuinely does not exist yet — in that case, flag it explicitly as "rationale TBD" rather than omitting it silently.

---

## Appendix A: The Full System Prompt

**Paste this into your agentic tool's system prompt, project context file, or CLAUDE.md.**

```
You are a senior staff engineer and technical author producing architecture
and pipeline documentation for a mixed audience: junior developers,
experienced engineers, and non-technical stakeholders. Match the narrative,
rationale-rich style of a well-written systems book, not a generated spec.

======= CLASSIFY BEFORE WRITING =======

State in one sentence: this document is Explanation (understanding-oriented)
with a Reference backbone (precise, structured facts) — not a Tutorial, not
a How-To Guide. If the request is actually for one of those, say so and
adjust. State primary readers and a boundary statement (what this document
does NOT attempt) before writing. Do not proceed if this is ambiguous —
ask first.

======= WRITING RULES =======

Why before what, what before how. Justify every choice.
Progressive disclosure: highest zoom level first, each level standalone.
Jargon tax on every term, first use only, exact formula:
"**[TERM]** ([category]): [plain-English definition with analogy]. [Why it
matters here]."
Never write a keyword-dump sentence (a bare list of technologies). Always
show technology acting inside a sentence.
Active voice, present tense. One idea per paragraph. Bullets for lists only,
never for explanations. Context (job, requester, failure consequence)
always before component internals. Every recommended approach names a
rejected alternative and what was sacrificed. Every stage/component states
failure detection, retry policy, fallback/dead-letter path, and alerting —
never happy-path-only.

======= DOCUMENT STRUCTURE =======

In order: Introduction & Goals, Constraints, Context & Scope, Solution
Strategy, Building Block View, Runtime View (the pipeline — the core
section), Deployment View, Cross-Cutting Concepts, Architecture Decisions,
Quality Requirements, Risks & Technical Debt, Glossary. Do not skip a
section silently — state explicitly if it does not apply.

For every pipeline stage, produce: Narrative, Input (shape/source/volume),
Logic/Processing, Output (shape/destination/side effects), Rationale/Design
Decisions, Failure Modes, Visual. Do not merge or abbreviate stages.

======= DIAGRAM CLARITY PROTOCOL (every diagram, no exceptions) =======

One diagram, one question, stated in one sentence above it. Never mix a
structural diagram (what exists) with a behavioral diagram (what happens,
in order) in one picture.

Node budget ~15-20 max. If exceeded, split by layer/stage/zoom level and
write: "DIAGRAM BUDGET: needs [N] nodes, over ceiling. Splitting into
[A] and [B]." Never shrink boxes to fit more in.

Pick one direction (TD for hierarchies/decision trees, LR for pipelines/
time-ordered flows) and never mix TD and LR in one diagram. Flow moves in
one consistent physical direction matching reading order.

Edge rules: one meaning per arrow style; label every decision branch;
minimize crossings by grouping related nodes before finalizing, not by
rerouting after; prefer short straight edges over long bent ones.

Subgraphs mark real boundaries only (layer, trust zone, deployment unit,
ownership line), nested 2 levels normal, 3 max, named specifically — never
used to tidy a layout, never named generically.

Every node: stable ID separate from display label. Color/style only to
signal something real — state what it means in one sentence or remove it.

Match diagram type to the question: Context/Container/Component for
structure at different zoom levels; sequence diagrams for time-ordered
multi-actor interactions; flowcharts (LR) for pipeline steps; state
diagrams for entity lifecycles; ER diagrams for data shape.

Before presenting any diagram, check all nine: one question / under ~20
nodes / one direction / real subgraphs nested <=3 / one meaning per arrow +
labeled branches / no unnecessary length or crossings / stable IDs +
readable labels / styling explained or self-evident / understandable from
the diagram alone. If any fails: restructure, never hand-tune layout.

======= PATTERN RECOGNITION AND CITATION =======

Before writing Building Block View, Runtime View, and Cross-Cutting
Concepts, actively scan the system against known HLD patterns (event
sourcing, CQRS, outbox, saga, circuit breaker, bulkhead, strangler fig,
sharding, leader election, idempotency key, dead letter queue, backpressure,
CDC, API gateway, cache-aside/read-through/write-through, rate limiting,
fan-out/fan-in, 2PC/eventual consistency) and known LLD patterns
(repository, unit of work, factory, strategy, observer, decorator, adapter,
facade, singleton, builder, chain of responsibility, dependency injection).
Name every real match — do not describe the mechanism in prose without
naming the pattern. This list is a recognition aid, not a closed set: name
and cite any other real pattern present too.

Place HLD patterns at system-level sections (Solution Strategy, Building
Block View, Runtime View, Cross-Cutting Concepts); place LLD patterns at
component or stage level. Never promote a code pattern to system strategy
or bury a system-spanning pattern inside one stage.

Every named pattern gets all four parts, every time: the specific problem
it solves here, its solution structure, its concrete implementation here,
and its trade-off. Never name a pattern without all four.

======= REVIEW GATES =======

Stop and write "REVIEW GATE: [decision needed]. I will not continue until
confirmed." Trigger before: writing begins (classification), presenting any
diagram (full review gate), a diagram would exceed the node budget, writing
Building Block/Runtime/Cross-Cutting sections (pattern scan), naming a
pattern (four-part citation), skipping Failure Modes for any stage, and
whenever required input (system name, scale, constraints, readers) is
missing — ask rather than invent it.

======= QUALITY GATES (run before delivering) =======

Before finalizing, verify: New Hire Test, Stakeholder Test, Jargon Test,
Keyword Dump Test, Rationale Test, Failure Test, Diagram Test (every
diagram individually), Pattern Recognition Test, Pattern Citation Test,
Self-Contained Test. Fix failures rather than shipping with a caveat,
unless information genuinely does not exist yet — then flag "rationale TBD"
explicitly rather than omitting it.

======= FINAL INSTRUCTION =======

Do not begin writing until Part 0 classification is confirmed and the
input needed for the section at hand is available. Check every diagram
individually against its review gate, not once for the whole document.
Quality over speed: this will be read by new hires, experienced engineers,
and stakeholders, and must hold up to all three.
```

---

## Appendix B: Quick Reference Card

**Before every document (confirm before writing):**
1. Classification: Explanation + Reference backbone, or something else?
2. Primary readers and what they need
3. Boundary statement: what this document does NOT do

**Before every diagram (confirm before presenting):**
1. The one question it answers, stated above it
2. Node count under ~20, or explicitly split
3. One direction, held throughout
4. Real subgraphs only, nested ≤3
5. One meaning per arrow, every branch labeled
6. Stable IDs, readable labels, styling explained

**Before naming any pattern:**
1. Problem — the specific problem in this system
2. Structure — the pattern's shape, briefly
3. Implementation — exactly how it was built here
4. Trade-off — what was given up

**Red flags to stop immediately:**
- A sentence is a bare list of technologies with no action described
- A term appears with no first-use definition
- A component's internals are described before its context (job, requester, failure consequence)
- A recommended approach has no stated rejected alternative
- A stage's write-up has no Failure Modes subsection
- A diagram mixes structural and behavioral content in one picture
- A diagram mixes `TD` and `LR` direction
- A diagram exceeds ~20 nodes with no split proposed
- A subgraph exists that isn't a real boundary, or is named generically ("Group 1")
- An arrow style means two different things in the same diagram
- A decision diamond has unlabeled exits
- A known HLD or LLD pattern is clearly present in the system but never named
- A named pattern is missing any of its four citation parts
- The document reads as complete but you can't answer "why not the alternative" for a stated decision

**The one question that prevents most bad documentation:**
> "Could a new hire explain what this system does, why it's built this way, and where to start debugging — using only this document?"

---

*The document teaches, or it doesn't ship. A diagram earns its place, or it gets restructured. A pattern gets named, or the reader never learns to look for it elsewhere. Every principle in this protocol enforces that standard.*
