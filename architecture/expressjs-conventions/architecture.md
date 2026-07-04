# Component & Folder Conventions

All code units in the Express service must live in self-contained folders with an `index.ts` entry file.

> [!IMPORTANT]
> **Factory Function Naming Rule:** Any factory function designed to return an instantiated module object (such as a model, repository, or controller) must be prefixed with `create` (e.g., `create{{EntityName}}Model`, `create{{EntityName}}Repository`, `create{{EntityName}}Controller`).

To ensure clean separation of concerns, testability, and scalability, a request execution traverses **six distinct layers**:

```text
Client Request
      │ (HTTP POST /{{entityNamePlural}})
      ▼
┌───────────────────────────┐
│     1. Routing Layer      │  -> src/routes/index.ts
└─────────────┬─────────────┘
              │ (Validates body)
              ▼
┌───────────────────────────┐
│    2. Validation Layer    │  -> src/validators/{{entityName}}Schema/index.ts
└─────────────┬─────────────┘
              │ (Invokes method)
              ▼
┌───────────────────────────┐
│    3. Controller Layer    │  -> src/controllers/create{{EntityName}}Controller/index.ts
└─────────────┬─────────────┘
              │ (Calls action)
              ▼
┌───────────────────────────┐
│    4. Repository Layer    │  -> src/repositories/create{{EntityName}}Repository/index.ts
└─────────────┬─────────────┘
              │ (Performs DB query)
              ▼
┌───────────────────────────┐
│      5. Model Layer       │  -> src/models/create{{EntityName}}Model/index.ts
└─────────────┬─────────────┘
              │ (ORM Mapping)
              ▼
┌───────────────────────────┐
│    6. Schema Definition   │  -> src/db/schema/{{entityNamePlural}}.ts
└─────────────┬─────────────┘
              │ (Executes SQL)
              ▼
    PostgreSQL Database
```

---

## Layer 1: Schema Definition (`src/db/schema/{{entityNamePlural}}.ts`)

Defines the database table structure, columns, types, and constraints using Drizzle ORM.

```typescript
import { InferSelectModel } from "drizzle-orm";
import { pgTable, serial, json, timestamp, varchar } from "drizzle-orm/pg-core";

export const {{entityNamePlural}} = pgTable("{{entityNamePlural}}", {
  id: serial("id").primaryKey(),
  name: varchar("name", { length: 256 }),
  data: json("data"),
  createdAt: timestamp("created_at", { mode: "string" }).defaultNow(),
  updatedAt: timestamp("updated_at", { mode: "string" }).defaultNow(),
  deletedAt: timestamp("deleted_at"),
});

export type {{EntityName}} = InferSelectModel<typeof {{entityNamePlural}}>;
```

---

## Derived Entity Types (`src/interfaces/`)

Define reusable CRUD types derived from `{{EntityType}}`. Each type must live in its own file under the `src/interfaces/` folder.

```text
src/
└── interfaces/
    ├── {{EntityName}}Create.ts
    ├── {{EntityName}}Update.ts
    ├── {{EntityName}}Read.ts
    ├── {{EntityName}}Delete.ts
    └── {{EntityName}}List.ts
```

### `src/interfaces/{{EntityName}}Create.ts`

Used when creating a new entity.

```typescript
import { {{EntityType}} } from "@/db/schema/{{entityNamePlural}}";

export type {{EntityName}}Create = Omit<
  {{EntityType}},
  "id" | "createdAt" | "updatedAt" | "deletedAt"
>;
```

### `src/interfaces/{{EntityName}}Update.ts`

Used when updating an existing entity.

```typescript
import { {{EntityType}} } from "@/db/schema/{{entityNamePlural}}";

export type {{EntityName}}Update = Partial<
  Omit<
    {{EntityType}},
    "id" | "createdAt" | "updatedAt" | "deletedAt"
  >
>;
```

### `src/interfaces/{{EntityName}}Read.ts`

Represents the complete entity returned by read operations.

```typescript
import { {{EntityType}} } from "@/db/schema/{{entityNamePlural}}";

export type {{EntityName}}Read = {{EntityType}};
```

### `src/interfaces/{{EntityName}}Delete.ts`

Represents the payload required for a delete (soft or hard) operation.

```typescript
export interface {{EntityName}}Delete {
  id: number;
}
```

### `src/interfaces/{{EntityName}}List.ts`

Represents a paginated list response.

```typescript
import { {{EntityName}}Read } from "./{{EntityName}}Read";

export interface {{EntityName}}List {
  data: {{EntityName}}Read[];
  total: number;
  page: number;
  pageSize: number;
}
```

> [!IMPORTANT]
> All CRUD layers (validators, controllers, repositories, and models) should consume these derived types instead of using `Omit`, `Partial`, or `InferSelectModel` directly. This centralizes the API contracts and avoids duplicating type transformations throughout the codebase.

---

## Layer 2: Payload Validator

Validates the incoming HTTP request payload schema. The validation middleware intercepts invalid requests before they reach the controllers.

### 1. Schema Validator (`src/validators/{{entityName}}Schema/index.ts`)

```typescript
import { object, string, array } from "yup";

export const {{entityName}}Schema = object().shape({
  name: string().required(),
  data: array().required().default([]),
});
```

### 2. Validation Middleware (`src/middlewares/validateBody/index.ts`)

```typescript
import { Request, Response, NextFunction } from "express";
import { AnySchema, ValidationError } from "yup";

export const validateBody = (schema: AnySchema) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.validate(req.body, { abortEarly: false });
      next();
    } catch (error) {
      if (error instanceof ValidationError) {
        return res.status(400).json({ errors: error.errors });
      }
      next(error);
    }
  };
};
```

---

## Layer 3: Model Layer (`src/models/create{{EntityName}}Model/index.ts`)

Encapsulates direct ORM operations (SQL insertions, pagination query filters, soft deletes) and hooks (such as timestamp modifiers).

```typescript
import { {{entityNamePlural}}, {{EntityName}} } from "@/db/schema/{{entityNamePlural}}";
import { eq } from "drizzle-orm";
import { db } from "@/db";

export const create{{EntityName}}Model = () => {
  const create = async (
    entity: Omit<{{EntityName}}, "createdAt" | "updatedAt" | "id">,
  ): Promise<{{EntityName}}> => {
    const [data] = await db.insert({{entityNamePlural}}).values(entity).returning();
    return data;
  };

  const read = async (id: number): Promise<{{EntityName}} | undefined> => {
    const [data] = await db
      .select()
      .from({{entityNamePlural}})
      .where(eq({{entityNamePlural}}.id, id));

    return data;
  };

  return { create, read };
};
```

---

## Layer 4: Repository Layer (`src/repositories/create{{EntityName}}Repository/index.ts`)

Handles storage strategies, file uploads, mapping data, and acts as the bridge between model-level data queries and controller business logic.

```typescript
import { {{EntityName}} } from "@/db/schema/{{entityNamePlural}}";
import { create{{EntityName}}Model } from "@/models/create{{EntityName}}Model";

const {{entityName}}Model = create{{EntityName}}Model();

export const create{{EntityName}}Repository = () => {
  const create = async (
    entityData: Omit<Omit<{{EntityName}}, "createdAt" | "updatedAt">, "id">,
  ) => {
    try {
      return await {{entityName}}Model.create(entityData);
    } catch (error) {
      throw new Error(`[Repo Error] Failed to create {{entityName}}: ${error}`);
    }
  };

  const read = async (id: number) => {
    try {
      const entity = await {{entityName}}Model.read(id);

      if (!entity) {
        throw new Error("{{EntityName}} with ID " + id + " not found");
      }

      return entity;
    } catch (error) {
      throw new Error(`[Repo Error] Failed to fetch {{entityName}}: ${error}`);
    }
  };

  return { create, read };
};
```

---

## Layer 5: Controller Layer (`src/controllers/create{{EntityName}}Controller/index.ts`)

Acts as the HTTP request and response handler. It extracts route params, invokes repositories, manages status codes, and formats API error payloads.

```typescript
import { Request, Response } from "express";
import { create{{EntityName}}Repository } from "@/repositories/create{{EntityName}}Repository";

const {{entityName}}Repository = create{{EntityName}}Repository();

export const create{{EntityName}}Controller = () => {
  const create = async (req: Request, res: Response) => {
    try {
      // Exclude ids or read-only properties from client payloads
      const { id: _, ...payload } = req.body;

      const data = await {{entityName}}Repository.create(payload);

      return res.status(201).json(data);
    } catch (error: any) {
      return res.status(500).json({ error: error.message });
    }
  };

  const read = async (req: Request, res: Response) => {
    try {
      const { id } = req.params;

      const data = await {{entityName}}Repository.read(Number(id));

      return res.status(200).json(data);
    } catch (error: any) {
      return res.status(500).json({ error: error.message });
    }
  };

  return { create, read };
};
```

---

## Layer 6: Routing Layer (`src/routes/index.ts`)

Declares the route endpoints, mapping verbs, and paths, and registers authorization/validation middlewares before calling controller handlers.

```typescript
import { Router } from "express";
import { validateBody } from "@/middlewares/validateBody";
import { {{entityName}}Schema } from "@/validators/{{entityName}}Schema";
import { create{{EntityName}}Controller } from "@/controllers/create{{EntityName}}Controller";

const apiRouter = Router();
const {{entityName}}Controller = create{{EntityName}}Controller();

// Create {{EntityName}} endpoint (with payload schema validation)
apiRouter.post(
  "/{{entityNamePlural}}",
  validateBody({{entityName}}Schema),
  {{entityName}}Controller.create,
);

// Read {{EntityName}} endpoint
apiRouter.get(
  "/{{entityNamePlural}}/:id",
  {{entityName}}Controller.read,
);

export { apiRouter };
```

---

# General Coding Conventions

The following conventions apply to every backend layer unless a library or framework explicitly requires a different approach.

## Function Declaration Rule

Prioritize **function expressions** over function declarations.

✅ Preferred

```typescript
const create = async () => {};
```

❌ Avoid

```typescript
async function create() {}
```

---

## Export Rule

Prioritize **named exports** and avoid `export default`.

✅ Preferred

```typescript
export const create{{EntityName}}Controller = () => {};
```

❌ Avoid

```typescript
export default create{{EntityName}}Controller;
```

---

## Import Rule

Prioritize **named imports** over default imports whenever possible. Modules should expose named exports so they can be imported consistently throughout the project.

✅ Preferred

```typescript
import { create{{EntityName}}Repository } from "@/repositories/create{{EntityName}}Repository";
```

❌ Avoid

```typescript
import create{{EntityName}}Repository from "@/repositories/create{{EntityName}}Repository";
```

[Go back to Table of Contents](./spec.md)
