# PNPM Monorepo Scaffolding Specification

## Overview

Create a scalable **pnpm workspace monorepo** that supports multiple applications and shared packages.

The repository should be organized into three top-level categories:

- **packages** — Shared libraries and tooling.
- **web** — Frontend applications.
- **api** — Backend services.

The structure should allow adding new workspaces without modifying the overall architecture.

---

## Repository Structure

```text
.
├── api/
│   ├── auth-service/
│   └── users-service/
│
├── web/
│   ├── admin/
│   └── marketing/
│
├── packages/
│   ├── ui/
│   ├── utils/
│   ├── types/
│   └── eslint-config/
│
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── .gitignore
└── README.md
```

---

## Workspace Configuration

```yaml
packages:
  - packages/*
  - web/*
  - api/*
```

---

## Package Responsibilities

### packages/

Contains reusable libraries shared across applications.

Examples:

- UI components
- Utilities
- Shared types
- API clients
- Configuration
- Shared tooling

### web/

Contains independent frontend applications.

Each application should be buildable and deployable independently.

### api/

Contains independent backend services.

Each service should own its own runtime, configuration, and deployment.

---

## Root package.json

The root package should be private and only orchestrate workspace tasks.

Example scripts:

```json
{
  "private": true,
  "packageManager": "pnpm",
  "scripts": {
    "dev": "pnpm -r dev",
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "lint": "pnpm -r lint",
    "format": "pnpm -r format",
    "typecheck": "pnpm -r typecheck",
    "clean": "pnpm -r clean"
  }
}
```

---

## General Guidelines

- Use **kebab-case** for workspace names.
- Keep applications independent.
- Place reusable code under `packages`.
- Avoid dependencies between applications.
- Share common tooling through workspace packages when appropriate.
- Keep the structure simple and scalable.
