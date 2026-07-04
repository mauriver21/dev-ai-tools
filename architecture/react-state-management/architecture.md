# React Redux Use Model - Developer Handbook

Welcome to the **react-redux-use-model** developer handbook. This guide covers how to set up, build API clients, define custom hook models, handle CRUD actions, utilize automatic pagination/caching, and query normalized state within React + Redux Toolkit applications.

---

## Table of Contents
1. [Overview](#1-overview)
2. [Why react-redux-use-model?](#2-why-react-redux-use-model)
3. [Installation & Setup](#3-installation--setup)
4. [Core Concepts & Actions](#4-core-concepts--actions)
5. [Defining an API Client](#5-defining-an-api-client)
6. [Defining a Custom Model Hook](#6-defining-a-custom-model-hook)
7. [Detailed CRUD Operations](#7-detailed-crud-operations)
   - [List Records & Pagination](#list-records--pagination)
   - [Create a Record](#create-a-record)
   - [Read a Record](#read-a-record)
   - [Update a Record](#update-a-record)
   - [Remove a Record](#remove-a-record)
8. [Selectors Reference](#8-selectors-reference)
9. [Full Quick-Start Example](#9-full-quick-start-example)

---

## 1. Overview

`react-redux-use-model` is a library built on top of **Redux Toolkit** and **react-redux**. It simplifies global state management in React applications by **automatically normalizing entity data**. 

By keeping a single normalized source of truth in Redux, changes made to a record (such as updates or deletions) automatically propagate to all components referencing that record, eliminating the need to duplicate state or manually synchronize multiple slices.

---

## 2. Why react-redux-use-model?

* **Automatic State Normalization**: Ensures structured, flat, and performance-optimized state.
* **Built-in Pagination & Caching**: Easily handles in-memory pagination and preloading via query keys and caching.
* **Simplified CRUD Operations**: Streamlines interactions with backend APIs and decreases boilerplate.
* **Query Keys System**: Lightweight caching system that stores responses in memory and aligns them with normalized data.
* **Intuitive hook APIs**: Access loaders, metadata, and data with simple selectors.

---

## 3. Installation & Setup

Ensure you have `@reduxjs/toolkit` and `react-redux` installed in your project:

```bash
npm install @reduxjs/toolkit react-redux
```

Then install the library:

```bash
npm install react-redux-use-model
```

### Configure the Redux Store (`store.ts`)
Register the library's `normalizedEntitiesState` reducer in your root reducer:

```typescript
import {
  configureStore,
  combineReducers as combineStates,
} from '@reduxjs/toolkit';
import { normalizedEntitiesState } from 'react-redux-use-model';

export const rootState = combineStates({
  normalizedEntitiesState,
});

export const store = configureStore({
  reducer: rootState,
});

export type RootState = ReturnType<typeof rootState>;
export default store;
```

### Wrap the Application (`main.tsx`)
Wrap your React root element with **both** Redux's `<Provider>` and the library's `<ModelProvider>`:

```tsx
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { ModelProvider } from 'react-redux-use-model';
import store from './store';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement);
root.render(
  <Provider store={store}>
    <ModelProvider store={store}>
      <App />
    </Provider>
  </Provider>
);
```

---

## 4. Core Concepts & Actions

The custom models hook (`useModel`) maps specific backend actions directly to the Redux store:

| Action Name | Description | HTTP Verb | TS Handler Type |
| :--- | :--- | :--- | :--- |
| `ENTITY:LIST` | Fetch a page of records from an API endpoint. | `GET` | `ListQueryHandler<T>` |
| `ENTITY:READ` | Fetch a single record from an API endpoint. | `GET` | `ReadQueryHandler<T>` |
| `ENTITY:CREATE` | Create a single record on the backend and insert/normalize in store. | `POST` | `CreateQueryHandler<T>` |
| `ENTITY:UPDATE` | Update a single record on the backend and update store references. | `PUT` | `UpdateQueryHandler<T>` |
| `ENTITY:REMOVE` | Delete a single record on the backend and remove it from store. | `DELETE` | `RemoveQueryHandler<T>` |

---

### 5. Defining an API Client

API client methods can be defined using a client library like Axios (e.g. `{{clientName}}Axios` from `@/utils`). All methods must meet the required type constraints to format response payloads correctly.

```typescript
import { {{clientName}}Axios } from '@/utils';
import { ListResponse, CreateResponse, UpdateResponse, RemoveResponse, ReadResponse, Id, PaginationParams } from 'react-redux-use-model';
import { {{EntityName}} } from '@/interfaces/{{EntityName}}';

export const use{{EntityName}}ApiClient = () => {
  const list = async (params: PaginationParams): Promise<ListResponse<{{EntityName}}>> => {
    const response = await {{clientName}}Axios.get<ListResponse<{{EntityName}}>>('/{{EndpointPath}}', { params });
    return response.data;
  };

  const create = async (entity: Omit<{{EntityName}}, 'id'>): Promise<CreateResponse<{{EntityName}}>> => {
    const response = await {{clientName}}Axios.post<{{EntityName}}>('/{{EndpointPath}}', entity);
    return response.data;
  };

  const update = async (id: Id, entity: {{EntityName}}): Promise<UpdateResponse<{{EntityName}}>> => {
    const response = await {{clientName}}Axios.put<{{EntityName}}>(`/{{EndpointPath}}/${id}`, entity);
    return response.data;
  };

  const read = async (id: Id): Promise<ReadResponse<{{EntityName}}>> => {
    const response = await {{clientName}}Axios.get<{{EntityName}}>(`/{{EndpointPath}}/${id}`);
    return response.data;
  };

  const remove = async (id: Id): Promise<RemoveResponse<{{EntityName}}>> => {
    const response = await {{clientName}}Axios.delete<{{EntityName}}>(`/{{EndpointPath}}/${id}`);
    return response.data;
  };

  return { list, create, update, read, remove };
};
```

---

## 6. Defining a Custom Model Hook

We recommend encapsulating the hook configuration inside a custom React hook (e.g. `use{{EntityName}}Model`) for each of your entities.

```typescript
import { useModel, EntityActionType, ListQueryHandler, CreateQueryHandler, UpdateQueryHandler, RemoveQueryHandler, ReadQueryHandler } from 'react-redux-use-model';
import { {{EntityName}} } from '@/interfaces/{{EntityName}}';
import { use{{EntityName}}ApiClient } from '@/api-clients/use{{EntityName}}ApiClient';
import { EntityName } from '@/constants/enums';

export const use{{EntityName}}Model = () => {
  const {{entityName}}ApiClient = use{{EntityName}}ApiClient();
  
  const model = useModel<
    {{EntityName}},
    {
      list: ListQueryHandler<{{EntityName}}>;
      create: CreateQueryHandler<{{EntityName}}>;
      update: UpdateQueryHandler<{{EntityName}}>;
      read: ReadQueryHandler<{{EntityName}}>;
      remove: RemoveQueryHandler<{{EntityName}}>;
    }
  >({
    entityName: EntityName.{{EntityName}}s, // Must be a unique string identifier
    config: {
      paginationSizeMultiplier: 5, // Optional. Pre-fetches 5x size records to keep in cache.
    },
    handlers: {
      list: {
        apiFn: {{entityName}}ApiClient.list,
        action: EntityActionType.LIST,
      },
      read: {
        apiFn: {{entityName}}ApiClient.read,
        action: EntityActionType.READ,
      },
      create: {
        apiFn: {{entityName}}ApiClient.create,
        action: EntityActionType.CREATE,
        onSuccess: (response) => console.log('Created:', response.data),
        onError: (err) => console.error('Failed to create:', err),
      },
      update: {
        apiFn: {{entityName}}ApiClient.update,
        action: EntityActionType.UPDATE,
      },
      remove: {
        apiFn: {{entityName}}ApiClient.remove,
        action: EntityActionType.REMOVE,
      },
    },
  });

  return model;
};
```

---

## 7. Detailed CRUD Operations

### List Records & Pagination
The list query handler supports backend-driven paginated lists.

```typescript
{{entityName}}Model.list({
  queryKey: '{{entityName}}s-list-key',
  paginationParams: { _page: 0, _size: 10, _filter: '' },
});
```

#### Pagination Size Multiplier
The `paginationSizeMultiplier` determines how many pages of records are pre-fetched and cached in Redux:
* **With Size Multiplier (e.g., `5`)**: If you request page 0 with size 10, `react-redux-use-model` will query the backend API for page 0 with size 50 (`paginationSizeMultiplier * _size`). It keeps these 50 records in memory and slice-renders only the 10 corresponding to the visual page. When the user clicks next page, if the records are within the preloaded block, the transition is instant with **no additional network requests**.
* **Without Size Multiplier (`1`)**: Setting it to 1 aligns the virtual and real API pagination. Every page transition triggers a new API request.

---

### Create a Record
Submits a creation payload to the API. Once the API responds, the record is normalized and injected into the store automatically, updating the cached query lists.

* **API Constraint**: Returns `Promise<CreateResponse<T>>`.
* **Execution**:
```typescript
{{entityName}}Model.create({ name: 'Generic Name' });
```

---

### Read a Record
Fetches a single record from the backend by ID and populates it in the store.

* **API Constraint**: Returns `Promise<ReadResponse<T>>`.
* **Execution**:
```typescript
{{entityName}}Model.read({{entityName}}Id);
```

---

### Update a Record
Sends an update payload (usually a `PUT` request). Once updated, the normalized record is updated in the Redux store, and all UI elements that render it will update simultaneously.

* **API Constraint**: Returns `Promise<UpdateResponse<T>>`.
* **Execution**:
```typescript
{{entityName}}Model.update({{entityName}}Id, { id: {{entityName}}Id, name: 'Generic Name (Updated)' });
```

---

### Remove a Record
Deletes a record on the backend (usually a `DELETE` request). Upon success, the record is removed from the store and references are updated.

* **API Constraint**: Returns `Promise<RemoveResponse<T>>`.
* **Execution**:
```typescript
{{entityName}}Model.remove({{entityName}}Id);
```

> [!NOTE]
> If a deletion causes the current page to become empty (for example, deleting the last element on page 2), the hook automatically decreases the page index in Redux to navigate the user to the previous page.

---

## 8. Selectors Reference

Use these selectors inside Redux's `useSelector` hook to retrieve data and query loading states:

### `selectEntity(state, id)`
Retrieves a normalized record from the store by its ID.
```typescript
const { data: {{entityName}}, loading } = useSelector((state: RootState) =>
  {{entityName}}Model.selectEntity(state, {{entityName}}Id)
);
```

### `selectEntities(state, ids)`
Retrieves multiple normalized records by their IDs.
```typescript
const {{entityName}}s = useSelector((state: RootState) =>
  {{entityName}}Model.selectEntities(state, {{entityName}}Ids)
); // Returns array of { id, data, loading }
```

### `selectAllEntities(state)`
Retrieves all currently cached/normalized records of this entity type.
```typescript
const all{{EntityName}}s = useSelector((state: RootState) =>
  {{entityName}}Model.selectAllEntities(state)
);
```

### `selectPaginatedQuery(state)`
Retrieves the paginated query metadata corresponding to the active `queryKey`.
```typescript
const {
  ids,               // IDs array representing the current visual page
  paginationParams,  // Current parameters ({ _page, _size, _filter })
  pagination,        // Pagination info ({ page, totalPages, totalElements })
  listing,           // Boolean loading state for list queries
  creating,          // Boolean loading state for creates
  updating,          // Boolean loading state for updates
  removing,          // Boolean loading state for deletes
} = useSelector({{entityName}}Model.selectPaginatedQuery);
```

---

## 9. Full Quick-Start Example

Here is a complete end-to-end integration for a generic CRUD application.

### 1. Define Entity (`interfaces/{{EntityName}}.ts`)
```typescript
import { Id } from 'react-redux-use-model';

export type {{EntityName}} = {
  id?: Id;
  name: string;
};
```

### 2. Define Enums (`constants/enums.ts`)
```typescript
export enum EntityName {
  {{EntityName}}s = '{{EntityName}}s',
}

export enum QueryKey {
  {{EntityName}}sCrud = '{{EntityName}}sCrud',
}
```

### 3. Create Axios Instance (`utils/{{clientName}}Axios.ts`)
```typescript
import axios from 'axios';

export const BASE_{{CLIENT_NAME}}_URL = import.meta.env.VITE_BASE_{{CLIENT_NAME}}_URL;
export const {{clientName}}Axios = axios.create({
  baseURL: BASE_{{CLIENT_NAME}}_URL,
});
```

### 4. Create API Client (`api-clients/use{{EntityName}}ApiClient.ts`)
```typescript
import { {{clientName}}Axios } from '@/utils/{{clientName}}Axios';
import { CreateResponse, Id, ListResponse, PaginationParams, ReadResponse, RemoveResponse, UpdateResponse } from 'react-redux-use-model';
import { {{EntityName}} } from '@/interfaces/{{EntityName}}';

export const use{{EntityName}}ApiClient = () => {
  const list = async (params: PaginationParams): Promise<ListResponse<{{EntityName}}>> => {
    const response = await {{clientName}}Axios.get<ListResponse<{{EntityName}}>>('/{{EndpointPath}}', { params });
    return response.data;
  };

  const create = async (entity: {{EntityName}}): Promise<CreateResponse<{{EntityName}}>> => {
    return {{clientName}}Axios.post<{{EntityName}}>('/{{EndpointPath}}', entity);
  };

  const update = async (id: Id, entity: {{EntityName}}): Promise<UpdateResponse<{{EntityName}}>> => {
    return {{clientName}}Axios.put<{{EntityName}}>(`/{{EndpointPath}}/${id}`, entity);
  };

  const read = async (id: Id): Promise<ReadResponse<{{EntityName}}>> => {
    return {{clientName}}Axios.get<{{EntityName}}>(`/{{EndpointPath}}/${id}`);
  };

  const remove = async (id: Id): Promise<RemoveResponse<{{EntityName}}>> => {
    return {{clientName}}Axios.delete<{{EntityName}}>(`/{{EndpointPath}}/${id}`);
  };

  return { list, create, update, read, remove };
};
```

### 5. Create custom Model Hook (`models/use{{EntityName}}Model.ts`)
```typescript
import { EntityName } from '@/constants/enums';
import { use{{EntityName}}ApiClient } from '@/api-clients/use{{EntityName}}ApiClient';
import { useModel, EntityActionType, ListQueryHandler, CreateQueryHandler, UpdateQueryHandler, RemoveQueryHandler, ReadQueryHandler } from 'react-redux-use-model';
import { {{EntityName}} } from '@/interfaces/{{EntityName}}';

export const use{{EntityName}}Model = () => {
  const {{entityName}}ApiClient = use{{EntityName}}ApiClient();
  const model = useModel<
    {{EntityName}},
    {
      list: ListQueryHandler<{{EntityName}}>;
      create: CreateQueryHandler<{{EntityName}}>;
      update: UpdateQueryHandler<{{EntityName}}>;
      read: ReadQueryHandler<{{EntityName}}>;
      remove: RemoveQueryHandler<{{EntityName}}>;
    }
  >({
    entityName: EntityName.{{EntityName}}s,
    config: {
      paginationSizeMultiplier: 5,
    },
    handlers: {
      list: { apiFn: {{entityName}}ApiClient.list, action: EntityActionType.LIST },
      read: { apiFn: {{entityName}}ApiClient.read, action: EntityActionType.READ },
      create: { apiFn: {{entityName}}ApiClient.create, action: EntityActionType.CREATE },
      update: { apiFn: {{entityName}}ApiClient.update, action: EntityActionType.UPDATE },
      remove: { apiFn: {{entityName}}ApiClient.remove, action: EntityActionType.REMOVE },
    },
  });

  const save = (data: {{EntityName}}) => {
    if (data.id) {
      return model.update(data.id, data);
    } else {
      return model.create(data);
    }
  };

  return {
    ...model,
    save,
    saveState: {
      isLoading: model.createState.isLoading || model.updateState.isLoading,
    },
  };
};
```

### 6. Create Entity Item Component (`components/{{EntityName}}Item.tsx`)
```tsx
import React from 'react';
import { Id } from 'react-redux-use-model';
import { useSelector } from 'react-redux';
import { RootState } from '@/store';
import { use{{EntityName}}Model } from '@/models/use{{EntityName}}Model';

export interface {{EntityName}}ItemProps {
  {{entityName}}Id: Id;
}

export const {{EntityName}}Item: React.FC<{{EntityName}}ItemProps> = ({ {{entityName}}Id }) => {
  const {{entityName}}Model = use{{EntityName}}Model();
  const { data: {{entityName}}, loading } = useSelector((state: RootState) =>
    {{entityName}}Model.selectEntity(state, {{entityName}}Id)
  );

  const updateRandom = () => {
    {{entityName}}Model.update({{entityName}}Id, {
      id: {{entityName}}Id,
      name: `${{{entityName}}?.name} (Updated)`
    });
  };

  if (loading) return <div>Loading details...</div>;

  return (
    <div style={{ display: 'flex', justifyContent: 'space-between', padding: '8px', border: '1px solid #ccc', margin: '4px 0' }}>
      <span>{{{{entityName}}Id}}. {{{entityName}}?.name}</span>
      <div>
        <button disabled={{{entityName}}Model.updateState.isLoading} onClick={updateRandom}>
          {{{{entityName}}Model.updateState.isLoading ? 'Updating...' : 'Rename'}
        </button>
        <button disabled={{{entityName}}Model.removeState.isLoading} onClick={() => {{entityName}}Model.remove({{entityName}}Id)}>
          {{{{entityName}}Model.removeState.isLoading ? 'Removing...' : 'Delete'}
        </button>
      </div>
    </div>
  );
};
```

### 7. Create Main Entities CRUD Container (`components/{{EntityName}}sCrud.tsx`)
```tsx
import React, { useEffect } from 'react';
import { useSelector } from 'react-redux';
import { QueryKey } from '@/constants/enums';
import { use{{EntityName}}Model } from '@/models/use{{EntityName}}Model';
import { {{EntityName}}Item } from './{{EntityName}}Item';
import { PaginationParams, useDebounce } from 'react-redux-use-model';

export const {{EntityName}}sCrud: React.FC = () => {
  const {{entityName}}Model = use{{EntityName}}Model();
  const {{entityName}}PaginatedQuery = useSelector({{entityName}}Model.selectPaginatedQuery);
  const {
    paginationParams = { _page: 0, _size: 10, _filter: '' },
    creating,
    ids,
    pagination,
  } = {{entityName}}PaginatedQuery;

  const create{{EntityName}} = () => {
    {{entityName}}Model.create({ name: `Random ${Math.floor(Math.random() * 1000)}` });
  };

  const list{{EntityName}}s = (params: PaginationParams) => {
    {{entityName}}Model.list({
      queryKey: QueryKey.{{EntityName}}sCrud,
      paginationParams: params,
    });
  };

  const debouncedList{{EntityName}}s = useDebounce(list{{EntityName}}s);

  useEffect(() => {
    list{{EntityName}}s(paginationParams);
  }, []);

  return (
    <div style={{ padding: '16px' }}>
      <h2>Database Container</h2>
      <div style={{ display: 'flex', gap: '8px', marginBottom: '16px' }}>
        <input
          type="text"
          placeholder="Filter..."
          onChange={(e) => debouncedList{{EntityName}}s({ ...paginationParams, _page: 0, _filter: e.target.value })}
        />
        <button disabled={creating} onClick={create{{EntityName}}}>
          {creating ? 'Adding...' : 'Add Random'}
        </button>
      </div>

      <div>
        {ids?.map((id) => (
          <{{EntityName}}Item key={id} {{entityName}}Id={id} />
        ))}
      </div>

      <div style={{ marginTop: '16px', display: 'flex', justifyContent: 'space-between' }}>
        <button
          disabled={paginationParams._page === 0}
          onClick={() => list{{EntityName}}s({ ...paginationParams, _page: paginationParams._page - 1 })}
        >
          Previous
        </button>
        <span>Page {paginationParams._page + 1} of {pagination?.totalPages || 1}</span>
        <button
          disabled={paginationParams._page + 1 >= (pagination?.totalPages || 1)}
          onClick={() => list{{EntityName}}s({ ...paginationParams, _page: paginationParams._page + 1 })}
        >
          Next
        </button>
      </div>
    </div>
  );
};
```

---

[Go back to Table of Contents](../README.md)
