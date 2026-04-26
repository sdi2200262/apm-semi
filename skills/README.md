# APM Semi Standalone Skills

Optional skills that live outside the main APM Semi bundles. These are used on demand for specific situations rather than installed by `apm custom`.

## Available Skills

### apm-customization

Guides an AI agent through customizing APM Semi templates, navigating the repository structure, making changes, building, and releasing. No installation needed - this skill is already present in any fork or template of this repository. The AI agent reads it directly from `skills/apm-customization/SKILL.md` when working within the repo. If your platform does not discover it automatically, copy it to the platform's skills directory (e.g., `.claude/skills/apm-customization/SKILL.md`).

The skill is adapted for APM Semi specifically - it covers the collaborative human-and-agent layer (sovereignty signals, User-claimable Tasks, standby collaboration, residual handling) on top of the standard APM customization workflow.

## Contributing

To propose a new standalone skill, open an issue or pull request on the [APM Semi repository](https://github.com/sdi2200262/apm-semi).
