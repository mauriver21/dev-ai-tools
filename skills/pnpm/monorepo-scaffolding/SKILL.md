# Monorepo Scaffolding Skill

## Purpose

Create a pnpm monorepo by following the specifications located at:

```text
specs/monorepo-scaffolding/
```

The skill should implement the documented architecture rather than making its own structural decisions.

## Responsibilities

- Read the monorepo scaffolding specification.
- Create the monorepo structure.
- Configure the pnpm workspace.
- Generate the root `package.json` and required root configuration files.
- Scaffold the `packages`, `web`, and `api` directories.
- Follow the conventions defined in the specification.

## Existing Project Migration

If the current repository already contains a standalone project:

- Detect the project type.
- Migrate the project into the appropriate workspace:
  - **Frontend applications** → `web/`
  - **Backend services** → `api/`
  - **Reusable libraries** → `packages/`

- Preserve the project's source code, configuration, and Git history whenever possible.
- Update workspace references and paths as needed.
- Ensure the migrated project builds successfully within the monorepo.

## Expected Output

A ready-to-use pnpm monorepo that matches the documented specification, with any existing standalone project seamlessly integrated into the appropriate workspace.
