# Softr skills

Skills for AI coding agents that work with [Softr](https://www.softr.io), in the [Agent Skills](https://agentskills.io)
format. Any client that reads `SKILL.md` files can use them; the repository is also a plugin for Claude Code, Cursor
and every [Agent Plugins](https://agent-plugins.org) client.

## Install

```bash
npx skills add softr-io/softr-io-skills
```

Then pick the skills to install. To install one directly:

```bash
npx skills add softr-io/softr-io-skills --skill softr-workflows
```

## Available skills

| Skill                                        | What it is for                                                                                        | Source                                                                                          |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| [`softr-workflows`](./skills/softr-workflows) | Build, test and publish Softr Workflows from local files with the `softr-workflows` CLI              | Synced from [softr-io/softr-workflows-cli](https://github.com/softr-io/softr-workflows-cli)     |

Each skill tells the agent what the tool is for, what it needs, how to install it and how to work with it, so adding
the skill is the only step you take. The documentation of Softr itself stays on [docs.softr.io](https://docs.softr.io);
the skills link to it instead of repeating it.

## Plugins

This repository follows the [Agent Plugins](https://agent-plugins.org) open standard: `plugin.json` at the root, skills
in `skills/`. It also carries the manifests of these platforms:

- **Claude Code**: `.claude-plugin/`
- **Cursor**: `.cursor-plugin/`

## Editing skills

Skills marked **Synced from** are copied here from their source repositories by
[`.github/workflows/sync-from-repo.yml`](./.github/workflows/sync-from-repo.yml), which opens a pull request on every
change. Do not edit them here: the next sync overwrites the copy. Edit them in the source repository, next to the tool
they describe.

A skill authored in this repository would be marked **Authored here** in the table above and edited directly.

## License

MIT
