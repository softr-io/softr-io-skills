---
name: softr-cli
description: >-
  Install, update and log in to the Softr CLI (@softr/cli, command softr), and use its account and
  workspace commands: whoami, workspaces, integrations. The CLI reaches Softr from the terminal, with
  a scope per area of Softr (softr workflows today, more to come), each with a skill of its own. Use
  when a Softr skill or the user needs the CLI, when "softr" is not found, when a login is needed, or
  to find the workspaces and the connected integrations a token reaches. Load before any softr command.
license: MIT
compatibility: >-
  Requires Node.js 22 or newer, network access to softr.io and a Softr account. The browser login
  needs a browser on the same machine; otherwise a personal API token from the Softr studio.
metadata:
  author: softr
  version: '0.1.0'
  homepage: https://docs.softr.io/softr-cli
  source: https://github.com/softr-io/softr-io-skills/tree/master/skills/softr-cli
---

# Softr CLI

`softr` works with the user's Softr workspace from the terminal. It is made for agents: every command accepts
`--json` for machine-readable output, exit code 1 means the command did not do what was asked, and nothing runs on the
user's machine except the CLI itself. The root commands cover the account and the workspace; each area of Softr is a
scope with a noun (`softr workflows ...`) and a skill of its own that builds on this one.

> **CRITICAL**: Never print a token or paste one into a file the user did not ask for. The CLI has exactly the rights
> the user has in Softr, never more.

## Install and update

Check first, then install or update only when needed:

```bash
softr --version                # not found? install it
npm install -g @softr/cli      # global install: the softr command
npm update -g @softr/cli       # update a global install
npx @softr/cli@latest --help   # or run the latest release without installing
```

When a command from a Softr skill is unknown to the installed CLI, update it.

## Log in once

```bash
softr login              # in a terminal: asks browser (recommended) or token; the browser path opens the studio consent page
softr login --no-browser # browser path without a terminal: prints the URL; open it for the user, the CLI waits for the approval
softr whoami             # who is logged in and what the token grants
softr logout             # forget the stored token
```

`login` stores the token in `~/.softr/credentials.json`, keyed by the environment, and renews it on its own. When the
browser flow is not possible, the user creates a personal API token in the studio (account menu, API tokens; it needs
the actions of the scope in its statements, `login` warns otherwise) and passes it with `login --token pat_...`.
Nothing else is accepted, the studio's session cookie included. On `401` run `login` again.

## Workspaces and integrations

```bash
softr workspaces                                      # the workspaces the token reaches, with the role and the default one
softr integrations [--workspace <id>] [--type <TYPE>] # the integrations connected in a workspace: id, name, type, account
```

Commands that need a workspace take `--workspace <id>`; without it they use `SOFTR_WORKSPACE_ID`, else the only or
the default workspace of the user. An integration's id is what a node input such as `integrationId` or a custom code
node's `integrations` list takes.

## Environment

For scripts, CI and agent runtimes the environment is the whole configuration; with it set the CLI prompts for nothing
and writes nothing to disk:

| Variable                                   | Meaning                                                                  |
| ------------------------------------------ | ------------------------------------------------------------------------ |
| `SOFTR_TOKEN`                              | The token to use instead of the stored one (`pat_...` or an OAuth token) |
| `SOFTR_WORKSPACE_ID`                       | The default workspace for commands that take `--workspace`               |
| `SOFTR_STUDIO_URL`, `SOFTR_STUDIO_API_URL` | Another Softr environment than production (the studio and its API)       |
| `SOFTR_WORKFLOWS_API_URL`                  | The workflows service of that environment                                |

## Scopes

| Scope             | Skill             | What it covers                                             |
| ----------------- | ----------------- | ---------------------------------------------------------- |
| `softr workflows` | `softr-workflows` | Build, test and publish Softr Workflows from files on disk |

`softr --help` and `softr <scope> --help` list the commands; the skill of the scope says how to use them.
