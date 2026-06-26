# xigua-skills

A collection of reusable agent skills for Claude Code and compatible skill installers.

## Available skills

### `business-memory`

Maintain and use a durable business knowledge base under `business_docs/`.

Use this skill when you need to:

- analyze business logic or feature flows
- record durable business behavior changes
- document business modules, key user flows, APIs, states, and UI effects
- maintain `business_docs/navigation.md`
- prepare business context for future AI coding sessions

Skill source:

```text
business-memory/SKILL.md
```

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
│   └── SKILL.md
├── .gitignore
└── README.md
```

Each skill should live in its own directory and include a required `SKILL.md` file with `name` and `description` frontmatter.

## Notes

- GitHub is the distribution source.
- `npx skills add` is the installer command.
- The skill itself is just a standard `SKILL.md` directory structure, not an npm package.
