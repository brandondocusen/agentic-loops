
# Agentic Loops: Language-Agnostic Best Practices

**Pillar by Pillar**

A practical guide to building **reliable agentic loops** and **loop engineering** patterns.

This guide distills hard-won lessons from real agent implementations into a zero-dependency, framework and language-agnostic design pattern. It focuses on what turns expensive drifting conversations into verifiable, compounding progress.

Review, comprehension, and final ownership stay with humans.
Built to replace the human as the iterator.

---

## Table of Contents

- [Core Definition](#core-definition)
- [Asynchronous Construction Order](#asynchronous-construction-order)
- [The Pillars](#the-pillars)
  - [Pillar 1: Trigger + Explicit Verifiable Goal](#pillar-1-trigger--explicit-verifiable-goal)
  - [Pillar 2: External Persistent State (The Memory Spine)](#pillar-2-external-persistent-state-the-memory-spine)
  - [Pillar 3: Compounding Project Knowledge (Skills)](#pillar-3-compounding-project-knowledge-skills)
  - [Pillar 4: The Observe-Reason-Plan-Act-Observe Cycle](#pillar-4-the-observe-reason-plan-act-observe-cycle)
  - [Pillar 5: Tools With Real Effect](#pillar-5-tools-with-real-effect)
  - [Pillar 6: Objective Verification Gate](#pillar-6-objective-verification-gate)
  - [Pillar 7: Guardrails and Termination Controls](#pillar-7-guardrails-and-termination-controls)
- [Composable Blocks](#composable-blocks)
- [Cross-Cutting Concerns](#cross-cutting-concerns)
- [Anti-Patterns](#anti-patterns)
- [References](#references)

---

## What This Guide Is

This is a **best practices blueprint** for **reliable agent loops** drawn from primary sources including Addy Osmani’s *Loop Engineering*, Simon Willison’s *Designing Agentic Loops*, Steve Kinney’s *Anatomy of an Agent Loop*, and recent 2026 discussions on agentic systems.

The core philosophy: treat the loop as a true design pattern. This is realized through files, git, processes, and human judgment rather than fragile prompts or black-box frameworks.

The goal is a **zero-dependency, framework and language-agnostic** description of what makes a loop effective for software engineering.

### Core Definition

A loop is distinguished from prompting or single-shot agent use by these invariants:

- It replaces the human as the iterator.
- It runs toward a **verifiable end state**, there is an identifiable finish line.
- State that survives context resets lives **outside** the model (filesystem, git, issue trackers, etc.).
- Every significant action is followed by an **objective signal** that is fed back.
- Guardrails bound cost, time, and drift regardless of what the model says.

If any of these are missing, you'll have an expensive drifting conversation, not a loop.

### Asynchronous Construction Order

Build in phases. Within and across phases, many pillars and blocks can be strengthened independently (asynchronously). Early pillars should deliver value before moving forward.

**Phase 0 — Non-Negotiable Foundations** (build these first and together)

- Pillar 1: Trigger + Explicit Verifiable Goal
- Pillar 2: External Persistent State
- Pillar 3: Compounding Project Knowledge
- Pillar 6: Objective Verification Gate (minimal version)

**Special Note about Phase 0: 
Why Phase 0 deliberately excludes the cycle (Pillar 4) and tools (Pillar 5):**  
These four are the smallest set that already delivers loop-like reliability *even with a weak or manual driver*. A human repeatedly feeding goal + state + last verification back to an agent is already a major upgrade over unstructured prompting — *provided* the goal is externally checkable, progress survives in files, project knowledge is re-injected, and an objective gate (not the actor) can say "not done yet." Adding an autonomous Observe-Reason-Plan-Act-Observe inner loop + powerful mutation tools before you have that independent gate is the classic failure mode the sources warn against: fast, confident iteration on the wrong thing or premature victory declarations. Phase 0 gets the external contracts and honesty mechanism in place first.

**Phase 1 — The Working Cycle** (once Phase 0 is reliable)

- Pillar 4: The Observe-Reason-Plan-Act-Observe Cycle
- Pillar 5: Tools With Real Effect
- Pillar 7: Guardrails and Termination Controls (first version)

**Phase 2 — Reliability Layer** (add as soon as the basic cycle runs without immediate catastrophe)

- Stronger verification signals
- Context management discipline
- No-progress and drift detection

**Phase 3 — Scaling and Power Blocks** (add only after the loop is observably useful and contained on small tasks)  
These blocks have weaker ordering dependencies among themselves and can be introduced in the order that matches your highest pain or highest leverage.

### The Pillars

#### Pillar 1: Trigger + Explicit Verifiable Goal

**Purpose**: Defines when the loop starts and what “done” actually means in observable terms.

**Best practices**:

- The goal must be stated in terms of **external, checkable outcomes**, not internal model satisfaction.
- Include explicit success criteria whenever possible (files that must exist and contain specific content, commands that must exit successfully with expected output, artifacts that must be present, user-visible behaviors that can be demonstrated).
- Make the goal the single source of truth for termination. Everything else (TODO lists, plans) is subordinate to it.
- Write the goal in a place the loop (and humans) will read on every invocation.

**Minimal zero-dependency realization**: A plain text file (GOAL.md or equivalent) or a clearly delineated section at the top of the persistent state file. The criteria can be natural language that a deterministic checker or a later independent reviewer can evaluate against the filesystem and command outputs.

**Common failure mode**: Vague goals (“make it better”, “improve the UI”). These produce infinite refinement or confident but unverified output.

**Asynchronous note**: You can start with a weak but explicit goal and strengthen the criteria over time as you learn what actually matters.

#### Pillar 2: External Persistent State (The Memory Spine)

**Purpose**: The model is stateless between sessions. The only thing that compounds is what lives on durable storage.

**Best practices**:

- State must survive process death, context resets, and model changes.
- State should be human-readable and human-editable (markdown or similarly transparent formats win).
- State should record not just “what to do” but what was tried, what failed, key decisions, and verification outcomes.
- Use the filesystem + git as the primary integration layer between loop runs and between humans and the loop.

**Minimal zero-dependency realization**: One or more markdown files (LOOP_STATE.md / TODO.md / PROGRESS.md) that are read at the start of every run and written after significant work. Git provides history and branching as a free side effect.

**Common failure modes**: Keeping all progress only in the current conversation window; using opaque vector stores or databases as the first (instead of last) persistence mechanism; letting the model “remember” important constraints only in its context.

#### Pillar 3: Compounding Project Knowledge (Skills)

**Purpose**: Prevent the loop from re-deriving the same project conventions, commands, gotchas, and intent on every invocation.

**Best practices**:

- Project knowledge must be read fresh on every new agent start or major context reset.
- It should contain exact commands, folder semantics, non-obvious constraints, past incidents, and success criteria patterns.
- It is human-curated and human-maintained. The loop can propose additions, but a human decides what becomes durable knowledge.
- Name and scope the knowledge narrowly so the right skill can be invoked or automatically selected.

**Minimal zero-dependency realization**: One primary file (AGENTS.md, SKILL.md, CLAUDE.md, or equivalent) at the project root, plus optional narrowly-scoped files in a skills/ directory when complexity grows. The loop prompt or invocation process always injects the current content of these files.

**Common failure mode**: Treating the skill file as optional documentation instead of required context that is mechanically re-injected.

#### Pillar 4: The Observe-Reason-Plan-Act-Observe Cycle

**Purpose**: The repeatable unit of work that turns a goal into progress.

**Best practices**:

- Every iteration should produce an observable change in the external world (files, command output, git state) or a deliberate decision to gather more information.
- Planning is not a separate phase that happens once; it is continuous and should be recorded in external state.
- After every mutating action, the next observation must include the direct result of that action plus any verification signal.
- The cycle must be able to continue across process restarts without losing its place.

**Minimal zero-dependency realization**: The loop process (whether a single long-running script, a shell while loop, or a human repeatedly invoking an agent) repeatedly performs: read current goal + state + last results → reason → record next plan or todo in state → take one or more actions via available tools → observe the direct outcome. The “tools” can be anything the environment already provides (shell, editor commands, git, build tools, etc.).

#### Pillar 5: Tools With Real Effect

**Purpose**: The loop must be able to change the world and receive accurate feedback about the change.

**Best practices**:

- Prioritize high-leverage, low-ambiguity tools over large numbers of overlapping ones.
- Prefer surgical operations (targeted edits) over bulk rewrites when possible.
- Tools should return clear, structured, bounded results (exit codes, stdout/stderr, before/after diffs, file existence).
- The same tool surface should be usable both by the loop and by a human reviewer for debugging.
- Tools must be scoped to the current working context (worktree, container, or clearly bounded directory).

**Minimal zero-dependency realization**: Direct access to the shell / command interpreter of the environment plus basic file operations (read, write, list, search). Everything else (build, test, lint, git, package managers) is invoked through the shell as the universal integration point.

#### Pillar 6: Objective Verification Gate

**Purpose**: This is the primary mechanism that prevents the loop from declaring victory prematurely. Self-assessment by the actor that made the change is weak.

**Best practices**:

- Verification must run after significant changes and its output must be treated as first-class context.
- Prefer deterministic, external signals (command exit codes, file contents, build artifacts, syntax checks that use the project’s own tools) over model judgment.
- When deterministic signals are weak or absent (greenfield work), the gate can include an independent reviewer process (different context window, restricted tool access, or a separate invocation) that evaluates against the original goal and explicit criteria.
- The gate should be able to say “not yet” even when the acting agent believes it is finished.
- The gate itself can be improved over time; early versions can be simple and still useful.

**Minimal zero-dependency realization**: After any mutating step, run a verification command or script (whatever the environment already supports for syntax, build, or smoke checks) and capture its full output. Compare against explicit criteria written in the goal or state file. A second, fresh invocation with read-only access + the goal + the verification report can serve as an independent checker when no stronger signal exists yet. The loop only considers the goal met when the gate (not the actor) agrees.

**Asynchronous note**: A weak but consistently applied gate is far better than a sophisticated one that is only used occasionally. You can bootstrap verification by letting the loop itself create the first smoke checks as part of the work.

#### Pillar 7: Guardrails and Termination Controls

**Purpose**: Loops that are not bounded will eventually do expensive, destructive, or infinite things.

**Best practices**:

- Hard limits on iterations, wall-clock time, and (where measurable) cost must exist outside the model’s control.
- No-progress detection (repeated identical or near-identical verification fingerprints, tool results, or state) is required in addition to iteration caps.
- The loop must have a way to stop cleanly and leave usable state behind even when it is terminated externally.
- Destructive or high-risk operations should require either explicit goal authorization or additional human confirmation layers.

**Minimal zero-dependency realization**: An outer driver (shell script, simple scheduler, or the loop process itself) enforces maximum iterations and timeouts on individual steps. A hash or normalized summary of the latest verification output detects stagnation. The persistent state file is always written before risky actions and on clean shutdown.

### Composable Blocks (Add These Asynchronously)

These can be introduced in almost any order once the pillars above are providing value. Choose based on the dominant failure mode you are seeing.

- **Independent Verifier / Sub-agent Separation**  
    A second process or invocation (different context, restricted tools, stronger model, or explicit rubric) evaluates whether the primary actor’s work satisfies the goal. This is the structural upgrade to Pillar 6.
    
- **Isolation Mechanisms**  
    Worktrees, separate clones, containers, or other environment boundaries so that parallel or experimental runs cannot corrupt each other or the main line of development.
    
- **Automations and Scheduling (The Heartbeat)**  
    Triggers that discover work, triage it, and start new loop instances without a human typing the goal each time. The outer scheduling layer is what turns a runnable loop into a running system.
    
- **Reflection and Memory Distillation**  
    After runs (or at explicit checkpoints), a process extracts durable lessons, failure patterns, successful strategies, or updated constraints and writes them into the project knowledge files or a dedicated patterns/skill area. This is how the loop gets smarter over calendar time rather than just within one run.
    
- **Connectors to External Systems**  
    Controlled access to issue trackers, code review systems, chat, databases, or deployment targets. These turn the loop from “edits files” into “participates in the actual development process.”
    
- **Parallelism and Orchestration**  
    Multiple loop instances or specialized sub-processes working on different parts of a goal, with explicit coordination through shared state or a manager process.
    
- **Instrumentation and Observability**  
    Structured logging of every loop run (goal, iterations, verification outcomes, cost proxies, final state). This is the data needed to improve the loop itself.
    
- **Context Compaction and Summarization Discipline**  
    Explicit strategies (state files, todo recitation, tool result truncation, sub-agent summaries) to keep the active context useful even as runs become long.
    

### Cross-Cutting Concerns

- **Human Judgment Remains the Ceiling**  
    The loop accelerates work the engineer already understands. It does not safely replace understanding. Review, comprehension, and final ownership stay with humans.
    
- **Start Tiny, Then Layer**  
    The highest-leverage first version is usually: one clear goal + state file + knowledge file + one verification gate + one bounded execution environment. Everything else is elaboration.
    
- **Measure What Matters**  
    Track cost per accepted change, time to first useful verification signal, and rate of no-progress terminations. Token count and raw iteration count are secondary.
    
- **The Environment Is the Integration Layer**  
    Because we prescribe zero dependencies, the loop is glued together by the filesystem, shell, git, and whatever build/test/run commands the project already uses. Any additional machinery must justify itself against this baseline.
    

### Anti-Patterns (What Turns Loops Into Expensive Failures)

- Relying on the actor agent to decide when it is done.
- Keeping all memory and plans only inside the current context window.
- Treating verification as optional or post-hoc.
- Adding sub-agents, parallelism, or fancy memory before a single reliable gate exists.
- Running without hard iteration or time bounds in the outer driver.
- Allowing the loop to operate with broad destructive permissions on a shared main working tree.
- Writing goals that cannot be evaluated without another human doing the same work the loop was supposed to do.

This structure is intentionally ordered by leverage and risk rather than by implementation difficulty. You can (and should) have a useful, safe, language-agnostic loop long before you have the full set of advanced blocks. The pillars above, built with nothing more than files, clear criteria, native shell access, and disciplined process, are sufficient to outperform unstructured agent use on well-scoped engineering tasks.

---

References: 

https://simonwillison.net/2025/Sep/30/designing-agentic-loops/

https://addyosmani.com/blog/loop-engineering/

https://datasciencedojo.com/blog/agentic-loops-explained-from-react-to-loop-engineering-2026-guide/

https://stevekinney.com/writing/agent-loops

https://blogs.oracle.com/developers/what-is-the-ai-agent-loop-the-core-architecture-behind-autonomous-ai-systems

https://x.com/RLanceMartin/status/2064397389189071163

https://x.com/0xCodez/status/2064374643729773029

https://x.com/sairahul1/status/2064277888216555684
