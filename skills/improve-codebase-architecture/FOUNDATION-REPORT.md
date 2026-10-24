# Foundation Report

For a project that does not exist yet, or exists only as a scaffold, and whose stack is already given. The review report asks "what is wrong with this code"; there is no code to be wrong yet, so this report answers different questions:

- Which deep modules will the system have, and where are their seams?
- How will the hard parts be built, concretely?
- What else could have been built instead, and why not?
- What to build first so the design meets reality early.

The review's improvement candidates and its **Fine as is** section do not appear here. Both describe existing code, and there is none.

Delivery (Artifact page or local file), the diagram primitives, style and tone are those of [HTML-REPORT.md](HTML-REPORT.md). The vocabulary is [LANGUAGE.md](LANGUAGE.md), unchanged: a new design is where *deep*, *seam* and *adapter* do the most good.

## Process

### 1. Understand the problem

Read whatever exists: the user's brief, a product doc, `CONTEXT.md`, ADRs, a scaffold's config. Then pin down, in the user's words where possible:

- **What the system must do**: three to six capabilities, each one sentence.
- **Constraints**: platforms, deployment, data volume, latency, offline use, security and privacy, licences, team skills, budget. Constraints decide most of the design, so ask about any that would change it and that you cannot infer.
- **The domain language**: the nouns the user uses. Start `CONTEXT.md` with them (see [CONTEXT-FORMAT.md](CONTEXT-FORMAT.md)); module names come from it.

### 2. Find the hard parts

A hard part is where a naive implementation would fail: concurrency and ordering, consistency across stores, an external system you do not control, a protocol or file format, performance under load, security, recovery after a crash, migration of stored data, anything with real money or real safety behind it. Most projects have two to five. If you find none, the design is probably simple enough not to need this report; say so.

For each hard part, check how others have solved it before inventing: the stack's own documentation, well-known libraries, reference implementations. Verify claims about versions, limits and behaviour against a source (fetch the doc page) and cite it in the report; do not rely on memory for anything the design depends on.

### 3. Draw the module map

Place the deep modules: each one owns one of the capabilities or hard parts behind a small interface. For each seam, name its dependency category from [DEEPENING.md](DEEPENING.md) (in-process, local-substitutable, ports & adapters, mock), because that decides how it is tested. Apply the seam discipline from the start: no port with only one adapter, unless the second (usually the test adapter) is already real.

### 4. Choose an implementation for each hard part, and record what you rejected

For every hard part and every structural decision, keep at least one serious alternative and the reason it lost. A reason names the constraint that kills the alternative ("needs a long-lived server; the app must run offline"), not a preference ("cleaner"). Note what would make you revisit the decision.

### 5. Write the report

Sections below. Then ask the user which hard part or decision they want to grill first (through a question tool when one exists). Decisions the grilling settles become ADRs ([ADR-FORMAT.md](ADR-FORMAT.md)); new terms go into `CONTEXT.md`. A greenfield project is the cheapest moment to write both.

## Report sections

### Header

Project name, date, the legend from [HTML-REPORT.md](HTML-REPORT.md), and a **brief** block: what the system must do (the capability list) and the constraints. This is the one place a short paragraph is allowed. When the stack came from a Stack report, link it.

### Module map

The centrepiece of the top half. One flow graph (see "Diagrams show flow" in [HTML-REPORT.md](HTML-REPORT.md)) of the deep modules, the calls and data between them, their seams (dashed), and the adapters at each seam (for example "HTTP in prod, in-memory in tests"). Under it, one row per module:

- **Module** — name from `CONTEXT.md`.
- **Interface** — one line: what a caller can ask of it.
- **Hides** — what the implementation absorbs so callers do not have to know.
- **Dependency category** — badge.

### Hard parts

One card per hard part, in the order they would hurt most if wrong. Each card:

- **Title** — the hard part, named by what makes it hard ("Edits from two devices must merge without a server").
- **Confidence badge** — `Settled` (emerald), `Needs a spike` (amber), `Open` (slate).
- **Why it is hard** — one sentence.
- **How it will work** — the diagram carries this: a sequence diagram for a protocol, a state machine for a lifecycle, a data-flow for a pipeline, a timeline for a crash and its recovery. Label the module and seam each step belongs to.
- **Interface it hides behind** — which module owns it, and the one-line interface.
- **Failure modes** — bullets: what goes wrong and what the design does about it.
- **How it is tested** — which adapter stands in at which seam, and what one test asserts.
- **Sources** — links for the facts the design depends on.

### Alternatives considered

One card per decision that had a real alternative (a hard part's implementation, the module split, a library, a storage model). This takes the place of the review's **Fine as is** section: instead of defending code that exists, it shows the roads not taken so a later reader does not reopen them by accident.

- **Decision** — one line.
- **Chosen** and **Rejected** — side by side, each a small flow graph (two to four nodes and the edges between them), so the difference in where the flow goes is visible, not just told.
- **Why rejected** — the constraint that kills it, one or two sentences.
- **Revisit if** — the change in circumstances that would flip the decision.
- **ADR** — whether this deserves an ADR now (most do, in a new project).

### First slice

The thinnest end-to-end path through the module map that exercises the riskiest hard part: what is built first, which adapters are fakes at that point, and what "it works" means. Draw it as the module map with the slice's path highlighted and everything else faded. List the spikes (`Needs a spike` cards) that should run before or alongside it.

### Top decision

One larger card at the end: the decision the user should confirm or challenge first, and why it is first.

## Tone

As in [HTML-REPORT.md](HTML-REPORT.md): diagrams carry the weight, prose is plain and sparse, glossary terms exactly. Two additions for a design that does not exist yet:

- Write in the future tense only where it is unclear whether something exists; otherwise describe the design in the present tense, as a plan.
- Say what is decided and what is not. A design report that sounds certain about an unverified assumption is worse than one that marks it `Needs a spike`.
