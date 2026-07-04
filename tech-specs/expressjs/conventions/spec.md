# Component & Folder Conventions

All code units in the Express service must live in self-contained folders with an `index.ts` entry file.

> [!IMPORTANT]
> **Factory Function Naming Rule:** Any factory function designed to return an instantiated module object (such as a model, repository, or controller) must be prefixed with `create` (e.g., `createItemModel`, `createItemRepository`, `createItemController`).

To ensure clean separation of concerns, testability, and scalability, a request execution traverses **six distinct layers**:

```
Client Request
      │ (HTTP POST /items)
      ▼
┌───────────────────────────┐
│     1. Routing Layer      │  -> src/routes/index.ts
└─────────────┬─────────────┘
              │ (Validates body)
              ▼
┌───────────────────────────┐
│    2. Validation Layer    │  -> src/validators/itemSchema/index.ts
└─────────────┬─────────────┘
              │ (Invokes method)
              ▼
┌───────────────────────────┐
│    3. Controller Layer    │  -> src/controllers/createItemController/index.ts
└─────────────┬─────────────┘
              │ (Calls action)
              ▼
┌───────────────────────────┐
│    4. Repository Layer    │  -> src/repositories/createItemRepository/index.ts
└─────────────┬─────────────┘
              │ (Performs DB query)
              ▼
┌───────────────────────────┐
│      5. Model Layer       │  -> src/models/createItemModel/index.ts
└─────────────┬─────────────┘
              │ (ORM Mapping)
              ▼
┌───────────────────────────┐
│    6. Schema Definition   │  -> src/db/schema/items.ts
└─────────────┬─────────────┘
              │ (Executes SQL)
              ▼
    PostgreSQL Database
```

---

## Layer 1: Schema Definition (`src/db/schema/items.ts`)

Defines the database table structure, columns, types, and constraints using Drizzle ORM.

```typescript
import { InferSelectModel } from "drizzle-orm";
import { pgTable, serial, json, timestamp, varchar } from "drizzle-orm/pg-core";

export const items = pgTable("items", {
  id: serial("id").primaryKey(),
  name: varchar("name", { length: 256 }),
  data: json("data"),
  createdAt: timestamp("created_at", { mode: "string" }).defaultNow(),
  updatedAt: timestamp("updated_at", { mode: "string" }).defaultNow(),
  deletedAt: timestamp("deleted_at"),
});

export type Item = InferSelectModel<typeof items>;
```

---

## Layer 2: Payload Validator

Validates the incoming HTTP request payload schema. The validation middleware intercepts invalid requests before they reach the controllers.

### 1. Schema Validator (`src/validators/itemSchema/index.ts`)

```typescript
import { object, string, array } from "yup";

export const itemSchema = object().shape({
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

## Layer 3: Model Layer (`src/models/createItemModel/index.ts`)

Encapsulates direct ORM operations (SQL insertions, pagination query filters, soft deletes) and hooks (such as timestamp modifiers).

```typescript
import { items, Item } from "@/db/schema/items";
import { eq } from "drizzle-orm";
import { db } from "@/db";

export const createItemModel = () => {
  const create = async (
    item: Omit<Item, "createdAt" | "updatedAt" | "id">,
  ): Promise<Item> => {
    const [data] = await db.insert(items).values(item).returning();
    return data;
  };

  const read = async (id: number): Promise<Item | undefined> => {
    const [data] = await db.select().from(items).where(eq(items.id, id));
    return data;
  };

  return { create, read };
};
```

---

## Layer 4: Repository Layer (`src/repositories/createItemRepository/index.ts`)

Handles storage strategies, file uploads, mapping data, and acts as the bridge between model-level data queries and controller business logic.

```typescript
import { Item } from "@/db/schema/items";
import { createItemModel } from "@/models/createItemModel";

const itemModel = createItemModel();

export const createItemRepository = () => {
  const create = async (
    itemData: Omit<Omit<Item, "createdAt" | "updatedAt">, "id">,
  ) => {
    try {
      const newItem = await itemModel.create(itemData);
      return newItem;
    } catch (error) {
      throw new Error(`[Repo Error] Failed to create item: ${error}`);
    }
  };

  const read = async (id: number) => {
    try {
      const item = await itemModel.read(id);
      if (!item) throw new Error(`Item with ID ${id} not found`);
      return item;
    } catch (error) {
      throw new Error(`[Repo Error] Failed to fetch item: ${error}`);
    }
  };

  return { create, read };
};
```

---

## Layer 5: Controller Layer (`src/controllers/createItemController/index.ts`)

Acts as the HTTP request and response handler. It extracts route params, invokes repositories, manages status codes, and formats API error payloads.

```typescript
import { Request, Response } from "express";
import { createItemRepository } from "@/repositories/createItemRepository";

const itemRepository = createItemRepository();

export const createItemController = () => {
  const create = async (req: Request, res: Response) => {
    try {
      // Exclude ids or read-only properties from client payloads
      const { id: _, ...payload } = req.body;
      const data = await itemRepository.create(payload);

      return res.status(201).json(data);
    } catch (error: any) {
      return res.status(500).json({ error: error.message });
    }
  };

  const read = async (req: Request, res: Response) => {
    try {
      const { id } = req.params;
      const data = await itemRepository.read(Number(id));

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
import { itemSchema } from "@/validators/itemSchema";
import { createItemController } from "@/controllers/createItemController";

const apiRouter = Router();
const itemController = createItemController();

// Create Item endpoint (with payload schema validation)
apiRouter.post("/items", validateBody(itemSchema), itemController.create);

// Read Item endpoint
apiRouter.get("/items/:id", itemController.read);

export { apiRouter };
```

[Go back to Table of Contents](./spec.md)
