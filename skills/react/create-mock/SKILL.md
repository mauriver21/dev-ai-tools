# Skill: Create API Mock (MSW)

This skill automates the creation of MSW mock database seeds, request interceptor handlers, and router configurations for a specific entity in a React application. It strictly adheres to the conventions defined in [mocks/spec.md](../../../tech-specs/react/specs/mocks/spec.md).

---

## 1. Goal
Generate MSW mock data seeds and REST API handlers under `src/mocks/` for a specific entity to enable offline development and testing.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **EntityName**: The PascalCase singular name of the entity (e.g., `Product`).
* **entityName**: The camelCase singular name of the entity (e.g., `product`).
* **EndpointPath**: The base API endpoint path for the entity (e.g., `products`).
* **clientName**: The camelCase Axios client prefix (e.g., `acme` for `acmeAxios`).
* **CLIENT_NAME**: The UPPERCASE Axios client prefix (e.g., `ACME` for `BASE_ACME_URL`).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Create Mock Data Seeds File
Create the seed data file at `src/mocks/data/{{entityName}}s.ts`.
Read the structure and boilerplate specifications directly from:
* [mocks/spec.md#3-mock-data-seed-boilerplate](../../../tech-specs/react/specs/mocks/spec.md#3-mock-data-seed-boilerplate)

Ensure you define the mutable mock array (e.g., `export let {{entityName}}s = [...]`) and a setter function (e.g., `export const set{{EntityName}}s = ...`).

### Step 2: Register Seeds in Global Store
Import the new entity seeds and setter functions inside the main mocks database file (`src/mocks/data/index.ts`):
1. Import the variables and setter from `./{{entityName}}s`.
2. Add getter/setter mappings to the consolidated mock database object (matching the existing database layout).

### Step 3: Create Request Handlers
Create the route interceptor file at `src/mocks/handlers/{{entityName}}Handler.ts`.
Do not duplicate code; implement the CRUD route handlers (list, create, update, read, remove) using the MSW boilerplate in:
* [mocks/spec.md#4-request-handler-boilerplate](../../../tech-specs/react/specs/mocks/spec.md#4-request-handler-boilerplate)

Make sure you replace:
* `BASE_{{CLIENT_NAME}}_URL` with the correct environment variable imported from `@/utils/{{clientName}}Axios`.
* `/items` with `/{{EndpointPath}}`.
* `Item` type imports and variables with your `{{EntityName}}` types.
* In-memory database mutations to reference `data.{{entityName}}s`.

### Step 4: Register Handler in Aggregator
Register the new route handlers by exporting them in the handlers index file (`src/mocks/handlers/index.ts`):
```typescript
export * from './{{entityName}}Handler';
```
*(Since `setupWorker`/`setupServer` aggregates handlers dynamically using `Object.values`, exporting the handler array is sufficient for activation).*
