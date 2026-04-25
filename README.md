# APM Semi

[![License: MPL-2.0](https://img.shields.io/badge/License-MPL_2.0-brightgreen.svg)](https://opensource.org/licenses/MPL-2.0)

*A custom APM adaptation for collaborative human-and-agent project execution.*

## What is APM Semi?

APM Semi is an official custom adaptation of [Agentic Project Management (APM)](https://github.com/sdi2200262/agentic-project-management) for collaborative human-and-agent project execution. In APM v1, the User mediates messages between fully autonomous AI agents. APM Semi gives the User more direct control by letting them pick up Tasks and execute them themselves while keeping APM's structure and context management.

In APM Semi, Workers are still the implementation team and execute Tasks dispatched by the Manager exactly as in APM v1. What is added is direct execution by the User: the User can claim any Task at any point and execute it themselves. When that happens, the respective Agent collaborates with the User as needed during their execution of the Task - answering questions, helping with validation, and finally logging on the User's behalf.

The respective Agent is the Manager when the Task is claimed during Manager coordination, including Tasks the Plan already assigned to the User. It is the Worker assigned to the Task when the User takes over mid-execution in the Worker's chat - the Worker pauses execution and collaborates until the User is done.

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

The Planner collaborates with you through project discovery and creates the planning documents. During discovery it listens for sovereignty signals - areas you want direct ownership over - alongside its existing focuses, and you can claim Tasks during Plan review. Once approved, open a new chat and run `/apm-2-initiate-manager` to begin the Implementation Phase.

## How It Works

APM Semi inherits APM v1 unchanged - same three agents, same two phases, same artifacts, same commands. The collaborative layer overlays it:

During Context Gathering, the Planner listens for sovereignty signals alongside its existing focuses and records them in the Memory Index for the Manager to read later. During Plan review the User can claim Tasks, and during the Implementation Phase the User can claim any Task at any point or take over an in-progress Worker Task in the Worker's chat. While the User holds a Task, the agent on their side stays on standby: it answers questions, runs validation when the User returns, iterates within scope on validation failures, and writes the Task Log on the User's behalf. Small mechanical leftovers after Task Review can be handled by the Manager inline or offered to the User instead of always dispatching a follow-up. At Stage boundaries the Manager proactively recommends Tasks for the User to claim based on accumulated session signal; in-Stage it stays quiet unless a concrete trigger surfaces. The Manager never marks a Task as User-owned without explicit confirmation.

## Trade-offs

APM Semi trades execution speed for direct authorship on the work that matters to you.

When the User claims a Task, an AI Worker is not writing the code - the User is. That is slower than full delegation, and it is intentional. APM Semi is for users who want to write the substantive code themselves and lean on AI for the straightforward or peripheral Tasks - a collaborative way of working agentically rather than a fully delegated one. If the User never claims a Task in a session, APM Semi behaves identically to APM v1, so nothing is lost by adopting it.

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
