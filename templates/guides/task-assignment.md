# APM Semi {VERSION} - Task Assignment Guide

## 1. Overview

**Reading Agent:** Manager

This guide defines how you construct and deliver Task Prompts for Workers, manage version control workspace isolation, and coordinate bus-based delivery. Task Prompts are self-contained - Workers receive everything needed to execute a Task without referencing the Spec or Plan.

### 1.1 Outputs

- *Task Prompt:* Content written to Task Bus for Worker to receive.
- *Follow-up Task Prompt:* Refined prompt when the review outcome determines retry.
- *Feature branches:* One branch per dispatch unit, created off the base branch.
- *Worktrees:* Isolated working directories under `.apm/worktrees/` for parallel dispatch.

---

## 2. Operational Standards

### 2.1 Dependency Context Standards

Tasks may depend on outputs from previous Tasks. The context you include depends on the Worker's familiarity with the producer's work.

**Same-agent dependencies:** The Worker previously completed the producer Task and has working familiarity. Provide light context - recall anchors, key file paths, brief reference to previous work. Detail increases with dependency complexity.

**Cross-agent dependencies:** A different Worker completed the producer Task. The Worker has zero familiarity. Provide comprehensive context - explicit file reading instructions, output summaries, integration guidance. Assume nothing.

**After Worker Handoff:** The incoming Worker only has current-Stage Task Logs loaded. Current-Stage same-agent dependencies remain same-agent. Previous-Stage same-agent dependencies are reclassified as cross-agent because the incoming Worker lacks that working context. Check cross-agent overrides in the Tracker during dependency analysis to determine which Tasks have been reclassified.

**Dependency identification:** Check the Task's Dependencies field in the Plan. Cross-agent dependencies are bolded. "None" indicates no dependencies.

**Chain reasoning:** Dependencies may have their own dependencies. Trace upstream when ancestors established patterns, schemas, or contracts the current Task must follow. Stop tracing when an intermediate node fully abstracts what came before. When uncertain whether an ancestor is relevant, include rather than risk missing critical context.

### 2.2 Task Prompt Content Standards

Task Prompts must be self-contained. Workers have the same tools as any agent but are intentionally scoped to their Task Prompt, Rules, and accumulated working context to keep them focused on execution. You enforce this scoping by extracting relevant content from the Spec, Plan, and authoritative sources into each prompt rather than referencing those documents by path. Never reference the Spec, Plan, Tracker, or Index by path - Workers should not read them. Task Prompt instructions and objectives do not reference Stage numbers, other Task IDs, or coordination-level concepts (dependency context sections reference producer Tasks by ID as needed). Validation criteria are Worker-scoped.

**Embed** content the Worker cannot discover from the codebase alone: design decisions and constraints from the Spec, Task definitions and guidance from the Plan, Task-relevant coordination context from the Tracker, observations from the Index, corrected findings from previous Tasks, and content from authoritative User documents the Spec references. Preserve specificity with exact constraints, not summaries. Present all embedded content as direct factual context. Never attribute content to its source artifact or use coordination-level vocabulary - Workers should not be aware of the Spec, Plan, Tracker, Index, or Memory - surfacing these concepts breaches their execution-focused scope.

**Reference with reading instructions** content that exists in the codebase: source files, existing patterns, configurations. Point the Worker to specific files and what to look for in them - the Worker reads them directly from their workspace. This applies to both dependency context and Spec content that references codebase patterns. The Manager identifies which files matter and what to look for, rather than embedding their contents.

**Exclude** content relating to other domains, providing background without actionable requirements, or already captured in the Task's Guidance field.

### 2.3 Follow-Up Standards

Follow-up Task Prompts occur when the review outcome determines retry after investigation. You arrive with: original Task Log findings, investigation results, understanding of what went wrong, and potentially modified planning documents.

**Content principle:** The follow-up is a new prompt - Objective, Instructions, Output, and Validation are refined based on what went wrong. Do not copy the previous prompt. The Worker operated with scoped context; your follow-up bridges the gap between what the Worker saw and what you now know from investigation, other Task completions, and planning document updates. Give the Worker concrete direction rather than restating the original Task Prompt.

**Log path continuity:** Use the same `log_path` as the original. The Worker overwrites the previous log. The Manager captures iteration patterns in Stage summaries when relevant.

### 2.4 Dispatch Standards

Before constructing individual Task Prompts, assess dispatch opportunities across Ready Tasks.

**Task readiness:** A Task is Ready when all its dependencies are Done. Read the Tracker for current statuses; cross-reference the Dependency Graph for newly unblocked Tasks.

**Dispatch modes.** Assess all Ready Tasks, group by Worker, and form dispatch units:
- *Batch:* Multiple Ready Tasks for the same Worker, dispatched together. Candidates either form a sequential chain (each depends only on the previous or already-complete Tasks) or are an independent group (no dependencies between them, all Ready simultaneously). When forming chains, weigh whether external Tasks depend on intermediate results - if so, dispatching individually allows earlier review and unblocks dependent Workers sooner. Soft guidance is 2-3 Tasks per batch.
- *Single:* one Ready Task for a Worker.
- *Parallel:* two or more dispatch units (any mix) with no unresolved cross-agent dependencies among them, dispatched simultaneously. Requires version control workspace isolation.

**Parallel dispatch prerequisites:** Version control must be initialized (established during Manager 1 initiation per `{COMMAND_PATH:apm-2-initiate-manager}` §2.1 First Manager Initiation). If version control is not active, fall back to sequential dispatch. Recommend the User configure platform tool approvals for Workers to minimize interactive wait times during parallel execution.

Before dispatching a ready unit, check whether a pending report would unlock Tasks that combine well with the current unit. If it is the only outstanding report, waiting costs little. If multiple reports are pending or no plausible combination exists, dispatch immediately.

**Wait state:** When no Tasks are Ready but Workers are still active, communicate what was processed, what is pending, and what the User should do next. Direct the User to return the next report - if a pending report would unlock a better dispatch combination, recommend prioritizing that report.

### 2.5 Version Control Standards

Version control provides workspace isolation during parallel dispatch. Each dispatch unit operates on its own feature branch, and you coordinate all merges during Task Review. When multiple repositories are listed in the Tracker's Version Control table, identify which repository each Task operates in from the Spec's Workspace section. If the User initially declined version control but later requests it mid-session, initialize it: run `git init` if needed, detect or confirm the base branch, establish conventions with the User, update Rules and the Tracker, then proceed with branch-based dispatch.

**Branch standards:** Every dispatch unit gets its own feature branch off the base branch per the branch convention in the Tracker. APM terminology (Task IDs, Stage numbers, agent identifiers) does not appear in branch names, commit messages, or worktree directory names - these reflect the actual work, not the framework managing it. A batch of sequential Tasks assigned to the same Worker shares one branch.

**Worktree standards:** Worktrees are created only for parallel dispatch. Each parallel dispatch unit gets its own worktree so all parallel Workers operate in isolated directories and the main working directory remains on the base branch for merge operations. For sequential dispatch, the Worker operates in the main working directory on their feature branch.

- *Layout:* Worktrees placed under `.apm/worktrees/` per §4.3 Branch and Worktree Standards.
- *Concurrency limit:* maximum 3-4 concurrent worktrees.
- *Lifecycle:* short-lived - created before dispatch, removed after merge.

Worktrees contain only tracked files; if a Worker needs untracked assets, note this in the Task Prompt. When `.apm/` is tracked (or partially tracked), the worktree may contain `.apm/` files but all APM runtime operations (Task Logs, bus communication) must target the project root's `.apm/`, not the worktree copy. You need to read Task Logs and bus files for review before merging, so they must be accessible from the main working directory. Include this guidance in the Task Prompt's Workspace section for worktree dispatch.

### 2.6 Delivery Standards

Bus directories and files are created by the Planner during the Planning Phase - do not re-create them. Before writing to a Worker's Task Bus, clear the Worker's Report Bus (`.apm/bus/<agent-slug>/report.md`) via terminal (e.g., `truncate -s 0` or shell redirection). Skip clearing on first Task Prompt to a Worker when no report exists. Read the Task Bus before writing to it per `{SKILL_PATH:apm-communication}` §4 Message Bus Protocol. When dispatching multiple sequential Tasks to the same Worker, send them as a batch in a single Task Bus message per §4.6 Batch Envelope Format.

### 2.7 Non-APM Agent Dispatch

When a non-APM agent has joined the session and you need to assign follow-up work to it, write a plain assignment to its Task Bus - not a full Task Prompt. Include what to do and what to produce, and instruct it to report back. Do not include log paths, logging instructions, or Handoff metadata - non-APM agents do not log to Memory or participate in Worker tracking.

### 2.8 Task Brief Standards

When a Task is owned by the User, deliver a Task Brief in chat instead of dispatching a Task Prompt to a Task Bus. The Task Brief is the conversational counterpart to a Task Prompt: built from the same three-source synthesis (relevant Spec content, dependency context, and Plan Task fields - Objective, Steps, Guidance, Output, Validation), but written as a brief to the User rather than as instructions to an executor. The same Brief shape is reused on the Worker side when the User takes over a Task mid-execution per `{GUIDE_PATH:task-execution}` §2.6 User Takeover Standards - the Worker presents the Brief with execution progress baked in instead of starting from scratch.

**What changes from Task Prompt to Task Brief:**

- *Audience.* The User is stepping in to execute the Task themselves - not a Worker. No bus slot, no Worker registry entry, no Rules to follow on the User's behalf, no Handoff procedure. Do not include Workspace branch instructions intended for Workers, Task Iteration guidance directed at iteration limits, Task Logging instructions, or Task Report instructions - these are mechanics for AI Workers writing through the bus. You write the Task Log on the User's behalf per `{GUIDE_PATH:task-review}` §3.6 User-Owned Task Collaboration.
- *Tone.* Conversational, not instructional. Present the Task as a brief: what is being asked, the design context, dependencies that bear on the work, the deliverable, and the validation criteria. The User decides how to do the work.
- *Coverage.* The same three-source synthesis still applies. The Brief carries everything the User would need to do the work themselves. Embed Spec content directly. Include dependency context at the same depth a Worker would receive (cross-Worker dependencies still warrant comprehensive context). Reference codebase files with reading instructions where a Worker would have received them.

**What stays the same:** dependency classification, the three-source synthesis itself, the embed-vs-reference choices for content. The User reads the Brief and works however they choose - no validation criteria are removed, no scope reduction.

**Workspace.** When the project uses version control, ask the User which branch they want to work on - the User may choose a feature branch, the base branch, or a worktree. The Manager does not pre-create branches for User-owned Tasks. The User commits or asks the Manager to commit per the project's commit conventions while collaborating with the User on the Task.

---

## 3. Task Assignment Procedure

Dispatch assessment followed by per-Task analysis and prompt construction for each Task in the dispatch plan. Follow-up prompts use a separate construction path when a review outcome requires retry.

### 3.1 Dispatch Assessment

Assess dispatch opportunities from current project state per §2.4 Dispatch Standards. Before each dispatch decision, assess the current project state visibly in chat under the header **Dispatch Assessment:** covering which Tasks are Ready, what dependency relationships exist among them, and what dispatch mode best serves progress and efficiency. Each dispatch cycle is a fresh assessment.

Perform the following actions:
1. Identify Ready Tasks from the Tracker. Cross-reference the Dependency Graph for newly unblocked Tasks.
2. Check whether a pending report would unlock Tasks that combine well with currently Ready Tasks. If waiting costs little, consider it. Otherwise proceed.
3. Group Ready Tasks by assigned Worker. Form dispatch units per §2.4 Dispatch Standards - assess all three modes (single, batch, parallel) before committing to a dispatch plan.
4. Assess parallel opportunity: if 2+ dispatch units exist with no unresolved cross-agent dependencies - parallel dispatch.
5. Formulate dispatch plan: which Workers receive which units, whether parallel. For each Task, continue to per-Task analysis. For Tasks with Owner `User`, route to §3.4 Task Brief Construction instead of bus dispatch - User-owned Tasks participate in dependency, Stage, and parallel-dispatch logic identically to Worker-owned Tasks; only the delivery mechanism differs.

### 3.2 Per-Task Analysis

Execute for each Task in the dispatch plan.

Perform the following actions:
1. Read the Task's Dependencies field from the Plan. If "None," skip dependency context steps.
2. For each dependency, determine context depth per §2.1 Dependency Context Standards - check Worker Handoff state and auto-compaction notes in the Tracker, classify as same-agent or cross-agent, check cross-agent overrides, and trace upstream when ancestors are relevant. For Workers that recovered from auto-compaction, provide more comprehensive same-agent dependency context since reconstructed context may lack working nuance.
3. For cross-agent dependencies, read unique producer Task Logs and note key outputs, file paths, and integration details. When multiple Tasks in this dispatch cycle depend on the same producer, read that log once and extract from context for subsequent Tasks.
4. Extract Spec content relevant to this Task per §2.2 Task Prompt Content Standards. The Spec is in context from session start and refreshed on any modification. A fresh read is warranted at the start of a new Stage's first dispatch; per-Task re-reads of an unchanged Spec are not needed.
5. Extract Task definition fields from the Plan: Objective, Steps, Guidance, Output, Validation. When Guidance references Spec sections, resolve those references and extract the referenced content per §2.2 Task Prompt Content Standards. Transform steps into actionable instructions, incorporating Guidance and relevant Spec content.

### 3.3 Task Prompt Construction

Assemble the Task Prompt and deliver via the Message Bus.

Perform the following actions:
1. Construct YAML frontmatter per §4.1 Task Prompt Format.
2. Construct prompt body: Task Reference, Context from Dependencies (if applicable), Objective, Detailed Instructions, Workspace, Expected Output, Validation Criteria, Instruction Accuracy, Task Iteration, Task Logging instructions, Reporting Instructions.
3. Create a feature branch off the repository's base branch per §2.5 Version Control Standards. For parallel dispatch, create a worktree: `git worktree add .apm/worktrees/<branch-slug> -b <branch-name>`. Include the branch name (sequential) or worktree path (parallel) in the Workspace section.
4. Record the branch name in the Task row's Branch column when updating the Tracker.
5. Clear the incoming Report Bus per §2.6 Delivery Standards.
6. Read the Worker's Task Bus, then write the Task Prompt to it: `.apm/bus/<agent-slug>/task.md`. For batches, use §4.6 Batch Envelope Format.
7. Direct the User to the Worker's chat per `{SKILL_PATH:apm-communication}` §2.1 Direct Communication:
   - If the Worker is not yet initialized - direct the User to start a new chat and run `/apm-3-initiate-worker <agent-id>`. The Worker detects the pending Task Prompt during init and begins executing. Only on first dispatch to this Worker.
   - If the Worker is already initialized - direct the User to run `/apm-4-check-tasks` in the Worker's chat.
   - For batch dispatch - summarize what the Worker will receive (number of Tasks, sequential execution).
   - For parallel dispatch - list each Worker with its required action.

### 3.4 Task Brief Construction

Execute when a User-owned Task becomes Ready, or when the User claims a Task you currently have context for. Take the User-owned branch instead of bus dispatch.

Perform the following actions:
1. Run §3.2 Per-Task Analysis as you would for a Worker - read dependencies, classify context depth, extract Spec content, extract Plan Task fields. The three-source synthesis is identical for User-owned Tasks.
2. Construct the Brief per §4.5 Task Brief Format. Embed Spec content directly. Include dependency context at the same depth a Worker would receive. Drop sections that are bus-and-Worker mechanics: Workspace branch instructions, Task Iteration guidance directed at iteration limits, Task Logging instructions, Task Report instructions per §2.8 Task Brief Standards.
3. Discuss workspace with the User per §2.8 Task Brief Standards. Ask which branch they want to work on. Do not pre-create a feature branch.
4. Present the Brief in chat. Update the Tracker per `{GUIDE_PATH:task-review}` §4.1 Task Tracking Format: set the Task's Status to Active and Owner to `User`. Do not write to a Task Bus.
5. Continue per `{GUIDE_PATH:task-review}` §3.6 User-Owned Task Collaboration from step 2 (standby).

### 3.5 Follow-Up Task Prompt Construction

Execute when the review outcome (per `{GUIDE_PATH:task-review}` §3.3 Review Outcome) determines follow-up is needed.

Perform the following actions:
1. Capture follow-up context: what went wrong, investigation findings, required refinement, any planning document modifications.
2. If planning documents were modified, extract relevant updated content per §3.2 Per-Task Analysis.
3. Refine all content sections per §2.3 Follow-Up Standards. Include a follow-up context section explaining the issue and required refinement.
4. Construct the follow-up prompt per §4.2 Follow-Up Format. Same `log_path` as the original.
5. Clear the incoming Report Bus per §2.6 Delivery Standards.
6. Read the Worker's Task Bus, then write to it: `.apm/bus/<agent-slug>/task.md`.
7. Direct the User to the Worker per §3.3 Task Prompt Construction step 7.

---

## 4. Structural Specifications

### 4.1 Task Prompt Format

Task Prompts are markdown files. Adapt based on Task needs - not all sections are required for every Task.

**YAML Frontmatter Schema:**
```yaml
---
stage: 1
task: 2
agent: frontend-agent
log_path: ".apm/memory/stage-01/task-01-02.log.md"
has_dependencies: true
---
```

**Field Descriptions:**
- `stage`: Stage number.
- `task`: Task number within Stage.
- `agent`: Worker identifier (kebab-case).
- `log_path`: Pre-constructed path for the Task Log. Path pattern: `.apm/memory/stage-<NN>/task-<NN>-<MM>.log.md` (relative to the project root). All Tasks in the same Stage share the same Stage directory. You construct the path; the Worker writes directly to it.
- `has_dependencies`: Whether dependency context is present.

**Prompt Body Sections:**
- *Title.* `#` heading using Task ID and title. Each section uses `##` heading:
- *Task Reference:* Task ID and assigned agent.
- *Context from Dependencies.* Included when `has_dependencies: true`. Format depends on dependency type per §2.1 Dependency Context Standards.
  - *Same-agent.* "Building on your previous work:" intro - `**From Task <N>.<M>:**` with key outputs and recall points - `**Integration Approach:**` with brief guidance.
  - *Cross-agent.* "This Task depends on work completed by [Producer Agent]:" intro - `**Integration Steps:**` numbered file reading instructions - `**Producer Output Summary:**` key features, files, interfaces, constraints - `**Upstream Context:**` for relevant ancestors.
- *Objective:* Single-sentence Task goal, optionally enhanced with coordination-level context.
- *Detailed Instructions:* Plan steps transformed into actionable instructions with integrated Spec content and guidance.
- *Workspace:* Working directory and branch name for sequential dispatch, or worktree path and project root for parallel dispatch. For worktree dispatch, instruct the Worker to perform code work in the worktree but resolve all `.apm/` paths (Task Log, bus files) from the project root. Worker operates in the specified workspace, commits there, and notes it in the Task Log. Workers do not merge.
- *Expected Output:* Deliverables from Plan Output field.
- *Validation Criteria:* From Plan Validation field.
- *Instruction Accuracy:* The objective and expected output are authoritative - deliver those. However, the detailed instructions and steps were constructed from planning documents and may contain inaccurate details, missed prerequisites, or outdated assumptions about the codebase. When a specific instruction contradicts what the codebase actually shows, validate the actual state rather than persisting with the instruction as written.
- *Task Iteration:* When validation fails, investigate before fixing - read error output, trace the cause, understand what went wrong. Apply one targeted change per iteration. When a fix does not resolve the issue, spawn a debug subagent with structured instructions: the error output, what you investigated and attempted, relevant file paths, and expected vs actual behavior. Direct it to trace the root cause and propose a fix. Validate the subagent's findings before applying. When the root cause could stem from multiple independent areas, spawn separate subagents in parallel. If unresolved after subagent investigation, report with Partial status.
- *Task Logging:* Path and reference to `{GUIDE_PATH:task-logging}` §3.1 Task Log Procedure.
- *Task Report:* Instruction to output a Task Report for User to return to Manager.

### 4.2 Follow-Up Format

Follow-up Task Prompts use the same structure as §4.1 Task Prompt Format with these modifications:
- *Title:* `APM Follow-Up Task: <Task Title>`
- *Follow-up context section* after Task Reference - previous issue, investigation findings, required refinement, additional guidance.
- *All content sections* refined based on what went wrong, not copied from the previous attempt.
- *Same `log_path`* as the original Task Prompt.

### 4.3 Branch and Worktree Standards

Branch naming follows the convention recorded in the Tracker Version Control table. Branch names are descriptive of the actual work; for batches, the name reflects the batch scope. Worktrees are placed under `.apm/worktrees/`. Each subdirectory name is derived from the branch name (e.g., replacing `/` with `-`). Each worktree directory contains a full checkout of all tracked files. Untracked files are not present.

### 4.4 Tracker VC Entry Format

VC configuration recorded in the Version Control table within the Tracker, with one row per repository. Branch state is tracked per-Task in the Task table's Branch column - an incoming Manager reads Task rows to rebuild working VC context.

**Format:**

```markdown
## Version Control

| Repository | Base Branch | Branch Convention | Commit Convention |
|-----------|-------------|-------------------|-------------------|
| <repo-name> | <branch-name> | <convention> | <convention> |
```

### 4.5 Task Brief Format

A Task Brief is a chat message presented to the User in the Manager's chat. It has no YAML frontmatter, no Task Bus file, and no log_path field - the Manager writes the Task Log on the User's behalf at the standard Memory path during the collaboration.

**Body Sections:**

- *Title.* Conversational header naming the Task (e.g., "Task 2.3 - Refresh the dashboard layout").
- *What is being asked.* A short framing of the deliverable in plain prose. Not a bulleted Objective in the Worker style.
- *Design context.* Spec content relevant to this Task, embedded directly per §2.2 Task Prompt Content Standards. Use natural prose - present the design decisions and constraints without coordination-level vocabulary.
- *Dependencies.* Context from any prior Tasks that bear on this work, at the same depth a Worker would receive (cross-Worker dependencies still warrant comprehensive context). Reference codebase files with reading instructions where a Worker would have received them.
- *Steps.* Plan steps as a sequence the User may follow or adapt - the User decides how to do the work.
- *Deliverable.* The expected output.
- *How we will validate.* The Task's validation criteria, with clear marking of which the User can check themselves and which the Manager will run on the User's return.

**Closing offer.** End the Brief with a natural-language invitation per `{SKILL_PATH:apm-communication}` §2.1 Direct Communication: tell the User you are available for questions and remaining validation while they work, and ask them to report back when finished with what was done, anything notable that came up, and which validation criteria they checked themselves versus which to leave to you. Pausing and resuming freely is fine; unclaiming back to a Worker is also fine and conversational.

### 4.6 Batch Envelope Format

When sending multiple Tasks to a Worker in a batch, the Task Bus file uses this structure:

**YAML Frontmatter Schema:**
```yaml
---
batch: true
batch_size: <N>
tasks:
  - stage: 1
    task: 1
    log_path: ".apm/memory/stage-01/task-01-01.log.md"
  - stage: 1
    task: 2
    log_path: ".apm/memory/stage-01/task-01-02.log.md"
---
```

**Field Descriptions:**
- `batch`: Always `true` for batch envelopes.
- `batch_size`: Total Tasks in the batch.
- `tasks[].stage`: Stage number.
- `tasks[].task`: Task number within Stage.
- `tasks[].log_path`: Pre-constructed path for the Task Log, following the same pattern as single Task Prompts.

**Body:** Individual Task Prompts separated by `---` delimiters. Each Task Prompt retains its full structure (YAML frontmatter and body) as if standalone.

---

## 5. Common Mistakes

- *Planning document paths in Task Prompts:* Workers are scoped to their Task Prompt and Rules - the Spec and Plan are not in their context. A reference like "see the Spec" or "check the Plan" breaks self-containedness. Extract and embed the relevant content instead.
- *Under-scoped cross-agent context:* Cross-agent dependencies require comprehensive context regardless of perceived simplicity. Workers do not interact with Memory and have no access to other Workers' work - the only cross-agent context they receive is what you embed in the Task Prompt.
- *Stale dependency classification after Handoff:* When a Worker Handoff is detected, previous-Stage same-agent dependencies must be reclassified as cross-agent. Check the Tracker's cross-agent overrides before constructing dependency context.
- *Shallow dependency chains:* A Task's direct dependency may itself depend on earlier work that established patterns, schemas, or contracts. Trace upstream until an intermediate node fully abstracts what came before.
- *Vague instructions:* "Implement the feature properly" vs "Implement POST /api/users with email validation using express-validator, returning 201 on success."
- *Dispatching before merging dependencies:* If Task B depends on Task A's output and A was on a separate branch, A must be merged before B's branch is created.
- *Assuming base branch name:* Read the base branch from the Tracker's Version Control table for the relevant repository. Do not assume `main` or `master`.
- *Forgetting VC state in Handoff:* Ensure Task rows reflect current branch state before Handoff. Include active branches, worktrees, and pending merges in the Handoff Log.
- *Committing build artifacts:* Do not commit generated files. Create or update `.gitignore` for build directories.

---

**End of Guide**
