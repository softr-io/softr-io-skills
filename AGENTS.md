# Softr skills

Distribution repository for Softr's agent skills. Skills are synced from the repositories of the tools they describe.

## Do not edit synced skills

The following skills are synced from other repositories and are overwritten on the next sync:

- `skills/softr-workflows/`: source [softr-io/softr-cli](https://github.com/softr-io/softr-cli),
  directory `skills/softr-workflows`

Edit them in their source repository instead.

## Safe to edit

- `README.md`, `AGENTS.md`, the plugin manifests and the workflow under `.github/`
- A skill authored here would live in `skills/<name>/` and be listed in the README as authored here

## Format

Every skill follows the [Agent Skills specification](https://agentskills.io/specification): `skills/<name>/SKILL.md`
with `name` (equal to the directory name), `description`, and optional `license`, `compatibility` and `metadata`;
on-demand detail in `references/`. Validate with `npx -y skills-ref@0.1.5 validate skills/<name>`.
