# Directory Structure & Dependencies

## 1. Directory Structure

All backend modules conform to a strict "Folder-as-Module" layout:

```text
src/
├── config/            # Server configuration, environment schemas, and secrets loader
├── constants/         # Static enums, error keys, and system constants
├── controllers/       # Route request handlers (camelCase folders)
├── db/                # Database connections and ORM schema configurations
│   ├── schema/        # Table schema definitions (entities)
│   └── index.ts       # Database client initialization
├── i18n/              # Localization files and translation setup
├── interfaces/        # Shared TypeScript interfaces and contracts
├── middlewares/       # Request interceptors (auth checks, logs, global error handling)
├── models/            # Domain classes and entities
├── providers/         # Access to infrastructure or contextual services
├── repositories/      # Database access layers (Drizzle queries and transactions)
├── routes/            # API routing entrypoints (divided into public and private)
├── singletons/        # Unique shared instances
├── utils/             # Helper libraries (file helpers, hashing, string formatters)
├── validators/        # Request payload validation schemas (e.g. Yup or Zod)
├── index.ts           # Server entrypoint bootstrapping Express & WebSockets
├── migrate.ts         # Direct database migration execution script
└── singleton.ts       # Singleton instance exports (app instance, socket connection)
```

## 2. Core Dependencies & Packages

| Category                 | Recommended Packages                                             | Description                                        |
| :----------------------- | :--------------------------------------------------------------- | :------------------------------------------------- |
| **Server Framework**     | `express`, `cors`                                                | HTTP server engine and CORS handling               |
| **Database & ORM**       | `drizzle-orm`, `pg`, `@types/pg`                                 | Type-safe TypeScript ORM and PostgreSQL driver     |
| **Realtime Web Sockets** | `socket.io`                                                      | Bidirectional event-based communication            |
| **Authentication**       | `jsonwebtoken`, `bcrypt`, `@types/jsonwebtoken`, `@types/bcrypt` | JWT creation/verification and password hashing     |
| **Validation**           | `yup` (or `zod`), `multer`, `uuid`                               | Input validation, multipart uploads, and UUIDs     |
| **Dev Tools**            | `typescript`, `ts-node-dev`, `tsx`                               | TS watcher, live reload compiler, and file runner  |
| **Path Alias**           | `tsconfig-paths`, `tsc-alias`                                    | Alias resolution for runtime and build compilation |
| **Migrations**           | `drizzle-kit`                                                    | Schema generator and migration CLI tool            |

## 3. Compilation & Dev Scripts (`package.json`)

```json
{
  "scripts": {
    "dev": "ts-node-dev -r tsconfig-paths/register src/index.ts",
    "build": "tsc && tsc-alias && cp package.json dist/ && cp -r drizzle dist/",
    "start": "node dist/src/index.js",
    "db-generate": "drizzle-kit generate",
    "db-migrate": "tsx src/migrate.ts"
  }
}
```

[Go back to Table of Contents](./spec.md)
