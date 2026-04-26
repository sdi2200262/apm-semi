---
name: apm-customization
description: Guides an AI agent through customizing APM Semi templates, building releases, and managing this custom APM repository.
---

# APM Semi Customization Skill

## 1. Overview

**Reading Agent:** Any AI assistant working within a forked or templated APM Semi repository

This skill guides the customization of APM Semi templates. It assumes the Agent is operating within the APM Semi codebase itself (a fork or template of this repository) and can explore the repository structure directly.

APM Semi is a custom adaptation of the official [Agentic Project Management (APM)](https://github.com/sdi2200262/agentic-project-management) framework that adds a collaborative human-and-agent execution layer: the User can claim any Task at any point and execute it directly while the agent on their side stays on standby. Customizations should preserve or thoughtfully extend this collaborative layer rather than ignore it.

### 1.1 Objectives

- Navigate the APM repository structure and understand what each part does
- Make targeted changes to templates, commands, guides, skills, or agent configurations
- Build and test changes locally
- Produce releases that can be installed via `apm custom`

### 1.2 How to Use

The User describes what they want to change or add. The Agent explores the relevant parts of the repository, proposes changes, and implements them after User approval. This skill provides the orientation needed to find the right files and understand how they connect.

---

## 2. Repository Structure

Explore the repository to understand the layout. The key directories are:

**`templates/`** - The source files that become the APM installation. Everything the User receives when running `apm init` or `apm custom` originates here. Changes to APM's workflow, procedures, communication patterns, or agent behavior happen in this directory.

**`templates/commands/`** - Slash commands the User sends to the model. These are the entry points for each Agent role (Planner, Manager, Worker) and for workflow actions (check-tasks, check-reports, handoff, recover, summarize).

**`templates/guides/`** - Procedural files Agents read autonomously. Each guide contains a single procedure with operational standards, step-by-step actions, output specifications, and content guidelines. Guides are the most detailed procedural documents in the system.

**`templates/skills/`** - Shared capabilities read by multiple Agent roles. Each skill lives in its own directory with a `SKILL.md` file and optional supporting files.

**`templates/agents/`** - Subagent configurations shipped with APM bundles.

**`templates/apm/`** - Artifact templates that become the `.apm/` directory (Spec, Plan, Tracker, Memory Index templates).

**`templates/_standards/`** - Development-time specifications that define how templates should be written. These files are not included in builds. Read them to understand the design rules:
- `WORKFLOW.md` - The formal workflow specification, including APM Semi's collaborative-execution overlay (sovereignty signals, User-claimable Tasks, standby collaboration, takeover protocol, residual handling). This is the source of truth for all behavior. Any change to the workflow must be reflected here first, then propagated to runtime files.
- `TERMINOLOGY.md` - Formal vocabulary and defined concepts, including APM Semi terms (Collaborating agent, Task Brief, Owner, sovereignty signal)
- `STRUCTURE.md` - Structural standards for each file type
- `WRITING.md` - Writing patterns, tone, formatting

**`build/`** - The build system that processes templates into platform-specific bundles. `build-config.json` defines the supported targets (Claude Code, Cursor, Copilot, Gemini CLI, OpenCode, Codex CLI) and their directory layouts. APM Semi keeps all six platforms supported.

**`skills/`** - Standalone skills (like this one) that are not part of the main APM Semi bundles. APM Semi installs via the official `agentic-pm` CLI and does not ship its own CLI source.

---

## 3. How Templates Become Installations

Templates use placeholders that the build system resolves per platform at build time. Understanding this is essential for making changes.

Explore `build/processors/placeholders.js` to see all supported placeholders. Common ones include:

- `{VERSION}` - Release version
- `{RULES_FILE}` - Platform-specific rules file name (e.g. `CLAUDE.md`, `AGENTS.md`)
- `{SKILL_PATH:name}` - Resolved path to a skill file
- `{GUIDE_PATH:name}` - Resolved path to a guide file
- `{COMMAND_PATH:name}` - Resolved path to a command file
- `{ARGS}` - Platform-specific argument syntax
- `{SUBAGENT_GUIDANCE}` - Platform-native subagent invocation

Explore `build/build-config.json` to see each target's directory layout, format (Markdown or TOML), and platform-specific values.

**Building locally:**

```bash
npm install
npm run build:release
```

This produces a `dist/` directory with ZIP bundles per assistant and an `apm-release.json` manifest. The User can test locally by extracting a bundle into a project.

---

## 4. Making Changes

When the User requests a change, identify which layer it affects. All workflow changes follow a top-down propagation:

1. **Update `WORKFLOW.md` first** - Any change that affects APM Semi's behavior, procedures, or coordination patterns must be reflected in the workflow specification before modifying runtime files. `WORKFLOW.md` is the source of truth and includes both the inherited APM v1 workflow and APM Semi's collaborative overlay.
2. **Propagate to runtime files** - Commands, guides, skills, and agent configurations implement the workflow spec. Update these to match the changes made in `WORKFLOW.md`, following the conventions in `STRUCTURE.md`, `WRITING.md`, and `TERMINOLOGY.md`.

Changes that do not affect the workflow (e.g. adjusting wording within existing procedures, adding examples to guidance fields) can be made directly in runtime files without updating `WORKFLOW.md`.

### Working with the Collaborative Layer

APM Semi adds collaborative-execution behaviors on top of APM v1: sovereignty signal detection in Context Gathering, User-claimable Tasks, takeover protocol for in-progress Worker Tasks, standby collaboration with validation iteration, residual handling, and proactive claim suggestions. When making changes, consider whether the change interacts with this layer:

- Changes to Worker procedures should preserve clean takeover (no partial Task Log on pause, takeover-time Task Brief with progress baked in)
- Changes to Manager procedures should preserve the dual posture (dispatcher toward Workers, direct collaborator toward the User)
- Changes to Task Logging should preserve User-owned Task Log additions (ownership field, Execution Breakdown, User Notes, Validation breakdown)
- Changes to artifact templates (Tracker, Memory Index, Plan) should preserve apm-semi-specific structure (Owner column, sovereignty observations in Plan notes, User behavior patterns and User preferences in Memory Notes)

### Template Content Changes

Most customizations involve modifying template files in `templates/`. The `_standards/` files define the conventions:

- Commands follow structural profiles defined in `STRUCTURE.md` (strict for initiation, lightweight for utility)
- Guides follow a five-section pattern (Overview, Operational Standards, Procedure, Structural Specifications, Content Guidelines)
- Skills have a required Overview section and free-form internal organization
- All files use the terminology defined in `TERMINOLOGY.md`
- Writing follows the patterns in `WRITING.md` (imperative mood, token efficiency, de-duplication)

Read the relevant `_standards/` file before making changes to understand the conventions in play.

### Adding New Files

When adding a new guide, skill, command, or agent:

1. Follow the structural conventions from `STRUCTURE.md` for that file type
2. Add YAML frontmatter where required (commands and skills require it, guides do not)
3. Use placeholders for any paths, rules file references, or platform-specific values
4. Update cross-references in other files if the new file should be loaded by an Agent

### Build Configuration Changes

If adding a new target (assistant), modify `build/build-config.json` to add the target definition with its directories, format, and platform-specific values. Explore existing targets as examples.

---

## 5. Releasing

After making changes, the User creates a release that can be installed via `apm custom`.

1. **Build** - Run `npm run build:release` to generate bundles in `dist/`
2. **Test** - Extract a bundle into a test project and verify the changes work
3. **Tag** - Create a git tag following the versioning convention
4. **Release** - Create a GitHub Release and attach all files from `dist/` (the ZIP bundles and `apm-release.json`)

The `apm-release.json` manifest is what the CLI reads to discover available assistants in the release. Explore `build/generators/manifest.js` to understand its structure.

Note: APM Semi ships its own `.github/workflows/release-templates.yml` workflow that handles tag, build, and release in one step (manually triggered via `workflow_dispatch`). It auto-increments the patch version or accepts an override, builds all six platform bundles, creates the tag, and publishes the GitHub Release with assets attached. For deeper-fork custom repositories that diverge further, the manual build-tag-release approach above remains valid.

Users install from the custom repository with:

```bash
apm custom -r owner/repo
```

---

## 6. Communicating Changes

When a custom repository diverges further from APM Semi (or APM Semi itself diverges from the official APM release), changes should be documented:

- Update the repository's README to describe what was customized and why
- If the changes affect the workflow (new procedures, modified coordination patterns), note how the customization differs from the official documentation
- If adding new commands or skills, document their purpose and usage

Custom repositories carry trust implications for anyone who installs from them. Bundles can write files anywhere within the project directory, and the templates define how AI assistants behave. When publishing a custom repository, document what the customization changes so users can make informed trust decisions. If the customization adds files outside the standard assistant config directory (e.g., source files, configuration), note this explicitly in the README.

---

**End of Skill**
