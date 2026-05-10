# Claude Code from Source — Architecture, Patterns & Internals
---
ingested: 2026-05-10
wiki_pages: [claude-code-bootstrap-pipeline, claude-code-state-architecture, claude-code-api-layer, claude-code-agent-loop, claude-code-tool-system, claude-code-concurrent-tools, claude-code-memory-system, claude-code-swarm, claude-code-terminal-ui, claude-code-input-system, claude-code-mcp, claude-code-remote-execution, claude-code-performance, claude-code-architecture, claude-code-subagent, claude-code-fork-subagent, claude-code-extensibility]
---

> How Anthropic built the most widely used AI coding agent

When Claude Code shipped on npm, the source maps came with it. This book distills the architecture, design decisions, and transferable patterns into 18 chapters you can learn from and apply to your own systems.

---

## Table of Contents

### Part 1: Foundations
*Before the agent can think, the process must exist.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [1](#chapter-1-the-architecture-of-an-ai-agent) | **The Architecture of an AI Agent** | The 6 key abstractions, data flow, permission system, build system |
| [2](#chapter-2-starting-fast--the-bootstrap-pipeline) | **Starting Fast — The Bootstrap Pipeline** | 5-phase init, module-level I/O parallelism, trust boundary |
| [3](#chapter-3-state--the-two-tier-architecture) | **State — The Two-Tier Architecture** | Bootstrap singleton, AppState store, sticky latches, cost tracking |
| [4](#chapter-4-talking-to-claude--the-api-layer) | **Talking to Claude — The API Layer** | Multi-provider client, prompt cache, streaming, error recovery |

### Part 2: The Core Loop
*The heartbeat of the agent: stream, act, observe, repeat.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [5](#chapter-5-the-agent-loop) | **The Agent Loop** | AsyncGenerator driving the entire system |
| [6](#chapter-6-tools--from-definition-to-execution) | **Tools — From Definition to Execution** | The 14-step pipeline from model request to tool result |
| [7](#chapter-7-concurrent-tool-execution) | **Concurrent Tool Execution** | Speculative execution, concurrent-safe batching |

### Part 3: Multi-Agent Orchestration
*One agent is powerful. Many agents working together are transformative.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [8](#chapter-8-spawning-sub-agents) | **Spawning Sub-Agents** | Process isolation, message passing |
| [9](#chapter-9-fork-agents-and-the-prompt-cache) | **Fork Agents and the Prompt Cache** | Cache sharing, 95% input token savings |
| [10](#chapter-10-tasks-coordination-and-swarms) | **Tasks, Coordination, and Swarms** | Swarm teams, mailbox messaging |

### Part 4: Persistence and Intelligence
*An agent without memory makes the same mistakes forever.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [11](#chapter-11-memory--learning-across-conversations) | **Memory — Learning Across Conversations** | File-based memory, Sonnet side-query |
| [12](#chapter-12-extensibility--skills-and-hooks) | **Extensibility — Skills and Hooks** | Two-phase skill loading, 27 lifecycle hooks |

### Part 5: The Interface
*Everything the user sees passes through this layer.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [13](#chapter-13-the-terminal-ui) | **The Terminal UI** | Rendering architecture, packed typed arrays, cell-level diffing |
| [14](#chapter-14-input-and-interaction) | **Input and Interaction** | Keyboard protocols, chord resolution, keybinding system |

### Part 6: Connectivity
*The agent reaches beyond localhost.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [15](#chapter-15-mcp--the-universal-tool-protocol) | **MCP — The Universal Tool Protocol** | Eight transports, seven config scopes, OAuth discovery |
| [16](#chapter-16-remote-control-and-cloud-execution) | **Remote Control and Cloud Execution** | Bridge v1/v2, Direct Connect, upstream proxy |

### Part 7: Performance Engineering
*Making it all fast enough that humans don't notice the machinery.*

| Chapter | Title | Description |
|---------|-------|-------------|
| [17](#chapter-17-performance--every-millisecond-and-token-counts) | **Performance — Every Millisecond and Token Counts** | Startup optimization, bitmap pre-filters, slot reservation |
| [18](#chapter-18-epilogue--what-we-learned) | **Epilogue — What We Learned** | Five architectural bets, what transfers, where agentic systems are heading |

---

## The 10 Patterns That Make It Work

1. **AsyncGenerator as agent loop** — yields Messages, typed Terminal return, natural backpressure and cancellation
2. **Speculative tool execution** — start read-only tools during model streaming, before the response completes
3. **Concurrent-safe batching** — partition tools by safety, run reads in parallel, serialize writes
4. **Fork agents for cache sharing** — parallel children share byte-identical prompt prefixes, saving ~95% input tokens
5. **4-layer context compression** — snip, microcompact, collapse, autocompact — each lighter than the next
6. **File-based memory with LLM recall** — Sonnet side-query selects relevant memories, not keyword matching
7. **Two-phase skill loading** — frontmatter only at startup, full content on invocation
8. **Sticky latches for cache stability** — once a beta header is sent, never unset mid-session
9. **Slot reservation** — 8K default output cap, escalate to 64K on hit (saves context in 99% of requests)
10. **Hook config snapshot** — freeze at startup to prevent runtime injection attacks

---

## About This Book

The source was extracted from npm source maps — the `.js.map` files that shipped with Claude Code contained a `sourcesContent` field with the full original TypeScript. Nearly two thousand files comprising the complete architecture.

36 AI agents analyzed and wrote the entire book in four phases:
- **Exploration** — 6 parallel agents read every file in the source tree
- **Analysis** — 12 agents wrote 494KB of raw technical documentation
- **Writing** — 15 agents rewrote everything from scratch as narrative chapters
- **Review & Revision** — 3 reviewers produced 900 lines of feedback; 3 agents applied all fixes

The entire process — from source extraction to final revised book — took approximately 6 hours.

*Purely educational. This book contains no source code from Claude Code — every code block is original pseudocode written to illustrate architectural patterns.*


---

# Chapter 1: The Architecture of an AI Agent

## What You're Looking At

A traditional CLI is a function. It takes arguments, does work, and exits. grep does not decide to also run sed. curl does not open a file and patch it based on what it downloaded. The contract is simple: one command, one action, deterministic output.

An agentic CLI breaks every part of that contract. It takes a natural language prompt, decides what tools to use, executes them in whatever order the situation demands, evaluates the results, and loops until the task is done or the user stops it. The "program" is not a fixed sequence of instructions — it is a loop around a language model that generates its own instruction sequence at runtime. The tool calls are the side effects. The model's reasoning is the control flow.

Claude Code is Anthropic's production implementation of this idea: a TypeScript monolith of nearly two thousand files that turns a terminal into a full development environment powered by Claude. It shipped to hundreds of thousands of developers, which means every architectural decision carries real-world consequences. This chapter gives you the mental model. Six abstractions define the entire system. A single data flow connects them. Once you internalize the golden path from keystroke to final output, every subsequent chapter is a zoom into one segment of that path.

What follows is a retrospective decomposition — these six abstractions were not designed upfront on a whiteboard. They emerged from the pressures of shipping a production agent to a large user base. Understanding them as they are, not as they were planned, sets the right expectations for the rest of the book.

## The Six Key Abstractions

Claude Code is built on six core abstractions. Everything else — the 400+ utility files, the forked terminal renderer, the vim emulation, the cost tracker — exists to support these six.

Here is what each one does and why it exists.

1. The Query Loop (query.ts, ~1,700 lines). An async generator that is the heartbeat of the entire system. It streams a model response, collects tool calls, executes them, appends results to the message history, and loops. Every interaction — REPL, SDK, sub-agent, headless --print — flows through this single function. It yields Message objects that the UI consumes. Its return type is a discriminated union called Terminal that encodes exactly why the loop stopped: normal completion, user abort, token budget exhaustion, stop hook intervention, max turns, or unrecoverable error. The generator pattern — rather than callbacks or event emitters — gives natural backpressure, clean cancellation, and typed terminal states. Chapter 5 covers the loop's internals in full.

2. The Tool System (Tool.ts, tools.ts, services/tools/). A tool is anything the agent can do in the world: read a file, run a shell command, edit code, search the web. That simplicity of purpose hides significant machinery. Each tool implements a rich interface covering identity, schema, execution, permissions, and rendering. Tools are not just functions — they carry their own permission logic, concurrency declarations, progress reporting, and UI rendering. The system partitions tool calls into concurrent and serial batches, and a streaming executor starts concurrency-safe tools before the model even finishes its response. Chapter 6 covers the full tool interface and execution pipeline.

3. Tasks (Task.ts, tasks/). Tasks are background work units — primarily sub-agents. They follow a state machine: pending -> running -> completed | failed | killed. The AgentTool spawns a new query() generator with its own message history, tool set, and permission mode. Tasks give Claude Code its recursive capability: an agent can delegate to sub-agents, which can delegate further.

4. State (two layers). The system maintains state at two levels. A mutable singleton (STATE) holds ~80 fields of session-level infrastructure: working directory, model configuration, cost tracking, telemetry counters, session ID. It is set once at startup and mutated directly — no reactivity. A minimal reactive store (34 lines, Zustand-shaped) drives the UI: messages, input mode, tool approvals, progress indicators. The separation is intentional: infrastructure state changes rarely and does not need to trigger re-renders; UI state changes constantly and must. Chapter 3 covers the two-tier architecture in depth.

5. Memory (memdir/). The agent's persistent context across sessions. Three tiers: project-level (CLAUDE.md files in the repo), user-level (~/.claude/MEMORY.md), and team-level (shared via symlinks). At session start, the system scans for all memory files, parses frontmatter, and an LLM selects which memories are relevant to the current conversation. Memory is how Claude Code "remembers" your codebase conventions, architectural decisions, and debugging history.

6. Hooks (hooks/, utils/hooks/). User-defined lifecycle interceptors that fire at 27 distinct events across 4 execution types: shell commands, single-shot LLM prompts, multi-turn agent conversations, and HTTP webhooks. Hooks can block tool execution, modify inputs, inject additional context, or short-circuit the entire query loop. The permission system itself is partially implemented through hooks — PreToolUse hooks can deny tool calls before the interactive permission prompt ever fires.

## The Golden Path: From Keystroke to Output

Trace a single request through the system. The user types "add error handling to the login function" and presses Enter.

Three things to notice about this flow.

First, the query loop is a generator, not a callback chain. The REPL pulls messages from it via for await, which means backpressure is natural — if the UI cannot keep up, the generator pauses. This is a deliberate choice over event emitters or observable streams.

Second, tool execution overlaps with model streaming. The StreamingToolExecutor does not wait for the model to finish before starting concurrency-safe tools. A Read call can complete and return its results while the model is still generating the rest of its response. This is speculative execution — if the model's final output invalidates the tool call (rare but possible), the result is discarded.

Third, the entire loop is re-entrant. When the model makes tool calls, the results are appended to the message history, and the loop calls the model again with the updated context. There is no separate "tool result handling" phase — it is all one loop. The model decides when it is done by simply not making any more tool calls.

## The Permission System

Claude Code runs arbitrary shell commands on your machine. It edits your files. It can spawn sub-processes, make network requests, and modify your git history. Without a permission system, this is a security catastrophe.

The system defines seven permission modes, ordered from most to least permissive:

| Mode | Behavior |
|------|----------|
| bypassPermissions | Everything allowed. No checks. Internal/testing only. |
| dontAsk | All allowed, but still logged. No user prompts. |
| auto | Transcript classifier (LLM) decides allow/deny. |
| acceptEdits | File edits auto-approved; all other mutations prompt. |
| default | Standard interactive mode. User approves each action. |
| plan | Read-only. All mutations blocked. |
| bubble | Escalate decision to parent agent (sub-agent mode). |

When a tool call needs permission, the resolution follows a strict chain:

The auto mode deserves special attention. It runs a separate, lightweight LLM call that classifies the tool invocation against the conversation transcript. The classifier sees a compact representation of the tool input and decides whether the action is consistent with what the user asked for. This is the mode that lets Claude Code work semi-autonomously — approving routine operations while flagging anything that looks like it deviates from the user's intent.

Sub-agents default to bubble mode, which means they cannot approve their own dangerous actions. Permission requests propagate up to the parent agent or ultimately to the user. This prevents a sub-agent from silently running destructive commands that the user never saw.

## Multi-Provider Architecture

Claude Code talks to Claude through four different infrastructure paths, all transparent to the rest of the system.

The key insight is that the Anthropic SDK provides wrapper classes for each cloud provider that present the same interface as the direct API client. The getAnthropicClient() factory reads environment variables and configuration to determine which provider to use, constructs the appropriate client, and returns it. From that point forward, callModel() and every other consumer treats it as a generic Anthropic client.

Provider selection is determined at startup and stored in STATE. The query loop never checks which provider is active. This means switching from Direct API to Bedrock is a configuration change, not a code change — the agent loop, tool system, and permission model are entirely provider-agnostic.

## The Build System

Claude Code ships as both an internal Anthropic tool and a public npm package. The same codebase serves both, with compile-time feature flags controlling what gets included.

```javascript
// Conditional imports guarded by feature flags
const reactiveCompact = feature('REACTIVE_COMPACT')
  ? require('./services/compact/reactiveCompact.js')
  : null
```

The feature() function comes from bun:bundle, Bun's built-in bundler API. At build time, each feature flag resolves to a boolean literal. The bundler's dead code elimination then strips the require() call entirely when the flag is false — the module is never loaded, never included in the bundle, and never shipped.

The pattern is consistent: a top-level feature() guard wrapping a require() call. The require() is used instead of import specifically because dynamic require() can be fully eliminated by the bundler when the guard is false, while dynamic import() cannot (it returns a Promise that the bundler must preserve).

There is an irony worth noting. The source maps published with early npm releases contained sourcesContent — the full original TypeScript source, including the internal-only code paths. The feature flags successfully stripped the runtime code but left the source in the maps. This is how the Claude Code source became publicly readable.

## How the Pieces Connect

The six abstractions form a dependency graph:

Memory feeds into the query loop as part of the system prompt. The query loop drives tool execution. Tool results feed back into the query loop as messages. Tasks are recursive query loops with isolated message histories. Hooks intercept the query loop at defined points. State is read and written by everything, with the reactive store bridging to the UI.

The circular dependency between the query loop and the tool system is the system's defining characteristic. The model generates tool calls. Tools execute and produce results. Results are appended to the message history. The model sees the results and decides what to do next. This cycle continues until the model stops generating tool calls or an external constraint (token budget, max turns, user abort) terminates it.

Here is how they connect to the chapters that follow: the golden path from input to output is the thread that runs through the entire book. Chapter 2 traces how the system boots to the point where this path can execute. Chapter 3 explains the two-tier state architecture that the path reads and writes. Chapter 4 covers the API layer that the query loop calls. Each subsequent chapter zooms into one segment of the path you have just seen end-to-end.

## Apply This

If you are building an agentic system — any system where an LLM decides what actions to take at runtime — here are the patterns from Claude Code's architecture that transfer.

The generator loop pattern. Use an async generator as your agent loop, not callbacks or event emitters. The generator gives you natural backpressure (consumers pull at their own pace), clean cancellation (.return() on the generator), and a typed return value for terminal states. The problem it solves: in callback-based agent loops, it is difficult to know when the loop is "done" and why. Generators make termination a first-class part of the type system.

The self-describing tool interface. Every tool should declare its own concurrency safety, permission requirements, and rendering behavior. Do not put this logic in a central orchestrator that "knows about" each tool. The problem it solves: a central orchestrator becomes a god object that must be updated every time a tool is added. Self-describing tools scale linearly — adding tool N+1 requires zero changes to existing code.

Separate infrastructure state from reactive state. Not all state needs to trigger UI updates. Session configuration, cost tracking, and telemetry belong in a plain mutable object. Message history, progress indicators, and approval queues belong in a reactive store. The problem it solves: making everything reactive adds subscription overhead and complexity to state that changes once at startup and is read a thousand times. Two tiers match two access patterns.

Permission modes, not permission checks. Define a small set of named modes (plan, default, auto, bypass) and resolve every permission decision through the mode. Do not scatter if (isAllowed) checks through tool implementations. The problem it solves: inconsistent permission enforcement. When every tool goes through the same mode-based resolution chain, you can reason about the system's security posture by knowing which mode is active.

Recursive agent architecture via tasks. Sub-agents should be new instances of the same agent loop with their own message history, not special-cased code paths. Permission escalation flows upward via bubble mode. The problem it solves: sub-agent logic that diverges from the main agent loop, leading to subtle differences in behavior and error handling. If the sub-agent is the same loop, it inherits all the same guarantees.


---

# Chapter 2: Starting Fast — The Bootstrap Pipeline

If Chapter 1 gave you the map of Claude Code's architecture, this chapter gives you the route it takes to reach a working state. Every component from the six abstractions — the query loop, the tool system, the state layers, hooks, memory — must be initialized before the user sees a cursor. The budget for all of it: 300 milliseconds.

Three hundred milliseconds is the threshold where humans perceive a tool as instant. Cross it, and the CLI feels sluggish. Miss it by a lot, and developers stop using it. Everything in this chapter exists to stay under that line.

Bootstrap must accomplish four things: validate the environment, establish security boundaries, configure the communication layer, and render the UI. It must do all four in under 300ms. The architectural insight is that these four jobs can be partially overlapped, carefully ordered, and aggressively pruned to fit inside a budget that feels impossible for a system this complex.

A note on methodology: the timestamps in this chapter are approximate, derived from the codebase's own profiling checkpoints. They represent typical warm-start timings on modern hardware. Cold starts are slower. The absolute numbers matter less than the relative structure: which operations overlap, which block, and which are deferred.

## The Shape of the Pipeline

The startup pipeline lives in five files, executed in sequence. Each file narrows the scope of what the system needs to do next:

Each file does the minimum work necessary before passing control to the next. cli.tsx tries to exit before importing anything heavy. main.tsx fires slow operations as side effects during import evaluation. init.ts resolves configuration and establishes the trust boundary. setup.ts registers capabilities. replLauncher.ts picks the right entry point and starts the UI.

Three parallelism strategies make this fast:

- Module-level subprocess dispatch. Fire keychain and MDM reads as side effects during import evaluation. The subprocesses run while the remaining ~135ms of static imports load.

- Promise parallelism in setup. Socket binding, hook snapshotting, command loading, and agent definition loading all run concurrently.

- Post-render deferred prefetches. Everything the user does not need before typing their first message — git status, model capabilities, AWS credentials — runs after the prompt is visible.

A fourth strategy is less visible but equally important: dynamic imports to defer module evaluation. The codebase uses await import('./module.js') in at least a dozen places to avoid loading code until it is needed. OpenTelemetry (400KB + 700KB gRPC) loads only when telemetry initializes. React components load only when rendering. Each dynamic import trades cold-path latency (first use triggers module evaluation) for hot-path speed (startup does not pay for modules it might never use).

## Phase 0: Fast-Path Dispatch (cli.tsx)

The first file the process enters, cli.tsx, has one job: determine whether the full bootstrap pipeline is needed at all. Many invocations — claude --version, claude --help, claude mcp list — need a specific answer and nothing else. Loading React, initializing telemetry, reading the keychain, and setting up the tool system would be pure waste.

The pattern is: check argv, dynamically import only the handler you need, and exit before the rest of the system loads.

```javascript
// Pseudocode for the fast-path pattern
if (args.length === 1 && args[0] === '--version') {
  const { printVersion } = await import('./commands/version.js')
  await printVersion()
  process.exit(0)
}
```

There are roughly a dozen fast paths covering version, help, configuration, MCP server management, and update checks. The specifics do not matter — the pattern does. Each path dynamically imports exactly one module, calls one function, and exits. The rest of the codebase never loads.

This is the first instance of a principle that recurs throughout bootstrap: do less by knowing more about intent. The argv array reveals the user's intent. If the intent is narrow, the execution path should be narrow too.

If no fast path matches, cli.tsx falls through to the full main.tsx import, and the real startup begins.

## Phase 1: Module-Level I/O (main.tsx)

When main.tsx is imported, its module-level side effects fire during evaluation — before any function in the file is called. This is the most performance-critical technique in the entire bootstrap:

```javascript
// These run at import time, not at call time
const mdmPromise = startMDMSubprocess()
const keychainPromise = readKeychainCredentials()
```

While the JavaScript engine evaluates the rest of main.tsx and its transitive imports (~138ms of module evaluation), these two promises are already in flight. The MDM (Mobile Device Management) subprocess checks organizational security policies. The keychain read fetches stored credentials. Both are I/O-bound operations that would otherwise serialize on the critical path.

The insight: module evaluation is not idle time — it is time you can overlap with I/O. By the time main.tsx's exported functions are first called, these promises are often already resolved.

This technique requires suppressing ESLint's top-level-await and side-effect-in-module-scope rules in the relevant files. The codebase has a custom ESLint rule specifically for process.env access patterns that allows controlled side effects at module scope while preventing uncontrolled ones elsewhere.

## Phase 2: Parse and Trust (init.ts)

The init() function is memoized — calling it multiple times is safe and returns the same result. This is important because multiple entry points (the REPL, print mode, SDK mode) may each call init(), and the memoization guarantees it runs exactly once.

The function resolves command-line arguments via Commander, loads configuration from multiple sources (global settings, project settings, environment variables), and then hits the most important boundary in the pipeline.

### The Trust Boundary

Before the trust boundary, the system operates in a restricted mode. After it, full capabilities are available. The boundary exists because Claude Code reads environment variables — and environment variables can be poisoned.

The trust boundary is not about the user trusting Claude Code. It is about Claude Code trusting the environment. A malicious .bashrc could set LD_PRELOAD to inject code into every subprocess. The trust dialog ensures the user explicitly consents to operating in a directory that may have been configured by someone else.

The system has ten distinct trust-sensitive operations. Before the user accepts the trust dialog, only safe operations run: TLS certificate configuration, theme preferences, telemetry opt-out. After trust, the system reads potentially dangerous environment variables (PATH, LD_PRELOAD, NODE_OPTIONS), executes git commands, and applies the full environment configuration.

### The preAction Hook

Commander's preAction hook is the architectural linchpin. Commander parses the command structure (flags, subcommands, positional arguments) without executing anything. The preAction hook fires after parsing but before the matched command handler runs:

```javascript
program.hook('preAction', async (thisCommand) => {
  await init(thisCommand)
})
```

This separation means fast-path commands (handled in cli.tsx before Commander loads) never pay the init() cost. Only commands that need the full environment trigger initialization.

## Phase 3: Setup (setup.ts)

After init() completes, setup() registers all the capabilities the system needs:

Commands, agents, hooks, and plugins all register in parallel where possible. The setup phase is where the system transitions from "I know my configuration" to "I have all my capabilities." After setup, every tool is registered, every hook is wired, and the system is ready to handle user input.

Setup also handles the security hook snapshot. The hook configuration is read from disk once, frozen into an immutable snapshot, and used for the rest of the session. Later modifications to the hooks configuration file on disk are ignored. This prevents an attacker from modifying hook rules after the session starts — the frozen snapshot is the only source of truth for permission decisions.

## Phase 4: Launch (replLauncher.ts)

Seven different code paths converge on replLauncher.ts: interactive REPL, print mode (--print), SDK mode, resume (--resume), continue (--continue), pipe mode, and headless. The launcher inspects the configuration produced by init() and dispatches to the right entry point.

Two examples illustrate the range:

Interactive REPL — the standard case. The launcher mounts the React/Ink component tree, starts the terminal renderer, and enters the event loop. The user sees a prompt and can start typing.

Print mode (--print) — a single prompt from argv. The launcher creates a headless query loop with no React tree, runs it to completion, streams the output to stdout, and exits. Same agent loop, different presentation.

The important detail: all seven paths eventually call query() — the same agent loop from Chapter 1. The launch path determines how the loop is presented (interactive terminal, single-shot, SDK protocol), not what it does. This convergence is what makes the architecture testable and predictable: regardless of how the user invokes Claude Code, the core behavior is identical.

## The Startup Timeline

Here is what the full pipeline looks like in time:

The critical path runs through module evaluation (the single longest phase at ~138ms), then Commander parse, init, and setup. The parallel I/O operations (MDM, keychain) overlap with module evaluation and are typically resolved before they are needed.

### The Performance Budget

| Phase | Time | What Happens |
|-------|------|--------------|
| Fast-path check | ~5ms | Check argv, exit early if possible |
| Module evaluation | ~138ms | Import tree, fire parallel I/O |
| Commander parse | ~3ms | Parse flags and subcommands |
| init() | ~14ms | Config resolution, trust boundary |
| setup() | ~35ms | Commands, agents, hooks, plugins |
| Launch + first render | ~25ms | Pick path, mount React, first paint |
| Total | ~240ms | Under 300ms budget |

The total is approximately 240ms on a modern machine — 60ms of headroom under the 300ms budget. Cold starts (first run after reboot, OS cache empty) can push module evaluation to 200ms+, bringing the total closer to the limit.

## The Migration System

A brief note on one subsystem that runs during init: schema migrations. Claude Code stores configuration and session data in local files and directories. When the format changes between versions, migrations run automatically at startup.

Each migration is a function with a version number. The system checks the current schema version against the highest migration version, runs pending migrations in order, and updates the version. Migrations are idempotent and fast (operating on small local files, not databases). The entire migration pass typically completes in under 5ms. If a migration fails, it logs the error and continues — availability beats strict consistency for local configuration.

## What Startup Teaches About System Design

The bootstrap pipeline is a study in narrowing scopes. Each phase reduces the space of possibilities:

- Phase 0 narrows from "any CLI invocation" to "needs full bootstrap"
- Phase 1 narrows from "everything must load" to "load in parallel with I/O"
- Phase 2 narrows from "unknown environment" to "trusted, configured environment"
- Phase 3 narrows from "no capabilities" to "fully registered"
- Phase 4 narrows from "seven possible modes" to "one concrete launch path"

By the time the REPL renders, every decision has been made. The query loop receives a fully configured environment with no ambiguity about what mode it is in, which tools are available, or what permissions apply. The 300ms budget is not just a performance target — it is a forcing function that prevents bootstrap from becoming a lazy initialization system where decisions are deferred and scattered throughout the codebase.

## Apply This

Overlap I/O with initialization. Fire slow operations (subprocess spawns, credential reads, network checks) at module evaluation time, before they are needed. The JavaScript engine is doing synchronous work anyway — use that time for parallel I/O. The pattern: const promise = startSlowThing() at the top of the file, await promise at the point of use.

Narrow scope as early as possible. The bootstrap pipeline's five files form a funnel: each phase eliminates work that subsequent phases do not need to do. Fast-path dispatch is the most dramatic example, but the principle applies everywhere. If you can determine at parse time that a code path is unnecessary, skip it.

Establish trust boundaries explicitly. If your application reads from an environment it does not control (environment variables, configuration files, shell settings), draw a clear line between "safe to read before the user consents" and "only read after consent." The trust boundary prevents a class of attacks where a malicious environment poisons the application before the user has a chance to evaluate it.

Memoize your init function. Make initialization idempotent — calling it twice produces the same result. This eliminates ordering bugs when multiple entry points may each trigger initialization. The memoization pattern is trivial but eliminates an entire class of double-initialization bugs.

Capture early input before yielding. In an event-driven system, user input that arrives during initialization can be lost. Claude Code captures the initial prompt from argv before any async work begins, ensuring that claude "fix the bug" does not drop the prompt if initialization takes longer than expected.


---

# Chapter 3: State — The Two-Tier Architecture

Chapter 2 traced the bootstrap pipeline from process start to first render. By the end, the system had a fully configured environment. But configured with what? Where does the session ID live? The current model? The message history? The cost tracker? The permission mode? Where does state live, and why does it live there?

Every long-running application eventually faces this question. For a simple CLI tool the answer is trivial — a few variables in main(). But Claude Code is not a simple CLI tool. It is a React application rendered through Ink, with a process lifecycle that spans hours, a plugin system that loads at arbitrary times, an API layer that must construct prompts from cached context, a cost tracker that survives process restarts, and dozens of infrastructure modules that need to read and write shared data without importing each other.

The naive approach — a single global store — fails immediately. If the cost tracker updated the same store that drives React re-renders, every API call would trigger a full component tree reconciliation. Infrastructure modules (bootstrap, context building, cost tracking, telemetry) cannot import React. They run before React mounts. They run after React unmounts. They run in contexts where no component tree exists at all. Putting everything into a React-aware store would create circular dependencies across the entire import graph.

Claude Code solves this with a two-tier architecture: a mutable process singleton for infrastructure state, and a minimal reactive store for UI state. This chapter explains both tiers, the side-effect system that bridges them, and the supporting subsystems that depend on this foundation. Every subsequent chapter assumes you understand where state lives and why it lives there.

## 3.1 Bootstrap State — The Process Singleton

### Why a Mutable Singleton

The bootstrap state module (bootstrap/state.ts) is a single mutable object created once at process start:

```javascript
const STATE: State = getInitialState()
```

The comment above this line reads: AND ESPECIALLY HERE. Two lines above the type definition: DO NOT ADD MORE STATE HERE - BE JUDICIOUS WITH GLOBAL STATE. These comments have the tone of engineers who learned the cost of an ungoverned global object the hard way.

A mutable singleton is the right choice here for three reasons. First, bootstrap state must be available before any framework initializes — before React mounts, before the store is created, before plugins load. Module-scope initialization is the only mechanism that guarantees availability at import time. Second, the data is inherently process-scoped: session IDs, telemetry counters, cost accumulators, cached paths. There is no meaningful "previous state" to diff against, no subscribers to notify, no undo history. Third, the module must be a leaf in the import dependency graph. If it imported React, or the store, or any service module, it would create cycles that break the bootstrap sequence described in Chapter 2. By depending on nothing but utility types and node:crypto, it remains importable from anywhere.

### The ~80 Fields

The State type contains approximately 80 fields. A sampling reveals the breadth:

Identity and paths — originalCwd, projectRoot, cwd, sessionId, parentSessionId. The originalCwd is resolved through realpathSync and NFC-normalized at process start. It never changes.

Cost and metrics — totalCostUSD, totalAPIDuration, totalLinesAdded, totalLinesRemoved. These accumulate monotonically through the session and persist to disk on exit.

Telemetry — meter, sessionCounter, costCounter, tokenCounter. OpenTelemetry handles, all nullable (null until telemetry initializes).

Model configuration — mainLoopModelOverride, initialMainLoopModel. The override is set when the user changes models mid-session.

Session flags — isInteractive, kairosActive, sessionTrustAccepted, hasExitedPlanMode. Booleans that gate behavior for the session duration.

Cache optimization — promptCache1hAllowlist, promptCache1hEligible, systemPromptSectionCache, cachedClaudeMdContent. These exist to prevent redundant computation and prompt cache busting.

### The Getter/Setter Pattern

The STATE object is never exported. All access goes through approximately 100 individual getter and setter functions:

```javascript
// Pseudocode — illustrates the pattern
export function getProjectRoot(): string {
  return STATE.projectRoot
}

export function setProjectRoot(dir: string): void {
  STATE.projectRoot = dir.normalize('NFC') // NFC normalization on every path setter
}
```

This pattern enforces encapsulation, NFC normalization on every path setter (preventing Unicode mismatches on macOS), type narrowing, and bootstrap isolation. The trade-off is verbosity — a hundred functions for eighty fields. But in a codebase where a stray mutation could bust a 50,000-token prompt cache, explicitness wins.

### The Signal Pattern

Bootstrap cannot import listeners (it is a DAG leaf), so it uses a minimal pub/sub primitive called createSignal. The sessionSwitched signal has exactly one consumer: concurrentSessions.ts, which keeps PID files in sync. The signal is exposed as onSessionSwitch = sessionSwitched.subscribe, letting callers register themselves without bootstrap knowing who they are.

### The Five Sticky Latches

The most subtle fields in bootstrap state are five boolean latches that follow the same pattern: once a feature is first activated during a session, a corresponding flag stays true for the rest of the session. They all exist for one reason: prompt cache preservation.

Claude's API supports server-side prompt caching. When consecutive requests share the same system prompt prefix, the server reuses cached computations. But the cache key includes HTTP headers and request body fields. If a beta header appears in request N but not request N+1, the cache is busted — even if the prompt content is identical. For a system prompt exceeding 50,000 tokens, a cache miss is expensive.

The five latches:

| Latch | What It Prevents |
|-------|------------------|
| afkModeHeaderLatched | Shift+Tab auto mode toggling flips the AFK beta header on/off |
| fastModeHeaderLatched | Fast mode cooldown enter/exit flips the fast mode header |
| cacheEditingHeaderLatched | Remote feature flag changes bust every active user's cache |
| thinkingClearLatched | Triggered on confirmed cache miss (>1h idle). Prevents re-enabling thinking blocks from busting freshly warmed cache |
| pendingPostCompaction | Consume-once flag for telemetry: distinguishes compaction-induced cache misses from TTL-expiry misses |

All five use a three-state type: boolean | null. The null initial value means "not yet evaluated." true means "latched on." They never return to null or false once set to true. This is the defining property of a latch.

The implementation pattern:

```javascript
function shouldSendBetaHeader(featureCurrentlyActive: boolean): boolean {
  const latched = getAfkModeHeaderLatched()
  if (latched === true) return true // Already latched -- always send
  if (featureCurrentlyActive) {
    setAfkModeHeaderLatched(true) // First activation -- latch it
    return true
  }
  return false // Never activated -- don't send
}
```

Why not just always send all beta headers? Because headers are part of the cache key. Sending an unrecognized header creates a different cache namespace. The latch ensures you only enter a cache namespace when you actually need it, then stay there.

## 3.2 AppState — The Reactive Store

### The 34-Line Implementation

The UI state store lives in state/store.ts:

The store implementation is approximately 30 lines: a closure over a state variable, an Object.is equality check to prevent spurious updates, synchronous listener notification, and an onChange callback for side effects. The skeleton looks like:

```javascript
// Pseudocode — illustrates the pattern
function makeStore(initial, onTransition) {
  let current = initial
  const subs = new Set()
  return {
    read: () => current,
    update: (fn) => { /* Object.is guard, then notify */ },
    subscribe: (cb) => { subs.add(cb); return () => subs.delete(cb) },
  }
}
```

Thirty-four lines. No middleware, no devtools, no time-travel debugging, no action types. Just a closure over a mutable variable, a Set of listeners, and an Object.is equality check. This is Zustand without the library.

The design decisions worth examining:

Updater function pattern. There is no setState(newValue) — only setState((prev) => next). Every mutation receives the current state and must produce the next state, eliminating stale-state bugs from concurrent mutations.

Object.is equality check. If the updater returns the same reference, the mutation is a no-op. No listeners fire. No side effects run. Critical for performance — components that spread-and-set without changing values produce no re-renders.

onChange fires before listeners. The optional onChange callback receives both old and new state and fires synchronously before any subscriber is notified. This is used for side effects (Section 3.4) that must complete before the UI re-renders.

No middleware, no devtools. This is not an oversight. When your store needs exactly three operations (get, set, subscribe), an Object.is equality check, and a synchronous onChange hook, 34 lines of code you own is better than a dependency. You control the exact semantics. You can read the entire implementation in thirty seconds.

### The AppState Type

The AppState type (~452 lines) is the shape of everything the UI needs to render. It is wrapped in DeepImmutable<> for most fields, with explicit exclusions for fields containing function types:

```typescript
export type AppState = DeepImmutable<{
  settings: SettingsJson
  verbose: boolean
  // ... ~150 more fields
}> & {
  tasks: { [taskId: string]: TaskState } // Contains abort controllers
  agentNameRegistry: Map<string, AgentId>
}
```

The intersection type lets most fields be deeply immutable while exempting fields that hold functions, Maps, and mutable refs. Full immutability is the default, with surgical escape hatches where the type system would fight the runtime semantics.

### React Integration

The store integrates with React through useSyncExternalStore:

```javascript
// Standard React pattern — useSyncExternalStore with a selector
export function useAppState<T>(selector: (state: AppState) => T): T {
  const store = useContext(AppStoreContext)
  return useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState()),
  )
}
```

The selector must return an existing sub-object reference (not a freshly constructed object) for Object.is comparison to prevent unnecessary re-renders. If you write useAppState(s => ({ a: s.a, b: s.b })), every render produces a new object reference, and the component re-renders on every state change. This is the same constraint Zustand users face — cheaper comparisons, but the selector author must understand reference identity.

## 3.3 How the Two Tiers Relate

The two tiers communicate through explicit, narrow interfaces.

Bootstrap state flows into AppState during initialization: getDefaultAppState() reads settings from disk (which bootstrap helped locate), checks feature flags (which bootstrap evaluated), and sets the initial model (which bootstrap resolved from CLI args and settings).

AppState flows back to bootstrap state through side effects: when the user changes the model, onChangeAppState calls setMainLoopModelOverride() in bootstrap. When settings change, credential caches in bootstrap are cleared.

But the two tiers never share a reference. A module that imports bootstrap state does not need to know about React. A component that reads AppState does not need to know about the process singleton.

A concrete example clarifies the data flow. When the user types /model claude-sonnet-4:

- The command handler calls store.setState(prev => ({ ...prev, mainLoopModel: 'claude-sonnet-4' }))
- The store's Object.is check detects a change
- onChangeAppState fires, detects the model changed, calls setMainLoopModelOverride() (updates bootstrap) and updateSettingsForSource() (persists to disk)
- All store subscribers fire — React components re-render to show the new model name
- The next API call reads the model from getMainLoopModelOverride() in bootstrap state

Steps 1-4 are synchronous. The API client in step 5 may run seconds later. But it reads from bootstrap state (updated in step 3), not from AppState. This is the two-tier handoff: the UI store is the source of truth for what the user chose, but bootstrap state is the source of truth for what the API client uses.

The DAG property — bootstrap depends on nothing, AppState depends on bootstrap for init, React depends on AppState — is enforced by an ESLint rule that prevents bootstrap/state.ts from importing modules outside its allowed set.

## 3.4 Side Effects: onChangeAppState

The onChange callback is where the two tiers synchronize. Every setState call triggers onChangeAppState, which receives both previous and new state and decides what external effects to fire.

Permission mode sync is the primary use case. Prior to this centralized handler, permission mode was synced to the remote session (CCR) by only 2 of 8+ mutation paths. The other six — Shift+Tab cycling, dialog options, slash commands, rewind, bridge callbacks — all mutated AppState without telling CCR. The external metadata drifted out of sync.

The fix: stop scattering notifications across mutation sites and instead hook the diff in one place. The comment in the source code lists every mutation path that was broken and notes that "the scattered callsites above need zero changes." This is the architectural benefit of centralized side effects — coverage is structural, not manual.

Model changes keep bootstrap state in sync with what the UI renders. Settings changes clear credential caches and re-apply environment variables. Verbose toggle and expanded view are persisted to global config.

The pattern — centralized side effects on a diffable state transition — is essentially the Observer pattern applied at the granularity of a state diff rather than individual events. It scales better than scattered event emissions because the number of side effects grows much more slowly than the number of mutation sites.

## 3.5 Context Building

Three memoized async functions in context.ts build the system prompt context prepended to every conversation. Each is computed once per session, not per turn.

getGitStatus runs five git commands in parallel (Promise.all), producing a block with the current branch, default branch, recent commits, and working tree status. The --no-optional-locks flag prevents git from taking write locks that could interfere with concurrent git operations in another terminal.

getUserContext loads CLAUDE.md content and caches it in bootstrap state via setCachedClaudeMdContent. This cache breaks a circular dependency: the auto-mode classifier needs CLAUDE.md content, but CLAUDE.md loading goes through the filesystem, which goes through permissions, which calls the classifier. By caching in bootstrap state (a DAG leaf), the cycle is broken.

All three context functions use Lodash's memoize (compute once, cache forever) rather than TTL-based caching. The reasoning: if git status were re-computed every 5 minutes, the change would bust the server-side prompt cache. The system prompt even tells the model: "This is the git status at the start of the conversation. Note that this status is a snapshot in time."

## 3.6 Cost Tracking

Every API response flows through addToTotalSessionCost, which accumulates per-model usage, updates bootstrap state, reports to OpenTelemetry, and recursively processes advisor tool usage (nested model calls within a response).

Cost state survives process restarts through save-and-restore to a project config file. The session ID is used as a guard — costs are only restored if the persisted session ID matches the session being resumed.

Histograms use reservoir sampling (Algorithm R) to maintain bounded memory while accurately representing distributions. The 1,024-entry reservoir produces p50, p95, and p99 percentiles. Why not a simple running average? Because averages hide distribution shape. A session where 95% of API calls take 200ms and 5% take 10 seconds has the same average as one where all calls take 690ms, but the user experience is radically different.

## 3.7 What We Learned

The codebase has grown from a simple CLI to a system with ~450 lines of state type definitions, ~80 fields of process state, a side-effect system, multiple persistence boundaries, and cache optimization latches. None of this was designed upfront. The sticky latches were added when cache busting became a measurable cost problem. The onChange handler was centralized when 6 of 8 permission sync paths were discovered to be broken. The CLAUDE.md cache was added when a circular dependency emerged.

This is the natural growth pattern of state in a complex application. The two-tier architecture provides enough structure to contain the growth — new bootstrap fields do not affect React rendering, new AppState fields do not create import cycles — while remaining flexible enough to accommodate patterns that were not anticipated in the original design.

## 3.8 State Architecture Summary

| Property | Bootstrap State | AppState |
|----------|----------------|----------|
| Location | Module-scope singleton | React context |
| Mutability | Mutable through setters | Immutable snapshots via updater |
| Subscribers | Signal (pub/sub) for specific events | useSyncExternalStore for React |
| Availability | Import time (before React) | After provider mounts |
| Persistence | Process exit handlers | Via onChange to disk |
| Equality | N/A (imperative reads) | Object.is reference check |
| Dependencies | DAG leaf (imports nothing) | Imports types from across codebase |
| Test reset | resetStateForTests() | Create new store instance |
| Primary consumers | API client, cost tracker, context builder | React components, side effects |

## Apply This

Separate state by access pattern, not by domain. Session ID belongs in the singleton not because it is "infrastructure" in the abstract, but because it must be readable before React mounts and writable without notifying subscribers. Permission mode belongs in the reactive store because changing it must trigger re-renders and side effects. Let the access pattern drive the tier, and the architecture follows naturally.

The sticky latch pattern. Any system that interacts with a cache (prompt cache, CDN, query cache) faces the same problem: feature toggles that change the cache key mid-session cause invalidation. Once a feature is activated, its cache key contribution stays active for the session. The three-state type (boolean | null, meaning "not evaluated / on / never off") makes the intent self-documenting. Especially valuable when the cache is not under your control.

Centralize side effects on state diffs. When multiple code paths can change the same state, do not scatter notifications across mutation sites. Hook the store's onChange callback and detect which fields changed. Coverage becomes structural (any mutation triggers the effect) rather than manual (each mutation site must remember to notify).

Prefer 34 lines you own over a library you do not. When your requirements are exactly get, set, subscribe, and a change callback, a minimal implementation gives you full control over the semantics. In a system where state management bugs can cost real money, that transparency has value. The key insight is recognizing when you do not need a library.

Use process exit as a persistence boundary with intention. Multiple subsystems persist state on process exit. The trade-off is explicit: non-graceful termination (SIGKILL, OOM) loses accumulated data. This is acceptable because the data is diagnostic, not transactional, and writing to disk on every state change would be too expensive for counters that increment hundreds of times per session.

The two-tier architecture established in this chapter — bootstrap singleton for infrastructure, reactive store for UI, side effects bridging them — is the foundation that every subsequent chapter builds on. The conversation loop (Chapter 4) reads context from the memoized builders. The tool system (Chapter 5) checks permissions from AppState. The agent system (Chapter 8) creates task entries in AppState while tracking costs in bootstrap state. Understanding where state lives, and why, is prerequisite to understanding how any of these systems work.

Some fields straddle the boundary. The main loop model exists in both tiers: mainLoopModel in AppState (for UI rendering) and mainLoopModelOverride in bootstrap state (for API client consumption). The onChangeAppState handler keeps them synchronized. This duplication is the cost of the two-tier split. But the alternative — having the API client import the React store, or having React components read from the process singleton — would violate the dependency direction that keeps the architecture sound. A small amount of controlled duplication, bridged by a centralized synchronization point, is preferable to a tangled dependency graph.


---

# Chapter 4: Talking to Claude — The API Layer

Chapter 3 established where state lives and how the two tiers communicate. Now we follow what happens when that state is put to use: the system needs to talk to a language model. Everything in Claude Code — the bootstrap sequence, the state system, the permission framework — exists to serve this moment.

This layer handles more failure modes than any other part of the system. It must route through four cloud providers via a single transparent interface. It must construct system prompts with byte-level awareness of how the server's prompt cache works, because a single misplaced section can bust a cache worth 50,000+ tokens. It must stream responses with active failure detection, because TCP connections die silently. And it must maintain session-stable invariants so that mid-conversation changes to feature flags do not cause invisible performance cliffs.

Let us trace a single API call from start to finish.

## The Multi-Provider Client Factory

The getAnthropicClient() function is the single factory for all model communication. It returns an Anthropic SDK client configured for whichever provider the deployment targets:

The dispatch is entirely environment-variable driven, evaluated in a fixed priority order. All four provider-specific SDK classes are cast to Anthropic via as unknown as Anthropic. The comment in the source is refreshingly honest: "we have always been lying about the return type." This deliberate type erasure means every consumer sees a uniform interface. The rest of the codebase never branches on provider.

Each provider SDK is dynamically imported — AnthropicBedrock, AnthropicFoundry, AnthropicVertex are heavy modules with their own dependency trees. The dynamic import ensures unused providers never load.

Provider selection is determined at startup and stored in bootstrap STATE. The query loop never checks which provider is active. Switching from Direct API to Bedrock is a configuration change, not a code change.

### The buildFetch Wrapper

Every outbound fetch gets wrapped to inject an x-client-request-id header — a UUID generated per request. When a request times out, the server never assigns a request ID to the response. Without the client-side ID, the API team cannot correlate the timeout with server-side logs. This header bridges that gap. It is only sent to first-party Anthropic endpoints — third-party providers might reject unknown headers.

## System Prompt Construction

The system prompt is the most cache-sensitive artifact in the entire system. Claude's API provides server-side prompt caching: identical prompt prefixes across requests can be cached, saving both latency and cost. A 200K-token conversation might have 50-70K tokens that are identical to the previous turn. Busting that cache forces the server to re-process all of it.

### The Dynamic Boundary Marker

The prompt is built as an array of string sections with a critical dividing line:

Everything before the boundary is identical across sessions, users, and organizations — it gets the highest tier of server-side caching. Everything after contains user-specific content and drops to per-session caching.

The naming convention for sections is deliberately loud. Adding a new section requires choosing between systemPromptSection (safe, cached) and DANGEROUS_uncachedSystemPromptSection (cache-breaking, requires a reason string). The _reason parameter is unused at runtime but serves as mandatory documentation — every cache-breaking section carries its justification in the source code.

### The 2^N Problem

A comment in prompts.ts explains why conditional sections must go after the boundary:

Each conditional here is a runtime bit that would otherwise multiply the Blake2b prefix hash variants (2^N).

Every boolean condition before the boundary doubles the number of unique global cache entries. Three conditionals create 8 variants; five create 32. The static sections are deliberately unconditional. Compile-time feature flags (resolved by the bundler) are acceptable before the boundary. Runtime checks (is this Haiku? does the user have auto mode?) must go after.

This is the kind of constraint that is invisible until you violate it. A well-intentioned engineer adding a user-setting-gated section before the boundary could silently fragment the global cache and double the fleet's prompt processing costs.

## Streaming

### Raw SSE Over SDK Abstractions

The streaming implementation uses the raw Stream<BetaRawMessageStreamEvent> rather than the SDK's higher-level BetaMessageStream. The reason: BetaMessageStream calls partialParse() on every input_json_delta event. For tool calls with large JSON inputs (file edits with hundreds of lines), this re-parses the growing JSON string from scratch on every chunk — O(n^2) behavior. Claude Code handles tool input accumulation itself, so the partial parsing is pure waste.

### The Idle Watchdog

TCP connections can die without notification. The server may crash, a load balancer may silently drop the connection, or a corporate proxy may time out. The SDK's request timeout only covers the initial fetch — once HTTP 200 arrives, the timeout is satisfied. If the streaming body stops, nothing catches it.

The watchdog: a setTimeout that resets on every received chunk. If no chunks arrive for 90 seconds, the stream is aborted and the system falls back to a non-streaming retry. A warning fires at the 45-second mark. When the watchdog fires, it logs the event with the client request ID for correlation.

### Non-Streaming Fallback

When streaming fails mid-response (network error, stall, truncation), the system falls back to a synchronous messages.create() call. This handles proxy failures where the proxy returns HTTP 200 with a non-SSE body, or truncates the SSE stream partway through.

The fallback can be disabled when streaming tool execution is active, since a fallback would re-execute the entire request and potentially run tools twice.

## Prompt Cache System

### Three Tiers

Prompt caching operates at three levels:

Ephemeral cache (default): Per-session caching with a server-defined TTL (~5 minutes). All users get this.

1-hour TTL: Eligible users get extended caching. Eligibility is determined by subscription status and latched in bootstrap state — the promptCache1hEligible sticky latch from Chapter 3 ensures a mid-session overage flip does not change the TTL.

Global scope: System prompt cache entries get cross-session, cross-organization sharing. The static portions of the prompt are identical for all Claude Code users, so a single cached copy serves everyone. Global scope is disabled when MCP tools are present, because MCP tool definitions are user-specific and would fragment the cache into millions of unique prefixes.

### The Sticky Latches in Action

The five sticky latches from Chapter 3 are evaluated here, during request construction. Each latch starts as null and, once set to true, remains true for the session. The comment above the latch block is precise: "Sticky-on latches for dynamic beta headers. Each header, once first sent, keeps being sent for the rest of the session so mid-session toggles don't change the server-side cache key and bust ~50-70K tokens."

See Chapter 3, Section 3.1 for the full explanation of the latch pattern, the five specific latches, and why always-send-all-headers is not the right solution.

## The queryModel Generator

The queryModel() function is an async generator (~700 lines) that orchestrates the entire API call lifecycle. It yields StreamEvent, AssistantMessage, and SystemAPIErrorMessage objects.

The request assembly follows a carefully ordered sequence:

- Kill switch check — safety valve for the most expensive model tier
- Beta header assembly — model-specific, with sticky latches applied
- Tool schema building — parallel via Promise.all(), deferred tools excluded until discovered
- Message normalization — repair orphaned tool_use/tool_result mismatches, strip excess media, remove stale blocks
- System prompt block construction — split at the dynamic boundary, assign cache scopes
- Retry-wrapped streaming — handles 529 (overloaded), model fallback, thinking downgrade, OAuth refresh

### Output Token Cap

The default output cap is 8,000 tokens, not the typical 32K or 64K. Production data showed that p99 output is 4,911 tokens — standard limits over-reserve by 8-16x. When a response hits the cap (<1% of requests), it gets one clean retry at 64K. This saves significant cost at fleet scale.

### Error Handling and Retry

The withRetry() function is itself an async generator that yields SystemAPIErrorMessage events so the UI can display retry status. Retry strategies:

- 529 (overloaded): Wait and retry, optionally downgrading fast mode
- Model fallback: Primary model fails, try a fallback (e.g., Opus to Sonnet)
- Thinking downgrade: Context window overflow triggers reduced thinking budget
- OAuth 401: Refresh token and retry once

The generator pattern means retry progress ("Server overloaded, retrying in 5s…") appears as a natural part of the event stream, not as a side-channel notification.

## Apply This

Treat prompt caching as an architectural constraint, not a feature toggle. Most LLM applications "turn on" caching. Claude Code treats it as a design constraint that shapes prompt ordering, section memoization, header latching, and configuration management. The difference between a well-structured prompt (cache hit on 50K tokens) and a poorly-structured one (full reprocessing every turn) is the single largest cost lever in the system.

Use the DANGEROUS naming convention for costly escape hatches. When a codebase has an invariant that is easy to violate accidentally, naming the escape hatch with a loud prefix does three things: makes violations visible in code review, forces documentation (the required reason parameter), and creates psychological friction toward the safe default. This generalizes beyond caching to any operation with invisible cost.

Build streaming with a watchdog, not just a timeout. The SDK's request timeout satisfies on HTTP 200, but the response body can stop arriving at any point. A setTimeout that resets on every chunk catches this. The non-streaming fallback handles proxy failure modes (HTTP 200 with non-SSE body, mid-stream truncation) that are more common than you expect in corporate environments.

Make retry strategies yield-based, not exception-based. By making the retry wrapper an async generator that yields status events, the caller displays retry progress as a natural part of the event stream. The model fallback pattern (Opus fails, try Sonnet) is particularly useful for production resilience.

Separate the fast path from the full pipeline. Not every API call needs tool search, advisor integration, thinking budgets, and streaming infrastructure. Claude Code's queryHaiku() function provides a streamlined path for internal operations (compaction, classification) that skips all agentic concerns. A separate function with a simplified interface prevents accidental complexity leakage.

## Looking Ahead

The API layer sits at the foundation of everything that follows. Chapter 5 will show how the query loop uses the streaming response to drive tool execution — including how tools begin executing before the model finishes its response. Chapter 6 will explain how the compaction system preserves cache efficiency when conversations approach the context limit. Chapter 7 will show how each agent thread gets its own message array and request chain.

All of those systems inherit the constraints established here: cache stability as an architectural invariant, provider transparency through the client factory, and session-stable configuration through the latch system. The API layer does not just send requests — it defines the rules by which every other system operates.


---

# Chapter 5: The Agent Loop

## The Beating Heart

Chapter 4 showed how the API layer transforms configuration into streaming HTTP requests — how the client is built, how system prompts are assembled, how responses arrive as server-sent events. That layer handles the mechanics of talking to the model. But a single API call is not an agent. An agent is a loop: call the model, execute tools, feed results back, call the model again, until the work is done.

Every system has a center of gravity. In a database, it is the storage engine. In a compiler, it is the intermediate representation. In Claude Code, it is query.ts — a single 1,730-line file containing the async generator that runs every interaction, from the first keystroke in the REPL to the last tool call of a headless --print invocation.

This is not an exaggeration. There is exactly one code path that talks to the model, executes tools, manages context, recovers from errors, and decides when to stop. That code path is the query() function. The REPL calls it. The SDK calls it. Sub-agents call it. The headless runner calls it. If you are using Claude Code, you are inside query().

The file is dense, but it is not complex in the way that tangled inheritance hierarchies are complex. It is complex in the way that a submarine is complex: a single hull with many redundant systems, each one added because the ocean found a way in. Every if branch has a story. Every withheld error message represents a real bug where an SDK consumer disconnected mid-recovery. Every circuit breaker threshold was tuned against real sessions that burned thousands of API calls in infinite loops.

This chapter traces the entire loop, start to finish. By the end, you will understand not just what happens, but why each mechanism exists and what breaks without it.

## Why an Async Generator

The first architectural question: why is the agent loop a generator and not a callback-based event emitter?

```javascript
// Simplified — shows the concept, not the exact types
async function* agentLoop(params: LoopParams): AsyncGenerator<Message | Event, TerminalReason>
```

The actual signature yields several message and event types and returns a discriminated union encoding why the loop stopped.

Three reasons, in order of importance.

Backpressure. An event emitter fires whether the consumer is ready or not. A generator yields only when the consumer calls .next(). When the REPL's React renderer is busy painting the previous frame, the generator naturally pauses. When an SDK consumer is processing a tool result, the generator waits. No buffer overflow, no dropped messages, no "fast producer / slow consumer" problem.

Return value semantics. The generator's return type is Terminal — a discriminated union encoding exactly why the loop stopped. Was it a normal completion? A user abort? A token budget exhaustion? A stop hook intervention? A max-turns limit? An unrecoverable model error? There are 10 distinct terminal states. Callers do not need to subscribe to an "end" event and hope the payload contains the reason. They get it as a typed return value from for await...of or yield*.

Composability via yield*. The outer query() function delegates to queryLoop() with yield*, which transparently forwards every yielded value and the final return. Sub-generators like handleStopHooks() use the same pattern. This creates a clean chain of responsibility without callbacks, without promises wrapping promises, without event forwarding boilerplate.

The choice has a cost — async generators in JavaScript cannot be "rewound" or forked. But the agent loop does not need either. It is a strictly forward-moving state machine.

One more subtlety: the function* syntax makes the function lazy. The body does not execute until the first .next() call. This means query() returns instantly — all the heavy initialization (config snapshot, memory prefetch, budget tracker) happens only when the consumer starts pulling values. In the REPL, this means the React rendering pipeline is already set up before the first line of the loop runs.

## What Callers Provide

Before tracing the loop, it helps to know what goes in:

```javascript
// Simplified — illustrates the key fields
type LoopParams = {
  messages: Message[]
  prompt: SystemPrompt
  permissionCheck: CanUseToolFn
  context: ToolUseContext
  source: QuerySource // 'repl', 'sdk', 'agent:xyz', 'compact', etc.
  maxTurns?: number
  budget?: { total: number } // API-level task budget
  deps?: LoopDeps // Injected for testing
}
```

The notable fields:

- querySource: A string discriminant like 'repl_main_thread', 'sdk', 'agent:xyz', 'compact', or 'session_memory'. Many conditionals branch on this. The compact agent uses querySource: 'compact' so the blocking limit guard does not deadlock (the compact agent needs to run to reduce the token count).

- taskBudget: The API-level task budget (output_config.task_budget). Distinct from the +500k auto-continue token budget feature. total is the budget for the whole agentic turn; remaining is computed per iteration from cumulative API usage and adjusted across compaction boundaries.

- deps: Optional dependency injection. Defaults to productionDeps(). This is the seam where tests swap in fake model calls, fake compaction, and deterministic UUIDs.

- canUseTool: A function that returns whether a given tool is allowed. This is the permission layer — it checks trust settings, hook decisions, and the current permission mode.

## The Two-Layer Entry Point

The public API is a thin wrapper around the real loop:

The outer function wraps the inner loop, tracking which queued commands were consumed during the turn. After the inner loop completes, consumed commands are marked as 'completed'. If the loop throws or the generator is closed via .return(), the completion notifications never fire — a failed turn should not mark commands as successfully processed. Commands queued during a turn (via / slash commands or task notifications) are marked 'started' inside the loop and 'completed' in the wrapper. If the loop throws or the generator is closed via .return(), the completion notifications never fire. This is intentional — a failed turn should not mark commands as successfully processed.

## The State Object

The loop carries its state in a single typed object:

```javascript
// Simplified — illustrates the key fields
type LoopState = {
  messages: Message[]
  context: ToolUseContext
  turnCount: number
  transition: Continue | undefined
  // ... plus recovery counters, compaction tracking, pending summaries, etc.
}
```

Ten fields. Each one earns its place:

| Field | Why It Exists |
|-------|---------------|
| messages | The conversation history, grown each iteration |
| toolUseContext | Mutable context: tools, abort controller, agent state, options |
| autoCompactTracking | Tracks compaction state: turn counter, turn ID, consecutive failures, compacted flag |
| maxOutputTokensRecoveryCount | How many multi-turn recovery attempts for output token limits (max 3) |
| hasAttemptedReactiveCompact | One-shot guard against infinite reactive compaction loops |
| maxOutputTokensOverride | Set to 64K during escalation, cleared after |
| pendingToolUseSummary | A promise from the previous iteration's Haiku summary, resolved during current streaming |
| stopHookActive | Prevents re-running stop hooks after a blocking retry |
| turnCount | Monotonic counter, checked against maxTurns |
| transition | Why the previous iteration continued — undefined on first iteration |

### Immutable Transitions in a Mutable Loop

Here is the pattern that appears at every continue statement in the loop:

```javascript
const next: State = {
  messages: [...messagesForQuery, ...assistantMessages, ...toolResults],
  toolUseContext: toolUseContextWithQueryTracking,
  autoCompactTracking: tracking,
  turnCount: nextTurnCount,
  maxOutputTokensRecoveryCount: 0,
  hasAttemptedReactiveCompact: false,
  pendingToolUseSummary: nextPendingToolUseSummary,
  maxOutputTokensOverride: undefined,
  stopHookActive,
  transition: { reason: 'next_turn' },
}
state = next
```

Every continue site constructs a complete new State object. Not state.messages = newMessages. Not state.turnCount++. A full reconstruction. The benefit is that every transition is self-documenting. You can read any continue site and see exactly which fields change and which are preserved. The transition field on the new state records why the loop is continuing — tests assert on this to verify that the correct recovery path fired.

## Context Management: Four Compression Layers

Before each API call, the message history passes through up to four context management stages. They run in a specific order, and that order matters.

### Layer 0: Tool Result Budget

Before any compression, applyToolResultBudget() enforces per-message size limits on tool results. Tools without a finite maxResultSizeChars are exempted.

### Layer 1: Snip Compact

The lightest operation. Snip physically removes old messages from the array, yielding a boundary message to signal the removal to the UI. It reports how many tokens were freed, and that number is plumbed into auto-compact's threshold check.

### Layer 2: Microcompact

Microcompact removes tool results that are no longer needed, identified by tool_use_id. For cached microcompact (which edits the API cache), the boundary message is deferred until after the API response. The reason: client-side token estimates are unreliable. The actual cache_deleted_input_tokens from the API response tells you what was really freed.

### Layer 3: Context Collapse

Context collapse replaces spans of conversation with summaries. It runs before auto-compact, and the ordering is deliberate: if collapse reduces the context below the auto-compact threshold, auto-compact becomes a no-op. This preserves granular context instead of replacing everything with a single monolithic summary.

### Layer 4: Auto-Compact

The heaviest operation: it forks an entire Claude conversation to summarize the history. The implementation has a circuit breaker — after 3 consecutive failures, it stops trying. This prevents the nightmare scenario observed in production: sessions stuck over the context limit burning 250K API calls per day in an infinite compact-fail-retry loop.

### Auto-Compact Thresholds

The thresholds are derived from the model's context window:

```
effectiveContextWindow = contextWindow - min(modelMaxOutput, 20000)
```

Thresholds (relative to effectiveContextWindow):
- Auto-compact fires: effectiveWindow - 13,000
- Blocking limit (hard): effectiveWindow - 3,000

| Constant | Value | Purpose |
|----------|-------|---------|
| AUTOCOMPACT_BUFFER_TOKENS | 13,000 | Headroom below effective window for auto-compact trigger |
| MANUAL_COMPACT_BUFFER_TOKENS | 3,000 | Reserves space so /compact still works |
| MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES | 3 | Circuit breaker threshold |

The 13,000-token buffer means auto-compact fires well before the hard limit. The gap between the auto-compact threshold and the blocking limit is where reactive compact operates — if the proactive auto-compact fails or is disabled, reactive compact catches the 413 error and compacts on demand.

## Model Streaming

### The callModel() Loop

The API call happens inside a while(attemptWithFallback) loop that enables model fallback:

```javascript
let attemptWithFallback = true
while (attemptWithFallback) {
  attemptWithFallback = false
  try {
    for await (const message of deps.callModel({ messages, systemPrompt, tools, signal })) {
      // Process each streamed message
    }
  } catch (innerError) {
    if (innerError instanceof FallbackTriggeredError && fallbackModel) {
      currentModel = fallbackModel
      attemptWithFallback = true
      continue
    }
    throw innerError
  }
}
```

When enabled, a StreamingToolExecutor starts executing tools as soon as their tool_use blocks arrive during streaming — not after the full response completes. How tools are orchestrated into concurrent batches is the subject of Chapter 7.

### The Withholding Pattern

This is one of the most important patterns in the file. Recoverable errors are suppressed from the yield stream:

```javascript
let withheld = false
if (contextCollapse?.isWithheldPromptTooLong(message)) withheld = true
if (reactiveCompact?.isWithheldPromptTooLong(message)) withheld = true
if (isWithheldMaxOutputTokens(message)) withheld = true
if (!withheld) yield yieldMessage
```

Why withhold? Because SDK consumers — Cowork, the desktop app — terminate the session on any message with an error field. If you yield a prompt-too-long error and then successfully recover via reactive compaction, the consumer has already disconnected. The recovery loop keeps running, but nobody is listening. So the error is withheld, pushed to assistantMessages so downstream recovery checks can find it. If all recovery paths fail, the withheld message is finally surfaced.

## Error Recovery: The Escalation Ladder

Error recovery in query.ts is not a single strategy. It is a ladder of increasingly aggressive interventions, each triggered when the previous one fails.

### The Death Spiral Guard

The most dangerous failure mode is an infinite loop. The code has multiple guards:

- hasAttemptedReactiveCompact: One-shot flag. Reactive compact fires once per error type.
- MAX_OUTPUT_TOKENS_RECOVERY_LIMIT = 3: Hard cap on multi-turn recovery attempts.
- Circuit breaker on auto-compact: After 3 consecutive failures, auto-compact stops trying entirely.
- No stop hooks on error responses: The code explicitly returns before reaching stop hooks when the last message is an API error. The comment explains: "error -> hook blocking -> retry -> error -> … (the hook injects more tokens each cycle)."
- Preserved hasAttemptedReactiveCompact across stop hook retries: When a stop hook returns blocking errors and forces a retry, the reactive compact guard is preserved. The comment documents the bug: "Resetting to false here caused an infinite loop burning thousands of API calls."

Each of these guards was added because someone hit the failure mode in production.

## Token Budgets

Users can request a token budget for a turn (e.g., +500k). The budget system decides whether to continue or stop after the model completes a response.

checkTokenBudget makes a binary continue/stop decision with three rules:

- Subagents always stop. Budget is a top-level concept only.
- Completion threshold at 90%. If turnTokens < total * 0.9, continue.
- When the decision is "continue," a nudge message is injected telling the model how much budget remains.

## Stop Hooks: Forcing the Model to Keep Working

Stop hooks run when the model finishes without requesting any tool use — it thinks it is done. The hooks evaluate whether it actually is done.

The pipeline runs template job classification, fires background tasks (prompt suggestion, memory extraction), and then executes stop hooks proper. When a stop hook returns blocking errors — "you said you were done, but the linter found 3 errors" — the errors are appended to the message history and the loop continues with stopHookActive: true. This flag prevents re-running the same hooks on the retry.

When a stop hook signals preventContinuation, the loop exits immediately with { reason: 'stop_hook_prevented' }.

## State Transitions: The Complete Catalog

Every exit from the loop is one of two types: a Terminal (the loop returns) or a Continue (the loop iterates).

### Terminal States (10 reasons)

| Reason | Trigger |
|--------|---------|
| blocking_limit | Token count at hard limit, auto-compact OFF |
| image_error | ImageSizeError, ImageResizeError, or unrecoverable media error |
| model_error | Unrecoverable API/model exception |
| aborted_streaming | User abort during model streaming |
| prompt_too_long | Withheld 413 after all recovery exhausted |
| completed | Normal completion (no tool use, or budget exhausted, or API error) |
| stop_hook_prevented | Stop hook explicitly blocked continuation |
| aborted_tools | User abort during tool execution |
| hook_stopped | PreToolUse hook stopped continuation |
| max_turns | Hit the maxTurns limit |

### Continue States (7 reasons)

| Reason | Trigger |
|--------|---------|
| collapse_drain_retry | Context collapse drained staged collapses on 413 |
| reactive_compact_retry | Reactive compact succeeded after 413 or media error |
| max_output_tokens_escalate | 8K cap hit, escalating to 64K |
| max_output_tokens_recovery | 64K still hit, multi-turn recovery (up to 3 attempts) |
| stop_hook_blocking | Stop hook returned blocking errors, must retry |
| token_budget_continuation | Token budget not exhausted, nudge message injected |
| next_turn | Normal tool-use continuation |

## Apply This: Building Your Own Agent Loop

Use a generator, not callbacks. The backpressure is free. The return value semantics are free. The composability via yield* is free. Agent loops are strictly forward-moving — you never need to rewind or fork.

Make state transitions explicit. Reconstruct the full state object at every continue site. The verbosity is the feature — it prevents partial-update bugs and makes each transition self-documenting.

Withhold recoverable errors. If your consumers disconnect on errors, do not yield errors until you know recovery has failed. Push them to an internal buffer, attempt recovery, and surface only on exhaustion.

Layer your context management. Light operations first (removal), heavy operations last (summarization). This preserves granular context when possible and falls back to monolithic summaries only when necessary.

Add circuit breakers for every retry. Every recovery mechanism in query.ts has an explicit limit: 3 auto-compact failures, 3 max-output recovery attempts, 1 reactive compact attempt. Without these limits, the first production session that triggers a retry-on-failure loop will burn your API budget overnight.

The minimal agent loop skeleton, if you are starting from scratch:

```javascript
async function* agentLoop(params) {
  let state = initState(params)
  while (true) {
    const context = compressIfNeeded(state.messages)
    const response = await callModel(context)
    if (response.error) {
      if (canRecover(response.error, state)) { state = recoverState(state); continue }
      return { reason: 'error' }
    }
    if (!response.toolCalls.length) return { reason: 'completed' }
    const results = await executeTools(response.toolCalls)
    state = { ...state, messages: [...context, response.message, ...results] }
  }
}
```

Every feature in Claude Code's loop is an elaboration of one of these steps. The four compression layers elaborate step 3 (compress). The withholding pattern elaborates the model call. The escalation ladder elaborates error recovery. Stop hooks elaborate the "no tool use" exit. Start with this skeleton. Add each elaboration only when you hit the problem it solves.

## Summary

The agent loop is 1,730 lines of a single while(true) that does everything. It streams model responses, executes tools concurrently, compresses context through four layers, recovers from five categories of errors, tracks token budgets with diminishing returns detection, runs stop hooks that can force the model back to work, manages prefetch pipelines for memory and skills, and produces a typed discriminated union of exactly why it stopped.

It is the most important file in the system because it is the only file that touches every other subsystem. The context pipeline feeds into it. The tool system feeds out of it. The error recovery wraps around it. The hooks intercept it. The state layer persists through it. The UI renders from it.

If you understand query(), you understand Claude Code. Everything else is a peripheral.


---

# Chapter 6: Tools — From Definition to Execution

## The Nervous System

Chapter 5 showed you the agent loop — the while(true) that streams model responses, collects tool calls, and feeds results back. The loop is the heartbeat. But the heartbeat is meaningless without the nervous system that translates "the model wants to run git status" into an actual shell command, with permission checks, result budgeting, and error handling.

The tool system is that nervous system. It spans 40+ tool implementations, a centralized registry with feature-flag gating, a 14-step execution pipeline, a permission resolver with seven modes, and a streaming executor that starts tools before the model finishes its response.

Every tool call in Claude Code — every file read, every shell command, every grep, every sub-agent dispatch — flows through the same pipeline. The uniformity is the point: whether the tool is a built-in Bash executor or a third-party MCP server, it gets the same validation, the same permission checks, the same result budgeting, the same error classification.

The Tool interface has approximately 45 members. That sounds overwhelming, but only five matter for understanding how the system works:

- call() — execute the tool
- inputSchema — validate and parse the input
- isConcurrencySafe() — can this run in parallel?
- checkPermissions() — is this allowed?
- validateInput() — does this input make semantic sense?

Everything else — the 12 rendering methods, the analytics hooks, the search hints — exists to support the UI and telemetry layers. Start with the five, and the rest falls into place.

## The Tool Interface

### Three Type Parameters

Every tool is parameterized over three types:

Tool<Input extends AnyObject, Output, P extends ToolProgressData>

Input is a Zod object schema that serves double duty: it generates the JSON Schema sent to the API (so the model knows what parameters to provide), and it validates the model's response at runtime via safeParse. Output is the TypeScript type of the tool's result. P is the progress event type the tool emits while running — BashTool emits stdout chunks, GrepTool emits match counts, AgentTool emits sub-agent transcripts.

### buildTool() and Fail-Closed Defaults

No tool definition directly constructs a Tool object. Every tool passes through buildTool(), a factory that spreads a defaults object under the tool-specific definition:

```javascript
// Pseudocode — illustrates the fail-closed defaults pattern
const SAFE_DEFAULTS = {
  isEnabled: () => true,
  isParallelSafe: () => false, // Fail-closed: new tools run serially
  isReadOnly: () => false, // Fail-closed: treated as writes
  isDestructive: () => false,
  checkPermissions: (input) => ({ behavior: 'allow', updatedInput: input }),
}

function buildTool(definition) {
  return { ...SAFE_DEFAULTS, ...definition } // Definition overrides defaults
}
```

The defaults are deliberately fail-closed where it matters for safety. A new tool that forgets to implement isConcurrencySafe defaults to false — it runs serially, never in parallel. A tool that forgets isReadOnly defaults to false — the system treats it as a write operation. A tool that forgets toAutoClassifierInput returns an empty string — the auto-mode security classifier skips it, which means the general permission system handles it instead of an automated bypass.

### Concurrency Is Input-Dependent

The signature isConcurrencySafe(input: z.infer<Input>): boolean takes the parsed input because the same tool can be safe for some inputs and unsafe for others. BashTool is the canonical example: ls -la is read-only and concurrency-safe, but rm -rf /tmp/build is not. The tool parses the command, classifies each subcommand against known-safe sets, and returns true only when every non-neutral part is a search or read operation.

### The ToolResult Return Type

Every call() returns a ToolResult<T>:

```typescript
type ToolResult<T> = {
  data: T
  newMessages?: (UserMessage | AssistantMessage | AttachmentMessage | SystemMessage)[]
  contextModifier?: (context: ToolUseContext) => ToolUseContext
}
```

data is the typed output that gets serialized into the API's tool_result content block. newMessages lets a tool inject additional messages into the conversation — AgentTool uses this to append sub-agent transcripts. contextModifier is a function that mutates the ToolUseContext for subsequent tools — this is how EnterPlanMode switches the permission mode. Context modifiers are only honored for non-concurrency-safe tools; if your tool runs in parallel, its modifier is queued until the batch completes.

## ToolUseContext: The God Object

ToolUseContext is the massive context bag threaded through every tool call. It has approximately 40 fields. It is, by any reasonable definition, a god object. It exists because the alternative is worse.

A tool like BashTool needs the abort controller, the file state cache, the app state, the message history, the tool set, MCP connections, and half a dozen UI callbacks. Threading these as individual parameters would produce function signatures with 15+ arguments. The pragmatic solution is a single context object, grouped by concern:

Configuration (options sub-object): The tool set, model name, MCP connections, debug flags. Set once at query start, mostly immutable.

Execution state: The abortController for cancellation, readFileState for the LRU file cache, messages for the full conversation history. These change during execution.

UI callbacks: setToolJSX, addNotification, requestPrompt. Only wired in interactive (REPL) contexts. SDK and headless modes leave them undefined.

Agent context: agentId, renderedSystemPrompt (frozen parent prompt for fork sub-agents — re-rendering could diverge due to feature flag warm-up and bust the cache).

## The Registry

### getAllBaseTools(): The Single Source of Truth

The function getAllBaseTools() returns the exhaustive list of every tool that could exist in the current process. Always-present tools come first, then conditionally-included tools gated by feature flags.

### assembleToolPool(): Merging Built-in and MCP Tools

The final tool set that reaches the model comes from assembleToolPool():

- Get built-in tools (with deny-rule filtering, REPL mode hiding, and isEnabled() checks)
- Filter MCP tools by deny rules
- Sort each partition alphabetically by name
- Concatenate built-ins (prefix) + MCP tools (suffix)

The sort-then-concatenate approach is not aesthetic preference. The API server places a prompt-cache breakpoint after the last built-in tool. A flat sort across all tools would interleave MCP tools into the built-in list, and adding or removing an MCP tool would shift built-in tool positions, invalidating the cache.

## The 14-Step Execution Pipeline

The function checkPermissionsAndCallTool() is where intent becomes action. Every tool call passes through these 14 steps.

### Steps 1-4: Validation

Tool Lookup falls back to getAllBaseTools() for alias matches, handling transcripts from older sessions where a tool was renamed. Abort Check prevents wasted computation on tool calls queued before Ctrl+C propagated. Zod Validation catches type mismatches; for deferred tools, the error appends a hint to call ToolSearch first. Semantic Validation goes beyond schema conformance — FileEditTool rejects no-op edits, BashTool blocks standalone sleep when MonitorTool is available.

### Steps 5-6: Preparation

Speculative Classifier Start kicks off the auto-mode security classifier in parallel for Bash commands, shaving hundreds of milliseconds off the common path. Input Backfill clones the parsed input and adds derived fields (expanding ~/foo.txt to absolute paths) for hooks and permissions, preserving the original for transcript stability.

### Steps 7-9: Permission

PreToolUse Hooks are the extension mechanism — they can make permission decisions, modify inputs, inject context, or stop execution entirely. Permission Resolution bridges hooks and the general permission system: if a hook already decided, that is final; otherwise canUseTool() triggers rule matching, tool-specific checks, mode-based defaults, and interactive prompts. Permission Denied Handling builds an error message and executes PermissionDenied hooks.

### Steps 10-14: Execution and Cleanup

Tool Execution runs the actual call() with the original input. Result Budgeting persists oversized output to ~/.claude/tool-results/{hash}.txt and replaces it with a preview. PostToolUse Hooks can modify MCP output or block continuation. New Messages are appended (sub-agent transcripts, system reminders). Error Handling classifies errors for telemetry, extracts safe strings from potentially mangled names, and emits OTel events.

## The Permission System

### Seven Modes

| Mode | Behavior |
|------|----------|
| default | Tool-specific checks; prompt user for unrecognized operations |
| acceptEdits | Auto-allow file edits; prompt for other operations |
| plan | Read-only — deny all write operations |
| dontAsk | Auto-deny anything that would normally prompt (background agents) |
| bypassPermissions | Allow everything without prompting |
| auto | Use the transcript classifier to decide (feature-flagged) |
| bubble | Internal mode for sub-agents that escalate to the parent |

### The Resolution Chain

When a tool call reaches permission resolution:

1. Hook decision: If a PreToolUse hook already returned allow or deny, that is final.
2. Rule matching: Three rule sets — alwaysAllowRules, alwaysDenyRules, alwaysAskRules — match on tool name and optional content patterns. Bash(git *) matches any Bash command starting with git.
3. Tool-specific check: The tool's checkPermissions() method. Most return passthrough.
4. Mode-based default: bypassPermissions allows everything. plan denies writes. dontAsk denies prompts.
5. Interactive prompt: In default and acceptEdits modes, unresolved decisions show a prompt.
6. Auto-mode classifier: A two-stage classifier (fast model, then extended thinking for ambiguous cases).

### Permission Rules and Matching

Permission rules are stored as PermissionRule objects with three parts: a source tracing provenance (userSettings, projectSettings, localSettings, cliArg, policySettings, session, etc.), a ruleBehavior (allow, deny, ask), and a ruleValue with the tool name and optional content pattern.

The ruleContent field enables fine-grained matching. Bash(git *) allows any Bash command starting with git. Edit(/src/**) allows edits only within /src. Fetch(domain:example.com) allows fetching from a specific domain. Rules without ruleContent match all invocations of that tool.

### Bubble Mode for Sub-Agents

Sub-agents in coordinator-worker patterns cannot show permission prompts — they have no terminal. The bubble mode causes permission requests to propagate up to the parent context. The coordinator agent, running in the main thread with terminal access, handles the prompt and sends the decision back down.

## Individual Tool Highlights

### BashTool: The Most Complex Tool

BashTool is the system's most complex tool by far. It parses compound commands, classifies subcommands as read-only or write, manages background tasks, detects image output by magic bytes, and implements a sed simulation for safe edit previews.

The compound command parsing is particularly interesting. splitCommandWithOperators() breaks a command like cd /tmp && mkdir build && ls build into individual subcommands. Each is classified against known-safe command sets (BASH_SEARCH_COMMANDS, BASH_READ_COMMANDS, BASH_LIST_COMMANDS). A compound command is read-only only if ALL non-neutral parts are safe. The neutral set (echo, printf) is ignored — they do not make a command read-only, but they also do not make it write-only.

The sed simulation (_simulatedSedEdit) deserves special attention. When a user approves a sed command in the permission dialog, the system pre-computes the result by running the sed command in a sandbox and capturing the output. The pre-computed result is injected into the input as _simulatedSedEdit. When call() executes, it applies the edit directly, bypassing shell execution. This guarantees that what the user previewed is exactly what gets written — not a re-execution that might produce different results if the file changed between preview and execution.

### FileEditTool: Staleness Detection

FileEditTool integrates with readFileState, the LRU cache of file contents and timestamps maintained across the conversation. Before applying an edit, it checks whether the file has been modified since the model last read it. If the file is stale — modified by a background process, another tool, or the user — the edit is rejected with a message telling the model to re-read the file first.

### FileReadTool: The Versatile Reader

FileReadTool is the only built-in tool with maxResultSizeChars: Infinity. If Read output were persisted to disk, the model would need to Read the persisted file, which could itself exceed the limit, creating an infinite loop. The tool instead self-bounds via token estimation and truncates at the source.

The tool is remarkably versatile: it reads text files with line numbers, images (returning base64 multimodal content blocks), PDFs (via extractPDFPages()), Jupyter notebooks (via readNotebook()), and directories (falling back to ls). It blocks dangerous device paths (/dev/zero, /dev/random, /dev/stdin) and handles macOS screenshot filename quirks.

## Result Budgeting

### Per-Tool Size Limits

Each tool declares maxResultSizeChars:

| Tool | maxResultSizeChars | Rationale |
|------|-------------------|-----------|
| BashTool | 30,000 | Enough for most useful output |
| FileEditTool | 100,000 | Diffs can be large but the model needs them |
| GrepTool | 100,000 | Search results with context lines add up fast |
| FileReadTool | Infinity | Self-bounds via its own token limits |

When a result exceeds the threshold, the full content is saved to disk and replaced with a preview. The model can then use Read to access the full output if needed.

## Apply This: Designing a Tool System

Fail-closed defaults. New tools should be conservative until explicitly marked otherwise. A developer who forgets to set a flag gets the safe behavior, not the dangerous one.

Input-dependent safety. isConcurrencySafe(input) and isReadOnly(input) take the parsed input because the same tool at different inputs has different safety profiles. A tool registry that marks BashTool as "always serial" is correct but wasteful.

Layer your permissions. Tool-specific checks, rule-based matching, mode-based defaults, interactive prompts, and automated classifiers each handle different cases. No single mechanism is sufficient.

Budget results, not just inputs. Token limits on input are standard. But tool results can be arbitrarily large and they accumulate across turns. Per-tool limits prevent individual explosions. Aggregate conversation limits prevent cumulative overflow.

Make error classification telemetry-safe. In minified builds, error.constructor.name is mangled. The classifyToolError() function extracts the most informative safe string available — telemetry-safe messages, errno codes, stable error names — without ever logging the raw error message to analytics.

## What Comes Next

This chapter traced how a single tool call flows from definition through validation, permission, execution, and result budgeting. But the model rarely requests just one tool at a time. How tools are orchestrated into concurrent batches is the subject of Chapter 7.


---

# Chapter 7: Concurrent Tool Execution

## The Cost of Waiting

Chapter 6 traced the lifecycle of a single tool call — from the raw tool_use block in the API response through input validation, permission checks, execution, and result formatting. That pipeline handles one tool. But the model rarely requests just one.

A typical Claude Code interaction involves three to five tool calls per turn. "Read these two files, grep for this pattern, then edit this function." The model emits all of those in a single response. If each tool takes 200 milliseconds, running them sequentially costs a full second. If the Read and Grep calls are independent — and they are — running them in parallel cuts that to 200 milliseconds. Five-to-one improvement, free.

But not all tools are independent. An Edit that modifies config.ts cannot run concurrently with another Edit that modifies config.ts. A Bash command that creates a directory must complete before a Bash command that writes a file into that directory. Concurrency is not a global property of a tool. It is a property of a specific tool invocation with specific inputs.

This is the insight that drives the entire concurrency system: safety is per-call, not per-tool-type. Bash("ls -la") is safe to parallelize. Bash("rm -rf build/") is not. The same tool, different inputs, different concurrency classification. The system must inspect the input before deciding.

Claude Code implements two layers of concurrency optimization. The first is batch orchestration: after the model's response is fully received, partition the tool calls into concurrent and serial groups, then execute each group appropriately. The second is speculative execution: start running tools while the model is still streaming its response, harvesting results before the response is even complete. Together, these two mechanisms eliminate most of the wall-clock time that would otherwise be spent waiting.

## The Partition Algorithm

The entry point is partitionToolCalls() in toolOrchestration.ts. It takes an ordered array of ToolUseBlock messages and produces an array of batches, where each batch is either "all concurrent-safe" or "a single serial tool."

```javascript
// Pseudocode — illustrates the partition algorithm
type Group = { parallel: boolean; calls: ToolCall[] }

function groupBySafety(calls, registry) {
  return calls.reduce((groups, call) => {
    const def = registry.lookup(call.name)
    const input = def?.schema.safeParse(call.input)
    const safe = input?.success
      ? tryCatch(() => def.isParallelSafe(input.data), false)
      : false
    if (safe && groups.at(-1)?.parallel) {
      groups.at(-1).calls.push(call)
    } else {
      groups.push({ parallel: safe, calls: [call] })
    }
    return groups
  }, [])
}
```

The algorithm walks the array left to right. For each tool call:

- Look up the tool definition by name.
- Parse the input with the tool's Zod schema via safeParse(). If parsing fails, the tool is conservatively classified as not concurrency-safe.
- Call isConcurrencySafe(parsedInput) on the tool definition. This is where per-input classification happens.
- Merge or create a batch. If the current tool is concurrency-safe AND the most recent batch is also concurrency-safe, append to that batch. Otherwise, start a new batch.

The result is a sequence of batches that alternates between concurrent groups and individual serial entries.

## Batch Execution

### Concurrent Batches

For a concurrent batch, runToolsConcurrently() fires all tools in parallel using an all() utility that caps active generators at the concurrency limit:

```javascript
async function* dispatchParallel(calls, context) {
  yield* boundedAll(
    calls.map(async function* (call) {
      context.markInProgress(call.id)
      yield* executeSingle(call, context)
      context.markComplete(call.id)
    }),
    MAX_CONCURRENCY, // Default: 10
  )
}
```

The concurrency limit defaults to 10. Context modifier queuing is the subtle part. Some tools produce context modifiers — functions that transform the ToolUseContext for subsequent tools. When tools run concurrently, you cannot apply these modifiers immediately. Instead, modifiers are collected and applied after the entire concurrent batch finishes, in tool-order (not completion-order), preserving deterministic context evolution.

### Serial Batches

Serial execution is straightforward. Each tool runs, its context modifiers are applied immediately, and the next tool sees the updated context. This is the critical difference. Serial tools can change the world for subsequent tools.

## The Streaming Tool Executor

Batch orchestration eliminates unnecessary serialization after the model's response arrives. But there is a bigger opportunity: the model's response takes time to stream. A typical multi-tool response might take 2-3 seconds to fully arrive. The first tool call is parseable after 500 milliseconds. Why wait for the remaining 2 seconds?

The StreamingToolExecutor class implements speculative execution. As the model streams its response, each tool_use block is handed to the executor the moment it is fully parsed. The executor starts running it immediately — while the model is still generating the next tool call. By the time the response finishes streaming, several tools may have already completed.

### Tool Lifecycle

Each tool tracked by the executor progresses through four states:

- queued: The tool_use block has been parsed and registered. Waiting for concurrency conditions.
- executing: The tool's call() function is running. Results accumulate in a buffer.
- completed: Execution finished. Results are ready to be yielded.
- yielded: Results have been emitted. Terminal state.

### The Admission Check

A tool can start executing if and only if:

- No tools are currently executing (the queue is empty), OR
- Both the new tool and all currently executing tools are concurrency-safe.

This is a mutual exclusion contract. A non-concurrent tool requires exclusive access — nothing else can be running. Concurrent tools can share the runway with other concurrent tools.

### Order Preservation

Results are yielded in the order tools were received, not the order they completed. This is a deliberate design choice. The model emitted them in a-b-c order. The conversation history must match the model's expectation, or the next turn will be confused about which result corresponds to which request.

## Tool Concurrency Properties

| Tool | Concurrency Safe | Condition | Rationale |
|------|-----------------|-----------|-----------|
| Read | Always | — | Pure read. No side effects. |
| Grep | Always | — | Pure read. Wraps ripgrep. |
| Glob | Always | — | Pure read. File listing. |
| Fetch | Always | — | HTTP GET. No local side effects. |
| WebSearch | Always | — | API call to search provider. |
| Bash | Sometimes | Read-only commands only | isReadOnly() parses the command. |
| Edit | Never | — | Modifies files. Two concurrent edits corrupt. |
| Write | Never | — | Creates or overwrites files. |

## Apply This

Partition by safety, not by type. The isConcurrencySafe(input) method receives the parsed input, not just the tool name. This per-invocation classification is more precise than a static "this tool type is always safe" declaration.

Speculative execution during I/O waits. The streaming executor starts tools while the API response is still arriving. The same pattern applies anywhere you have a slow producer and fast consumers.

Preserve submission order in results. Buffer completed results and release them in the order they were requested. The implementation cost is a simple array walk; the correctness benefit is absolute.


---

# Chapter 8: Spawning Sub-Agents

## The Multiplication of Intelligence

A single agent is powerful. It can read files, edit code, run tests, search the web, and reason about the results. But there is a hard ceiling on what one agent can do in a single conversation: the context window fills up, the task branches in directions that demand different capabilities, and the serial nature of tool execution becomes a bottleneck. The solution is not a bigger model. It is more agents.

Claude Code's sub-agent system lets the model request help. When the parent agent encounters a task that would benefit from delegation — a codebase search that should not pollute the main conversation, a verification pass that demands adversarial thinking, a set of independent edits that could run in parallel — it calls the Agent tool. That call spawns a child: a fully independent agent with its own conversation loop, its own tool set, its own permission boundary, and its own abort controller. The child does its work and returns a result. The parent never sees the child's internal reasoning, only the final output.

This chapter traces the path from the model's "I need help" to a fully operational child agent. We will examine the tool definition that the model sees, the fifteen-step lifecycle that creates the execution environment, the six built-in agent types and what each optimizes for, the frontmatter system that lets users define custom agents, and the design principles that emerge from all of it.

## The AgentTool Definition

The AgentTool is registered under the name "Agent" with a legacy alias "Task" for backward compatibility. It is built with the standard buildTool() factory, but its schema is more dynamic than any other tool in the system.

### The Input Schema

The input schema is constructed lazily via lazySchema(). There are two layers: a base schema and a full schema that adds multi-agent and isolation parameters.

The base fields are always present:

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| description | string | Yes | Short 3-5 word summary of the task |
| prompt | string | Yes | The full task description for the agent |
| subagent_type | string | No | Which specialized agent to use |
| model | enum('sonnet','opus','haiku') | No | Model override for this agent |
| run_in_background | boolean | No | Launch asynchronously |

The full schema adds multi-agent parameters (when swarm features are active) and isolation controls.

### The call() Decision Tree

Before runAgent() is ever invoked, the call() method routes the request through a decision tree:

1. Is this a teammate spawn? (team_name + name both set) → spawnTeammate()
2. Resolve effective agent type from subagent_type
3. Is this the fork path? (effectiveType === undefined) → Recursive fork guard check
4. Resolve agent definition from activeAgents list
5. Check required MCP servers (wait up to 30s for pending)
6. Resolve isolation mode (param overrides agent def)
7. Determine sync vs async
8. Assemble worker tool pool
9. Build system prompt and prompt messages
10. Execute (async → registerAsyncAgent; sync → iterate runAgent)

## The runAgent Lifecycle

runAgent() in runAgent.ts is an async generator that drives a sub-agent's entire lifecycle. It yields Message objects as the agent works. Every sub-agent — fork, built-in, custom, coordinator worker — flows through this single function. The function is approximately 400 lines.

Here are the fifteen steps:

### Step 1: Model Resolution

The resolution chain is: caller override > agent definition > parent model > default. The getAgentModel() function handles special values like 'inherit' (use whatever the parent uses).

### Step 2: Agent ID Creation

Agent IDs follow the pattern agent-<hex> where the hex part is derived from crypto.randomUUID().

### Step 3: Context Preparation

Fork agents and fresh agents diverge here. Fork agents get the parent's conversation history cloned. Fresh agents start with an empty context.

### Step 4: CLAUDE.md Stripping

Read-only agents like Explore and Plan have omitClaudeMd: true. CLAUDE.md files contain project-specific instructions about commit messages, PR conventions, lint rules, and coding standards. A read-only search agent does not need any of this.

### Step 5: Permission Isolation

Each agent gets a custom getAppState() wrapper. Four distinct concerns are layered together:

- Permission mode cascade: If the parent is in bypassPermissions, acceptEdits, or auto mode, the parent's mode always wins.
- Prompt avoidance: Background agents cannot show permission dialogs.
- Automated check ordering: Background agents that can show prompts set awaitAutomatedChecksBeforeDialog.
- Tool permission scoping: When allowedTools is provided, it replaces the session-level allow rules entirely.

### Step 6: Tool Resolution

Fork agents use useExactTools: true, passing the parent's tool array through unchanged for cache-identical tool definitions. For normal agents, resolveAgentTools() applies a layered filter.

### Step 7: System Prompt

Fork agents receive the parent's pre-rendered system prompt via override.systemPrompt. For normal agents, getAgentSystemPrompt() calls the agent definition's getSystemPrompt() function.

### Step 8: Abort Controller Isolation

Three behaviors:
- Override: Used when resuming a backgrounded agent.
- Async agents get a new, unlinked controller. Escape kills the parent but not the background agent.
- Sync agents share the parent's controller. Escape kills both.

### Step 9: Hook Registration

Agent definitions can declare their own hooks in frontmatter. These hooks are scoped to the agent's lifecycle via the agentId.

### Step 10: Skill Preloading

Agent definitions can specify skills in their frontmatter. Loaded skills become user messages prepended to the agent's conversation.

### Step 11: MCP Initialization

Agents can define their own MCP servers in frontmatter, additive to the parent's clients.

### Step 12: Context Creation

createSubagentContext() assembles the new ToolUseContext with deliberate isolation decisions.

### Step 13: Cache-Safe Params Callback

This callback is consumed by background summarization, enabling periodic progress summaries without disturbing the main conversation.

### Step 14: The Query Loop

The same query() function from Chapter 5 drives the sub-agent's conversation. Each yielded message is recorded to a sidechain transcript.

### Step 15: Cleanup

The finally block runs on normal completion, abort, or error. Every subsystem the agent touched gets cleaned up: MCP connections, hooks, cache tracking, file state, perfetto tracing, todo entries, and orphaned shell processes.

## Built-In Agent Types

Six built-in agents ship with the system:

### General-Purpose

The default agent. Full tool access, no CLAUDE.md omission. This is the workhorse.

### Explore

A read-only search specialist. Uses Haiku (the cheapest, fastest model). Omits CLAUDE.md and git status. Has FileEdit, FileWrite, NotebookEdit, and Agent removed from its tool pool. The most aggressively optimized built-in — 34 million times per week across the fleet.

### Plan

A software architect agent. Same read-only tool set as Explore but uses 'inherit' for its model. Its system prompt guides it through a structured four-step process: Understand Requirements, Explore Thoroughly, Design Solution, Detail the Plan.

### Verification

The adversarial tester. Read-only tools, 'inherit' model, always runs in background. Its system prompt explicitly lists excuses the model might reach for and instructs it to "recognize them and do the opposite."

### Claude Code Guide

A documentation-fetching agent for questions about Claude Code itself. Uses Haiku, runs with dontAsk permission mode.

### Statusline Setup

A specialized agent for configuring the terminal status line. Uses Sonnet, limited to Read and Edit tools only.

## Agent Definitions from Frontmatter

Users and plugins can define custom agents by placing markdown files in .claude/agents/. The frontmatter schema supports the full range of agent configuration. The markdown body becomes the agent's system prompt.

Four sources of agent definitions exist, in priority order:
1. Built-in agents — hardcoded in TypeScript
2. User agents — markdown files in .claude/agents/
3. Plugin agents — loaded via loadPluginAgents()
4. Policy agents — loaded via organizational policy settings

## Apply This: Designing Agent Types

The built-in agents demonstrate a pattern language for agent design across five dimensions:

1. **What Can It See?** — Context is not free. Every token costs money and displaces working memory.
2. **What Can It Do?** — Tool access determines capability boundaries.
3. **How Does It Decide?** — Model choice and system prompt shape reasoning quality.
4. **How Does It Report?** — Output format determines parent's ability to synthesize.
5. **How Does It Persist?** — Lifecycle management determines resource efficiency.


---

# Chapter 9: Fork Agents and the Prompt Cache

## The Ninety-Five Percent Insight

When a parent agent spawns five child agents in parallel, the overwhelming majority of each child's API request is identical. The system prompt is the same. The tool definitions are the same. The conversation history is the same. The only thing that differs is the final directive.

On a typical fork with a warm conversation, the shared prefix might be 80,000 tokens. The per-child directive might be 200 tokens. That is 99.75% overlap. Anthropic's prompt cache gives a 90% discount on cached input tokens. If you can make those 80,000 tokens hit the cache for children 2 through 5, you just cut the input cost of those four requests by 90%.

The catch is that prompt caching is byte-exact. Not "similar enough." The bytes must match, character for character. One extra space, one reordered tool definition, one stale feature flag — and the cache misses.

Fork agents are Claude Code's answer to this constraint. Every design decision in the fork system traces back to one question: how do we guarantee byte-identical prefixes across parallel children?

## What a Fork Child Inherits

A fork agent inherits four things from its parent, by reference or byte-exact copy:

1. **The system prompt** — threaded, not regenerated. The parent's already-rendered system prompt bytes are passed via override.systemPrompt.
2. **The tool definitions** — the useExactTools flag passes the parent's assembled tool array directly. No filtering, no reordering.
3. **The conversation history** — every message cloned into the child's context via forkContextMessages.
4. **The thinking configuration and model** — model: 'inherit' resolves to the parent's exact model.

## The Byte-Identical Prefix Trick

Three layers are frozen:

**Layer 1: System prompt via threading.** When the parent's system prompt was rendered, the result was captured in toolUseContext.renderedSystemPrompt. The fork child receives this exact string. Why not call getSystemPrompt() again? Because GrowthBook flags transition from cold to warm state. A flag that returned false during the parent's first turn might return true by the time the fork child spins up. Cache busted.

**Layer 2: Tool definitions via exact passthrough.** Normal sub-agents go through resolveAgentTools(). Fork agents skip this entirely with useExactTools: true. The child gets the parent's tool pool as-is — including keeping the Agent tool in the pool even though the child cannot use it.

**Layer 3: Message array construction.** buildForkedMessages() constructs the final two messages: a cloned assistant message with placeholder tool results (identical across children), followed by a user message with the per-child directive wrapped in boilerplate.

## The Fork Boilerplate Tag

Each child's directive is wrapped in a boilerplate XML tag with approximately 10 rules:

- Override the parent's forking instruction: "You ARE the fork. Do NOT spawn sub-agents."
- Execute silently, report once. No conversational text between tool calls.
- Stay within scope. The child must not expand beyond its directive.
- Structured output format: Scope/Result/Key files/Files changed/Issues.

## Recursive Fork Prevention

Two guards prevent infinite recursion:

**Primary guard:** querySource check. Fork children have querySource: 'agent:builtin:fork'. The call() method checks this before allowing the fork path.

**Fallback guard:** message scanning. Scans conversation history for the boilerplate XML tag. Belt and suspenders.

## The Sync-to-Async Transition

A fork child starts in the foreground. If it takes too long, it can be backgrounded mid-execution. The mechanism: the foreground iterator races between "next message" and "background signal." When the background signal fires, the iterator is gracefully terminated, and a new runAgent() instance is spawned with isAsync: true using the same agent ID and message history.

## The Economics

With fork (byte-identical prefixes):
- Child 1: 48,700 tokens at full price (cache miss)
- Children 2-5: 48,500 tokens at 10% price (cache hit) + 200 tokens at full price each
- Effective cost for children 2-5: ~5,050 tokens each

The savings scale with context size and child count. For a warm session with 100K tokens of history spawning 8 parallel forks, the cache savings can exceed 90%.

## Design Tensions

- **Isolation vs. cache efficiency** — Fork children inherit everything, including irrelevant history.
- **Safety vs. cache efficiency** — The Agent tool stays in the child's pool despite being forbidden.
- **Simplicity vs. cache efficiency** — Placeholder tool results are a lie chosen for brevity and uniformity.

## Apply This

1. Thread rendered prompts, do not recompute.
2. Freeze the tool array.
3. Maximize the shared prefix, minimize the per-child suffix.
4. Use constant placeholders for variable content.
5. Measure the break-even.


---

# Chapter 10: Tasks, Coordination, and Swarms

## The Limits of a Single Thread

A single agent loop can accomplish a remarkable amount of work. But it hits a ceiling — not of intelligence, but of parallelism and scope. A developer working on a large refactoring needs to update 40 files, run tests after each batch, and verify nothing broke. These are not harder problems — they are wider ones.

Claude Code's answer is a layered stack of orchestration patterns: background tasks for fire-and-forget commands, coordinator mode for manager-worker hierarchies, swarm teams for peer-to-peer collaboration, and a unified communication protocol that ties them all together.

## The Task State Machine

Every background operation is tracked as a task. Seven task types, five statuses, and a unified state model.

### Seven Types

| Type | Prefix | Description |
|------|--------|-------------|
| local_bash | b | Background shell commands |
| local_agent | a | Background sub-agents |
| remote_agent | r | Remote sessions |
| in_process_teammate | t | Swarm teammates |
| local_workflow | w | Workflow script executions |
| monitor_mcp | m | MCP server monitors |
| dream | d | Speculative background thinking |

### Five Statuses

pending → running → completed | failed | killed

No cycles. isTerminalTaskStatus() guards against interacting with dead tasks.

### The Base State

Every task carries: id, type, status, description, toolUseId, startTime, endTime, totalPausedMs, outputFile, outputOffset, notified.

The outputFile is the bridge between async execution and the parent's conversation. notified prevents duplicate completion messages.

## Communication Patterns

### Foreground: The Generator Chain

When an agent runs synchronously, the parent iterates its runAgent() async generator directly. The background escape hatch races between "next message" and "background signal" using Promise.race.

### Background: Three Channels

1. **Disk output files** — every task writes to an outputFile path. The parent reads incrementally via outputOffset.
2. **Task notifications** — XML notifications injected as user-role messages when agents complete.
3. **Command queue** — pendingMessages array drained at tool-round boundaries.

## Coordinator Mode

Coordinator mode transforms Claude Code from a single agent with background helpers into a true manager-worker architecture.

### Tool Restrictions

The coordinator gets exactly three tools: Agent, SendMessage, TaskStop. No file reading. No code editing. No shell commands. This restriction is the core design principle — the coordinator thinks, plans, decomposes, and synthesizes. Workers do the work.

### The 370-Line System Prompt

Key teachings:
- "Never delegate understanding." — The coordinator must synthesize research findings into specific prompts with file paths, line numbers, and exact changes.
- "Parallelism is your superpower." — Read-only tasks run freely in parallel. Write-heavy tasks serialize per file set.
- Four phases: Research → Synthesis → Implementation → Verification.
- The continue-vs-spawn decision depends on context overlap.

### The Scratchpad

A shared filesystem location where workers can read and write without permission prompts. The coordinator moves information by reference, not by value.

## The Swarm System

Peer-to-peer alternative to coordinator mode. Multiple Claude Code instances working as a team with a leader coordinating through message passing.

### In-Process Teammates

Run in the same Node.js process, isolated via AsyncLocalStorage. Messages capped at 50 entries for memory safety. The isIdle flag enables work-stealing patterns. The currentWorkAbortController enables "redirect" without killing the teammate.

### The Mailbox

File-based communication. Messages written to recipient's mailbox file on disk. Durable, inspectable, cheap. No message broker needed for tens of messages per session.

### Permission Forwarding

Workers operate with restricted permissions but can escalate to the leader via the mailbox system when approval is needed.

## Inter-Agent Communication: SendMessage

The universal communication primitive handles four routing modes:

1. **Bridge messages** — cross-machine via Anthropic's Remote Control servers
2. **UDS messages** — local inter-process via Unix Domain Sockets
3. **In-process subagent routing** — most common path, with auto-resume
4. **Team mailbox** — fallback when team context is active

### The Auto-Resume Pattern

When SendMessage targets a completed or killed agent, instead of returning an error, it resurrects the agent from its disk transcript. From the coordinator's perspective, sending a message to a dead agent and sending to a live agent are the same operation.

## Choosing Between Patterns

| Scenario | Pattern | Why |
|----------|---------|-----|
| Run tests while editing | Simple delegation | One background task, no coordination |
| Refactor 40 files across 3 modules | Coordinator | Research → synthesis → parallel implementation |
| Multi-day feature development | Swarm | Long-lived agents, plan approval, peer communication |
| Fix a bug with known location | Neither — single agent | Orchestration overhead exceeds benefit |

## Apply This

- Design your task state machine as a simple directed graph with no cycles.
- Use file-based communication for durability and debuggability.
- Make the simplest pattern your starting point. Sophisticated patterns exist for the 5% of genuinely wide problems.


---

# Chapter 11: Memory — Learning Across Conversations

## The Stateless Problem

Every chapter so far has described machinery that exists within a single session. When the process exits, all of it vanishes. The next conversation starts with zero knowledge of what happened before.

A developer corrects the model's testing approach on Monday, and on Tuesday the model makes the same mistake. Without memory, every session starts at zero. The agent is perpetually a new hire on their first day.

Claude Code's memory system is a different bet: files on disk, Markdown format, LLM-powered recall, no infrastructure. The bet is that simplicity in storage, combined with intelligence in retrieval, produces a better system than sophistication in both.

The design philosophy: human-readable, human-editable, version-controllable, zero infrastructure, debuggable.

## The Four-Type Taxonomy

Not everything is worth remembering. The memory system constrains all memories to exactly four types:

| Type | Description |
|------|-------------|
| user | Information about the person: role, goals, expertise level |
| feedback | Guidance about how to approach work — corrections and confirmations |
| project | Ongoing work context — who is doing what, why, by when |
| reference | Bookmarks — pointers to where information lives in external systems |

The taxonomy is designed around a single criterion: is this knowledge derivable from the current project state? Code patterns, architecture, file structure — all derivable, all excluded.

### Frontmatter as Contract

Every memory file uses YAML frontmatter:

```yaml
---
name: {{memory name}}
description: {{one-line description}}
type: {{user, feedback, project, reference}}
---
```

The description is the most load-bearing field — it is what the relevance selector uses to decide whether to surface this memory.

## The Write Path

Writing a memory is a two-step process:
1. Write the memory file with YAML frontmatter
2. Update the index (MEMORY.md) with a one-line pointer

## The Recall Path

The recall system operates in two tiers:
- MEMORY.md index is always loaded at session start
- Individual memory files are surfaced on-demand through an LLM-powered relevance query

### The Sonnet Side-Query

The manifest is sent to a Sonnet model as a side-query. The selector is conservative: include only memories useful for the current query, skip if uncertain, avoid selecting memories for tools already in active use.

This approach trades latency for precision. Keyword matching would be fast but has no understanding of context. Embedding similarity struggles with negation. The Sonnet side-query understands semantic relevance, handles negation, and requires zero infrastructure.

## Staleness

The staleness system addresses stale memories being asserted as fact. Memories from today or yesterday get no warning. Everything older gets a caveat: "This memory is N days old. Code behavior claims or file:line citations may be outdated."

The human-readable format exists because models are poor at date arithmetic. "47 days ago" triggers better staleness reasoning than a raw ISO timestamp.

## MEMORY.md as the Always-Loaded Index

Two hard caps: 200 lines and 25,000 bytes. The two-tier architecture — lightweight always-on index plus heavy on-demand content — allows memory to scale.

## Team Memory

Team memory is a subdirectory gated behind a feature flag. Defense in depth uses three layers: input sanitization, string-level path validation, and symlink resolution. All validation failures produce a PathTraversalError. Fail closed.

## KAIROS Mode: Append-Only Daily Logs

For long-running sessions, the model appends to date-named log files. Consolidation runs via /dream in four phases: Orient, Gather, Consolidate, Prune.

## The Memory Extraction Agent

A forked agent — sharing the parent's prompt cache — analyzes recent messages and writes any memories the main agent missed. The interaction is cooperative: when the main agent saves, the background agent defers.

## Apply This

- Files beat databases for agent memory — inspectable, editable, version-controllable.
- Constrain what gets saved using the derivability test.
- Use an LLM for recall, not keywords or embeddings.
- Warn about staleness, do not expire.
- Build a safety net for capture with a background extraction agent.


---

# Chapter 12: Extensibility — Skills and Hooks

## Two Dimensions of Extension

Skills extend what the model can do — markdown files that become slash commands, injecting new instructions. Hooks extend when and how things happen — lifecycle interceptors that fire at over two dozen distinct points.

The separation is clean: skills are content, hooks are control flow.

## Skills: Teaching the Model New Tricks

### Two-Phase Loading

Phase 1 reads each SKILL.md file, extracts frontmatter metadata into the system prompt. Phase 2 fires on invocation, loading full content with variable substitution.

### Seven Sources with Priority

| Priority | Source | Location |
|----------|--------|----------|
| 1 | Managed (Policy) | Enterprise-controlled |
| 2 | User | ~/.claude/skills/ |
| 3 | Project | .claude/skills/ |
| 4 | Additional Dirs | Via --add-dir flag |
| 5 | Legacy Commands | .claude/commands/ |
| 6 | Bundled | Compiled into binary |
| 7 | MCP | MCP server prompts |

### The MCP Security Boundary

MCP skills never execute inline shell commands. The trust boundary is absolute.

## Hooks: Controlling When Things Happen

### Four User-Configurable Types

- **Command hooks** — spawn a shell process
- **Prompt hooks** — single LLM call
- **Agent hooks** — multi-turn agentic loop (max 50 turns)
- **HTTP hooks** — POST to a URL

### The Five Most Important Lifecycle Events

1. **PreToolUse** — fires before every tool execution. Can block, modify input, auto-approve.
2. **PostToolUse** — fires after successful execution. Can inject context or replace output.
3. **Stop** — fires before Claude concludes. A blocking hook forces continuation.
4. **SessionStart** — fires at session beginning. Can set environment variables.
5. **UserPromptSubmit** — fires when user submits. Can block processing.

### Exit Code Semantics

| Exit Code | Meaning | Blocks |
|-----------|---------|--------|
| 0 | Success | No |
| 2 | Blocking error, stderr shown | Yes |
| Other | Non-blocking warning | No |

## The Snapshot Security Model

Hooks configuration is frozen at startup. captureHooksConfigSnapshot() is called once. From that point, executeHooks() reads from the snapshot, never re-reading settings files implicitly. This prevents TOCTOU attacks.

## Integration: Where Skills Meet Hooks

When a skill is invoked, its frontmatter-declared hooks register as session-scoped hooks. The skillRoot becomes CLAUDE_PLUGIN_ROOT for the hook's shell commands.

## Apply This

- Separate content from control flow.
- Freeze configuration at trust boundaries.
- Use uncommon exit codes for semantic signals (exit 2, not 1).
- Optimize for the common case with fast paths for internal callbacks.


---

# Chapter 13: The Terminal UI

## Why Build a Custom Renderer?

The terminal is not a browser. There is no DOM, no CSS engine, no compositor. Everything between the two byte streams — layout, styling, diffing, scrolling, selection — has to be invented from scratch.

Claude Code started with Ink, then forked it beyond recognition. The stock version allocates one JavaScript object per cell per frame — on a 200x120 terminal, that is 24,000 objects created and garbage-collected every 16ms. For an LLM agent streaming tokens at 60fps, it is a non-starter.

What remains is a custom rendering engine: packed typed arrays instead of object-per-cell, pool-based string interning, double-buffered rendering with cell-level diffing, and an optimizer that merges adjacent terminal writes into minimal escape sequences.

## The Custom DOM

Seven element types map to terminal rendering concepts:

- **ink-root** — the document root
- **ink-box** — a flexbox container
- **ink-text** — a text node with Yoga measure function
- **ink-virtual-text** — nested styled text
- **ink-link** — hyperlink via OSC 8 escape sequences
- **ink-progress** — progress indicator
- **ink-raw-ansi** — pre-rendered ANSI content (zero-cost after initial highlighting)

Each DOMElement carries: yogaNode, style, attributes, childNodes, dirty flag, _eventHandlers (separated to avoid dirty-marking on handler changes), scrollTop, stickyScroll.

The markDirty() function walks up through ancestors, setting dirty = true on each element. Sibling subtrees remain clean and can be blitted from the previous frame.

## The Rendering Pipeline

Every frame traverses seven stages:

1. **React commit and Yoga layout** — resolves flexbox tree
2. **DOM-to-screen** — walks DOM tree, writes characters into Screen buffer
3. **Overlay** — text selection and search highlighting modify buffer in-place
4. **Diff** — cell-by-cell comparison against front frame (two integer comparisons per cell)
5. **Optimize** — adjacent patches merged, redundant cursor moves eliminated
6. **Write** — ANSI escape sequences written to stdout in single write() call with BSU/ESU synchronized update markers
7. **Frame timing** — per-stage instrumentation reported via FrameEvent

## Double-Buffer Rendering

Two frame buffers (frontFrame, backFrame) swap after each render. Eliminates allocation — the swap is a pointer assignment.

Render scheduling uses lodash throttle at 16ms (~60fps). Scroll operations use a separate setTimeout at 4ms for faster scroll frames, bypassing React entirely.

## Pool-Based Memory: Why Interning Matters

A 200x120 terminal has 24,000 cells. At 60fps, that is 5.76 million allocations per second without interning.

### Cell Layout

Two Int32 words per cell in a contiguous Int32Array:
- word0: charId (32 bits, index into CharPool)
- word1: styleId | hyperlinkId | width

A parallel BigInt64Array view enables bulk operations — clearing a row is a single fill() call.

### Three Interning Pools

- **CharPool** — character strings to integer IDs. Fast path for ASCII: 128-entry Int32Array.
- **StylePool** — ANSI style codes to integer IDs. Bit 0 encodes visibility on space characters. Caches pre-serialized transition strings.
- **HyperlinkPool** — OSC 8 hyperlink URIs.

All three pools are shared across front and back frames — interned IDs are valid across frames, enabling blit optimization without re-interning.

## Streaming Markdown

Three optimizations:

1. **Token caching** — module-level LRU cache (500 entries) stores marked.lexer() results
2. **Fast-path detection** — hasMarkdownSyntax() checks first 500 chars; plain text bypasses parser
3. **Lazy syntax highlighting** — code block highlighting loaded via React Suspense

The StreamingMarkdown component handles incomplete code fences, partial bold markers, and truncated list items. Transitions to static Markdown when streaming completes.

## Selection and Search Highlighting

Text selection operates as screen-buffer overlays in alt-screen mode. Mouse tracking uses SGR 1003 mode. Multi-click detection: double-click selects word, triple-click selects line (5000ms timeout, 1-cell tolerance).

Search highlighting uses two parallel mechanisms: scan-based (walks visible cells) and position-based (pre-computed MatchPosition[] offsets).

## Apply This

- Intern everything — integer comparison is one CPU instruction, string comparison is a loop.
- Diff at the right level — cell-level diffing with packed typed arrays.
- Separate the hot path from React — scroll mutations bypass the reconciler.
- Clean up periodically — 5-minute pool reset cycle bounds growth.
- Track damage bounds — skip unchanged regions entirely.


---

# Chapter 14: Input and Interaction

## Raw Bytes, Meaningful Actions

When you press Ctrl+X followed by Ctrl+K, the terminal sends two byte sequences. The input system must recognize that these constitute the chord ctrl+x ctrl+k, which maps to chat:killAgents. Between the raw bytes and the killed agents, six systems activate.

The difficulty is the combinatorial explosion of terminal diversity. The input system must produce correct ParsedKey objects from all of them.

Design philosophy: progressive enhancement with graceful degradation.

## The Key Parsing Pipeline

### The Tokenizer

Maintains a state machine that buffers partial escape sequences and emits complete tokens. The incomplete-sequence problem: when the tokenizer sees a lone \x1b, it buffers the byte and starts a 50ms timer.

All parsed keys from a single read() are processed in one reconciler.discreteUpdates() call — pasting 100 characters produces one re-render, not 100.

### Multi-Protocol Support

Five distinct protocols handled simultaneously:

| Protocol | Format | Notes |
|----------|--------|-------|
| CSI u (Kitty) | ESC [ codepoint [; modifier] u | Modern standard, unambiguous |
| xterm modifyOtherKeys | ESC [ 27 ; modifier ; keycode ~ | Fallback for Ghostty over SSH |
| Legacy sequences | ESC O/ESC [ patterns | VT100/VT220/xterm variations |
| SGR mouse | ESC [ < button ; col ; row M/m | Click, drag, wheel events |
| Bracketed paste | ESC [200~ ... ESC [201~ | Identifies pasted content |

Terminal identification via XTVERSION query survives SSH connections.

## The Keybinding System

Separates three concerns: bindings (what key triggers what action), handlers (what happens), contexts (which bindings are active).

### 16 Contexts

| Context | When Active |
|---------|-------------|
| Global | Always |
| Chat | Prompt input focused |
| Autocomplete | Completion menu visible |
| Confirmation | Permission dialog showing |
| Scroll | Alt-screen with scrollable content |
| Task | Background task running |
| Help | Help overlay displayed |
| ... | 9 more contexts |

The last matching binding wins — user overrides take precedence over defaults.

### Chord Support

The ctrl+x ctrl+k binding uses a state machine with 1000ms timeout. ChordInterceptor handles edge cases: non-matching characters fall through to normal input (only the chord prefix is discarded).

## Vim Mode

### The State Machine

A discriminated union with 12 variants:

```typescript
export type CommandState =
  | { type: 'idle' }
  | { type: 'count'; digits: string }
  | { type: 'operator'; op: Operator; count: number }
  | { type: 'operatorCount'; op: Operator; count: number; digits: string }
  | { type: 'operatorFind'; op: Operator; count: number; find: FindType }
  | { type: 'operatorTextObj'; op: Operator; count: number; scope: TextObjScope }
  | { type: 'find'; find: FindType; count: number }
  | { type: 'g'; count: number }
  | { type: 'operatorG'; op: Operator; count: number }
  | { type: 'replace'; count: number }
  | { type: 'indent'; dir: '>' | '<'; count: number }
```

TypeScript's exhaustive checking ensures every switch handles all 12 cases. The type system makes impossible states unrepresentable.

### Transitions as Pure Functions

The transition() function dispatches on current state type. Each returns a TransitionResult with optional next state and optional execute closure. Side effects are returned, not executed — the caller decides when to run them.

## Virtual Message List

VirtualMessageList renders only visible messages plus a buffer. Height cache per message, jump handle for search navigation, sticky prompt tracking. Scroll mutations go directly to DOM node properties, bypassing React's reconciler.

## Apply This

- Separate bindings from handlers — bindings are data, handlers are code.
- Context as a first-class concept — activate/deactivate based on application state.
- Chord state as an explicit machine with timeout and cancellation.
- Make the type system enforce your state machine.
- Push complexity to the boundaries, keep the interior clean.


---

# Chapter 15: MCP — The Universal Tool Protocol

## Why MCP Matters Beyond Claude Code

Every other chapter in this book is about Claude Code's internals. This one is different. The Model Context Protocol is an open specification that any agent can implement, and Claude Code's MCP subsystem is one of the most complete production clients in existence. If you are building an agent that needs to call external tools — any agent, in any language, on any model — the patterns in this chapter transfer directly.

The core proposition is straightforward: MCP defines a JSON-RPC 2.0 protocol for tool discovery and invocation between a client (the agent) and a server (the tool provider). The client sends tools/list to discover what a server offers, then tools/call to execute. The server describes each tool with a name, description, and JSON Schema for its inputs. That is the entire contract. Everything else — transport selection, authentication, config loading, tool name normalization — is the implementation work that turns a clean spec into something that survives contact with the real world.

Claude Code's MCP implementation spans four core files: types.ts, client.ts, auth.ts, and InProcessTransport.ts. Together they support eight transport types, seven configuration scopes, OAuth discovery across two RFCs, and a tool wrapping layer that makes MCP tools indistinguishable from built-in ones — the same Tool interface covered in Chapter 6. This chapter walks through each layer.

## Eight Transport Types

The first design decision in any MCP integration is how the client talks to the server. Claude Code supports eight transport configurations:

Three design choices are worth noting. First, stdio is the default — when type is omitted, the system assumes a local subprocess. This is backwards-compatible with the earliest MCP configs. Second, the fetch wrappers stack: timeout wrapping outside step-up detection, outside the base fetch. Each wrapper handles one concern. Third, the ws-ide branch has a Bun/Node runtime split — Bun's WebSocket accepts proxy and TLS options natively, while Node requires the ws package.

When to use which. For local tools (filesystem, database, custom scripts), stdio — no network, no auth, just pipes. For remote services, http (Streamable HTTP) is the current spec recommendation. sse is legacy but widely deployed. The sdk, IDE, and claudeai-proxy types are internal to their respective ecosystems.

## Configuration Loading and Scoping

MCP server configs load from seven scopes, merged and deduplicated:

| Scope | Source | Trust |
|-------|--------|-------|
| local | .mcp.json in working directory | Requires user approval |
| user | ~/.claude.json mcpServers field | User-managed |
| project | Project-level config | Shared project settings |
| enterprise | Managed enterprise config | Pre-approved by org |
| managed | Plugin-provided servers | Auto-discovered |
| claudeai | Claude.ai web interface | Pre-authorized via web |
| dynamic | Runtime injection (SDK) | Programmatically added |

Deduplication is content-based, not name-based. Two servers with different names but the same command or URL are recognized as the same server. The getMcpServerSignature() function computes a canonical key: stdio:["command","arg1"] for local servers, url:https://example.com/mcp for remote ones. Plugin-provided servers whose signature matches a manual config are suppressed.

## Tool Wrapping: From MCP to Claude Code

When a connection succeeds, the client calls tools/list. Each tool definition is transformed into Claude Code's internal Tool interface — the same interface used by built-in tools. After wrapping, the model cannot distinguish between a built-in tool and an MCP tool.

The wrapping process has four stages:

1. **Name normalization.** normalizeNameForMCP() replaces invalid characters with underscores. The fully qualified name follows `mcp__{serverName}__{toolName}`.

2. **Description truncation.** Capped at 2,048 characters. OpenAPI-generated servers have been observed dumping 15-60KB into tool.description — roughly 15,000 tokens per turn for a single tool.

3. **Schema passthrough.** The tool's inputSchema passes directly to the API. No transformation, no validation at wrapping time. Schema errors surface at call time, not registration time.

4. **Annotation mapping.** MCP annotations map to behavior flags: readOnlyHint marks tools safe for concurrent execution (as discussed in Chapter 7's streaming executor), destructiveHint triggers extra permission scrutiny. These annotations come from the MCP server — a malicious server could mark a destructive tool as read-only. This is an accepted trust boundary, but one worth understanding: the user opted into the server, and a malicious server marking destructive tools as read-only is a real attack vector. The system accepts this tradeoff because the alternative — ignoring annotations entirely — would prevent legitimate servers from improving the user experience.

## OAuth for MCP Servers

Remote MCP servers often require authentication. Claude Code implements the full OAuth 2.0 + PKCE flow with RFC-based discovery, Cross-App Access, and error body normalization.

### Discovery Chain

The authServerMetadataUrl escape hatch exists because some OAuth servers implement neither RFC.

### Cross-App Access (XAA)

When an MCP server config has `oauth.xaa: true`, the system performs federated token exchange through an Identity Provider — one IdP login unlocks multiple MCP servers.

### Error Body Normalization

The normalizeOAuthErrorBody() function handles OAuth servers that violate the spec. Slack returns HTTP 200 for error responses with the error buried in the JSON body. The function peeks at 2xx POST response bodies, and when the body matches OAuthErrorResponseSchema but not OAuthTokensSchema, rewrites the response to HTTP 400. It also normalizes Slack-specific error codes (invalid_refresh_token, expired_refresh_token, token_expired) to the standard invalid_grant.

## In-Process Transport

Not every MCP server needs to be a separate process. The InProcessTransport class enables running an MCP server and client in the same process:

```
class InProcessTransport implements Transport {
  async send(message: JSONRPCMessage): Promise<void> {
    if (this.closed) throw new Error('Transport is closed')
    queueMicrotask(() => { this.peer?.onmessage?.(message) })
  }
  async close(): Promise<void> {
    if (this.closed) return
    this.closed = true
    this.onclose?.()
    if (this.peer && !this.peer.closed) {
      this.peer.closed = true
      this.peer.onclose?.()
    }
  }
}
```

The entire file is 63 lines. Two design decisions deserve attention. First, send() delivers via queueMicrotask() to prevent stack depth issues in synchronous request/response cycles. Second, close() cascades to the peer, preventing half-open states. The Chrome MCP server and Computer Use MCP server both use this pattern.

## Connection Management

### Connection States

Each MCP server connection exists in one of five states: connected, failed, needs-auth (with a 15-minute TTL cache to prevent 30 servers from independently discovering the same expired token), pending, or disabled.

### Session Expiry Detection

MCP's Streamable HTTP transport uses session IDs. When a server restarts, requests return HTTP 404 with JSON-RPC error code -32001. The isMcpSessionExpiredError() function checks both signals — note that it uses string inclusion on the error message to detect the error code, which is pragmatic but fragile:

```
export function isMcpSessionExpiredError(error: Error): boolean {
  const httpStatus = 'code' in error ? (error as any).code : undefined
  if (httpStatus !== 404) return false
  return error.message.includes('"code":-32001') ||
         error.message.includes('"code": -32001')
}
```

On detection, the connection cache clears and the call retries once.

### Batched Connections

Local servers connect in batches of 3 (spawning processes can exhaust file descriptors), remote servers in batches of 20. The React context provider MCPConnectionManager.tsx manages the lifecycle, diffing current connections against new configs.

## Claude.ai Proxy Transport

The claudeai-proxy transport illustrates a common agent integration pattern: connecting through an intermediary. Claude.ai subscribers configure MCP "connectors" through the web interface, and the CLI routes through Claude.ai's infrastructure which handles vendor-side OAuth.

The createClaudeAiProxyFetch() function captures the sentToken at request time, not re-read after a 401. Under concurrent 401s from multiple connectors, another connector's retry might have already refreshed the token. The function also checks for concurrent refreshes even when the refresh handler returns false — the "ELOCKED contention" case where another connector won the lockfile race.

## Timeout Architecture

MCP timeouts are layered, each protecting against a different failure mode:

| Layer | Duration | Protects Against |
|-------|----------|-----------------|
| Connection | 30s | Unreachable or slow-starting servers |
| Per-request | 60s (fresh per request) | Stale timeout signal bug |
| Tool call | ~27.8 hours | Legitimately long operations |
| Auth | 30s per OAuth request | Unreachable OAuth servers |

The per-request timeout deserves emphasis. Early implementations created a single AbortSignal.timeout(60000) at connection time. After 60 seconds of idle time, the next request would abort immediately — the signal was already expired. The fix: wrapFetchWithTimeout() creates a fresh timeout signal for every request. It also normalizes the Accept header as a last-step defense against runtimes and proxies that drop it.

## Apply This: Integrating MCP Into Your Own Agent

**Start with stdio, add complexity later.** StdioClientTransport handles everything: spawn, pipe, kill. One line of config, one transport class, and you have MCP tools.

**Normalize names and truncate descriptions.** Names must match `^[a-zA-Z0-9_-]{1,64}$`. Prefix with `mcp__{serverName}__` to avoid collisions. Cap descriptions at 2,048 characters — OpenAPI-generated servers will waste context tokens otherwise.

**Handle auth lazily.** Do not attempt OAuth until a server returns 401. Most stdio servers need no auth.

**Use in-process transport for built-in servers.** createLinkedTransportPair() eliminates subprocess overhead for servers you control.

**Respect tool annotations and sanitize output.** readOnlyHint enables concurrent execution. Sanitize responses against malicious Unicode (bidirectional overrides, zero-width joiners) that could mislead the model.

The MCP protocol is deliberately minimal — two JSON-RPC methods. Everything between those methods and a production deployment is engineering: eight transports, seven config scopes, two OAuth RFCs, and timeout layering. Claude Code's implementation shows what that engineering looks like at scale.

The next chapter examines what happens when the agent reaches beyond localhost: the remote execution protocols that let Claude Code run in cloud containers, accept instructions from web browsers, and tunnel API traffic through credential-injecting proxies.


---

# Chapter 16: Remote Control and Cloud Execution

## The Agent Reaches Beyond Localhost

Every chapter so far has assumed that Claude Code runs on the same machine where the code lives. The terminal is local. The filesystem is local. The model responses stream back to a process that owns both the keyboard and the working directory.

That assumption breaks the moment you want to control Claude Code from a browser, run it inside a cloud container, or expose it as a service on your LAN. The agent needs a way to receive instructions from a web browser, a mobile app, or an automated pipeline — forward permission prompts to someone who is not sitting at the terminal, and tunnel its API traffic through infrastructure that might inject credentials or terminate TLS on the agent's behalf.

Claude Code solves this with four systems, each addressing a different topology:

These systems share a common design philosophy: reads and writes are asymmetric, reconnection is automatic, and failures degrade gracefully.

## Bridge v1: Poll, Dispatch, Spawn

The v1 bridge is the environment-based remote control system. When a developer runs `claude remote-control`, the CLI registers with the Environments API, polls for work, and spawns a child process per session.

Before registration, a gauntlet of pre-flight checks runs: runtime feature gate, OAuth token validation, organization policy check, dead token detection (a cross-process backoff after three consecutive failures with the same expired token), and proactive token refresh that eliminates roughly 9% of registrations that would otherwise fail on the first attempt.

Once registered, the bridge enters a long-poll loop. Work items arrive as sessions (with a secret field containing session tokens, API base URL, MCP configs, and environment variables) or healthchecks. The bridge throttles "no work" log messages to every 100 empty polls.

Each session spawns a child Claude Code process communicating via NDJSON on stdin/stdout. Permission requests flow through the bridge transport to the web interface where the user approves or denies. The round-trip must complete within roughly 10-14 seconds.

## Bridge v2: Direct Sessions and SSE

The v2 bridge eliminates the entire Environments API layer — no registration, no polling, no acknowledgment, no heartbeat, no deregistration. The motivation: v1 required the server to know the machine's capabilities before dispatching work. V2 collapses the lifecycle to three steps:

- **Create session:** POST /v1/code/sessions with OAuth credentials.
- **Connect bridge:** POST /v1/code/sessions/{id}/bridge. Returns a worker_jwt, api_base_url, and worker_epoch. Each /bridge call bumps the epoch — it IS the registration.
- **Open transport:** SSE for reads, CCRClient for writes.

The transport abstraction (ReplBridgeTransport) unifies v1 and v2 behind a common interface, so message handling does not need to know which generation it is talking to.

When the SSE connection drops due to a 401, the transport rebuilds with fresh credentials from a new /bridge call while preserving the sequence number cursor — no messages are lost. The write path uses per-instance getAuthToken closures instead of process-wide environment variables, preventing JWT leakage across concurrent sessions.

### The FlushGate

A subtle ordering problem: the bridge needs to send conversation history while accepting live writes from the web interface. If a live write arrives during the history flush, messages could be delivered out of order. The FlushGate queues live writes during the flush POST and drains them in order when it completes.

### Token Refresh and Epoch Management

The v2 bridge proactively refreshes worker JWTs before expiry. A new epoch tells the server this is the same worker with fresh credentials. Epoch mismatches (409 responses) are handled aggressively: both connections close and an exception unwinds the caller, preventing split-brain scenarios.

## Message Routing and Echo Deduplication

Both bridge generations share handleIngressMessage() as the central router:

- Parse JSON, normalize control message keys.
- Route control_response to permission handler, control_request to request handler.
- Check UUID against recentPostedUUIDs (echo dedup) and recentInboundUUIDs (re-delivery dedup).
- Forward validated user messages.

### BoundedUUIDSet: O(1) Lookup, O(capacity) Memory

The bridge has an echo problem — messages may echo back on the read stream or be delivered twice during transport switches. BoundedUUIDSet is a FIFO-bounded set backed by a circular buffer:

```
class BoundedUUIDSet {
  private buffer: string[]
  private set: Set<string>
  private head = 0

  add(uuid: string): void {
    if (this.set.size >= this.capacity) {
      this.set.delete(this.buffer[this.head])
    }
    this.buffer[this.head] = uuid
    this.set.add(uuid)
    this.head = (this.head + 1) % this.capacity
  }

  has(uuid: string): boolean { return this.set.has(uuid) }
}
```

Two instances run in parallel, each with capacity 2000. O(1) lookup via the Set, O(capacity) memory via circular buffer eviction, no timers or TTLs. Unknown control request subtypes get an error response, not silence — preventing the server from waiting for a response that never comes.

## The Asymmetric Design: Persistent Reads, HTTP POST Writes

The CCR protocol uses asymmetric transport: reads flow through a persistent connection (WebSocket or SSE), writes go through HTTP POST. This reflects a fundamental asymmetry in the communication pattern.

Reads are high-frequency, low-latency, server-initiated — hundreds of small messages per second during token streaming. A persistent connection is the only sensible choice. Writes are low-frequency, client-initiated, and require acknowledgment — messages per minute, not per second. HTTP POST provides reliable delivery, idempotency via UUIDs, and natural integration with load balancers.

Trying to unify them on a single WebSocket creates coupling: if the WebSocket drops during a write, you need retry logic and must distinguish "not sent" from "sent but acknowledgment lost." Separate channels let each be optimized independently.

## Remote Session Management

The SessionsWebSocket manages the client side of a CCR WebSocket connection. Its reconnection strategy discriminates between failure types:

| Failure | Strategy |
|---------|----------|
| 4003 (unauthorized) | Stop immediately, no retries |
| 4001 (session not found) | Max 3 retries, linear backoff (transient during compaction) |
| Other transient | Exponential backoff, max 5 attempts |

The isSessionsMessage() type guard accepts any object with a string `type` field — deliberately permissive. A hardcoded allowlist would silently drop new message types before the client is updated.

## Direct Connect: The Local Server

Direct Connect is the simplest topology: Claude Code runs as a server and clients connect via WebSocket. No cloud intermediary, no OAuth tokens.

Sessions have five states: starting, running, detached, stopping, stopped. Metadata persists to `~/.claude/server-sessions.json` for resume across server restarts. The `cc://` URL scheme provides clean addressing for local connections.

## Upstream Proxy: Credential Injection in Containers

The upstream proxy runs inside CCR containers and solves a specific problem: injecting organization credentials into outbound HTTPS traffic from a container where the agent might execute untrusted commands.

The setup sequence is carefully ordered:

1. Read the session token from `/run/ccr/session_token`.
2. Set `prctl(PR_SET_DUMPABLE, 0)` via Bun FFI — blocking same-UID ptrace of the process heap. Without this, a prompt-injected `gdb -p $PPID` could scrape the token from memory.
3. Download the upstream proxy CA certificate and concatenate with system CA bundle.
4. Start a local CONNECT-to-WebSocket relay on an ephemeral port.
5. Unlink the token file — the token now exists only on the heap.
6. Export environment variables for all subprocesses.

Every step fails open: errors disable the proxy rather than killing the session. The correct tradeoff — a failed proxy means some integrations will not work, but core functionality remains available.

### Protobuf Hand-Encoding

Bytes through the tunnel are wrapped in UpstreamProxyChunk protobuf messages. The schema is trivial — `message UpstreamProxyChunk { bytes data = 1; }` — and Claude Code encodes it by hand in ten lines rather than pulling in a protobuf runtime:

```
export function encodeChunk(data: Uint8Array): Uint8Array {
  const varint: number[] = []
  let n = data.length
  while (n > 0x7f) { varint.push((n & 0x7f) | 0x80); n >>>= 7 }
  varint.push(n)
  const out = new Uint8Array(1 + varint.length + data.length)
  out[0] = 0x0a // field 1, wire type 2
  out.set(varint, 1)
  out.set(data, 1 + varint.length)
  return out
}
```

Ten lines replace a full protobuf runtime. A single-field message does not justify a dependency — the maintenance burden of the bit manipulation is far lower than the supply chain risk.

## Apply This: Designing Remote Agent Execution

**Separate read and write channels.** When reads are high-frequency streams and writes are low-frequency RPCs, unifying them creates unnecessary coupling. Let each channel fail and recover independently.

**Bound your deduplication memory.** The BoundedUUIDSet pattern provides fixed-memory deduplication. Any at-least-once delivery system needs a bounded dedup buffer, not an unbounded Set.

**Make reconnection strategy proportional to the failure signal.** Permanent failures should not retry. Transient failures should retry with backoff. Ambiguous failures should retry with a low cap.

**Keep secrets heap-only in adversarial environments.** Reading the token from a file, disabling ptrace, and unlinking the file eliminates both filesystem and memory-inspection attack vectors.

**Fail open for auxiliary systems.** The upstream proxy fails open because it provides enhanced functionality (credential injection), not core functionality (model inference).

The remote execution systems encode a deeper principle: the agent's core loop (Chapter 5) should be agnostic about where instructions come from and where results go. The bridge, Direct Connect, and upstream proxy are transport layers. The message handling, tool execution, and permission flows above them are identical regardless of whether the user is sitting at the terminal or on the other side of a WebSocket.

The next chapter examines the other operational concern: performance — how Claude Code makes every millisecond and token count across startup, rendering, search, and API costs.


---

# Chapter 17: Performance — Every Millisecond and Token Counts

## The Senior Engineer's Playbook

Performance optimization in an agentic system is not one problem. It is five:

- **Startup latency** — the time from keystroke to first useful output. Users abandon tools that feel slow to launch.
- **Token efficiency** — the fraction of the context window consumed by useful content versus overhead. The context window is the most constrained resource.
- **API cost** — the dollar amount per turn. Prompt caching can reduce this by 90%, but only if the system preserves cache stability across turns.
- **Rendering throughput** — the frames per second during streaming output. Chapter 13 covered the rendering architecture; this chapter covers the performance measurements and optimizations that keep it fast.
- **Search speed** — the time to find a file in a 270,000-path codebase on every keystroke.

Claude Code attacks all five with techniques ranging from the obvious (memoization) to the subtle (26-bit bitmaps for pre-filtering fuzzy search). A note on methodology: these are not theoretical optimizations. Claude Code ships with 50+ startup profiling checkpoints, sampled at 100% of internal users and 0.5% of external users. Every optimization below was motivated by data from this instrumentation, not by intuition.

## Saving Milliseconds at Startup

### Module-Level I/O Parallelism

The entry point main.tsx deliberately violates "no side effects at module scope":

```
profileCheckpoint('main_tsx_entry');
startMdmRawRead();       // fires plutil/reg-query subprocesses
startKeychainPrefetch(); // fires both macOS keychain reads in parallel
```

Two macOS keychain entries would otherwise cost ~65ms of sequential synchronous spawns. By launching both as fire-and-forget promises at the module level, they execute in parallel with ~135ms of module loading during which the CPU would otherwise be idle.

### API Preconnection

apiPreconnect.ts fires a HEAD request to the Anthropic API during initialization, overlapping the TCP+TLS handshake (100-200ms) with setup work. In interactive mode, the overlap is unbounded — the connection warms while the user types. The request fires after applyExtraCACertsFromConfig() and configureGlobalAgents() so the warmed connection uses the correct transport configuration.

### Fast-Path Dispatch and Deferred Imports

The CLI entry point contains early-return paths for specialized subcommands — `claude mcp` never loads the React REPL, `claude daemon` never loads the tool system. Heavy modules are loaded via dynamic import() only when needed: OpenTelemetry (~400KB + ~700KB gRPC), event logging, error dialogs, upstream proxy. LazySchema defers Zod schema construction to first validation, pushing the cost past startup.

## Saving Tokens in the Context Window

### Slot Reservation: 8K Default, 64K Escalation

The most impactful single optimization:

The default output slot reservation is 8,000 tokens, escalating to 64,000 on truncation. The API reserves max_output_tokens of capacity for the model's response. The default SDK value is 32K-64K, but production data shows p99 output length is 4,911 tokens. The default over-reserves by 8-16x, wasting 24,000-59,000 tokens per turn. Claude Code caps at 8K and retries at 64K on the rare truncation (<1% of requests). For a 200K window, this is a 12-28% improvement in usable context — for free.

### Tool Result Budgeting

| Limit | Value | Purpose |
|-------|-------|---------|
| Per-tool characters | 50,000 | Results persisted to disk when exceeded |
| Per-tool tokens | 100,000 | ~400KB text upper bound |
| Per-message aggregate | 200,000 chars | Prevents N parallel tools from blowing the budget in one turn |

The per-message aggregate is the key insight. Without it, "read all files in src/" could produce 10 parallel reads each returning 40K characters.

### Context Window Sizing

The default 200K-token window is expandable to 1M via the [1m] suffix on model names or experiment treatment. When usage approaches the limit, a 4-layer compaction system progressively summarizes older content. Token counting is anchored on the API's actual usage field, not client-side estimation — accounting for prompt caching credits, thinking tokens, and server-side transformations.

## Saving Money on API Calls

### The Prompt Cache Architecture

Anthropic's prompt cache operates on exact prefix matching. If a single token changes mid-prefix, everything after is a cache miss. Claude Code structures the entire prompt so stable parts come first and volatile parts come last.

When shouldUseGlobalCacheScope() returns true, system prompt entries before the dynamic boundary get `scope: 'global'` — two users running the same Claude Code version share the prefix cache. Global scope is disabled when MCP tools are present, since MCP schemas are per-user.

### Sticky Latch Fields

Five boolean fields use a "sticky-on" pattern — once true, they remain true for the session:

| Latch Field | What It Prevents |
|-------------|-----------------|
| promptCache1hEligible | Mid-session overage flip changing cache TTL |
| afkModeHeaderLatched | Shift+Tab toggling busting cache |
| fastModeHeaderLatched | Cooldown enter/exit double-busting cache |
| cacheEditingHeaderLatched | Mid-session config toggles busting cache |
| thinkingClearLatched | Flipping thinking mode after confirmed cache miss |

Each corresponds to a header or parameter that, if changed mid-session, would bust ~50,000-70,000 tokens of cached prompt. The latches sacrifice mid-session toggling to preserve the cache.

### Memoized Session Date

```
const getSessionStartDate = memoize(getLocalISODate)
```

Without this, the date would change at midnight, busting the entire cached prefix. A stale date is cosmetic; a cache bust reprocesses the entire conversation.

### Section Memoization

System prompt sections use a two-tier cache. Most content uses `systemPromptSection(name, compute)`, cached until /clear or /compact. The nuclear option `DANGEROUS_uncachedSystemPromptSection(name, compute, reason)` recomputes every turn — the naming convention forces developers to document WHY cache-breaking is necessary.

## Saving CPU in Rendering

Chapter 13 covered the rendering architecture in depth — the packed typed arrays, pool-based interning, double buffering, and cell-level diffing. Here we focus on the performance measurements and adaptive behaviors that keep it fast.

The terminal renderer throttles at 60fps via `throttle(deferredRender, FRAME_INTERVAL_MS)`. When the terminal is blurred, the interval doubles to 30fps. Scroll drain frames run at quarter interval for maximum scroll speed. This adaptive throttling ensures rendering never consumes more CPU than necessary.

The React Compiler (react/compiler-runtime) auto-memoizes component renders throughout the codebase. Manual useMemo and useCallback are error-prone; the compiler gets it right by construction. Pre-allocated frozen objects (Object.freeze()) eliminate allocations for common render-path values — one allocation saved per frame in alt-screen mode compounds over thousands of frames.

For the full rendering pipeline details — the CharPool/StylePool/HyperlinkPool interning system, the blit optimization, the damage rectangle tracking, the OffscreenFreeze component — see Chapter 13.

## Saving Memory and Time in Search

The fuzzy file search runs on every keystroke, searching 270,000+ paths. Three optimization layers keep it under a few milliseconds.

### The Bitmap Pre-Filter

Every indexed path gets a 26-bit bitmap of which lowercase letters it contains:

```
// Pseudocode — illustrates the 26-bit bitmap concept
function buildCharBitmap(filepath: string): number {
  let mask = 0
  for (const ch of filepath.toLowerCase()) {
    const code = ch.charCodeAt(0)
    if (code >= 97 && code <= 122) mask |= 1 << (code - 97)
  }
  return mask // Each bit represents presence of a-z
}
```

At search time: `if ((charBits[i] & needleBitmap) !== needleBitmap) continue`. Any path missing a query letter fails instantly — one integer comparison, no string operations. Rejection rate: ~10% for broad queries like "test," 90%+ for queries with rare letters. Cost: 4 bytes per path, ~1MB for 270,000 paths.

### Score-Bound Rejection and Fused indexOf Scan

Paths surviving the bitmap face a score ceiling check before the expensive boundary/camelCase scoring. If the best-case score cannot beat the current top-K threshold, the path is skipped.

The actual matching fuses position finding with gap/consecutive bonus computation using String.indexOf(), which is SIMD-accelerated in both JSC (Bun) and V8 (Node). The engine's optimized search is significantly faster than manual character loops.

### Async Indexing with Partial Queryability

For large codebases, loadFromFileListAsync() yields to the event loop every ~4ms of work (time-based, not count-based — adapting to machine speed). It returns two promises: queryable (resolves on first chunk, enabling immediate partial results) and done (full index complete). The user can start searching within 5-10ms of the file list becoming available.

The yield check uses `(i & 0xff) === 0xff` — a branchless modulo-256 to amortize the cost of performance.now().

## The Memory Relevance Side-Query

One optimization sits at the intersection of token efficiency and API cost. As described in Chapter 11, the memory system uses a lightweight Sonnet model call — not the main Opus model — to select which memory files to include. The cost (256 max output tokens on a fast model) is negligible compared to the tokens saved by not including irrelevant memory files. A single irrelevant 2,000-token memory costs more in wasted context than the side query costs in API calls.

## Speculative Tool Execution

The StreamingToolExecutor begins executing tools as they stream in, before the full response completes. Read-only tools (Glob, Grep, Read) can execute in parallel; write tools require exclusive access. The partitionToolCalls() function groups consecutive safe tools into batches: [Read, Read, Grep, Edit, Read, Read] becomes three batches — [Read, Read, Grep] concurrent, [Edit] serial, [Read, Read] concurrent.

Results are always yielded in the original tool order for deterministic model reasoning. A sibling abort controller kills parallel subprocesses when a Bash tool errors, preventing resource waste.

## Streaming and the Raw API

Claude Code uses the raw streaming API instead of the SDK's BetaMessageStream helper. The helper calls partialParse() on every input_json_delta — O(n²) in tool input length. Claude Code accumulates raw strings and parses once when the block is complete.

A streaming watchdog (CLAUDE_STREAM_IDLE_TIMEOUT_MS, default 90 seconds) aborts and retries if no chunks arrive, with fallback to non-streaming messages.create() on proxy failure.

## Apply This: Performance for Agentic Systems

**Audit your context window budget.** The gap between your max_output_tokens reservation and your actual p99 output length is wasted context. Set a tight default and escalate on truncation.

**Design for cache stability.** Every field in your prompt is stable or volatile. Put stable first, volatile last. Treat any mid-conversation change to the stable prefix as a bug with a dollar cost.

**Parallelize startup I/O.** Module loading is CPU-bound. Keychain reads and network handshakes are I/O-bound. Launch I/O before imports.

**Use bitmap pre-filters for search.** A cheap pre-filter rejecting 10-90% of candidates before expensive scoring is a significant win at 4 bytes per entry.

**Measure where it matters.** Claude Code has 50+ startup checkpoints, sampled at 100% internally and 0.5% externally. Performance work without measurement is guesswork.

A final observation: most of these optimizations are not algorithmically sophisticated. Bitmap pre-filters, circular buffers, memoization, interning — these are CS fundamentals. The sophistication is in knowing where to apply them. The startup profiler tells you where the milliseconds are. The API usage field tells you where the tokens are. The cache hit rate tells you where the money is. Measurement first, optimization second, always.


---

# Chapter 18: Epilogue — What We Learned

## Five Architectural Bets

Claude Code is not the only agentic system. It is not the first. But it made five architectural bets that distinguish it from the landscape of agent frameworks, and after nearly two thousand files and seventeen chapters, those bets deserve examination.

### Bet 1: The Generator Loop Over Callbacks

Most agent frameworks give you a pipeline: define tools, register handlers, let the framework orchestrate. The developer writes callbacks. The framework decides when to call them.

Claude Code does the opposite. The query() function is an async generator — the developer owns the loop. The model streams a response, the generator yields tool calls, the caller executes them, appends results, and the generator loops. There is one function, one data flow, one place where every interaction passes through. The 10 terminal states and 7 continuation states of the generator's return type encode every possible outcome. The loop is the system.

The bet was that a single generator function, even one that grew to 1,700 lines, would be more comprehensible than a distributed callback graph. After studying the source, the bet paid off. When you want to understand why a session ended, you look at one function. When you want to add a new terminal state, you add one variant to one discriminated union. The type system enforces exhaustive handling. A callback architecture would scatter this logic across dozens of files, and the interactions between callbacks would be implicit rather than visible in the control flow.

### Bet 2: File-Based Memory Over Databases

Chapter 11 made the case in detail, but the architectural significance extends beyond memory. The decision to use plain Markdown files instead of SQLite, a vector database, or a cloud service was a bet on transparency over capability. A database would support richer queries, faster lookups, and transactional guarantees. Files provide none of that. What files provide is trust.

A user who opens `~/.claude/projects/myapp/memory/MEMORY.md` in vim and sees exactly what the agent remembers about them has a fundamentally different relationship with the system than a user who must ask the agent "what do you remember?" and hope the answer is complete. The file-based design makes the agent's knowledge state externally observable, not just self-reported. This matters more than query performance. The LLM-powered recall system compensates for the storage simplicity with retrieval intelligence — a Sonnet side-query selecting five relevant memories from a manifest is more precise than embedding similarity and requires zero infrastructure.

### Bet 3: Self-Describing Tools Over Central Orchestrators

Agent frameworks typically provide a tool registry: you describe your tools in a central configuration, and the framework presents them to the model. Claude Code's tools describe themselves. Each Tool object carries its own name, description, input schema, prompt contribution, concurrency safety flag, and execution logic. The tool system's job is not to describe tools to the model — it is to let tools describe themselves.

This bet pays off in extensibility. MCP tools (Chapter 15) become first-class citizens by implementing the same interface. A tool from an MCP server and a built-in tool are indistinguishable to the model. The system does not need a separate "MCP tool adapter" layer — the wrapping produces a standard Tool object, and from that point forward, the existing tool pipeline handles it: permission checking, concurrent execution, result budgeting, hook interception.

### Bet 4: Fork Agents for Cache Sharing

Chapter 9 covered the fork mechanism: a sub-agent that starts with the parent's full conversation in its context window, sharing the parent's prompt cache. This is not a convenience optimization — it is an architectural bet that the cache sharing model is worth the complexity of fork lifecycle management.

The alternative — spawning a fresh agent with a summary of the conversation — is simpler but expensive. Every fresh agent pays the full cost of processing its context from scratch. A forked agent gets the parent's cached prefix for free (a 90% discount on input tokens), making it economical to spawn agents for small tasks: memory extraction, code review, verification passes. The background memory extraction agent (Chapter 11) runs after every query loop turn, and its cost is marginal precisely because it shares the parent's cache. Without fork-based cache sharing, that agent would be prohibitively expensive.

### Bet 5: Hooks Over Plugins

Most extensibility systems use plugins — code that registers capabilities and runs within the host process. Claude Code uses hooks — external processes that run at lifecycle points and communicate through exit codes and JSON on stdin/stdout.

The bet is that process isolation is worth the overhead. A plugin can crash the host. A hook crashes its own process. A plugin can leak memory into the host's heap. A hook's memory dies with its process. A plugin requires an API surface that must be versioned and maintained. A hook requires stdin, stdout, and an exit code — a protocol that has been stable since 1971.

The overhead is real: spawning a process per hook invocation costs milliseconds that an in-process callback would not. The -70% fast path for internal callbacks (Chapter 12) shows that the system knows this cost matters. But for external hooks — user scripts, team linters, enterprise policy servers — the isolation guarantee makes the system safer to extend. An enterprise can deploy hook-based policy enforcement without worrying that a malformed hook script will crash their developers' sessions.

## What Transfers, What Does Not

Not every pattern in Claude Code generalizes. Some are consequences of scale, resources, or specific constraints that other agent builders may not share.

### Patterns That Transfer to Any Agent

**The generator loop pattern.** Any agent that needs to stream responses, handle tool calls, and manage multiple terminal states benefits from making the loop explicit rather than hiding it behind callbacks. The discriminated union return type — encoding exactly why the loop stopped — is a pattern that eliminates an entire class of "why did the agent stop?" debugging sessions.

**File-based memory with LLM recall.** The specific implementation details are Claude Code's, but the principle — simple storage combined with intelligent retrieval — applies to any agent that needs to persist knowledge across sessions. The four-type taxonomy (user, feedback, project, reference) and the derivability test ("can this be re-derived from the current project state?") are reusable design heuristics.

**Asymmetric read/write channels for remote execution.** When reads are high-frequency streams and writes are low-frequency RPCs, separating them is correct regardless of the specific transport protocol.

**Bitmap pre-filters for search.** Any agent searching a large file index benefits from a 26-bit letter bitmap as a pre-filter. Four bytes per entry, one integer comparison per candidate — the cost-to-benefit ratio is remarkable.

**Prompt cache stability as an architectural concern.** If your agent uses an API with prompt caching, structuring the prompt with stable content first and volatile content last is not an optimization — it is an architectural decision that determines your cost structure.

### Patterns Specific to Claude Code's Scale

**The forked terminal renderer.** Claude Code forked Ink and reimplemented the rendering pipeline with packed typed arrays, pool-based interning, and cell-level diffing because it needed 60fps streaming in a terminal. Most agents render to a web interface or a simple log output. The engineering investment only makes sense when terminal rendering is your primary UI and you are streaming at high frequency.

**The 50+ startup profiling checkpoints.** Meaningful when you have hundreds of thousands of users and 0.5% sampling produces statistically significant data. For a smaller agent, a simpler timing system suffices.

**Eight MCP transport types.** Claude Code supports stdio, SSE, HTTP, WebSocket, SDK, two IDE variants, and a Claude.ai proxy because it must integrate with every deployment topology. Most agents need stdio and HTTP.

**The hooks snapshot security model.** Freezing hook configuration at startup and never re-reading it implicitly is a defense against a specific threat: malicious repository code modifying hooks after the user accepts the trust dialog. This matters when your agent runs in arbitrary repositories with untrusted `.claude/` configurations. An agent that only runs in trusted environments can use simpler hook management.

## The Cost of Complexity

Nearly two thousand files. What does that buy, and what does it cost?

The file count is misleading as a complexity metric. Much of it is test infrastructure, type definitions, configuration schemas, and the forked Ink renderer. The actual behavioral complexity concentrates in a small number of high-density files: query.ts (1,700 lines, the agent loop), hooks.ts (4,900 lines, the lifecycle interception system), REPL.tsx (5,000 lines, the interactive orchestrator), and the memory system's prompt building functions.

The complexity comes from three sources, each with a different character:

**Protocol diversity.** Supporting five terminal keyboard protocols, eight MCP transport types, four remote execution topologies, and seven configuration scopes is inherently complex. Each additional protocol is a linear addition to the codebase, not an exponential one — but the sum is large. This complexity is accidental in the Brooksian sense: it comes from the environment (terminal fragmentation, MCP transport evolution, remote deployment topologies), not from the problem being solved.

**Performance optimization.** The pool-based rendering, bitmap search pre-filters, sticky cache latches, and speculative tool execution each add complexity in exchange for measurable performance gains. This complexity is justified by measurement — every optimization was preceded by profiling data that identified the bottleneck. The risk is that optimizations accumulate and interact in ways that make the hot paths harder to modify.

**Behavioral tuning.** The memory system's prompt instructions, the staleness warnings, the verification protocol, the "ignore memory" anti-pattern instruction — these are not code complexity. They are prompt complexity, and they carry a different maintenance burden. When the model's behavior changes between versions, prompt instructions that were carefully tuned through evals may need re-tuning. The eval infrastructure (referenced throughout the codebase as case numbers and eval scores) is the defense against regression, but it requires ongoing investment.

The maintenance burden of this system is significant. A new engineer reading the codebase must understand not just the code paths but the eval outcomes that motivated specific prompt phrasings, the production incidents that motivated specific security checks, and the performance profiles that motivated specific optimizations. The code comments are thorough — many include eval case numbers and before/after measurements — but thorough comments in nearly two thousand files are themselves a reading burden.

## Where Agentic Systems Are Heading

Four trends are visible from the patterns in Claude Code, and they point toward where the field is going.

### MCP as the Universal Protocol

Chapter 15 described Claude Code as one of the most complete MCP clients. The significance is not Claude Code's implementation — it is that MCP exists at all. A standardized protocol for tool discovery and invocation means that tools built for one agent work with any agent. The ecosystem effects are obvious: an MCP server for Postgres, once built, serves every agent that speaks MCP. The developer's investment in tool integration is portable.

The implication for agent builders: if you are defining a custom tool protocol, you are probably making a mistake. MCP is good enough, it is getting better, and the ecosystem advantages of a standard protocol compound over time. Build an MCP client, contribute to the spec, and let the protocol evolve through community feedback.

### Multi-Agent Coordination

Claude Code's sub-agent system (Chapter 8), task coordination (Chapter 10), and fork mechanism (Chapter 9) are early implementations of multi-agent patterns. They solve specific problems — cache sharing, parallel exploration, structured verification — but they also reveal the fundamental challenge: coordination overhead.

Every message between agents consumes tokens. Every fork shares a cache but adds a conversation branch that the parent must eventually reconcile. The Task system's state machine (queued, running, completed, failed, cancelled) is coordination machinery that adds complexity without adding capability. As agents become more capable, the pressure will shift from "how do we coordinate multiple agents?" to "how do we make one agent capable enough that coordination is unnecessary?"

The current evidence suggests both approaches will coexist. Simple tasks will use single agents. Complex tasks will use coordinated multi-agent systems. The engineering challenge is making the coordination overhead low enough that the crossover point favors multi-agent for genuinely parallel work, not just for tasks that are complex.

### Persistent Memory

Claude Code's memory system is version 1 of persistent agent memory. The file-based design, the four-type taxonomy, the LLM-powered recall, the staleness system, and the KAIROS mode for long-running sessions are all first-generation solutions to a problem that will evolve significantly.

Future memory systems will likely add structured retrieval (the current system retrieves whole files; future systems might retrieve specific facts), cross-project transfer learning (user preferences that apply everywhere, project conventions that do not), and collaborative memory (Chapter 11's team memory is a first step, but the sync, conflict resolution, and access control are minimal).

The open question is whether the file-based approach scales. At 200 memories per project, it works. At 2,000 memories per project, the Sonnet side-query's manifest becomes too large, the consolidation becomes too expensive, and the index exceeds its caps. The architectural bet on files-over-databases will face its hardest test as usage grows.

### Autonomous Operation

The KAIROS mode, the background memory extraction agent, the auto-dream consolidation, the speculative tool execution — these are all steps toward autonomous operation. The agent does useful work without being asked: it remembers what you forgot to tell it to remember, it consolidates its own knowledge while you sleep, it starts executing the next tool before the current response is complete.

The trajectory is clear. Future agents will be less reactive and more proactive. They will notice patterns the user has not described, suggest corrections the user has not requested, and maintain their own knowledge without explicit /remember commands. Claude Code's memory system, with its background extraction safety net and its prompt-engineered "what to save" heuristics, is the prototype for this future.

The constraint is trust. Autonomous operation requires the user to trust that the agent will do the right thing when unattended. The file-based memory, the observable hook system, the staleness warnings, the permission dialogs — all of these exist because trust must be earned, not assumed. The path to more autonomous agents runs through more transparent agents.

## Closing

Seventeen chapters. Six core abstractions. A generator loop at the center, tools extending outward, memory reaching backward through time, hooks guarding the perimeter, a rendering engine translating it all into characters on a screen, and MCP connecting it to the world beyond the codebase.

The deepest pattern in Claude Code is not any single technique. It is the recurring decision to push complexity to the boundaries. The rendering system pushes complexity to the pools and the diff — inside the pipeline, everything is integer comparisons. The input system pushes complexity to the tokenizer and the keybinding resolver — inside the handlers, everything is typed actions. The memory system pushes complexity to the write protocol and the recall selector — inside the conversation, everything is context. The agent loop pushes complexity to the terminal states and the tool system — inside the loop, it is just: stream, collect, execute, append, repeat.

Each boundary absorbs chaos and exports order. Raw bytes become ParsedKey. Markdown files become recalled memories. MCP JSON-RPC becomes Tool objects. Hook exit codes become permission decisions. On one side of each boundary, the world is messy — five keyboard protocols, fragile OAuth servers, stale memories, untrusted repository hooks. On the other side, the world is typed, bounded, and exhaustively handled.

If you are building an agentic system, this is the transferable lesson. Not the specific techniques — you may not need pool-based rendering or KAIROS mode or eight MCP transports. But the principle: define your boundaries, absorb complexity there, and keep everything between them clean. The boundaries are where the engineering is hard. The interior is where the engineering is pleasant. Design for pleasant interiors, and invest your complexity budget at the edges.

The source code is open. The crab has the map in its claw. Go read it.
