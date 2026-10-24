# Stack Report

For a new project when the user asks you to choose what it is built on: language, runtime, framework, storage, deployment. A stack is not chosen on its own merits; it is chosen for how well the project's architecture and hard parts fit it. So this report lists the plausible stacks **each with the architecture it implies**, compares them against the project's constraints, and recommends one.

Delivery, diagram primitives, style and tone are those of [HTML-REPORT.md](HTML-REPORT.md); the vocabulary is [LANGUAGE.md](LANGUAGE.md).

## Process

### 1. Pin down what decides the choice

Do steps 1 and 2 of [FOUNDATION-REPORT.md](FOUNDATION-REPORT.md) first: the capabilities, the constraints, the domain language, and the hard parts. The hard parts matter most here: a stack that makes the hardest part easy usually beats one that makes everything else slightly nicer.

Turn the constraints into **criteria**, each with a weight the user would agree with. Typical criteria: target platforms, what the team already knows, the hard parts' fit, ecosystem maturity for the libraries the design needs, deployment and operating cost, performance headroom, licensing, how long the project must live. Ask about any weight you cannot infer; a wrong weight picks the wrong stack.

### 2. Pick two to four candidate stacks

Candidates must all be able to meet the hard constraints. Prefer a real spread (for example a managed full-stack framework, a lean runtime with chosen libraries, a native or compiled option) over three variants of one idea. Stacks that fail a hard constraint go straight to the rejected list with that constraint named.

### 3. Draw each candidate's architecture

For each candidate, sketch the module map it implies (as in the Foundation report): which modules the framework gives you, which you write, where the seams fall, and where each hard part lands. The same system drawn under different stacks is the core of this report; it shows the trade-off instead of asserting it.

### 4. Check the facts

Versions, licences, platform support, hosting limits and library maturity change. Verify each claim the comparison depends on against a source (fetch the project's docs or release notes) and cite it. Mark anything you could not verify.

### 5. Write the report, then ask

Sections below. End by asking the user (through a question tool when one exists) which stack to take forward, with your recommendation first. On their pick, offer a Foundation report for that stack, and record the choice as an ADR ([ADR-FORMAT.md](ADR-FORMAT.md)) with the rejected stacks and their reasons.

## Report sections

### Header

Project name, date, legend, and the **brief** block: capabilities, constraints, and the hard parts in one line each.

### Criteria

A short table: criterion, weight, and the constraint it comes from. This is what every later judgement refers back to.

### Candidate cards

One card per candidate stack:

- **Title** — the stack, named by its parts ("TypeScript · Node 24 · SQLite · single binary").
- **Fit badge** — `Recommended` (emerald), `Viable` (amber), `Stretch` (slate).
- **Components** — monospaced list: language, runtime, framework, storage, deployment, notable libraries, each with the version you checked.
- **Architecture under this stack** — the module map as a flow graph (see "Diagrams show flow" in [HTML-REPORT.md](HTML-REPORT.md)), with the framework-provided modules drawn differently from the ones you would write.
- **Hard parts** — one row per hard part: how it is solved on this stack, and whether that is a library, a framework feature or your own code.
- **Strengths** and **Costs** — bullets, each tied to a criterion.
- **Risks** — what is unverified or immature.
- **Sources** — links.

### Comparison

A matrix: criteria down the side, candidates across, each cell a short verdict. Draw it as a table or a scored grid drawn to scale; if you score, show the weights and keep scores coarse (three or four levels), since false precision is worse than none.

### Rejected stacks

Brief rows: the stack, the constraint that rules it out, and what would bring it back.

### Recommendation

One larger card: the recommended stack, the two or three reasons that decided it (in terms of the criteria), and what would change the answer. Then the offer of a Foundation report.

## Tone

As in [HTML-REPORT.md](HTML-REPORT.md). Compare stacks on the project's criteria, not on popularity or taste; a reason that does not trace back to a criterion does not belong in the report.
