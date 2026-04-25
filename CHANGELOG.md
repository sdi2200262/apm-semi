# Changelog

All notable changes to this project will be documented in this file.

---

## [1.0.0] - 2026-04-25

Custom adaptation of [APM v1.0.0](https://github.com/sdi2200262/agentic-project-management) for collaborative human-and-agent project execution.

### Architecture

- **User-claimable Tasks** - The User can claim any Task at any point and execute it directly. The User is not a Worker - no bus slot, no Worker registry entry, no Task Prompt delivered, no Rules to follow on the User's behalf, and no Handoff procedure.
- **Standby collaborator posture** - When the User holds a Task, whichever agent has context for it (the Manager for unassigned or Manager-hosted Tasks, the Worker for in-progress Tasks) hosts execution as a standby collaborator: answering questions, running validation when the User returns, and writing the Task Log on the User's behalf. Manager-hosted and Worker-hosted hosting are identical in shape; only the flavor of context the host can offer differs.
- **Two postures for the Manager** - Toward Workers, the Manager is a dispatcher exactly as in v1. Toward the User, it is a direct collaborator in chat with no message bus between them. The two postures run in parallel.

### Changes from Official APM

**Added**

- Sovereignty signal detection during Context Gathering. Signals are recorded as durable Memory notes in the Index that the Manager reads during the Implementation Phase.
- User-claimable Tasks during Plan and Spec review and dynamically during the Implementation Phase.
- User takeover protocol for in-progress Worker Tasks. Workers pause cleanly without writing a partial Task Log and become standby collaborators.
- Task Briefs - the conversational counterpart to a Task Prompt, presented in chat to the User by the hosting agent for User-owned Tasks.
- Validation iteration heuristic with bounded scope, systemic-or-persistent escalation, and cleaned-up state on escalation.
- Leftover handling heuristic for Manager-side residuals: handle inline within scope, offer to the User, dispatch to a Worker, or escalate.
- Active recommendation behavior: actively at Stage boundaries and quietly in-Stage based on accumulated session signal. Manager never marks a Task as User-owned without explicit confirmation.
- Owner column in the Tracker accepting either an AI Worker slug or `User`.
- User-owned Task Log additions: ownership field, takeover marker, Execution Breakdown, User Notes, and Validation breakdown distinguishing User-checked from agent-checked criteria.
- Baseline Collaborative Execution section in the APM_RULES block, written for AI agents and present regardless of project context.
- Manager Handoff carries collaborative context: User-owned Task state, sovereignty signals not yet promoted, behavior patterns, pending recommendations.

**Modified**

- Tracker Task Tracking column renamed from Agent to Owner.
- Memory Index Memory Notes section accepts three first-class categories alongside general observations: sovereignty signals, User behavior patterns, and User preferences.
- Manager Initiation surfaces sovereignty signals and User-owned Tasks during the understanding summary.
- Recovery for the Manager reconstructs collaborative coordination state from the Index, the Tracker, and chat history.

**Removed**

- `src/` (CLI source) - APM Semi installs via the official `agentic-pm` CLI.
- `SECURITY.md`, `VERSIONING.md`, `templates/_standards/NOTES.md` - specific to the main APM repository.
