# Configurations & Server Bootstrap

## 1. TypeScript Configuration (`tsconfig.json`)

Configure absolute path mapping for the backend project:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./",
    "baseUrl": "./",
    "paths": {
      "@/config": ["src/config"],
      "@/constants/*": ["src/constants/*"],
      "@/controllers/*": ["src/controllers/*"],
      "@/db": ["src/db"],
      "@/middlewares/*": ["src/middlewares/*"],
      "@/models/*": ["src/models/*"],
      "@/repositories/*": ["src/repositories/*"],
      "@/routes": ["src/routes"],
      "@/schemas/*": ["src/db/schema/*"],
      "@/singleton": ["src/singleton"],
      "@/validators/*": ["src/validators/*"],
      "@/utils/*": ["src/utils/*"]
    }
  },
  "include": ["src/**/*", "scripts/**/*"],
  "exclude": ["node_modules"]
}
```

## 2. Server Bootstrap Flow (`src/index.ts`)

```typescript
import express from "express";
import cors from "cors";
import { dbClient } from "@/db";
import { publicRouter, privateRouter } from "@/routes";

const app = express();
const port = process.env.PORT || 3000;

const startServer = async () => {
  // 1. Initialize database connection
  await dbClient.connect();

  // 2. Setup Global Middlewares
  app.use(express.json());
  app.use(cors({ origin: true, credentials: true }));

  // 3. Register Routing Layers
  app.use("/api/public", publicRouter);
  app.use("/api/private", privateRouter);

  // 4. Start HTTP Server
  app.listen(port, () => {
    console.log(`[server]: API running on port ${port}`);
  });
};

startServer().catch((error) => {
  console.error("[server]: Bootstrapping error:", error);
  process.exit(1);
});
```

---

## 3. Code Formatting Config (`.prettierrc`)

To enforce uniform styles across the codebase, check in a root `.prettierrc` file configured to match the developer handbook rules (2-space indents, single quotes, required semicolons):

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "es5",
  "printWidth": 80
}
```

[Go back to Table of Contents](./spec.md)
