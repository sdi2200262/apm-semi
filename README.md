# APM Semi

[![License: MPL-2.0](https://img.shields.io/badge/License-MPL_2.0-brightgreen.svg)](https://opensource.org/licenses/MPL-2.0)

*A custom APM adaptation for collaborative human-and-agent project execution.*

## What is APM Semi?

APM Semi is an official custom adaptation of [Agentic Project Management (APM)](https://github.com/sdi2200262/agentic-project-management) for collaborative human-and-agent project execution. In APM v1, the User mediates messages between fully autonomous AI agents. APM Semi gives the User more direct control by letting them pick up Tasks and execute them themselves while keeping APM's structure and context management.

In APM Semi, Workers are still the implementation team and execute Tasks dispatched by the Manager exactly as in APM v1. What is added is direct execution by the User: the User can claim any Task at any point and execute it themselves. When that happens, the respective Agent collaborates with the User as needed during their execution of the Task - answering questions, helping with validation, and finally logging on the User's behalf.

The respective Agent is the Manager when the Task is claimed during Manager coordination, including Tasks the Plan already assigned to the User. It is the Worker assigned to the Task when the User takes over mid-execution in the Worker's chat - the Worker pauses execution and collaborates until the User is done.

If the User never claims a Task in a given session, APM Semi behaves identically to APM v1.

## Installation

Navigate to your project directory, initialize through the `agentic-pm` CLI:

```bash
npm install -g agentic-pm
apm custom -r sdi2200262/apm-semi
```

Open your AI assistant and start planning:

```
/apm-1-initiate-planner
```

The Planner collaborates with you through project discovery and creates the planning documents. During discovery it listens for sovereignty signals - areas you want direct ownership over - alongside its existing focuses, and you can claim Tasks during Plan and Spec review. Once approved, open a new chat and run `/apm-2-initiate-manager` to begin the Implementation Phase.

## How It Works

APM Semi inherits the APM v1 architecture: the three-agent model (Planner, Manager, Workers), the two-phase workflow (Planning, Implementation), the Message Bus, the planning documents (Spec, Plan, Rules), the Memory system (Tracker, Index, Task Logs, Stage summaries), the Handoff system, the recovery command, and the Ad-Hoc subagent dispatch.

The collaborative layer adds:

- **Sovereignty signal detection** during Context Gathering. Signals are recorded as durable Memory notes in the Index that the Manager reads during the Implementation Phase.
- **User-claimable Tasks** during Plan and Spec review and dynamically at any point during the Implementation Phase. The User can also take over an in-progress Worker Task by signaling in the Worker's chat.
- **Task Briefs** in chat instead of Task Prompts on the bus when the Manager is collaborating with the User on a Task. The Brief carries the same content as a Task Prompt, written conversationally to the User rather than as instructions to an executor.
- **Standby collaborator posture** for the AI agent on the User's side (the Manager or, after a takeover, the Worker assigned to the Task). The agent does not work the Task autonomously while the User holds it; it is available for questions, runs validation when the User returns, and writes the Task Log on the User's behalf.
- **Validation iteration** with bounded scope and systemic-or-persistent escalation. When validation fails on User-completed work, the agent on standby attempts to resolve within scope and escalates with cleaned-up code state if the failure is architectural, design-level, or not resolving across attempts.
- **Leftover handling** by the Manager. Small mechanical residuals after Task Review can be handled inline or offered to the User instead of always dispatching a follow-up.
- **Active recommendations** from the Manager based on accumulated session signal: actively at Stage boundaries, quietly in-Stage. The Manager never marks a Task as User-owned without explicit confirmation.

## Trade-offs

APM Semi is an additive overlay on APM v1. It loses nothing if you do not claim Tasks; it adds collaborative ownership when you do.

| | APM Semi | Official APM |
|---|---|---|
| **User role** | Mediates agent communication and can claim and execute any Task directly | Mediates agent communication |
| **Worker model** | Same as v1; Workers execute dispatched Tasks | Workers execute dispatched Tasks |
| **Task Brief** | Manager presents Task Briefs in chat for User-owned Tasks | No equivalent |
| **Validation iteration** | Agent on standby corrects within scope, escalates on systemic failures | Worker iterates within scope, returns Partial when stalled |
| **Leftover handling** | Manager can handle inline within scope, offer to User, or dispatch | Manager dispatches a follow-up |
| **Active recommendations** | Manager surfaces recommendations at Stage boundaries and on signal | None |
| **Platforms** | Same six as v1 | Six platforms supported |

## Commands

APM Semi keeps the v1 command surface unchanged. Claim, unclaim, takeover, takeover return, and report-done are all conversational and use natural language.

| # | Command | Agent | Purpose |
|---|---------|-------|---------|
| 1 | `/apm-1-initiate-planner` | Planner | Planning Phase |
| 2 | `/apm-2-initiate-manager` | Manager | Implementation Phase |
| 3 | `/apm-3-initiate-worker` | Worker | Worker initialization |
| 4 | `/apm-4-check-tasks` | Worker | Task Bus check |
| 5 | `/apm-5-check-reports` | Manager | Report Bus check |
| 6 | `/apm-6-handoff-manager` | Manager | Manager Handoff |
| 7 | `/apm-7-handoff-worker` | Worker | Worker Handoff |
| 8 | `/apm-8-summarize-session` | Standalone | Session summary and archival |
| 9 | `/apm-9-recover` | Manager or Worker | Reconstruct context after compaction |

## Documentation

For the full APM workflow documentation, see [agentic-project-management.dev](https://agentic-project-management.dev). APM Semi shares APM's core concepts (planning documents, Stages, Tasks, Memory, Handoff) with the collaborative-execution layer added on top.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines. Report bugs or suggest features via [GitHub Issues](https://github.com/sdi2200262/apm-semi/issues).

## License

Licensed under the **Mozilla Public License 2.0 (MPL-2.0)**. See [LICENSE](LICENSE) for full details.

Based on [Agentic Project Management](https://github.com/sdi2200262/agentic-project-management) by [CobuterMan](https://github.com/sdi2200262).
