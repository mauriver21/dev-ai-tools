---
name: expressjs-scaffold-app
description: Scaffold a new Express.js application by following the architecture, conventions, and implementation details documented in tech-specs/expressjs.
---

## Steps

- Read and understand every specification under `tech-specs/expressjs` before generating code.
- Detect whether the current repository is already a monorepo.
- If the repository is a monorepo:
  - Identify the monorepo tooling and conventions (workspace configuration, package manager, folder layout, etc.).
  - Determine the appropriate location for backend/server applications according to the existing monorepo structure.
  - If an Express application already exists, extend it without breaking the current architecture.
  - If no Express application exists, ask the user for the application name before scaffolding it.
  - Scaffold the new Express application inside the appropriate workspace folder.
  - Register the new application in the monorepo (workspace configuration, build system, task runner, package manager, or any other required configuration).
  - Ensure the new application follows all existing monorepo conventions.
- If the repository is not a monorepo:
  - Detect whether an Express application already exists.
  - If one exists, extend it without breaking the current architecture.
  - If no Express application exists, ask the user for the application name before creating it.
- Follow the documented project structure, naming conventions, coding standards, and architectural decisions defined in `tech-specs/expressjs`.
- Reuse existing files whenever possible instead of replacing them.
- Generate only the files required by the specifications.
- Validate that the resulting project matches the documented folder structure, dependencies, scripts, and configuration.
- Do not introduce technologies, libraries, patterns, or tooling that are not defined in the specifications unless they are required to make the project functional or to integrate correctly with an existing monorepo.