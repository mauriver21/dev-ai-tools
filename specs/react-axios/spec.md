# Axios Configuration Specification

This specification outlines the configuration and conventions for HTTP client instances in the application. HTTP clients are pre-configured using Axios to simplify requests, inject default configurations, and target specific API services.

---

## 1. Directory & File Conventions

All custom Axios instances reside inside the `src/utils/` directory, following the **Folder-as-Module** layout:

```text
src/utils/
└── {{clientName}}Axios/
    └── index.ts          # Axios instance configuration & exports
```

### Naming Guidelines

- **Folder Name**: camelCase matching the client name and ending with `Axios` (e.g., `{{clientName}}Axios`).
- **File Name**: Always `index.ts`.
- **Instance Export**: Named export matching the folder name (e.g., `export const {{clientName}}Axios = ...`).

---

## 2. Axios Configuration

The Axios client instance is initialized with a base URL sourced from Vite environment variables.

### Configuration Template

#### File: `src/utils/{{clientName}}Axios/index.ts`

```typescript
import axios from 'axios';

export const BASE_{{CLIENT_NAME}}_URL = import.meta.env.VITE_BASE_{{CLIENT_NAME}}_URL;

export const {{clientName}}Axios = axios.create({
  baseURL: BASE_{{CLIENT_NAME}}_URL,
});
```

---

## 3. Usage Guidelines

- **Environment Variables**: The environment variable `VITE_BASE_{{CLIENT_NAME}}_URL` must point to the target backend service and be declared in environment files (e.g., `.env.dev`, `.env.prod`) using Vite's nomenclature (`VITE_` prefix).
- **Interceptors**: Additional headers, token injections, and response parsing must be configured on the specific `{{clientName}}Axios` instance using Axios interceptors if needed.
- **Global Path Mapping**: Import the instance utilizing the `@/utils` path alias (e.g., `import { {{clientName}}Axios } from '@/utils'`).

---

[Go back to Table of Contents](../README.md)
