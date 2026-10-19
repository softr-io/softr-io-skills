---
name: softr-db
description: >-
  Manage Softr Databases from the terminal with the `softr db` commands of the Softr CLI (@softr/cli):
  databases, their tables, fields and views, one directory per database holding softr-db.jsonc and a
  schema cache with every table and field id. Use when the user wants to create or change a Softr
  database, table or field from the terminal or a repository, needs the ids of tables and fields (for
  example for a workflow), or mentions softr-db.jsonc or `softr db`. Load this before running any
  `softr db` command; the softr-cli skill covers install and login.
license: MIT
compatibility: >-
  Requires Node.js 22 or newer, network access to softr.io and a Softr account. The browser login
  needs a browser on the same machine; otherwise a personal API token from the Softr studio.
metadata:
  author: softr
  version: '0.1.0'
  homepage: https://docs.softr.io/softr-cli
  source: https://github.com/softr-io/softr-io-skills/tree/master/skills/softr-db
---

# Softr Databases from the terminal

`softr db` works on [Softr Databases](https://docs.softr.io/softr-databases/softr-databases-overview.md) through the public
[Softr Database API](https://docs.softr.io/softr-api/softr-database-api.md): databases, tables, fields and views.
Every change goes to Softr at once; there is no local draft and no publish step.

> **CRITICAL**: `delete` removes data for everyone and cannot be undone. `--force` also removes every table of a
> database or every record of a table. Never delete, and never pass `--force`, without the user's explicit consent for
> that item.
> **CRITICAL**: Record values are keyed by field id, not by field name. Take the ids from `.softr-db/schema.json` or
> from `softr db tables get <table>`; never guess them.

## When to use it, and when not

Use it to create a database for a project, to shape its tables and fields from a script or an agent, and to give a
project folder the table and field ids that its workflows and code read. Views can only be listed; create and change
them in the [Softr studio](https://docs.softr.io/softr-databases/softr-databases-overview.md) or through the
[Softr MCP server](https://docs.softr.io/mcp/databases.md).

## Preconditions

- The `softr-cli` skill: the CLI installed and logged in.
- The token grants `databases.records:read` to read and `databases.schema:write` to create or change databases,
  tables and fields. A browser login grants both; a personal API token needs them in its statements. `softr workspaces`
  shows the scopes per workspace.

## The database directory

One directory holds one database. A project folder holds database and workflow directories side by side.

```text
crm/
  softr-db.jsonc               # { "databaseId": "…" }: the user's file, committed
  .softr-db/schema.json        # the CLI's cache: the database, its tables and fields with ids, types and options
  .softr-db/views/<tableId>.json  # the views of each table read so far
```

- `softr db get <id or exact name>` outside a database directory checks that the token reaches the database and writes
  `./<name-slug>/` with both files. `softr db create <name>` does the same for a new database.
- Inside the directory, or with `--dir <path>`, the commands work on that database without its id. A database id as an
  argument (`--database <id>` for tables, fields and views) or in `SOFTR_DATABASE_ID` comes first.
- Every command that reads or changes the schema rewrites the cache; `softr db get` in the directory refreshes it on
  purpose. Read the cache for ids instead of asking the server again. Never edit it, and add `.softr-db/` to
  `.gitignore`.
- `softr db views list <table>` writes that table's views file. A missing file means the views were not read yet, not
  that the table has none. Records are never written to disk.

## Commands

```bash
softr db list                                         # databases the token reaches
softr db get CRM                                      # ./crm/ with softr-db.jsonc and the cache
cd crm
softr db tables list
softr db tables create Deals --field "Title:SINGLE_LINE_TEXT" --field "Due date:DATE"
softr db tables get Deals                             # fields with ids, types, primary/required/read-only flags
softr db fields add Deals Stage --type SELECT --options '{"choices":[{"label":"Open"},{"label":"Won"}]}'
softr db fields update Deals Stage --name Phase       # what is not passed stays as it is
softr db views list Deals
```

- Tables and fields take an id or an exact name. A name that two items share is an error that lists their ids.
- `tables create` needs at least one field. The first field names a record unless `--primary-field` names another.
- `--field <name>:<TYPE>` suits types without options. For a type with options (SELECT choices, LINKED_RECORD target,
  NUMBER precision, ...) pass `tables create --fields` a JSON array of fields with `name`, `type` and `options` (inline,
  `@<file>` or `-`), or add the field later with `fields add --options <json>`. The types and their options:
  [field reference](https://docs.softr.io/softr-api/softr-database-api/table-fields/add-table-field.md).
- `fields update --options` replaces the options as a whole. Read the current ones with `fields get` first. The
  other settings of the field (AI fill, default value) stay as they are.
- Computed types (FORMULA, LOOKUP, ROLLUP, CREATED_AT, AUTONUMBER, ...) are read-only in records.
- `softr db <noun> --help` lists every command and option.

## When something goes wrong

| Message or situation                                  | What to do                                                                              |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `Which database?`                                     | `cd` into a database directory, pass `--dir`, `--database` or `softr db get <database>` |
| `has no table with the id or name`                    | `softr db tables list`; the name must match exactly                                     |
| `Several tables are named` / `Several fields are ...` | Pass one of the listed ids                                                              |
| `belongs to database ...`                             | The directory holds another database; run from the parent directory or pass `--dir`     |
| `cannot be deleted because it has ...`                | Ask the user; only with their consent repeat with `--force`                             |
| `403` with a scope hint                               | The token lacks the scope; a new token or a browser login                               |
| `429`                                                 | Rate limit per token; wait a second and repeat                                          |
