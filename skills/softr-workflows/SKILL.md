---
name: softr-workflows
description: >-
  Build, test and publish Softr Workflows from local files with the softr-workflows CLI
  (@softr/cli): one directory per workflow holding softr-workflows.jsonc and a real .js or
  .py file per Run custom code node, the init, add, push, test, outputs, verify, publish loop, and
  placeholders that reference other nodes' outputs. Use when the user wants to create, edit, test or
  publish a Softr workflow from the terminal or a repository, wants workflow logic as JavaScript or
  Python files, or mentions softr-workflows, softr-workflows.jsonc or the Softr Workflows CLI. Load
  this before running any softr-workflows command; it also says how to install and update the CLI.
license: MIT
compatibility: >-
  Requires Node.js 22 or newer, network access to softr.io and a Softr account. The browser login
  needs a browser on the same machine; otherwise a personal API token from the Softr studio.
metadata:
  author: softr
  version: '0.2.0'
  homepage: https://docs.softr.io/softr-cli
  source: https://github.com/softr-io/softr-cli
---

# Softr Workflows CLI

`softr-workflows` treats a [Softr workflow](https://docs.softr.io/workflows/workflows.md) like a small project: the
definition is a `softr-workflows.jsonc` file, every **Run custom code** node is a real `.js` or `.py` file next to it,
and the CLI moves both to and from the Softr workflows service. Most workflows built this way are a trigger plus a few
custom code nodes, with branches, filters, loops and waits between them.

> **CRITICAL**: Nothing runs locally. Every test and every execution happens in the Softr workflows service. The only
> way to learn what a node outputs is `softr-workflows test <nodeId>`.
> **CRITICAL**: Never hand-write a placeholder such as `{outputs.x:::$.y}`. Test the upstream node first, then let
> `softr-workflows placeholder <nodeId> <keys...>` build and check the token.
> **CRITICAL**: Never write a placeholder inside a code file. Map it under `inputs.inputData` in the config and read it
> from `inputData` in the code.
> **CRITICAL**: `push` saves a draft. Only `publish` enables the workflow. Never pass `--force` to `publish` without
> the user's explicit consent.

## When to use it, and when not

Use the CLI when the workflow's logic belongs in code files the user keeps, versions and reviews: enrichment and
scoring in JavaScript or Python, webhooks answered by code, schedules that call an API. It builds deterministically
from files, so it suits agents and repositories.

Do not use it for workflows made of many third-party integration steps (Slack, Airtable, Gmail and the like): the CLI
offers only a small set of node types on purpose (see [Node types on offer](#node-types-on-offer)). Those workflows are
built in the [Softr studio](https://docs.softr.io/workflows/workflows.md) or through the
[Softr MCP server](https://docs.softr.io/mcp/workflows.md), which reaches every node type.

## Preconditions

- A Softr account with access to a workspace where Workflows are available. The user must be able to open Workflows in
  the studio; the CLI has the same rights the user has, never more.
- Node.js 22 or newer on the machine that runs the CLI, and network access to `*.softr.io`.
- For the browser login, a browser on the same machine. Without one, the user creates a personal API token in the
  studio and passes it to `login --token`.
- The commands only ever touch the workflows service and the studio API; nothing runs on the user's machine except the
  CLI itself.

## Install and update

Check first, then install or update only when needed:

```bash
softr-workflows --version                # not found? install it
npm install -g @softr/cli      # global install: the softr command (softr-workflows works too)
npm update -g @softr/cli       # update a global install
npx @softr/cli@latest --help   # or run the latest release without installing
```

When a command from this skill is unknown to the installed CLI, update it. Every command accepts `--json` for
machine-readable output, and exit code 1 means the command did not do what was asked.

## Log in once

```bash
softr-workflows login              # in a terminal: asks browser (recommended) or token; the browser path opens the studio consent page
softr-workflows login --no-browser # browser path without a terminal: prints the URL; open it for the user, the CLI waits for the approval
softr-workflows whoami             # who is logged in and what the token grants
```

`login` stores the token in `~/.softr/credentials.json` and renews it on its own. When the browser flow is not
possible, the user creates a personal API token in the studio (account menu, API tokens; it needs a `workflows:*`
action in its statements, `login` warns otherwise) and passes it with `login --token pat_...`. Nothing else is
accepted, the studio's session cookie included. `SOFTR_WORKFLOWS_TOKEN` overrides the stored token for scripts and CI.
On `401` run `login` again. Never print a token or paste one into a file the user did not ask for.

## Project layout

One directory holds exactly one workflow. A parent directory may group several.

```text
lead-scoring/
  softr-workflows.jsonc      # the workflow: title, workspaceId, triggers, actions, paths, configuration, id
  actions/
    score.py                 # body of the CUSTOM_CODE node "score", wrapped in def main(inputData):
    notify.js                # body of the CUSTOM_CODE node "notify", wrapped in export default async function (inputData)
  .softr-workflows/          # CLI state: server version and draft, cached test outputs, JSON schema. Add to .gitignore
```

Conventions in `softr-workflows.jsonc`:

- Any string value `"@@/relative/path"` is replaced by that file's content on `push`. Every `CUSTOM_CODE` node's
  `inputs.code` is such a reference.
- Node ids are readable slugs such as `score`. They must be unique and must not contain `:::`.
- `$schema` points at `.softr-workflows/schema.json`, which `pull` and `init` write. Editors validate and complete the
  file with it. The service stays the authority: `push` accepts incomplete drafts, `publish` validates everything.
- Comments and key order in the file survive `add`, `replace` and `remove`.

## The loop

1. **Start.** `softr-workflows init "<title>"` creates the workflow on the server with a `WEBHOOK` trigger named
   `trigger` and the local directory. `--trigger <TYPE>` picks another trigger, `--workspace <id>` a workspace when the
   user has several. To work on an existing workflow: `softr-workflows pull` lists them, `softr-workflows pull <id>`
   downloads one.
2. **Add nodes.** `softr-workflows add CUSTOM_CODE --id score --lang PYTHON --title "Score lead"` appends a node after
   the last one and creates `actions/score.py` with an empty function. `--after <nodeId>` places it elsewhere,
   `--in-loop <loopId>` inside a loop body. `softr-workflows spec` lists the node types on offer, `spec <TYPE>` shows
   inputs, output, notes, an example node and the docs URL. Node types that take integration ids (the Softr Tables triggers and `CALL_API` through `integrationId`, `CUSTOM_CODE` through its `integrations` list) get them from a connected integration: `softr-workflows integrations` lists them for the
   workflow's workspace, `--type SLACK` narrows the list. `replace <nodeId> <TYPE>` changes a node's type,
   `remove <nodeId>` deletes it with its paths.
3. **Write the code and its inputs.** Edit the function in `actions/<id>.js|py` (see
   [references/custom-code.md](references/custom-code.md)) and
   map what it needs under the node's `inputs.inputData`. Values are literals or placeholders.
4. **Push.** `softr-workflows push` inlines the files and saves a new draft. It sends nothing when the files match the
   last sync and refuses when the server copy changed since then; read the message before choosing `pull --force`
   (take the server copy, local edits are lost) or `push --force` (overwrite the server).
5. **Test, upstream first.** `softr-workflows test trigger` proposes recent trigger samples and saves one (`--sample
<n>` picks another; a webhook needs one real request first). `softr-workflows test <nodeId>` runs one action
   against the saved samples of its predecessors. `softr-workflows test --all` runs the trigger and then every action
   in execution order, skipping branches, filters and loops. Tests run against the server copy: the output names
   nodes whose local edits are not pushed.
6. **Look at the outputs.** `softr-workflows outputs` shows which nodes are tested. `softr-workflows outputs <nodeId>`
   lists the node's referenceable variables with types and examples. `softr-workflows placeholder <nodeId> body
email` builds `{outputs.<nodeId>:::$.body.email}` and fails with the existing keys when the path is not in the test
   data. Paste the token into `inputData`, push and test the consuming node.
7. **Verify.** `softr-workflows verify` prints one report: code files parse, local files match the server, the
   service accepts the definition for publishing, every node is tested on current data, every placeholder is backed
   by test data. Exit code 1 means something needs attention; fix it and run again. `--test-missing` first runs the
   tests that are missing.
8. **Publish.** `softr-workflows publish` enables the latest version. It refuses unpushed edits and a changed server
   copy, and stops on validation problems the service reports. For untested or stale nodes it asks in a terminal and
   needs `--force` from a script. `unpublish` disables the workflow; `status` shows local, server and live state.

The trigger's URL and the workflow in the studio are in the output of `push`, `init` and `status`.

## Placeholders

A placeholder references another node's output at run time. The forms the CLI builds and checks:

| Reference                                          | Meaning                                                                   |
| -------------------------------------------------- | ------------------------------------------------------------------------- |
| `{outputs.<nodeId>:::$.body.email}`                | A field of the node's output. A custom code result sits under `body`      |
| `{outputs.<nodeId>:::$.items[*].id}`               | Every element's field, an array                                           |
| `{loopActionGroup.<loopId>:::loopVariables.items}` | The current item inside a loop body; `placeholder <loopId> ...` builds it |

A node can reference only its predecessors, and the CLI can only type-check a path against saved test data. So the order
is always: test upstream, `placeholder`, edit `inputData`, push, test downstream. A placeholder as the whole value keeps
its JSON type; inside a longer string it becomes text.

## Node types on offer

`spec` lists them. Triggers: `WEBHOOK`, `RECURRING_SCHEDULE`, `ONE_TIME_SCHEDULE`, Softr Tables events, Softr Apps
events including the app's **Run workflow** action, incoming email. Actions: `CUSTOM_CODE`, `RESPONDED_TO_WEBHOOK`,
`CALL_API`, `BRANCH`, `FILTER`, `LOOP_ACTION_GROUP`, `WAIT`. Other integrations stay in the studio. `spec <TYPE>`
points to the public docs page of each type; the docs index for agents is <https://docs.softr.io/llms.txt>, and every
docs page is also served as `.md`.

## When something goes wrong

| Message or situation                           | What to do                                                                          |
| ---------------------------------------------- | ----------------------------------------------------------------------------------- |
| `command not found: softr-workflows`           | `npm install -g @softr/cli`, or use `npx @softr/cli@latest`                         |
| `No token stored` or `401`                     | Run `softr-workflows login` (or `login --no-browser` and open the URL for the user) |
| `The server copy changed since your last sync` | Someone edited in the studio. Ask which side wins: `pull --force` or `push --force` |
| `not backed by test data`                      | Test the referenced node first, then rebuild the token with `placeholder`           |
| `Code files that do not parse`                 | The file must hold exactly one function; the message names `file:line:col`          |
| `verify` exits with 1                          | Read the `NOT` sections; fix, push, test, verify again                              |
| `The service would reject publishing`          | A definition problem, not a test problem. Fix the config; `--force` never helps     |
| A node reads something the code needs          | Map it in `inputData`; code has no other input                                      |

## References

| Task                                            | File                                                   |
| ----------------------------------------------- | ------------------------------------------------------ |
| Write or fix the JavaScript or Python of a node | [references/custom-code.md](references/custom-code.md) |

Public documentation of Softr Workflows, also served as Markdown for agents (append `.md` to a page URL; index at
<https://docs.softr.io/llms.txt>): [Workflows](https://docs.softr.io/workflows/workflows.md),
[Trigger types](https://docs.softr.io/workflows/trigger-types.md),
[Advanced concepts](https://docs.softr.io/workflows/advanced-concepts.md) (Branch, Loop, Run custom code, Call API).
