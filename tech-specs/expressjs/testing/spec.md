# Testing Setup

## 1. Vitest & Supertest Packages (pnpm)

```bash
pnpm add -D vitest supertest @types/supertest node-mocks-http
```

## 2. Integration Test Boilerplate (`src/controllers/userController/index.test.ts`)

Hit Express routes directly in-memory using `supertest` to run integration tests:

```typescript
import { describe, it, expect } from "vitest";
import request from "supertest";
import express from "express";
import userController from "./index";

const app = express();
app.use(express.json());
app.use("/users", userController);

describe("User Controller Integration Tests", () => {
  it("GET /users/:id should return user details", async () => {
    const res = await request(app).get("/users/123");

    expect(res.status).toBe(200);
    expect(res.body).toHaveProperty("id", "123");
    expect(res.body).toHaveProperty("name", "John Doe");
  });
});
```

[Go back to Table of Contents](./spec.md)
