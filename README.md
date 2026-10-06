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
| [`softr-workflows`](./skills/softr-workflows) | Build, test and publish Softr Workflows from local files with the Softr CLI (`softr`)              | Synced from [softr-io/softr-cli](https://github.com/softr-io/softr-cli)     |

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

The sync runs with the Softr Skills Sync GitHub App: the source repository mints an installation token to start the
workflow here, and this workflow mints one to read the source. Both use the org secrets `SKILLS_SYNC_APP_ID` (the App's
client id) and `SKILLS_SYNC_APP_PRIVATE_KEY`, scoped to this repository and every source repository.

A skill authored in this repository would be marked **Authored here** in the table above and edited directly.

## License

MIT
