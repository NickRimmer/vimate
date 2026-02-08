# Autonomous Agent Framework Concept (v0)

> Implementation plan: see [v0-implementation-plan.md](v0-implementation-plan.md).

## Intro
This project explores an autonomous-agent model that is different from the usual prompt-response loop.

The agent is a long-lived runtime with its own inner work cycle. User messages are external triggers, not the center of execution. Communication to user is done through a tool, when the agent decides it is needed.

## Core Idea
Two circles:

- Inner circle: Brain (`Core AI` + pluggable `Subminds`), internal dialog/thread, context bucket.
- Outer circle: Triggers and tools.

The core AI works continuously on its job. Incoming data from triggers is delivered through context items. Subminds can help with prioritization and context management while core is working.

## Design Principles (Agreed)
- No hardcoded "ask user" logic.
- Sending a message to user is just one tool call.
- Keep v0 models minimal.

## Framework Shape (Target)
This is a library/framework for building agents with this runtime model.

Developer experience target:
- create an agent via a builder/composition API,
- register triggers, tools, knowledge modules (pluggable), and subminds,
- start the agent as a long-lived async process (for example `StartAsync(...)` returning a `Task`).

## v0 Models

### Context item
`ContextItem { Source, Content, Priority }`

Purpose: represent incoming messages from outer circle into core AI.

Priority levels for v0:
- `Interruption`: must stop current inference loop and inject now (cancel execution if possible).
- `ForNextTurn`: inject on next turn after AI model response or tool execution completes.
- `InTheEnd`: inject when current loop is completed.

### Context bucket
A prioritized bucket/queue of incoming context items for core AI.

Purpose: buffer and order incoming data while the agent is running.

Ordering rules for v0:
- `Interruption` before `ForNextTurn` before `InTheEnd`.
- FIFO inside each priority level.

### Injection orchestrator (non-AI)
Context bucket monitoring and injection are handled by non-AI runtime logic.

Responsibilities:
- monitor incoming context items,
- inject context according to priority semantics,
- request cancellation of current turn for `Interruption` items when required (best-effort, based on model/tool cancellation support).

### Plan
`Plan { steps[] }`

`PlanStep { Index, Content, Status }`

- `Index`: sequence index for ordering.
- `Content`: step text (v0).
- `Status`: enum flag for v0 states:
  - `New` (planned/todo)
  - `Active` (doing/in progress)
  - `Done` (completed/finished)

Plan is controlled through tools and can be updated by the agent during execution.

## v0 Triggers
Triggers are external events that bring new context into the agent runtime.

For the first version, external event type is:
- user incoming message.

Trigger handling model for v0:
- runtime keeps a trigger event queue,
- a dedicated submind monitors trigger events,
- this submind maps each trigger event into a `ContextItem`,
- this submind assigns the correct priority (`Interruption`, `ForNextTurn`, `InTheEnd`) and adds item to the context bucket.

## v0 Tools
- `user.sendMessage`
- `plan.read`
- `plan.update`
- `memory.search`
- `memory.add`
- `memory.remove`

Memory tool behavior for v0:
- `memory.add` and `memory.remove` manage markdown memory files (for example `MEMORY.md`, `AGREEMENTS.md`).
- `memory.search` can query directly against vectorized memory index.

## Memory Concept (v0)
Memory source of truth is markdown files.

Initial memory files can be:
- `MEMORY.md`: general facts about user and facts the user asked to remember.
- `AGREEMENTS.md`: instructions, rules, and agreements the user asked the agent to remember/follow.

Vector search availability is provided by a background indexing service:
- creates embeddings for memory markdown,
- rebuilds vectors when source files change,
- keeps vector index ready for `memory.search`.

Open question (intentionally unresolved in v0):
- retrieval unit for search results: chunk-level vs whole-markdown return.

## Core Execution Behavior (v0)
Plan-driven cycle:

1. Injection orchestrator delivers next context item to Core AI (by priority), if any.
2. Core AI thinks and decides next action.
3. Core AI maintains its own plan:
   - all work must be planned,
   - all thoughts and execution steps in flow must be reasoned and guided by the plan.
4. Core AI uses tools when needed.
5. Core AI reviews plan actuality and adjusts plan if needed.
6. Continue until no more planned actions.

Operational pattern:

`Plan -> Step execution -> Review/Adjust plan -> Next step -> ... -> Review results -> Add more steps if needed -> End`

## Subminds Concept
Subminds are pluggable support parts around core AI. They can be AI-based or non-AI logic.

Possible responsibilities:
- assign incoming context priority,
- combine/manage context items,
- monitor plan drift and inject high-priority reminders from non-AI control logic.
- memory controller (AI core submind):
  - observes core AI tool usage/results,
  - extracts memory mutations that should be persisted,
  - updates/removes data in markdown memory files through memory tools.

Additional submind idea (v0+):
- proactive copilot (non-AI):
  - reads core AI thread/context,
  - runs relevant search in vectorized memory DB,
  - adds short context items for `ForNextTurn` with format: `additional knowledge: ...`,
  - avoids duplicates if similar knowledge was already added in the last X minutes.

Subminds can be triggered by runtime events such as:
- new trigger came,
- new context item added,
- core mind turn completed,
- core mind completed thinking.

Subminds should have capabilities to:
- add items to context bucket,
- read core internal thread/context,
- keep own state (threaded or single-run memory).

For AI-based subminds this can be exposed as tools; for non-AI subminds as services.
