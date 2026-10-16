# Softr skills

Skills for AI coding agents that work with [Softr](https://www.softr.io), in the [Agent Skills](https://agentskills.io)
format. Any client that reads `SKILL.md` files can use them; the repository is also a plugin for Claude Code, Cursor
and every [Agent Plugins](https://agent-plugins.org) client.

## Install

### Claude Code

Install the repository as a plugin. Claude Code keeps it updated: it refreshes the marketplace in the background and
picks up new versions of the skills.

```text
/plugin marketplace add softr-io/softr-io-skills
/plugin install softr@softr-io-skills
```

### Every other agent

The [`skills`](https://github.com/vercel-labs/skills) CLI copies a skill into the directories your agents read (Codex,
Cursor, Gemini CLI, GitHub Copilot, OpenCode and many more read `.agents/skills/`):

```bash
npx skills add softr-io/softr-io-skills --skill softr-workflows
```

Run `npx skills update` to update installed skills; they do not update on their own. As agents add a plugin mechanism
with updates of their own, they get a section of their own here.

## Available skills

| Skill                                        | What it is for                                                                                        | Source                                                                                          |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `softr-cli` | Install the Softr CLI, log in, and find workspaces and integrations; the base for every `softr` scope | Synced from the Softr CLI repository |
| `softr-workflows` | Build, test and publish Softr Workflows from local files with the Softr CLI (`softr workflows`) | Synced from the Softr CLI repository |
| `softr-db` | Manage Softr Databases from the Softr CLI (`softr db`): databases, tables, fields, views and records | Synced from the Softr CLI repository |

Each skill tells the agent what the tool is for, what it needs, how to install it and how to work with it, so adding
the skill is the only step you take. The documentation of Softr itself stays on [docs.softr.io](https://docs.softr.io);
the skills link to it instead of repeating it.

## Plugins

This repository follows the [Agent Plugins](https://agent-plugins.org) open standard: `plugin.json` at the root, skills
in `skills/`. It also carries the manifests of these platforms:

- **Claude Code**: `.claude-plugin/`
- **Cursor**: `.cursor-plugin/`

## Editing skills

Skills marked **Synced from** are copied here from their source repositories: a workflow that runs in the source
repository compares each skill with the copy here and opens a pull request on every change. Do not edit them here:
the next sync overwrites the copy. Edit them in the source repository, next to the tool they describe.

The sync pushes its branches with a token of the Softr Skills Sync GitHub App that can write to this repository and
nothing else. This repository holds no secret and no workflow for it.

A skill authored in this repository would be marked **Authored here** in the table above and edited directly.

## License

MIT
