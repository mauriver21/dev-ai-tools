---
name: expressjs-create-api
description: Create a complete CRUD API following the Express.js conventions defined in expressjs/conventions/spec.md.
---

# Create API

Create a complete CRUD API for `{{EntityName}}` by following the conventions documented in:

- `expressjs/conventions/spec.md`

Do not generate your own architecture or patterns. Follow the specification exactly.

## Steps

### 1. Create the database schema

- Create `src/db/schema/{{entityNamePlural}}.ts`
- Define the table using Drizzle ORM.
- Include audit fields.
- Support logical deletion through `deletedAt`.

---

### 2. Create the payload validator

- Create `src/validators/{{entityName}}Schema/index.ts`
- Define the validation schema for create and update operations.

---

### 3. Create the model

- Create `src/models/create{{EntityName}}Model/index.ts`
- Implement the database access layer.
- Implement CRUD operations.
- Ensure delete operations perform soft deletes by updating `deletedAt`.

---

### 4. Create the repository

- Create `src/repositories/create{{EntityName}}Repository/index.ts`
- Consume the model.
- Handle repository-level errors.
- Expose CRUD operations.

---

### 5. Create the controller

- Create `src/controllers/create{{EntityName}}Controller/index.ts`
- Implement HTTP handlers for:
  - Create
  - Read
  - Update
  - Delete (logical)
  - List
- Return appropriate HTTP responses.

---

### 6. Register the routes

Update:

- `src/routes/index.ts`

Register endpoints for:

- POST `/{{entityNamePlural}}`
- GET `/{{entityNamePlural}}`
- GET `/{{entityNamePlural}}/:id`
- PUT `/{{entityNamePlural}}/:id`
- DELETE `/{{entityNamePlural}}/:id`

Apply validation middleware where appropriate.

---

## Required Conventions

Follow every convention described in:

- `expressjs/conventions/spec.md`

Including, but not limited to:

- Folder structure
- Layer responsibilities
- Factory function naming (`create{{EntityName}}...`)
- Function expressions
- Named exports
- Named imports
- Self-contained folders with `index.ts`
- Logical deletion using `deletedAt`

Do not duplicate boilerplate or explanations from the specification. Use it as the implementation reference.
