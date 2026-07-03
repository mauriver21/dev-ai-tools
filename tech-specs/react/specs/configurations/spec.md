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

#### 2. Scaffold Initial Application Structure & Layout

Immediately after scaffolding, configure the initial pages, layouts, and routing arrays. This splits the theme selector and mode toggle header from the home page content, routing via React Router DOM.

#### 1. Home Page Component (`src/pages/Home/index.tsx`)
```tsx
import React from 'react';
import { Card, CardContent, Typography, Container } from '@mui/material';

export const Home: React.FC = () => {
  return (
    <Container
      maxWidth="lg"
      sx={{
        flexGrow: 1,
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        py: 4,
      }}
    >
      <Card sx={{ minWidth: 300, textAlign: 'center', p: 4, borderRadius: 2 }}>
        <CardContent>
          <Typography variant="h4" component="h1" gutterBottom sx={{ fontWeight: 'bold' }}>
            Hello World!
          </Typography>
          <Typography color="text.secondary">
            Welcome to the clean Vite + React + Material UI boilerplate.
          </Typography>
        </CardContent>
      </Card>
    </Container>
  );
};
```

#### 2. Main Layout Component (`src/layouts/MainLayout/index.tsx`)
This layout wraps child views with a header bar that provides the global theme select dropdown and light/dark mode switch, rendering child components inside the `<Outlet />` element matching [hello-world-layout.png](../../../skills/react/scaffold-app/resources/images/hello-world-layout.png):
```tsx
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { Outlet } from 'react-router-dom';
import { 
  Box, 
  AppBar, 
  Toolbar, 
  Select, 
  MenuItem, 
  FormControl, 
  InputLabel, 
  Switch, 
  FormControlLabel 
} from '@mui/material';
import { useAppModel } from '@/models/useAppModel';

export const MainLayout: React.FC = () => {
  const dispatch = useDispatch();
  const appModel = useAppModel();
  const appState = useSelector(appModel.selectAppState);

  const activeThemeName = appState.theme || 'default';
  const activeMode = appState.mode || 'light';

  const handleThemeChange = (event: any) => {
    dispatch(appModel.setTheme(event.target.value));
  };

  const handleModeChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const nextMode = event.target.checked ? 'dark' : 'light';
    dispatch(appModel.setMode(nextMode));
  };

  return (
    <Box sx={{ flexGrow: 1, minHeight: '100vh', display: 'flex', flexDirection: 'column', bgcolor: 'background.default' }}>
      <AppBar position="static" color="default" elevation={1}>
        <Toolbar sx={{ justifyContent: 'flex-end', gap: 2 }}>
          <FormControl size="small" sx={{ minWidth: 120 }}>
            <InputLabel id="theme-select-label">Theme</InputLabel>
            <Select
              labelId="theme-select-label"
              id="theme-select"
              value={activeThemeName}
              label="Theme"
              onChange={handleThemeChange}
            >
              <MenuItem value="default">Default</MenuItem>
              <MenuItem value="ocean">Ocean</MenuItem>
              <MenuItem value="sunset">Sunset</MenuItem>
            </Select>
          </FormControl>
          <FormControlLabel
            control={
              <Switch
                checked={activeMode === 'dark'}
                onChange={handleModeChange}
                color="primary"
              />
            }
            label={activeMode === 'dark' ? 'Dark' : 'Light'}
          />
        </Toolbar>
      </AppBar>
      <Box component="main" sx={{ flexGrow: 1, display: 'flex', flexDirection: 'column' }}>
        <Outlet />
      </Box>
    </Box>
  );
};
```

#### 3. Routing Configuration (`src/routes/index.tsx`)
```tsx
import React from 'react';
import { RouteObject } from 'react-router-dom';
import { MainLayout } from '@/layouts/MainLayout';
import { Home } from '@/pages/Home';

export const routes: RouteObject[] = [
  {
    path: '/',
    element: <MainLayout />,
    children: [
      {
        index: true,
        element: <Home />,
      },
    ],
  },
];
```

#### 4. App Entrypoint (`src/App.tsx`)
`App.tsx` serves as the router mount point, consuming and resolving the active route configurations:
```tsx
import React from 'react';
import { useRoutes } from 'react-router-dom';
import { routes } from '@/routes';

export const App: React.FC = () => {
  const element = useRoutes(routes);
  return element;
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
