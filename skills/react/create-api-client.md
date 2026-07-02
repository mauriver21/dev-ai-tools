# Skill: Create React API Client

This skill automates the creation of a standardized API client hook for a specific entity/domain in a React application. It strictly adheres to the conventions defined in [api-clients.md](../../tech-specs/react/specs/api-clients.md).

---

## 1. Goal
Generate a self-contained API client module (hook and unit tests) for a given entity type under `src/api-clients/`.

---

## 2. Inputs Needed
Before executing, the agent must identify or request:
* **EntityName**: The PascalCase name of the entity/type (e.g., `Product`, `User`, `Category`).
* **EndpointPath**: The resource endpoint path without leading slash (e.g., `products`, `users`, `categories`).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Validate Entity Type
Ensure that the interface/type for `{{EntityName}}` is defined in the project, typically under `src/interfaces/` (e.g., `src/interfaces/{{EntityName}}.ts` or exported via `src/interfaces/index.ts`). If it is not defined, request definition details or create the interface first.

### Step 2: Create Folder
Create the target module directory:
```text
src/api-clients/use{{EntityName}}ApiClient/
```

### Step 3: Write Implementation File (`index.ts`)
Write the custom hook code to `src/api-clients/use{{EntityName}}ApiClient/index.ts` using the template below. Replace all instances of:
* `{{EntityName}}` with the PascalCase entity name.
* `{{EndpointPath}}` with the API endpoint path.

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

### Step 4: Write Unit Test File (`index.test.ts`)
Create a Vitest test suite at `src/api-clients/use{{EntityName}}ApiClient/index.test.ts` using **MSW (Mock Service Worker)** to intercept network requests and verify the hook outputs:

```typescript
import { describe, it, expect } from "vitest";
import { renderHook } from "@testing-library/react";
import { http, HttpResponse } from "msw";
import { server } from "@/mocks/server"; // Adjust path to project MSW server
import { use{{EntityName}}ApiClient } from "./index";

describe("use{{EntityName}}ApiClient", () => {
  it("should query paginated list with defaults", async () => {
    const mockData = [{ id: "1", name: "Mock Resource" }];
    
    // Intercept GET requests to /{{EndpointPath}}
    server.use(
      http.get("*/{{EndpointPath}}", ({ request }) => {
        const url = new URL(request.url);
        expect(url.searchParams.get("_limit")).toBe("10");
        expect(url.searchParams.get("_page")).toBe("0");

        return HttpResponse.json(mockData, {
          headers: {
            "x-total-count": "25",
          },
        });
      })
    );

    const { result } = renderHook(() => use{{EntityName}}ApiClient());
    const res = await result.current.list();

    expect(res).toEqual({
      data: mockData,
      pagination: { count: 25, limit: 10, page: 0 },
    });
  });

  it("should make a create POST request", async () => {
    const payload = { name: "New Resource" };

    server.use(
      http.post("*/{{EndpointPath}}", async ({ request }) => {
        const body = await request.json();
        expect(body).toEqual(payload);
        return HttpResponse.json({ id: "2", ...payload }, { status: 201 });
      })
    );

    const { result } = renderHook(() => use{{EntityName}}ApiClient());
    const res = await result.current.create(payload as any);

    expect(res).toEqual({ id: "2", ...payload });
  });

  it("should make an update PUT request", async () => {
    const payload = { id: "1", name: "Updated Resource" };

    server.use(
      http.put("*/{{EndpointPath}}/1", async ({ request }) => {
        const body = await request.json();
        expect(body).toEqual(payload);
        return HttpResponse.json(payload);
      })
    );

    const { result } = renderHook(() => use{{EntityName}}ApiClient());
    const res = await result.current.update("1", payload as any);

    expect(res).toEqual(payload);
  });

  it("should make a read GET request", async () => {
    const mockData = { id: "1", name: "Resource" };

    server.use(
      http.get("*/{{EndpointPath}}/1", () => {
        return HttpResponse.json(mockData);
      })
    );

    const { result } = renderHook(() => use{{EntityName}}ApiClient());
    const res = await result.current.read("1");

    expect(res).toEqual(mockData);
  });

  it("should make a remove DELETE request", async () => {
    const mockData = { id: "1", success: true };

    server.use(
      http.delete("*/{{EndpointPath}}/1", () => {
        return HttpResponse.json(mockData);
      })
    );

    const { result } = renderHook(() => use{{EntityName}}ApiClient());
    const res = await result.current.remove("1");

    expect(res).toEqual(mockData);
  });
});
```

### Step 5: Export the New Client Hook
Export the new hook from the main API clients entry point (`src/api-clients/index.ts`) so it can be cleanly imported across the app:

```typescript
export * from "./use{{EntityName}}ApiClient";
```
*(If `src/api-clients/index.ts` does not exist, create it).*
