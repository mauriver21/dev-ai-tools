# Skill: Create React API Client

This skill automates the creation of a standardized API client hook for a specific entity/domain in a React application.

---

## 1. Goal
Generate a self-contained API client module (hook and unit tests) for a given entity type under `src/api-clients/`.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **EntityName**: The PascalCase name of the entity/type (e.g., `Product`, `User`, `Category`).
* **EndpointPath**: The resource endpoint path without leading slash (e.g., `products`, `users`, `categories`).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Validate Entity Type
Ensure that the interface/type for `{{EntityName}}` is defined in the project under `src/interfaces/` (e.g., `src/interfaces/{{EntityName}}.ts` or exported via `src/interfaces/index.ts`).

### Step 2: Create Folder
Create the target module directory:
```text
src/api-clients/use{{EntityName}}ApiClient/
```

### Step 3: Write Implementation File (`index.ts`)
Write the custom hook code to `src/api-clients/use{{EntityName}}ApiClient/index.ts`.
Read the template code directly from section 3 ("Reference Boilerplate") of:
* [api-clients.md](../../../tech-specs/react/specs/api-clients/spec.md#3-reference-boilerplate)

Replace all placeholder variables:
* `{{EntityName}}` with the PascalCase entity name.
* `{{EndpointPath}}` with the API endpoint path.
* `{{clientName}}` with the configured Axios client prefix (typically `acme` or custom name).

### Step 4: Write Unit Test File (`index.test.ts`)
Create a Vitest test suite at `src/api-clients/use{{EntityName}}ApiClient/index.test.ts` using MSW.
Read the unit test code template directly from section 6 ("Unit Testing Boilerplate (MSW)") of:
* [api-clients.md](../../../tech-specs/react/specs/api-clients/spec.md#6-unit-testing-boilerplate-msw)

Replace:
* `{{EntityName}}` with the PascalCase entity name.
* `{{EndpointPath}}` with the API endpoint path.

### Step 5: Export the New Client Hook
Export the new hook from the main API clients entry point (`src/api-clients/index.ts`):
```typescript
export * from "./use{{EntityName}}ApiClient";
```
*(If `src/api-clients/index.ts` does not exist, create it).*
