# ViMate v0 Implementation Plan

## Purpose
This document is the decision-complete implementation plan for the first ViMate framework skeleton.  
It is intended to be consumed by a new AI coding agent without additional clarification.

## Scope
Build a first runnable skeleton with:

- framework-owned runtime lifecycle
- push-based generic triggers
- submind-driven trigger/context orchestration
- internal prioritized context bucket with interruption semantics
- MS Agents Framework (`AIAgent`) for inference and tool execution
- initial MSAF workflow structure for the plan-driven loop
- console sample with minimal interactive chat flow

Out of scope for this iteration:

- Azure OpenAI provider support
- dedicated test project (sample-run validation only)
- advanced memory vector indexing implementation

## Finalized Architectural Decisions

### Agent API
Public runtime API:

- `Task StartAsync()`
- `Task StopAsync()`
- `Task ReceiveTriggerAsync(TriggerEvent trigger)`

Rules:

- no external `CancellationToken` on `StartAsync/StopAsync`
- runtime owns internal `CancellationTokenSource`
- start/stop are the only external lifecycle controls

### Trigger Model
Use only a generic trigger envelope:

- `TriggerEvent` (general event model)

No specialized `UserMessageTriggerEvent` in v0.

### Submind Model
Use an abstract base class with overridable handlers:

- `SubmindBase` with virtual async hooks for runtime events

Runtime dispatches lifecycle events to registered subminds.

### Context Bucket and Priority
Context bucket is runtime-internal (not externally configurable).

Priority rules:

- `Interruption` before `ForNextTurn` before `InTheEnd`
- FIFO inside each priority

Idle and running semantics:

- if idle and context arrives: start a new inference cycle immediately
- if running and `Interruption` arrives: request cancellation of current turn and prioritize injection
- if running and `ForNextTurn` arrives: inject on next turn boundary
- if running and `InTheEnd` arrives: inject after current loop completion

### Builder API
Expose only composition points aligned with the concept:

- `.RegisterSubminds(params ISubmind[] subminds)`
- `.RegisterTools(params IAgentTool[] tools)`
- `.UseOpenAi(...)`
- `.Build()`

Do not expose:

- `.WithCore(...)`
- `.WithTriggerSource(...)`
- `.WithTriggerMapper(...)`
- `.WithContextBucket(...)`

## MSAF and OpenAI Decisions

### Inference/Tool Execution
Inference and tool invocation are owned by MSAF `AIAgent`.  
ViMate runtime must not duplicate a separate custom tool-execution loop.

### Provider
v0 provider is direct OpenAI API.

### Workflow in v0
Workflow is included now (not postponed), with a skeletal but real executable structure representing:

1. think
2. create/update plan
3. execute step
4. validate/adjust plan
5. loop decision
6. finalize/clean plan
7. decide whether to notify user (`user.sendMessage`)

## Framework Structure Plan

Create/organize under `src/ViMate/ViMate.Framework`:

- `Abstractions/`
- `Models/`
- `Runtime/`
- `Subminds/`
- `Builder/`
- `Msaf/`

Primary additions:

- `IAgent`, `ISubmind`, `IAgentTool`
- `TriggerEvent`
- `ContextItem`, `ContextPriority`
- `Plan`, `PlanStep`, `PlanStepStatus`
- internal trigger queue and context bucket
- injection/orchestration runtime services
- `SubmindBase`
- MSAF `AIAgent` + workflow adapter
- fluent `ViMateAgentBuilder`

## Console Sample Plan

Update sample project and `Program.cs` to:

- configure/build/start agent
- read user input from console
- convert each line into generic `TriggerEvent`
- submit via `ReceiveTriggerAsync(...)`
- print agent replies through `user.sendMessage`
- stop on `/exit`

## Concept Clarifications to Keep in Docs
Ensure `docs/concept.md` clearly states:

- triggers are push-based external calls into agent API
- runtime owns trigger queue and context bucket
- submind maps/prioritizes/moves events into context
- runtime handles idle-vs-running injection policy
- MSAF `AIAgent` is core inference/tool engine
- workflow pattern is used for plan-driven execution loop

## Validation Commands
From repo root:

1. `dotnet restore src/ViMate/ViMate.slnx`
2. `dotnet build src/ViMate/ViMate.slnx -c Debug`
3. `dotnet run --project src/ViMate/Samples/ViMate.Samples.Console`

Acceptance:

- build succeeds
- console loop works until `/exit`
- trigger push path functions
- priority semantics and interruption path are present
- MSAF `AIAgent` integration path is active
- workflow skeleton is present and executable
