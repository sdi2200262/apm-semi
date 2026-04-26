# Changelog

All notable changes to this project will be documented in this file.

---

## [1.0.0] - 2026-04-25

Custom adaptation of [APM v1.0.0](https://github.com/sdi2200262/agentic-project-management) for collaborative human-and-agent project execution.

### Architecture

- **User-claimable Tasks** - The User can claim any Task at any point and execute it directly. The User is not a Worker - no bus slot, no Worker registry entry, no Task Prompt delivered, no Rules to follow on the User's behalf, and no Handoff procedure. APM Semi's typical pattern is the User taking on critical or high-stakes work and AI Workers handling boilerplate or peripheral Tasks that depend on it.
- **Collaborating agent** - When the User holds a Task, an AI agent serves as the collaborating agent on the User's side. The Manager fills this role for any Task the User claims while it has not yet been dispatched, including Tasks the Plan assigned to the User. The Worker fills this role when the User takes over a Task that Worker is currently executing - the Worker pauses cleanly. Either way the collaborating agent stays on standby: answers questions, runs validation when the User returns, and writes the Task Log on the User's behalf.
- **Two postures for the Manager** - Toward Workers, the Manager is a dispatcher exactly as in v1. Toward the User, it is a direct collaborator in chat with no message bus between them. The two postures run in parallel.

### Changes from Official APM

**Added**

- Sovereignty signal detection during Context Gathering. Signals carry into Plan Analysis - those that align with specific Tasks become User-owned assignments in the Plan; remaining observations are recorded in Plan notes for the Manager.
- User-claimable Tasks during Plan review and dynamically during the Implementation Phase.
- User takeover protocol for in-progress Worker Tasks. Workers pause cleanly without writing a partial Task Log, present a takeover-time Task Brief that hands the Task back with execution progress baked in, then stay on standby.
- Task Briefs - the conversational counterpart to a Task Prompt, presented in chat to the User by the collaborating agent for User-owned Tasks. The same Brief shape is reused on the Worker side at takeover time with progress baked in.
- Validation on User-completed work with bounded scope, systemic-or-persistent escalation, and cleaned-up state on escalation.
- Residual handling for Manager-side residuals: handle inline within scope, offer to the User, dispatch to a Worker, or escalate.
- Proactive claim suggestions: actively at Stage boundaries and quietly in-Stage based on accumulated session signal. Manager never marks a Task as User-owned without explicit confirmation.
- Owner column in the Tracker accepting either an AI Worker slug or `User`.
- User-owned Task Log additions: ownership field, takeover marker, Execution Breakdown, User Notes, and Validation breakdown distinguishing User-checked from agent-checked criteria.
- User-owned Task workspace in the main working directory on whatever branch and git flow the User chooses. AI Worker dispatches concurrent with a User Task always run in worktrees, and the Manager performs their merges in a dedicated merge worktree at `.apm/worktrees/.merge/` to leave the User's main directory undisturbed.
- Conversational mechanics standard in the communication skill: agents orient the User on claim, unclaim, takeover, takeover return, and report-done at the appropriate moments, the same way they orient the User on which command to run.
- Manager Handoff carries collaborative context: User-owned Task state, evolving sovereignty observations beyond what Plan notes already carry, behavior patterns, pending claim suggestions.

**Modified**

- Tracker Task Tracking column renamed from Agent to Owner.
- Memory Index Memory Notes section accepts two first-class categories alongside general observations: User behavior patterns and User preferences. Sovereignty observations live in Plan notes.
- Manager Initiation surfaces sovereignty observations from Plan notes and User-owned Tasks during the understanding summary.

**Removed**

- `src/` (CLI source) - APM Semi installs via the official `agentic-pm` CLI.
- `SECURITY.md`, `VERSIONING.md`, `templates/_standards/NOTES.md` - specific to the main APM repository.
