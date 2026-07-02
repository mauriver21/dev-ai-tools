# Skill: Configure Axios Client

This skill automates the configuration of a standardized Axios HTTP client instance in the `src/utils/` directory. It strictly adheres to the conventions defined in [axios.md](../../../tech-specs/react/specs/axios/spec.md).

---

## 1. Goal
Generate a custom pre-configured Axios instance under `src/utils/{{clientName}}Axios/` for making HTTP requests to specific backend services.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **clientName**: The camelCase name representing the client type/purpose (e.g., `local` to create `localAxios`, `public` to create `publicAxios`).
* **CLIENT_NAME**: The UPPERCASE representation of the clientName (e.g., `LOCAL` or `PUBLIC`).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Create Folder
Create the target utility directory:
```text
src/utils/{{clientName}}Axios/
```

### Step 2: Write Configuration File (`index.ts`)
Write the Axios instance configuration code to `src/utils/{{clientName}}Axios/index.ts`.
Do not duplicate code; read the template and naming conventions directly from:
* [axios.md](../../../tech-specs/react/specs/axios/spec.md#2-axios-configuration)

Replace:
* `{{clientName}}` with the camelCase client name (e.g., `local`).
* `{{CLIENT_NAME}}` with the UPPERCASE representation (e.g., `LOCAL`).

### Step 3: Declare Environment Variable
Verify or add the environment variable declaration in the project's environment configuration files (e.g., `.env.dev`, `.env.prod`) using Vite's prefix convention:
```text
VITE_BASE_{{CLIENT_NAME}}_URL=http://localhost:3000
```

### Step 4: Export the Custom Client
Export the configured Axios instance from the main utilities index file (`src/utils/index.ts`) for clean importing:
```typescript
export * from "./{{clientName}}Axios";
```
*(If `src/utils/index.ts` does not exist, create it).*
