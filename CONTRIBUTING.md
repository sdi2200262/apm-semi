# Contributing to APM Semi

Thank you for considering contributing to APM Semi! This adaptation extends APM v1 with collaborative human-and-agent execution, and contributions help refine that overlay.

## Ways to Contribute

### Reporting Bugs and Workflow Issues

- Search existing issues first: [GitHub Issues](https://github.com/sdi2200262/apm-semi/issues)
- For new bug reports, include:
  - The AI assistant used (Claude Code, Cursor, Copilot, Gemini CLI, OpenCode, or Codex CLI)
  - The agent role experiencing the issue (Planner, Manager, or Worker)
  - Whether the User had claimed a Task at the time and which (if any)
  - Step-by-step reproduction
  - Expected vs actual behavior

### Suggesting Improvements

- Refinements to the collaborative-execution behaviors: sovereignty signal detection, Task Brief construction, validation iteration, leftover handling, active recommendations
- Edge cases in claim and unclaim flows, takeover handling, or User-owned Task Logging
- Documentation improvements: clearer explanations, additional examples
- Platform-specific issues across the six supported assistants

### Template Standards

The template authoring standards live in `templates/_standards/`:

- `WORKFLOW.md` - The formal workflow specification. Source of truth for behavior. Any change to coordination patterns must be reflected here first, then propagated to runtime files.
- `TERMINOLOGY.md` - Formal vocabulary and defined concepts.
- `STRUCTURE.md` - Structural standards for each file type.
- `WRITING.md` - Writing patterns, tone, formatting.

Contributions to these files affect every runtime template downstream.

## Development

### Repository Structure

- `templates/` - Source files that become the APM Semi installation. Commands, guides, skills, agent definitions, and artifact templates.
- `templates/_standards/` - Development-time specifications. Not included in builds.
- `build/` - Build system that processes templates into platform-specific bundles.
- `skills/apm-customization/` - Standalone skill for AI agents helping with further customization of this repository.

APM Semi installs via the official `agentic-pm` CLI; this repository does not ship its own CLI source.

### Building Locally

```bash
npm install
npm run build:release
```

This produces a `dist/` directory with bundles per assistant and an `apm-release.json` manifest.

### Testing Changes

Extract a built bundle into a test project and run an APM Semi session end-to-end:

1. Initiate the Planner and confirm sovereignty signals are detected and recorded in the Memory Index.
2. Claim Tasks during Plan and Spec review, then run the Manager and verify the Task Brief path.
3. Take over an in-progress Worker Task mid-execution and verify the Worker pauses cleanly without writing a partial Task Log.
4. Trigger a validation failure on User-completed work and verify the hosting agent iterates within scope and escalates with cleaned-up state when systemic.

### Releasing

After making changes:

1. Build with `npm run build:release`.
2. Test a bundle locally.
3. Tag the release.
4. Create a GitHub Release attaching the ZIP bundles and `apm-release.json`.

The CLI reads `apm-release.json` to discover available assistants. Users install with:

```bash
apm custom -r sdi2200262/apm-semi
```

## License

By contributing, you agree that your contributions will be licensed under the Mozilla Public License 2.0.
