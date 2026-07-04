# API Mocking (MSW) Architecture & Boilerplate

API Mocking enables offline local development, rapid UI prototyping, and sandbox integration testing without relying on active backend services.

The mocking layer leverages **Mock Service Worker (MSW)**. MSW intercepts requests at the browser's network level using a Service Worker, routing them to local mock handlers instead of hitting a live server.

```
React App
   │ (HTTP Request /api/items)
   ▼
┌───────────────────────────┐
│    MSW Service Worker     │
└─────────────┬─────────────┘
              │ (Intercepts)
              ▼
┌───────────────────────────┐
│     Mock Interceptors     │  <-> In-Memory Mock Data
└─────────────┬─────────────┘
              │ (Simulated HTTP Response)
              ▼
┌───────────────────────────┐
│    MSW Service Worker     │
└─────────────┬─────────────┘
              │ (Resolves Promise)
              ▼
          React App
```

### Key Principles

1. **Stateful In-Memory Mocking**: Mock data is loaded into memory (simulating a lightweight database). Write requests (POST, PUT, DELETE) mutate this memory, allowing the frontend to experience realistic CRUD behavior.
2. **Network Latency Simulation**: Mock handlers introduce synthetic delays (e.g. using `delay(200)`) to test loading animations, skeletons, and UI race conditions.
3. **Parity with Backend Spec**: Request endpoints, search queries, and response payloads match production API specs exactly.

## 1. Service Worker Initialization

Before configuring MSW, the service worker file (`mockServiceWorker.js`) must be generated in the public directory of the application using the MSW CLI tool. This worker script is responsible for intercepting real HTTP traffic in the browser.

Run the following command in the application's root directory:

```bash
npx msw init public --save
```

This places the worker script inside `public/` and links it to your `package.json` to keep it updated automatically with future MSW updates.

---

## 2. Directory Layout

All mock configurations, data, and request interceptors reside in the `src/mocks/` directory:

```text
src/mocks/
├── data/
│   ├── index.ts         # Aggregates mock datasets
│   ├── items.ts         # Item entity seed mock data
│   └── users.ts         # User entity seed mock data
├── handlers/
│   ├── index.ts         # Combines all request interceptors
│   ├── itemHandler.ts   # Mock endpoints for items
│   └── userHandler.ts   # Mock endpoints for users
├── browser.ts           # MSW service worker configuration (browser)
└── server.ts            # MSW mock server configuration (node/testing)
```

---

## 3. Mock Data Seed Boilerplate

Define the mock datasets as mutable arrays or objects, allowing handlers to perform in-memory database updates.

### File: `src/mocks/data/items.ts`

```typescript
import { Item } from '@/interfaces/Item';

export let items: Item[] = [
  { id: '1', name: 'Item One', description: 'This is the first generic item.' },
  {
    id: '2',
    name: 'Item Two',
    description: 'This is the second generic item.',
  },
];

export const setItems = (updated: Item[]) => {
  items = updated;
};
```

### File: `src/mocks/data/index.ts`

```typescript
import { items, setItems } from './items';

export const data = {
  get items() {
    return items;
  },
  set items(value) {
    setItems(value);
  },
};
```

---

## 4. Request Handler Boilerplate

Handlers define the route path and request interceptor logic, simulating common database CRUD patterns.

### File: `src/mocks/handlers/itemHandler.ts`

```typescript
import { HttpResponse, delay, http } from 'msw';
import { BASE_{{CLIENT_NAME}}_URL } from '@/utils/{{clientName}}Axios';
import { paginateData } from '@/utils/paginateData';
import { data } from '../data';
import { Item } from '@/interfaces/Item';
import { v4 as uuid } from 'uuid';

export const itemHandler = [
  // 1. GET List (With Pagination Support)
  http.get(`${BASE_{{CLIENT_NAME}}_URL}/items`, async ({ request }) => {
    const url = new URL(request.url);
    const size = Number(url.searchParams.get('_size') || 10);
    const page = Number(url.searchParams.get('_page') || 0);

    const { content, totalPages } = paginateData(data.items, {
      limit: size,
      page,
    });

    await delay(300); // Simulate network latency

    return HttpResponse.json(
      {
        content,
        pagination: {
          size,
          page,
          totalElements: data.items.length,
          totalPages,
        },
      },
      { status: 200 }
    );
  }),

  // 2. GET Single Record by ID
  http.get(`${BASE_{{CLIENT_NAME}}_URL}/items/:id`, async ({ params }) => {
    const { id } = params;
    const item = data.items.find((i) => i.id === id);

    await delay(200);

    if (!item) {
      return HttpResponse.json(
        { message: `Item with ID ${id} not found` },
        { status: 404 }
      );
    }
    return HttpResponse.json(item, { status: 200 });
  }),

  // 3. POST Create
  http.post(`${BASE_{{CLIENT_NAME}}_URL}/items`, async ({ request }) => {
    const payload = (await request.json()) as Item;
    const newItem = { id: uuid(), ...payload };

    data.items = [...data.items, newItem]; // Push to in-memory database
    await delay(1000);

    return HttpResponse.json(newItem, { status: 201 });
  }),

  // 4. PUT Update
  http.put(`${BASE_{{CLIENT_NAME}}_URL}/items/:id`, async ({ request, params }) => {
    const { id } = params;
    const exists = data.items.some((i) => i.id === id);

    if (!exists) {
      return HttpResponse.json({ message: 'Item not found' }, { status: 404 });
    }

    const payload = (await request.json()) as Item;
    const updatedItem = { ...payload, id };

    data.items = data.items.map((i) => (i.id === id ? updatedItem : i));
    await delay(1000);

    return HttpResponse.json(updatedItem, { status: 200 });
  }),

  // 5. DELETE Remove
  http.delete(`${BASE_{{CLIENT_NAME}}_URL}/items/:id`, async ({ params }) => {
    const { id } = params;
    const item = data.items.find((i) => i.id === id);

    if (!item) {
      return HttpResponse.json({ message: 'Item not found' }, { status: 404 });
    }

    data.items = data.items.filter((i) => i.id !== id);
    await delay(500);

    return HttpResponse.json(item, { status: 200 });
  }),
];
```

### File: `src/mocks/handlers/index.ts`

```typescript
export * from './itemHandler';
```

---

## 5. Worker & Server Configuration

### Browser Service Worker (`src/mocks/browser.ts`)

The browser worker setup wraps handlers for runtime interception in the browser:

```typescript
import { setupWorker } from 'msw/browser';
import * as handlers from './handlers';

export const worker = setupWorker(...Object.values(handlers).flat());
```

### Node Mock Server (`src/mocks/server.ts`)

The mock server setup wraps the same handlers to run inside Node environment during Vitest tests:

```typescript
import { setupServer } from 'msw/node';
import * as handlers from './handlers';

export const server = setupServer(...Object.values(handlers).flat());
```

### Bootstrapping in `main.tsx`

Ensure worker execution only starts inside development/mock environments:

```typescript
import { isDev } from '@/utils/isDev';

const bootstrap = async () => {
  if (isDev()) {
    const { worker } = await import('./mocks/browser');
    await worker.start({ onUnhandledRequest: 'bypass' });
  }
  // Initialize React...
};

bootstrap();
```

[Go back to Table of Contents](../README.md)
