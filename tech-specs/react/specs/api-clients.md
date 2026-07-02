# API Client Specification & Conventions

API clients in the application handle HTTP communication with backend services. They encapsulate requests, structure request payloads, process responses (including headers), and export clean, typed async methods to be consumed by state models, hooks, or components.

This spec outlines the design, folder layout, TypeScript definitions, and best practices for creating and maintaining API client hooks using generic patterns.

---

## 1. Directory & File Conventions

All API clients must reside in the `src/api-clients/` directory. Each client should follow the **Folder-as-Module** convention:

```text
src/api-clients/
├── use{{EntityName}}ApiClient/
│   ├── index.ts          # Main hook implementation & exports
│   └── index.test.ts     # Unit tests for the API client
```

### Naming Guidelines

- **Folder Name**: camelCase starting with `use` and ending with `ApiClient` (e.g., `use{{DomainName}}ApiClient` or `use{{EntityName}}ApiClient`).
- **Hook Name**: Matching the folder name (e.g., `export const use{{EntityName}}ApiClient = ...`).
- **File Name**: Always `index.ts` within the self-contained folder.

---

## 2. Core Architectural Design

API clients are defined as **React Hooks** returning a standard set of async operations (CRUD) for a specific domain. This hook pattern allows:

1. **Access to Context**: Accessing React context, environment variables, or other hooks (like notification hooks, authentication, or localized configurations) if needed.
2. **Axios Integration**: Leveraging pre-configured Axios instances (e.g., `localAxios` from `@utils`) for headers, interceptors, and request/response transformation.

### Generic CRUD Hook Interface

A standard API client provides five main CRUD functions:

| Operation | HTTP Method | Endpoint        | Return Type (Promise)                   |
| :-------- | :---------- | :-------------- | :-------------------------------------- |
| `list`    | `GET`       | `/{{EndpointPath}}`     | `{ data: T[], pagination: Pagination }` |
| `create`  | `POST`      | `/{{EndpointPath}}`     | `T`                                     |
| `read`    | `GET`       | `/{{EndpointPath}}/:id` | `T`                                     |
| `update`  | `PUT`       | `/{{EndpointPath}}/:id` | `T`                                     |
| `remove`  | `DELETE`    | `/{{EndpointPath}}/:id` | `T`                                     |

---

## 3. Reference Boilerplate

The following boilerplate provides the blueprint for writing standard API clients. Replace all `{{Placeholders}}` with the respective entity or domain-specific names.

### File: `src/api-clients/use{{EntityName}}ApiClient/index.ts`

```typescript
import { localAxios } from "@utils";
import { EntityParams, Id, {{EntityName}} } from "@interfaces";

/**
 * Custom hook representing the API client for {{EntityName}} entities.
 * Handles CRUD operations with automatic pagination mapping for the list endpoint.
 */
export const use{{EntityName}}ApiClient = () => {
  /**
   * Fetches a paginated list of {{EntityName}} records.
   * Merges custom query params with default pagination limit (10) and page (0).
   * Parses the pagination count from response headers (`x-total-count`).
   */
  const list = async (params?: EntityParams<{{EntityName}}>) => {
    const paginatedParams = { _limit: 10, _page: 0, ...params };

    const { data, headers } = await localAxios.get<{{EntityName}}[]>("/{{EndpointPath}}", {
      params: paginatedParams,
    });

    return {
      data,
      pagination: {
        count: Number(headers["x-total-count"] || 0),
        limit: paginatedParams._limit,
        page: paginatedParams._page,
      },
    };
  };

  /**
   * Creates a new {{EntityName}} record.
   */
  const create = async (data: {{EntityName}}) => {
    const response = await localAxios.post<{{EntityName}}>("/{{EndpointPath}}", data);
    return response.data;
  };

  /**
   * Updates an existing {{EntityName}} record by ID.
   */
  const update = async (id: Id, data: {{EntityName}}) => {
    const response = await localAxios.put<{{EntityName}}>(`/{{EndpointPath}}/${id}`, data);
    return response.data;
  };

  /**
   * Reads a single {{EntityName}} record details by ID.
   */
  const read = async (id: Id) => {
    const { data } = await localAxios.get<{{EntityName}}>(`/{{EndpointPath}}/${id}`);
    return data;
  };

  /**
   * Deletes a {{EntityName}} record by ID.
   */
  const remove = async (id: Id) => {
    const { data } = await localAxios.delete<{{EntityName}}>(`/{{EndpointPath}}/${id}`);
    return data;
  };

  return {
    list,
    create,
    update,
    read,
    remove,
  };
};
```

---

## 4. Key Design Patterns & Requirements

### A. Pagination and Header Parsing

The `list` endpoint assumes the server uses custom pagination headers:

- **Default Values**: Standardize fallback values: `_limit: 10` and `_page: 0`.
- **Total Count Extraction**: The backend returns the total record count in the `x-total-count` header. Always cast this value to a `Number` safely (falling back to `0` if empty or missing).
- **Return Structure**: The returned payload must conform to the pagination wrapper schema:
  ```typescript
  export interface PaginatedResult<T> {
    data: T[];
    pagination: {
      count: number; // Total elements matching criteria
      limit: number; // Size/Limit of the page
      page: number; // Current page number (0-indexed or 1-indexed depending on API)
    };
  }
  ```

### B. Standard Type Usage

Always import core entity structures and helper types from `@interfaces`:

- **`Id`**: Unified identifier type (e.g. `string | number`) to avoid type mismatching.
- **`EntityParams<T>`**: Wraps search query parameters, filtering, sorting, and pagination options.
- **`T` (Entity Type)**: Represented by the generic or specific entity definition (e.g., `{{EntityName}}`).

### C. Error Handling

- Do not wrap API methods in individual `try/catch` blocks unless you need to transform or recover from errors locally.
- Let the errors bubble up so that the state management layer (Redux, React Query, or local state controllers) and global axios interceptors can handle them uniformly (e.g., displaying error toasts).

---

## 5. Directory Mapping and Imports

To maintain a clean module architecture:

- Define shared Axios instances (like `localAxios`) inside `src/utils/` and import them using the path alias `@utils`.
- Define types (like `EntityParams`, `Id`, and entity schemas) in `src/interfaces/` and import using `@interfaces`.

---

[Go back to Table of Contents](./spec.md)
