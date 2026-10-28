# xigua-skills

A collection of reusable agent skills for Claude Code and compatible skill installers.

## Available skills

### `business-memory`

Restore business context at workflow start and assess durable memory updates at workflow finish.

- **Start:** read relevant memory, verify key code paths and core rules, inspect related history, and preserve an initial change baseline.
- **Finish:** compare requirements, task-specific changes, final code and available verification; update business memory when warranted.
- **Storage:** Markdown describes current business knowledge; JSONL preserves meaningful business change events. Local runtime state is stored separately.

Skill source:

```text
business-memory/SKILL.md
```

## Workflow integration

The workflow invokes this skill explicitly at both boundaries. Installing the skill does not install hooks or arrange those calls.

Example start input (workflow data, not an executable command):

```json
{
  "phase": "start",
  "project_root": "/path/to/project",
  "run_id": "refund-window-42",
  "requirement": "Extend the refund application window from 7 to 15 days"
}
```

After development and appropriate checks, invoke finish with the same identity:

```json
{
  "phase": "finish",
  "project_root": "/path/to/project",
  "run_id": "refund-window-42",
  "implementation_summary": "Updated the refund rule and its boundary cases",
  "verification_results": ["Refund policy unit tests passed"]
}
```

Provide `state_dir` at both calls when reusing workflow storage. Otherwise state defaults to `<project_root>/.business-memory/runs/<run_id>/`; keep it local and excluded from commits through project ignore conventions. The skill does not change ignore configuration automatically. Start retries preserve the original baseline.

Durable memory lives under `business_docs/`: module and flow Markdown, `navigation.md`, optional graphs and detailed change explanations, and `changes/events.jsonl`. Create files only when needed; preserve existing documentation and Markdown change history.

Finish reports `updated`, `no_update_needed`, or `insufficient_evidence`, plus pending writes if any. It accounts for pre-existing edits, task commits and final worktree changes. Without a reliable baseline or explicit task range, it may repair verified current facts but cannot invent task history. Finish retries reuse event identities to avoid duplicates.

See [the skill](business-memory/SKILL.md) and [the persistence contract](business-memory/references/persistence.md) for the lifecycle and event schema. The workflow handles implementation, test execution and deployment.

## Install

This repository can be installed directly from GitHub. It does **not** need to be published as an npm package.

List available skills without installing:

```bash
npx skills add paihuai00/skills -l
```

Install the `business-memory` skill:

```bash
npx skills add paihuai00/skills --skill business-memory
```

Install globally:

```bash
npx skills add paihuai00/skills --skill business-memory -g
```

Install for Claude Code explicitly:

```bash
npx skills add paihuai00/skills --skill business-memory --agent claude-code
```

Skip confirmation prompts:

```bash
npx skills add paihuai00/skills --skill business-memory -y
```

## Repository structure

```text
.
├── business-memory/
│   ├── SKILL.md
│   └── references/
│       └── persistence.md
├── .gitignore
└── README.md
```

Each skill should live in its own directory and include a required `SKILL.md` file with `name` and `description` frontmatter.

## Notes

- GitHub is the distribution source.
- `npx skills add` is the installer command.
- The skill itself is just a standard `SKILL.md` directory structure, not an npm package.
