# APM Workflow

This document is the formal specification of the APM workflow. It defines the phases, systems, and procedures that govern how agents coordinate to deliver project outcomes. This is a development-time specification: agents do not read this file during runtime. All commands, guides, and skills derive their behavior from this specification - the rules defined here take effect through those implementation files.

Terms used here are defined in `TERMINOLOGY.md`. Writing conventions follow `WRITING.md`.

---

## 1. Design

APM is a multi-agent project management framework that coordinates agents through file-based communication and structured planning documents.

**User-mediated coordination** - The User initiates agents, triggers bus checks, approves key decisions, and triggers Handoffs. All inter-agent communication passes through the file system; agents never communicate directly. The User decides when messages are delivered by running trigger commands in the appropriate agent's chat.

**Platform agnosticism** - Core concepts (the Message Bus, Memory, planning documents) are platform-independent. Platform-specific capabilities are additive and delivered through the build pipeline's conditional placeholder system.

**Context scoping** - All agents are identical - same tools, same capabilities. There is no mechanism that prevents an agent from reading any file. Context scoping is entirely behavioral, shaped by what each agent reads and does during its session: the initiation command, the guides and skills it references, what those guides direct the agent to read, what arrives through the bus, and everything the agent reads, executes, and accumulates during its work. Workers are scoped to their Task Prompts, Rules, and accumulated working context - they should not be aware of the Spec, Plan, Tracker, Index, or any coordination-level concepts. This scoping holds because nothing in the Worker's reading chain mentions these documents and no Task Prompt references them. The Manager holds coordination-level context (planning documents, Memory, the full project picture) and enforces Worker scoping by extracting relevant content into self-contained Task Prompts rather than referencing coordination-level documents. The Planner holds initial project context during the Planning Phase but does not participate during implementation.

**Structured memory** - Memory captures project history in a hierarchical file structure that enables Handoff continuity and efficient progress tracking.

**Subagent usage** - Agents spawn platform-native subagents for isolated, focused work: debug subagents (full access) for complex bugs, and research subagents (read-only) for knowledge gaps. These are behavioral expectations, not a closed set. The build pipeline replaces subagent guidance placeholders with platform-specific invocations at build time. Findings are integrated into the spawning agent's context; Workers log subagent usage in their Task Log.

**User-claimable Tasks** - Workers are the implementation team and execute Tasks dispatched by the Manager. The User may claim any Task at any point and execute it themselves. When the User holds a Task, an AI agent stays on the User's side - answering questions, running validation when the User returns, and writing the Task Log on the User's behalf. Which agent depends on where the User picks the Task up: the Manager when the Task is claimed during Manager coordination (including Tasks the Plan already assigned to the User), or the Worker assigned to the Task when the User takes over in the Worker's chat (in which case the Worker pauses execution). This AI agent on the User's side is referred to throughout this document as the **collaborating agent**. The User is not a Worker - no bus slot, no Worker registry entry, no Task Prompt delivered, no Rules to follow, and no Handoff procedure. APM Semi's typical pattern is the User claiming the critical or high-stakes work they want full personal ownership over and AI Workers handling boilerplate or peripheral Tasks that depend on it - though any Task is claimable at any point and the User may claim none, in which case full Manager-Worker dispatch operates unchanged.

---

## 2. Phases

### 2.1 Planning Phase

The Planner transforms User requirements into planning documents through two sequential procedures: Context Gathering, then Work Breakdown. During Context Gathering, the Planner detects sovereignty signals (areas the User wants direct ownership over) alongside its existing focuses, captured as observations about the work itself. During Plan Analysis, sovereignty observations inform per-Task assignment - Tasks aligned with clear signals are assigned to the User, and any remaining observations are recorded in Plan notes for the Manager. During Plan review the User can claim or unclaim Tasks conversationally as part of normal review feedback. After the User approves all three planning documents, the Planner initializes the Message Bus and directs the User to start the Implementation Phase by initiating the Manager.

### 2.2 Implementation Phase

The Manager and Workers transform the Plan into completed deliverables. The Implementation Phase begins when the User initiates the first Manager. The Manager determines its init path from the Handoff Bus: if the bus has content, the Manager is an incoming instance and processes the handoff; otherwise, it is the first instance. The first Manager reads planning documents, explores the workspace's git state, then presents an understanding summary alongside proposed version control conventions (combining Planner observations from the Spec with its own detection) for User approval - including any Stage boundaries where holistic verification may be warranted. On approval, the Manager writes VC conventions to the Tracker and Rules, initializes the Tracker with Task and Worker tracking, and enters a continuous coordination loop of dispatching Tasks, reviewing results, and maintaining project state. Each Task progresses through: Task Assignment → Task Execution → Task Logging → Task Review. Stages execute sequentially; Tasks within a Stage can be dispatched individually, in batches, or in parallel when version control provides workspace isolation. After all Stages complete, the Manager presents a project completion summary. When agent context window limits approach, Handoff transfers context to a successor instance.

The Manager simultaneously maintains two postures. Toward Workers, it is a dispatcher - writing Task Prompts to Task Buses, reading Task Reports, reviewing Task Logs. Toward the User, it is a direct collaborator in chat with no message bus between them - answering questions, surfacing recommendations, and collaborating on any Task the User claims during Manager coordination. The two postures run in parallel. Coordination of AI Workers continues while the Manager collaborates with the User on a claimed Task. A Worker may also become the collaborating agent when the User takes over an in-progress Task in the Worker's chat.

---

## 3. Planning Documents

| Document | Purpose | Location | Access |
| -------- | ------- | -------- | ------ |
| Spec | Define what is being built | `.apm/spec.md` | Manager reads directly; relevant content extracted into Task Prompts by Manager |
| Plan | Define how work is organized | `.apm/plan.md` | Manager reads directly; Task definitions extracted into Task Prompts by Manager |
| Rules | Define how work is performed | `{RULES_FILE}` at workspace root | All agents |

### 3.1 Spec

The Spec defines what is being built - project-specific design decisions and constraints. It resides at `.apm/spec.md` with a free-form markdown structure determined by project needs. The Spec informs the Plan.

Workers are not given the Spec - the Manager extracts relevant content into Task Prompts during Task Assignment, making them self-contained. The Manager may update the Spec during the Implementation Phase when execution findings warrant it.

### 3.2 Plan

The Plan defines how work is organized - Stages, Tasks, agent assignments, a dependency graph, and validation criteria. It resides at `.apm/plan.md`.

The Plan contains a header (project name, agents, Stages, dependency graph) followed by Stage sections containing Tasks. Each Task specifies an objective, output, validation criteria, guidance, dependencies, and steps. The dependency graph is a mermaid diagram that visualizes Task dependencies and agent assignments, enabling the Manager to identify batch candidates, parallel dispatch opportunities, and critical path bottlenecks.

The Plan accepts the User as a valid Task assignee. When the User claims a Task at planning time, the Plan records the assignee as the User and the Worker that would have owned it does not gain a Worker registry entry on the User's behalf. If a Worker ends up with no Tasks because the User claimed everything in its domain, the Planner removes that Worker from the registry. The User is never added to the Worker registry. Dependency graph, Stage structure, and parallel-dispatch logic are unchanged for User-owned Tasks - they participate in dependencies and parallel-dispatch decisions exactly like Worker-owned Tasks; the Manager simply collaborates with the User on them in chat instead of dispatching to a bus.

Task dependencies are listed in each Task's dependencies field. Cross-agent dependencies - between different Workers - are bolded for visual distinction.

Workers are not given the Plan - the Manager extracts Task content into Task Prompts during Task Assignment. The Manager may update the Plan during the Implementation Phase, updating the dependency graph when modifications affect Task dependencies.

### 3.3 Rules

Rules define how work is performed - universal execution patterns applied during Task Execution. They are stored within the APM Rules block in `{RULES_FILE}` at the workspace root. Content outside this block is user-managed and preserved. When relevant standards already exist outside the block, the APM Rules block references them rather than duplicating.

All agents have direct access to this file. Both the Manager and Workers may update Rules during the Implementation Phase.

Rules are read directly by every Worker in the project - including Workers assigned to different domains. A frontend Worker and a backend Worker read the same file. Content is applicable to all or most Tasks. Rules are framed conditionally on the work being performed rather than on a specific Worker or domain, so any Worker can determine whether a rule applies to its current work. Narrowly domain-specific execution patterns belong in Task Prompts via Spec extraction, not here.

Version control is the Manager's operational domain. The Manager creates a feature branch for every AI Worker dispatch, Workers always operate on their assigned branch, and parallel dispatch uses worktrees for workspace isolation. User-owned Tasks operate in the main working directory on whatever branch the User chooses, with whatever git flow the User prefers - the Manager does not pre-create branches or worktrees for the User. When a User-owned Task is Active concurrently with AI Worker dispatch, the Manager uses a dedicated merge worktree at `.apm/worktrees/.merge/` to perform AI Worker merges without disturbing the User's main directory; concurrent AI Worker dispatches always run in worktrees regardless of single/batch/parallel classification, since main is held by the User. The Manager establishes all VC conventions during first init: it explores the workspace's git state, reads any VC observations the Planner recorded in the Spec, and confirms conventions with the User. When no conventions are detected or specified, the Manager proposes lightweight defaults: `type: description` commits (feat, fix, refactor, docs, test, chore) and `type/short-description` branches. APM does not push to remotes by default. When `.apm/` resides inside a repository directory, the Manager adds `.apm/` to `.gitignore` by default and asks the User if they want to track any `.apm/` artifacts in git (planning documents, Memory). If the User declines version control entirely, the Manager notes that parallel dispatch is unavailable. Commit conventions are universal execution patterns (every Worker commits following the same format) and belong in Rules. Branch naming conventions are recorded in the Tracker's Version Control section. When the workspace contains multiple repositories with different conventions, per-repository conventions are captured in the Tracker and only conventions that are genuinely universal across all repositories belong in Rules.

### 3.4 Document Modification

The Spec and Plan have bidirectional influence - changes to one may require adjustments in the other. Rules are generally isolated from architecture and coordination-level documents. When modifying any planning document, the Manager assesses cascade implications before executing changes.

**Manager authority** (small contained changes):

- Single Task clarification or correction
- Adding a missing dependency
- Isolated Spec addition
- Minor Rules adjustment

**User collaboration required** (significant changes):

- Multiple Tasks affected
- Design direction change
- Scope change
- New Stage addition or major restructure
- Multiple small modifications that together represent significant change

---

## 4. Communication System

**Runtime:** `skills/apm-communication/SKILL.md`, `skills/apm-communication/bus-integration.md`

### 4.1 Communication Models

Agent communication follows three models based on audience:

**Agent-to-user communication.** Agents explain decisions and actions to Users in natural language. No framework vocabulary - section references (§N.M), procedure step names, checkpoint labels, decision categories - is exposed. Only terms defined in `TERMINOLOGY.md` are used formally. When describing decisions, agents explain what happened, what was decided, and what happens next - not which procedure branch was taken. When directing Users to take action, agents provide specific actionable guidance naturally: which command to run, in which agent's chat, with what arguments. Commands are presented in code format for easy copying. Communication at workflow transitions orients the User on what was completed and what comes next.

**Visible reasoning.** Agents present analysis visibly in chat so the User can understand, review, and audit their decisions - explaining assessments, justifying choices, and surfacing trade-offs. Internal reasoning may reach conclusions before visible output begins, but visible analysis in chat must still walk through the reasoning for User audit. When a procedure defines a reasoning frame with a header and labeled aspects, the agent presents the header visibly and addresses the aspects as natural analytical prose beneath it. Some frames are hierarchical with sub-headers for distinct phases. When no frame is defined, agents present their assessment, key considerations, and conclusion. The communication skill defines the standards.

**Agent-to-agent and agent-to-system communication.** Structured per schemas and format specifications. Bus messages, artifact writing, memory logs. Governed by the communication skill (bus protocol) and each guide's structural specifications (artifact formats).

### 4.2 Message Bus

The Message Bus is a file-based communication mechanism in `.apm/bus/`. The Planner initializes it at the end of the Planning Phase. Each Worker has a bus directory containing three bus files. The Manager has a bus directory containing only a Handoff Bus.

| Bus File | File Name | Direction | Contains |
| -------- | --------- | --------- | -------- |
| Task Bus | `task.md` | Manager → Worker | Task Prompts (single or batched) |
| Report Bus | `report.md` | Worker → Manager | Task Reports |
| Handoff Bus | `handoff.md` | Outgoing → incoming agent | Handoff prompt content |

A bus file is either empty (no message present) or contains a message awaiting delivery. Before writing to an outgoing bus file, an agent clears its incoming bus file. This prevents stale messages from accumulating and signals that the previous message was processed. Agents always read a bus file before writing to it to ensure cross-platform file tool compatibility.

Workers read their Task Bus when the User runs `/apm-4-check-tasks` in the Worker's chat. The Manager reads Report Buses when the User runs `/apm-5-check-reports`. Both commands accept optional agent identifier arguments for targeted delivery.

When dispatching multiple sequential Tasks to the same Worker, the Manager sends them as a batch in a single Task Bus message. Each Task Prompt within the batch retains its full standalone structure.

### 4.3 Communication Flow

1. Manager writes a Task Prompt to a Worker's Task Bus and provides the User with specific action guidance - which command to run, in which agent's chat, and whether the Worker needs initialization first.
2. User runs the indicated command(s) in the Worker's context.
3. Worker executes the Task, writes a Task Log, writes a Task Report to the Report Bus, and directs the User to deliver the report - including the agent identifier for targeted retrieval.
4. User runs `/apm-5-check-reports` in the Manager's chat.
5. Manager reviews the report and log, determines next steps.

The User is the trigger puller at every boundary - there is no direct agent-to-agent communication. Each agent provides concise, actionable guidance covering only their end of the exchange.

### 4.4 Non-APM Agent Participation

Agents not managed by APM can participate in bus communication by creating their own agent directory under `.apm/bus/`. They operate at the communication level only - receiving assignments and reporting results. Non-APM agents do not log to Memory, perform Handoff, or appear in Worker tracking. The Manager assigns follow-up work through the agent's Task Bus and processes reports through standard bus delivery. The communication skill's bus integration guide provides setup details.

### 4.5 User Collaboration

The User-Manager relationship is direct chat with no message bus between them. The Manager has full project context visible to the User in conversation. During an active session, the User may also chat directly with any Worker in that Worker's chat - by default for delivering Manager-mediated Task Prompts, and additionally to take over an in-progress Task. Direct User-Worker chat during a takeover is conversational and does not pass through the bus.

Whenever the User holds a Task, an AI agent serves as the collaborating agent on the User's side. The Manager fills this role for any Task the User claims while it has not yet been dispatched - including Tasks the Plan assigned to the User and Tasks the User claims later in the Manager's chat. The Worker fills this role when the User takes over a Task that Worker is currently executing - the Worker pauses cleanly, does not write a partial Task Log, and does not report the takeover to the Manager at takeover time (the Manager learns when the eventual Task Report and Task Log arrive, which include a marker noting that a takeover occurred and at what point).

In either case the collaborating agent opens the collaboration with a Task Brief - the conversational counterpart to a Task Prompt, built from the same three-source synthesis (relevant Spec content, dependency context, and Plan Task fields - objective, steps, guidance, output, validation criteria) and written as a brief to the User rather than as instructions to an executor. The Manager-side Brief carries everything the User needs to do the work from scratch. The Worker-side Brief is the same Brief shape but with progress baked in: what was done before the takeover, files touched, decisions made, what remains, and what the Worker was about to do next. The flavor of in-the-moment context each side can offer differs - the Manager has coordination-level context (cross-Worker contracts, Spec ambiguities, dependency implications, Stage timing), the Worker has execution-level context - but the collaboration shape is identical.

While the User executes, the collaborating agent stays on standby - answering questions at its level, running validation when the User returns, and writing the Task Log on the User's behalf. The Manager continues to coordinate AI Workers on independent Tasks in parallel. The Task is held in Active state with an ownership marker for the User; the User may pause and resume freely.

**Communicating collaboration mechanics.** Claim, unclaim, takeover, takeover return, and report-done have no commands - they are conversational. Agents orient the User on the conversational mechanic relevant to the moment the same way they orient the User on which command to run at workflow boundaries. The communication skill defines the phrasing standards.

---

## 5. Memory

### 5.1 Structure

Project persistence resides in `.apm/` with this hierarchy:

```text
.apm/
├── tracker.md
├── memory/
│   ├── index.md
│   ├── stage-<NN>/
│   │   └── task-<NN>-<MM>.log.md
│   └── handoffs/
│       ├── manager/
│       │   └── handoff-<NN>.log.md
│       └── <agent>/
│           └── handoff-<NN>.log.md
```

**Tracker** (`tracker.md`) is the live project state document. It contains Task tracking, Worker tracking, version control state, and working notes. The Manager updates it throughout the Implementation Phase as the operational view for dispatch decisions, dependency analysis, and Handoff continuity.

**Index** (`memory/index.md`) is the project's durable memory. It contains Memory notes (observations with lasting impact on future coordination) and Stage summaries (historical record of each Stage, appended after completion). Memory notes come first - forward-looking knowledge is hit first by incoming Managers. Memory notes include two first-class categories alongside general observations: User behavior patterns observed during the session (claim and unclaim tendencies, validation-handling preferences, areas where the User has been particularly engaged or hands-off), and User preferences and stated positions (timing constraints on claimed Tasks, specific coordination preferences). The Manager seeds awareness from sovereignty observations carried into the Plan (User-owned Tasks and Plan notes) and accumulates evolving session signal as Memory notes during the Implementation Phase. These notes inform the Manager's claim-suggestion behavior and survive Manager Handoffs.

**Task tracking** (within Tracker) records Task statuses, ownership, and branch state per Stage. The Manager updates it after each Task Review. Tasks progress through lifecycle states per `TERMINOLOGY.md` §4: Waiting, Ready, Active, Done. Each Task carries a current owner - either an AI Worker (the default) or the User. Ownership transitions are reflected in the Tracker as they happen. Historical claim and unclaim cycles are not tracked in the Tracker; the Tracker holds current state. History lives in Task Logs and as durable Memory notes.

**Worker tracking** (within Tracker) records Worker identifiers, instance numbers, and notes. Workers start as uninitialized and transition to Instance N when initialized. The Manager updates it when Workers are first dispatched to, and when Handoffs are detected. Cross-agent overrides are recorded below the Worker table when Worker Handoffs reclassify dependencies.

**Working notes** (within Tracker) are coordination context accumulated during the Stage - User preferences, coordination insights, technical observations, patterns noticed during reviews. At Stage end, working notes are distilled: observations with lasting impact become Memory notes, Stage-specific observations become Stage summary prose. Working notes are inserted and removed as context evolves.

**Stage directories** (`memory/stage-<NN>/`) contain Task Logs for each Stage. Workers write Task Logs directly to the specified `log_path`; the file write operation creates parent directories as needed.

**Task Logs** are structured logs written by Workers after Task completion. They capture outcome status, validation results, deliverables, and flags.

**Handoff Logs** (`memory/handoffs/<agent>/`) are logs created during Handoff containing working context not captured elsewhere.

### 5.2 Task Log Flags

Workers set flags based on what they observe during execution. The Manager interprets these flags with full project awareness:

| Flag | Interpretation |
| ---- | -------------- |
| `important_findings` | The Worker observed something potentially beyond the Task's scope. The Manager assesses whether it affects planning documents or other Tasks. |
| `compatibility_issues` | The Worker observed dependency or system conflicts. The Manager assesses whether this indicates issues with the Plan, Spec, or Rules. |

When uncertain whether a finding warrants a flag, Workers set it to true. False negatives harm coordination more than false positives.

### 5.3 Task Outcome Status

Task outcome status reflects whether the objective was achieved:

| Status | Definition |
| ------ | ---------- |
| Success | Objective achieved, all validation passed |
| Partial | Some progress made; the Worker needs guidance to continue |
| Failed | Objective not achieved; the Worker attempted but could not resolve the issue |

---

## 6. Planning Phase

### 6.1 Context Gathering

**Runtime:** `commands/apm-1-initiate-planner.md` (§2), `guides/context-gathering.md`

The Planner gathers project requirements through three progressive rounds of questions, deriving technical formalization from natural User responses rather than asking Users to produce technical content directly.

**Workspace assessment** - Before question rounds begin, the Planner scans the workspace to orient itself: directory structure, git repositories (commit history, branch structure as project signals), `{RULES_FILE}` contents, and the location of existing materials (documentation, requirements, specs). The Planner reads `{RULES_FILE}` freely as structural orientation. When the User's initiation context references specific documents or describes the project with enough specificity to imply document authority, the Planner treats those materials as authoritative - reads them and may explore based on them. For materials discovered during scanning without initiation context authority, the Planner lists them for the User alongside the first round of questions, asking which are current and relevant. On confirmation, the Planner reads them and may use them as a basis for deeper exploration. The workspace assessment produces an initial understanding of the project environment that informs question rounds and feeds into the Spec's Workspace section during Work Breakdown.

**Round 1 - Existing Materials and Vision.** Project type, problem, scope, skills, existing documentation, current vision.

**Round 2 - Technical Requirements.** Design decisions and constraints, work structure, dependencies, technical requirements, validation criteria.

**Round 3 - Implementation Approach and Quality.** Technical constraints, workflow preferences, quality standards, coordination needs, domain organization, design decisions and constraints.

The Planner operates with progressive awareness of the workflow - understanding that gathered context feeds into three planning documents with different audiences, that the Manager handles all coordination at runtime, and that technical research needed for accurate planning must be resolved during Context Gathering rather than deferred to execution. This awareness shapes question selection toward project substance and away from coordination mechanics.

Each round follows an iteration cycle: ask initial questions, assess gaps after each response, follow up until understanding is complete, present a round summary, advance. Before advancing, the Planner includes an open-ended question targeting gaps the focused questions may have missed. When User responses reference codebase elements, the Planner proactively explores before continuing - subagent usage is encouraged to avoid context bloat. After subagent results return, the Planner verifies critical claims by reading referenced files or running referenced commands directly - subagent summaries compress and sometimes misrepresent details that shape planning. When verification contradicts the subagent's findings or reveals important context that needs deeper investigation, the Planner dispatches a follow-up subagent with a more targeted scope to resolve the discrepancy, then verifies the follow-up's claims in turn. The Planner then presents verified findings to the User before incorporating them into round reasoning. When gathered context reveals multiple viable approaches to an architectural or design question, the Planner presents alternatives with a recommendation rather than selecting autonomously. When exploring a codebase that is a git repository, the Planner checks git history and branch structure as project signals alongside other exploration. The Planner captures validation criteria for each requirement, proposing concrete measures when the User does not specify them. Version control signals and coordination preferences observed during Context Gathering are noted as factual observations - placement into planning documents happens during Work Breakdown. Context Gathering produces signals and constraints about the project - never decomposition structures, agent assignments, or planning vocabulary (Stages, Tasks, Workers, tracks, phases, task sizing).

After all rounds, the Planner presents a consolidated understanding summary for User review. Modifications loop back through targeted follow-ups. User approval triggers transition to Work Breakdown.

**Sovereignty signal detection.** Alongside its existing focuses, the Planner listens for signals that the User wants direct ownership over part of the work - domain preferences, areas the User has stressed about, parts the User describes as critical or wants full personal knowledge of, parts described with strong personal language, or explicit statements that the User intends to do certain work themselves. APM Semi's typical pattern is the User taking on critical or high-stakes work and AI Workers handling boilerplate or peripheral work that depends on it - this lens shapes how the Planner reads signals. Sovereignty signals are not a separate question round and not a domain-by-domain procedure; they surface organically through the existing rounds. The Planner may surface a recommendation alongside normal questioning when a signal appears, but remains a question-asker and does not pressure the User toward claiming work. Signals are captured as observations about the work itself, framed in the same vocabulary as other gathered context - decomposition is not yet underway. They feed into Work Breakdown the same way other gathered context does.

### 6.2 Work Breakdown

**Runtime:** `commands/apm-1-initiate-planner.md` (§3-4), `guides/work-breakdown.md`

The Planner decomposes gathered context into planning documents through visible reasoning - thinking is presented in chat before file output. The User sees decomposition decisions and can redirect before artifacts are written. The Planner's workflow awareness deepens at this stage: understanding how each document is consumed (the Manager extracts Spec content per-Task into Task Prompts, enriches them at runtime, and Workers focus on their Task Prompt and Rules by design), shaping content placement decisions.

**Sequence:**

1. **Spec Analysis** - The Planner analyzes design decisions, writes the Spec (including a Workspace section capturing the project environment: directory structure, working repositories, reference repositories, authoritative documents, and existing Rules file content), and presents it for User approval. The Planner may include blockquote notes after the Spec header separator with observations about the project environment for the Manager: version control observations, workspace constraints, and User preferences that affect execution.
2. **Plan Analysis** - The Planner identifies work domains and maps them to Workers, reasons about how domain characteristics create stage boundaries, identifies the Tasks each Stage requires, then deepens the analysis per Task. Per-Task analysis includes a sovereignty consideration aspect: the Planner reasons from the captured sovereignty signals whether the Task should be assigned to the User instead of an AI Worker - clear alignment yields a User assignment, borderline alignment leaves the Task with its Worker and is surfaced as a recommendation during Plan review. After dependency verification and completeness checks, the Planner writes the full Plan and presents it for User approval. The Planner may include blockquote notes after the Plan header separator with work structure observations (why certain boundaries exist, natural groupings or sequencing patterns, critical path, convergence points, Stage boundaries where holistic verification may be valuable) and any sovereignty observations from Context Gathering that did not translate into User-owned Tasks. Notes are awareness for the Manager - observations and reasoning that inform judgment, not instructions.
3. **Rules Analysis** - The Planner extracts universal execution patterns from gathered context (excluding version control conventions, which the Manager handles), writes the APM Rules block, and presents it for User approval. Rules govern Worker execution only and are not read by the User; Tasks already assigned to the User in the Plan therefore do not contribute to which rules end up in the block.

**User claims during review.** Plan review is a normal modification gate - the User reviews the Plan and may request changes. Claim and unclaim are conversational - the User can ask to take any Task or hand back any Task assigned to them, and the Planner applies the change. If the User claims no Tasks at planning, the Plan looks identical to a non-collaborative Plan and the Implementation Phase begins as a normal session - until and unless the User claims something dynamically later.

**Task decomposition principles.** Each Task produces a meaningful deliverable with clear boundaries, scoped to a single Worker's domain, with specified validation criteria. Steps within Tasks support failure tracing but have no independent validation. Subagent steps are included when investigation or research is needed. Decomposition granularity adapts to project size and complexity - smaller projects warrant lighter breakdown.

---

## 7. Implementation Phase

### 7.1 Task Assignment

**Runtime:** `guides/task-assignment.md`, `commands/apm-4-check-tasks.md`

The Manager assesses readiness, determines dispatch mode, constructs Task Prompts, and delivers them via the Task Bus.

**Dispatch assessment** - The Manager identifies Ready Tasks from the Tracker, groups them by Worker, and forms dispatch units. Three dispatch modes exist:

| Mode | Description | Prerequisites |
| ---- | ----------- | ------------- |
| Single | One Task dispatched to one Worker | Task is Ready |
| Batch | Multiple Tasks dispatched to the same Worker in one message | Tasks form a sequential chain or are independent same-Worker Tasks all Ready simultaneously |
| Parallel | Dispatch units sent to different Workers simultaneously | No unresolved cross-agent dependencies; version control workspace isolation |

For parallel dispatch, the Manager recommends the User configure platform tool approvals for Workers to minimize interactive wait times during execution.

Before dispatching, the Manager checks whether a pending report would unlock Tasks that combine well with the current unit - if waiting costs little and a plausible combination exists, the Manager may wait. When no Tasks are Ready but Workers are active, the Manager communicates what was processed, what is pending, and what the User should do next.

**Per-Task analysis** - For each Task, the Manager synthesizes content from three sources into the Task Prompt: dependency context (familiarity classification, producer Task Log content when applicable, integration guidance), relevant Spec content (design decisions and constraints for this Task, extracted inline), and Plan Task fields (objective, steps, guidance, output, validation criteria). Rules are not included in Task Prompts - Workers read `{RULES_FILE}` directly and the Task Prompt assumes those standards are in effect. Content from planning documents and authoritative sources is embedded directly - the Manager never references the Spec or Plan by path. Content that exists in the codebase (source files, patterns, configurations) is referenced through targeted reading instructions - pointing the Worker to specific files and what to look for rather than embedding file contents. Dependency context depth depends on Worker familiarity with the producer's work. Same-agent dependencies receive light context: recall anchors and file paths. Cross-agent dependencies receive comprehensive context: file reading instructions, output summaries, and integration guidance. After a Worker Handoff, previous-Stage same-agent dependencies become cross-agent because the incoming Worker lacks that working context. For Workers that recovered from auto-compaction, the Manager provides more comprehensive same-agent dependency context since reconstructed context may lack working nuance. The Manager traces dependency chains upstream when ancestors established patterns, schemas, or contracts the current Task must follow.

**Task Prompt construction** - The Manager assembles each Task Prompt as a self-contained document with metadata (Stage, Task, agent identifier, log path, dependency indicator) and a body containing the Task objective, dependency context, detailed instructions, expected output, validation criteria, instruction accuracy guidance, iteration guidance, logging instructions, and reporting instructions. For parallel dispatch, the Task Prompt includes the workspace path where the Worker operates. Workers commit to their assigned branch following conventions from Rules and note the workspace in their Task Log. Workers only commit - the Manager handles branch creation, merging, and cleanup. For each dispatch, the Manager writes to the Worker's Task Bus and directs the User to the Worker's chat with specific action guidance. For uninitialized Workers, the Manager directs the User to create a new chat and initialize with the agent identifier; for initialized Workers, to deliver the Task Prompt. For batch dispatch, the Manager summarizes what the Worker will receive. For parallel dispatch, the Manager lists each Worker chat with its required action.

**Follow-up Task Prompts** - When a Task Review determines retry is needed, the Manager issues a follow-up. The follow-up is a new Task Prompt with objective, instructions, output, and validation refined based on what went wrong. It uses the same log path as the original (the Worker overwrites the previous log) and includes context explaining the issue and required refinement.

**User-owned Task path** - When a User-owned Task becomes Ready, the Manager does not dispatch to a Task Bus. Instead, in the Manager's chat, it presents a Task Brief built from the same three-source synthesis as a Task Prompt (relevant Spec content, dependency context, Plan Task fields including objective, steps, guidance, output, and validation criteria), but written conversationally as a brief rather than as instructions to an executor. The Task Brief carries everything the User would need to do the work. The User-owned Task workspace is the main working directory - the User picks their branch and follows whatever git flow they prefer; the Manager does not pre-create branches or place the User in a worktree per §3.3 Rules. The Manager then stays on standby for that Task while continuing to coordinate AI Workers on independent Tasks in parallel - any AI Worker dispatch concurrent with the User Task runs in a worktree, and merges are performed in the merge worktree per §3.3 to leave main undisturbed. The same Brief construction applies on the Worker side when the User takes over mid-execution per §7.2 Task Execution - the Worker presents a Brief with execution progress baked in instead of starting from scratch. See `{GUIDE_PATH:task-assignment}` for Task Brief construction.

### 7.2 Task Execution

**Runtime:** `commands/apm-3-initiate-worker.md`, `guides/task-execution.md`, `guides/task-logging.md`

The Worker executes Task instructions, validates results, iterates if needed, logs the outcome, and reports back.

**Worker registration** - A Worker binds to an agent identity during initiation by resolving the provided agent identifier against `.apm/bus/` directory names. This identity persists for the duration of the Implementation Phase for this Worker instance. The Task Prompt's agent identifier field is used for cross-validation, not identity binding. After registration, the Worker checks bus state to determine the init path: if the Handoff Bus has content, the Worker is an incoming instance and processes the handoff; if the Task Bus has content (with or without a preceding handoff), the Worker reads the Task Prompt and begins executing immediately; if neither has content, the Worker awaits Task delivery via `/apm-4-check-tasks`.

**Execution flow** - The Worker integrates dependency context if present, executes steps sequentially, then validates per the Task Prompt's validation criteria. The Worker validates autonomously first (running checks, verifying outputs), then pauses when criteria require User involvement (judgment or action) - the Worker does not involve the User until autonomous checks pass. When validation fails, the Worker investigates the root cause before attempting a correction - reading error output, tracing the failure, and identifying what specifically went wrong. If the correction does not resolve the issue, the Worker spawns a debug subagent to investigate root causes and iterate on fixes in a fresh context rather than continuing in the main context. The Worker validates the subagent's findings before applying them. When a Task includes subagent steps, the Worker spawns the relevant subagent and integrates findings into execution.

**Batch execution** - When receiving a batch of Tasks, the Worker executes them sequentially. After each Task, a Task Log is written immediately before proceeding to the next. If any Task results in a Failed status, execution stops. After completing all Tasks (or stopping on failure), the Worker writes a batch Task Report to the Report Bus containing outcomes for each Task.

**User corrections and Rules integration** - When the User provides a correction or directive during execution, the Worker complies immediately and continues without interruption. At Task completion, the correction is noted in the Task Log as an important finding - the Manager sees it during Task Review regardless. After logging, reporting, and directing the User, the Worker asks at the end of its turn whether the correction should become a Rule for all Workers. If approved, the Worker updates Rules and the Task Log. If not, the Manager still has visibility through the important findings flag.

**Completion** - After execution, the Worker commits work to the assigned branch following conventions from Rules (if version control is active), writes a Task Log, clears the incoming bus file, writes a Task Report to the Report Bus, and directs the User to deliver the report - providing both the targeted command with agent identifier and the general command, since multiple Workers may finish concurrently. For large Tasks, Workers may commit at logical intermediate points during execution rather than only at completion - each commit follows the conventions from Rules and represents a coherent unit of change.

**Mid-execution takeover** - At any point during Worker execution, the User may take over by signaling so in natural language in the Worker's chat. There is no command for this. The Worker pauses cleanly: it does not write a partial Task Log and does not report the takeover to the Manager at takeover time. It then presents a takeover-time Task Brief - the same Brief construction the Manager uses for User-owned Tasks, but with progress baked in: what was done so far, files touched, decisions made, what remains, and what the Worker was about to do next. From there, the Worker is the collaborating agent for the remainder of the Task - on standby for execution-level questions while the User does the remaining work however they choose. When the User reports back, the Worker proceeds to validation per `{GUIDE_PATH:task-execution}` and then to Task Logging.

**Reporting done** - When the User finishes a claimed Task (in either the Manager's chat or a Worker's chat), the User reports back via natural language. The report is expected to include what was done, any narrative context worth capturing, and validation status - which validation criteria the User checked themselves and which the User wants the collaborating agent to handle. The collaborating agent does not enforce a strict report format; it interprets the User's message, cross-references the Task's validation criteria, and proceeds to validation iteration. If the User's report omits validation status entirely, the collaborating agent must explicitly check before proceeding rather than silently assuming completeness. The collaborating agent peeks at the git diff (or equivalent ground-truth signal) before writing the log so the log accurately reflects what changed even if the User's narrative is incomplete.

**Validating User-completed work** - After the report, the collaborating agent runs whichever validation steps remain. On failure, it attempts to resolve the issue itself within a bounded scope: small surface area, mechanical or near-mechanical, no new design decisions, no domain logic the User owns. Within that scope it iterates with one targeted change per attempt, leveraging Ad-Hoc subagent dispatch (debug, research, verification) when the work would otherwise consume too much agent context. The agent stops and escalates as soon as the failure looks systemic (architectural, design-level, multi-area) or persistent (not resolving across iterations even when the agent understands the issue) - no fixed iteration count is required. When escalating, the agent presents what was attempted, what worked, what did not, hypotheses about why, and the cleaned-up state of the code: changes that produced real progress are kept; exploratory or unsuccessful changes are reverted. If nothing produced progress, the code returns to exactly what the User delivered. The User never cleans up after the agent and decides whether to continue iterating or hand back with new direction. This applies whether the collaborating agent is a Manager or a Worker - each handles within its own scope and consults the other only when the failure points there.

**Task Logging for User-owned Tasks** - The collaborating agent writes the Task Log for any Task the User owned, at the standard Memory path for that Task (Stage directory, standard file naming). The log follows the standard Task Log format with additions: an ownership field indicating the User owned this Task, a takeover marker noting where the User took over and what was complete at that point if applicable, a breakdown of who did what during execution and validation (which steps the User performed, which the collaborating agent performed during validation iteration, what was attempted, what was kept, what was reverted), and the User's narrative context. The log remains compatible with all downstream consumption (Stage summaries, dependency context for downstream Tasks, Handoffs) without special-casing. When the Manager is the collaborating agent, the Manager writes the log directly to the standard Memory path; format and consumption are identical to a Worker-written log. When a Worker is the collaborating agent (after a takeover), the Worker writes the log following its standard procedure with the same additions.

### 7.3 Task Review

**Runtime:** `guides/task-review.md`, `commands/apm-5-check-reports.md`

The Manager reviews Worker results, determines review outcomes, modifies planning documents when needed, and updates the Tracker.

**Report processing** - The Manager reads the Task Report from the Report Bus. For batch reports, each Task's outcome is processed individually. Unstarted Tasks from a stopped batch re-enter the dispatch pool.

**Handoff detection** - An incoming Worker (post-Handoff) indicates in its first Task Report that it is a new instance and lists the specific Task Logs it loaded during Handoff initialization and notes that previous-Stage logs were not loaded. When the Manager detects this, it verifies the Handoff Log exists, compares the loaded logs against all Tasks previously completed by this Worker, and records cross-agent overrides in the Tracker for any completed Tasks whose logs were not loaded. The Manager checks these overrides during future Task Assignments to determine dependency context depth.

**Log review** - The Manager reads the Task Log, interprets status, flags, and body content. Consistency between claimed status and actual content is assessed.

**Review outcome** - If the log shows Success with no flags and the content supports the status, the Manager proceeds to the next Task. If version control is active and the Worker's Task was successful but changes remain uncommitted on the task branch, the Manager commits on behalf following the conventions from Rules. If something needs attention (flags raised, non-Success status, or inconsistencies), the Manager investigates - directly for contained checks, using a subagent for context-intensive issues. After investigation, three outcomes are possible: no issues found, follow-up needed (the Manager issues a follow-up Task Prompt), or planning document modification needed (the Manager assesses cascade per §3.4 Document Modification). For non-Success outcomes with uncommitted changes, the follow-up notes that the Worker did not commit. When investigation reveals deficiencies in previously-Done work, the Manager creates a new Task through plan modification rather than reopening the original - Done is terminal per `TERMINOLOGY.md` §4. Every outcome path ends with updating the Tracker. During parallel dispatch, reports arrive asynchronously; the Manager processes each as it arrives, merges the completed Task's branch, reassesses readiness, and dispatches newly Ready Tasks in a single turn.

**Stage verification** - After all Tasks in a Stage complete, the Manager assesses whether the Stage's deliverables require holistic verification before proceeding - based on what the User confirmed during the understanding summary, observations accumulated during Task Reviews, and Planner notes. When verification is needed, the Manager executes checks and examines the deliverables. When verification reveals issues, the Manager responds based on scope: fixing contained issues directly, dispatching a subagent for focused investigation, creating a new Task through Plan modification for Worker-level work, or presenting findings to the User when the direction is unclear.

**Stage summary** - After all Tasks in a Stage complete and any verification concludes, the Manager reviews the Stage's Task Logs and appends a Stage summary to the Index capturing Stage-level outcomes, agents involved, notable findings, and references to individual logs.

**Handling residuals** - Residuals identified after Task Review (or during ongoing coordination) are judged for inline handling rather than automatic follow-up dispatch. Inline handling qualifies when the residual is small in surface area, mechanical or near-mechanical, with no new design decisions, no new abstractions, and no domain logic owned by a Worker's specialization. Within that scope, the Manager either handles the residual directly (action recorded in the parent Task's log or as a Tracker note) or offers it to the User - choice informed by the same session signal that drives proactive claim suggestions. Beyond that scope, the Manager dispatches a follow-up to the appropriate Worker, or escalates to the User if the residual implies a Plan or Spec change.

**Proactive claim suggestions** - Beyond per-Task review, the Manager surfaces opportunities for the User to claim Tasks based on accumulated session signal: sovereignty observations from Context Gathering (User-owned Tasks the Planner already assigned and additional observations recorded in Plan notes), claim and unclaim history during the session, observed behavior patterns (ownership tendencies, validation handling, engagement across areas), and stated positions the User has expressed. The Plan-borne observations seed the Manager's awareness; session-derived signals accumulate as Memory notes in the Index and survive Manager Handoffs. At Stage boundaries the Manager actively reviews upcoming Tasks against these inputs and offers claims for relevant ones before dispatching; in-Stage it stays quiet and only volunteers a suggestion when triggered by a concrete signal (a residual that qualifies for offer-to-User, a qualifying follow-up, a Task Review finding touching a sovereignty area). It does not nag. Two hard constraints: a Task is never marked User-owned without explicit User confirmation; and these are suggestions - the User can decline, defer, or counter at any time, and the Manager proceeds with whatever the User decides.

**Routing follow-up work** - When Task Review reveals follow-up work that exceeds the inline-handling scope, the Manager presents the follow-up to the User with the available paths: dispatch to the responsible AI Worker, the User takes it themselves, or (if it actually does qualify for inline handling on closer look) the Manager handles it directly. The Manager's recommendation informs which path it leans toward, but the User makes the call.

### 7.4 Claim and Unclaim Mechanics

The User can claim any Task at any point. Whichever agent currently has context becomes the collaborating agent for that Task: the Manager for unassigned or planning-time-claimed Tasks, the Worker for in-progress Tasks. The User can unclaim any Task they hold. Whatever the User did so far becomes context the Manager packages into a normal Task Prompt for the responsible AI Worker, which then resumes the Task. If no Worker was originally assigned (the User claimed at planning), the Manager picks the best-fit Worker from the registry, possibly with User input. The User does nothing else. The User cannot claim and never return: agents need input to proceed, so a non-responding User leaves the relevant agent in standby. The Implementation Phase pauses on the User. There is no timeout, no auto-unclaim, and no escalation - those would violate ownership. A Task already marked Done is not claimable; if the User attempts it, the Manager notes the Task is complete and asks whether the User wants to revisit anything specific. Claims and unclaims happen via natural language in chat with the appropriate agent.

### 7.5 Stage Boundary Handling

At Stage boundaries, in addition to standard Stage-completion behavior (verification when needed, Stage summary written to the Index, proceeding to the next Stage's first dispatch), the Manager reviews the upcoming Stage's Tasks against accumulated session signal and proactively surfaces claim suggestions to the User per §7.3 before beginning the new Stage's dispatches.

---

## 8. Handoff and Continuity

### 8.1 Handoff

**Runtime:** `commands/apm-6-handoff-manager.md`, `commands/apm-7-handoff-worker.md`

Handoff transfers context between successive instances of the same agent role when context window limits approach. It applies to the Manager and Workers only - the Planner operates as a single instance. Handoff is User-initiated and can occur at any point (mid-Task, between Tasks, while awaiting reports) as long as the handoff prompt captures comprehensive current state.

**Two artifacts** - Handoff produces two artifacts with distinct purposes:

| Artifact | Location | Content | Lifecycle |
| -------- | -------- | ------- | --------- |
| Handoff prompt | Handoff Bus | Current state and continuation: outstanding Tasks, mid-Task progress, pointers to logs, continuation guidance | Ephemeral; cleared after incoming agent processes it |
| Handoff Log | Memory | Past actions: working context, decisions made, approaches tried, technical notes | Persistent archival context |

**Worker and Manager asymmetry** - Workers and the Manager have different Handoff characteristics due to their bus clearing behavior. A mid-Task Worker Handoff occurs while the Task Bus still contains the original Task Prompt (the bus has not been cleared yet); the handoff prompt references the intact Task Prompt directly. A mid-batch Worker Handoff occurs while the Task Bus still contains the batch envelope; the handoff prompt describes the state of each Task in the batch - which are complete, which is in progress and how far, and which have not been started. A between-Tasks Worker Handoff occurs after the Task Bus was cleared; the handoff prompt states readiness to await the next Task. A Manager Handoff must describe outstanding Tasks in full because Workers may have already cleared their Task Buses.

**Context rebuilding** - An incoming Manager reads the Handoff Log, the Tracker, the Index (Stage summaries and Memory notes), and relevant recent Task Logs to reconstruct working context. An incoming Worker reads the Handoff Log and their own current-Stage Task Logs only. When previous Stages exist, the incoming Worker notes that previous-Stage logs were not loaded. The Manager accounts for this limited context in future Task Prompts by treating previous-Stage same-agent dependencies as cross-agent.

**Manager Handoff carries collaborative context** - In addition to the standard Manager Handoff content, the Manager's Handoff prompt and Handoff Log carry current Task ownership state for any User-held Tasks, evolving sovereignty observations as they currently stand, observed User behavior patterns, and pending claim suggestions the Manager planned to surface but has not yet. The Plan notes carry the original sovereignty observations from Context Gathering and the Index carries User behavior patterns and preferences accumulated during the session; the Handoff prompt fills in any working-knowledge gaps. The new Manager instance, on initiation, reads the Plan (including notes), the Index, and the Handoff prompt, and resumes both postures - coordinator toward Workers and collaborator toward the User - without dropping context on either side.

**Worker Handoff during a takeover** - A Worker may be in a paused-for-takeover state when its context fills. The Handoff proceeds normally; the new Worker instance picks up the paused-waiting state and resumes when the User returns to the Worker's chat.

### 8.2 Recovery

**Runtime:** `commands/apm-9-recover.md`

Recovery reconstructs context after platform auto-compaction, manual compaction, or a cleared or lost conversation. The User invokes the recovery command with the agent's role. The agent re-reads its initiation command and follows its document loading instructions to rebuild procedural knowledge, then explores project artifacts and the codebase to reconstruct operational state. Recovery does not increment the instance number. The agent notes the recovery event in subsequent communications and in its eventual Handoff Log.

Recovery for a Manager that has been collaborating with the User on User-owned Tasks reconstructs that state from the Index, the Tracker's ownership fields, and any in-progress Task Briefs visible in chat history.

### 8.3 Session Continuation

**Runtime:** `commands/apm-8-summarize-session.md`, `agents/apm-archive-explorer.md`, `guides/context-gathering.md` (§3.1)

Session continuation archives the current session's artifacts for future reference. After archival, the user reinitializes APM to begin a new session with fresh templates while retaining read access to previous session context.

**Archive structure** - Archived sessions reside in `.apm/archives/`. Each archive is a directory named `session-YYYY-MM-DD-NNN` (zero-padded daily counter) containing the session's planning documents, Tracker, and Memory. The bus directory is not archived - bus state is ephemeral and session-specific.

```text
.apm/archives/
├── index.md
├── session-2026-03-04-001/
│   ├── metadata.json
│   ├── plan.md
│   ├── spec.md
│   ├── tracker.md
│   ├── session-summary.md     # optional
│   └── memory/
│       ├── index.md
│       ├── stage-01/
│       └── handoffs/
└── session-2026-03-05-001/
    └── ...
```

**Archive marker** - `metadata.json` within each archive directory is the canonical archive marker. Its presence identifies a valid archive. It contains the original installation metadata with archival timestamp.

**Session summary** - An optional artifact (`session-summary.md`) produced by a standalone agent via the summarization command. When present, it provides a point-in-time snapshot of the session: project scope, stage outcomes, key deliverables, notable findings, known issues, and current codebase state including how deliverables relate to `.apm/` artifacts. Can be produced at any point during a session, not only after completion. When absent, the archive contains raw artifacts sufficient for future Planners to examine.

**Archive index** - `.apm/archives/index.md` is a table listing all archived sessions with date, scope, stages, and tasks. The summarization command updates it; if absent or malformed, it is recreated.

**Planner archive detection** - During Context Gathering (§6.1), the Planner checks for `.apm/archives/`. If archives exist, the Planner presents them to the User and asks about relevance. If the User indicates archives are relevant, the Planner uses the `apm-archive-explorer` custom subagent to examine indicated archives, then verifies findings against the current codebase before integrating into question rounds.

**Manager completion recommendation** - After project completion (§2.2), the Manager recommends running the summarization command in a new chat to produce a session summary. The summarization agent may offer archival as a follow-up step; archival is also available directly via the CLI (`apm archive`).

**`apm-archive-explorer` subagent** - A custom subagent shipped with APM bundles. It understands archive structure and efficiently extracts relevant context from archived sessions. The Planner spawns it during Context Gathering when archived sessions are relevant.

**No secondary archival** - Archives accumulate in `.apm/archives/`. There is no mechanism to archive archives. Users manage cleanup manually.

---

**End of Workflow Specification**
