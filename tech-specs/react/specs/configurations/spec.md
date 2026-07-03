# Configuration Files

## 1. TypeScript Paths Configuration (`tsconfig.app.json`)

To enable clean path imports, use path alias configurations pointing to `@/*`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable", "dom", "esnext"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": "./",
    "paths": {
      "@/api-clients/*": ["src/api-clients/*"],
      "@/assets/*": ["src/assets/*"],
      "@/components/*": ["src/components/*"],
      "@/constants/*": ["src/constants/*"],
      "@/contexts/*": ["src/contexts/*"],
      "@/form-schemas/*": ["src/form-schemas/*"],
      "@/hocs/*": ["src/hocs/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/interfaces/*": ["src/interfaces/*"],
      "@/i18n/*": ["src/i18n/*"],
      "@/layouts/*": ["src/layouts/*"],
      "@/mocks/*": ["src/mocks/*"],
      "@/models/*": ["src/models/*"],
      "@/pages/*": ["src/pages/*"],
      "@/routes": ["src/routes"],
      "@/states": ["src/states"],
      "@/store": ["src/store.ts"],
      "@/utils/*": ["src/utils/*"]
    }
  },
  "include": ["src"]
}
```

## 2. Vite & Vitest Configuration (`vite.config.ts`)

The build pipeline loads environment variables, automatically checks Typescript types, and registers the `tsconfig.json` path mappings with Vite. It also houses the **Vitest** configuration:

```typescript
/// <reference types="vitest" />
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';
import EnvironmentPlugin from 'vite-plugin-environment';
import checker from 'vite-plugin-checker';
import * as dotenv from 'dotenv';

// Load environment variables according to NODE_ENV
const envPaths = {
  development: '.env.dev',
  production: '.env.prod',
};

dotenv.config({
  path: envPaths[process.env.NODE_ENV as keyof typeof envPaths] || '.env.dev',
});

export default defineConfig({
  plugins: [
    react(),
    // Perform type checking asynchronously in a separate process
    checker({
      typescript: {
        tsconfigPath: './tsconfig.app.json',
      },
      overlay: false,
    }),
    // Resolve TS path mappings automatically in Vite imports
    tsconfigPaths(),
    EnvironmentPlugin('all'),
  ],
  server: {
    port: 5173,
  },
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/setupTests.ts'],
  },
});
```

---

## 3. React App Cleanup

Immediately after scaffolding the React application using Vite, perform the following cleanup tasks to establish a clean starting point:

### 1. Remove Default Boilerplate Assets

- Delete `src/App.css` (we will use Material UI CSS Baseline or styled components).
- Delete `src/assets/react.svg`.
- Empty the contents of `src/index.css` and use it only for root global style resets (e.g., margins, box-sizing).

### 2. Scaffold a Simple Hello World Layout (`src/App.tsx`)

Replace `src/App.tsx` with a basic layout structure rendering a hello world message:

```tsx
import React from 'react';
import { Box, Typography, Container } from '@mui/material';

export const App: React.FC = () => {
  return (
    <Container maxWidth="lg">
      <Box
        sx={{
          my: 4,
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
        }}
      >
        <Typography variant="h4" component="h1" gutterBottom>
          Hello World
        </Typography>
        <Typography variant="body1">
          Welcome to the clean Vite + React + Material UI boilerplate.
        </Typography>
      </Box>
    </Container>
  );
};
```

---

## 4. Code Formatting Config (`.prettierrc`)

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

---

## 5. Version Control Exclusions (`.gitignore`)

Configure exclusions in the root `.gitignore` file to ensure development logs, system files, dependencies, build folders, and Storybook/MSW cached assets are not checked into Git:

```text
# Logs and crash dumps
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Runtime diagnostic reports
report.[0-9]*.[0-9]*.[0-9]*.[0-9]*.json

# Dependency folders
node_modules
/dist
/dist-ssr
*.local

# System/IDE settings
.vscode/*
!.vscode/extensions.json
.idea/
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

# Testing and Coverage
/coverage

# Storybook compiled outputs
/storybook-static

# MSW generated service worker
src/mocks/mockServiceWorker.js
```

---

## 6. Monorepo Root Version Control Exclusions (Root `.gitignore`)

At the **monorepo root directory**, place a simplified global `.gitignore` file. Since each application and package manages its own build, coverage, and mock outputs locally, the root configuration only needs to filter global dependencies, logs, environments, and editor configurations:

```text
# Dependency folders
node_modules/

# Editor/System settings
.vscode/*
!.vscode/extensions.json
.idea/
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

# Environment configs
*.local

# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
```

[Go back to Table of Contents](../README.md)
