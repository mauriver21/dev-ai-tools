# Skill: Scaffold React Application

This skill automates the scaffolding and bootstrapping of a complete, premium React TypeScript application. It orchestrates project initialization and configures the codebase to conform strictly to the specifications defined in the repository.

---

## 1. Goal
Initialize a standard, production-ready React web application in a target directory with all necessary configurations and architectural layers.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **projectName**: The directory name to initialize the app in, or `./` for the current workspace.
* **packageManager**: The package manager to use (e.g., `npm`, `pnpm`, default to `pnpm` if monorepo, or `npm` otherwise).

---

## 3. Scaffolding Runbook

### Step 1: Initialize Vite Project
Create a TypeScript-enabled React application using Vite. Run the command in non-interactive mode.
If initializing in the current directory `./`:
```bash
npx -y create-vite@latest ./ --template react-ts
```
If initializing in a new directory:
```bash
npx -y create-vite@latest {{projectName}} --template react-ts
```

### Step 2: Install Core Dependencies
Install the required packages categorized by their architectural layers:
* **Production Packages**: `@mui/material`, `@emotion/react`, `@emotion/styled`, `@reduxjs/toolkit`, `react-redux`, `redux-persist`, `react-redux-use-model`, `react-router-dom`, `axios`, `i18next`, `react-i18next`, `uuid`.
* **Dev Packages**: `vitest`, `@testing-library/react`, `@testing-library/jest-dom`, `@testing-library/user-event`, `jsdom`, `msw`, `vite-tsconfig-paths`, `vite-plugin-environment`, `vite-plugin-checker`, `dotenv`, `eslint`, `prettier`, `@storybook/react-vite`.

Refer to the recommended package catalog in:
* [structure.md#2-core-dependencies--packages](../../../tech-specs/react/specs/structure.md#2-core-dependencies--packages)

### Step 3: Setup Directory Layout
Scaffold the standard "Folder-as-Module" directory structure under `src/` (creating directories like `api-clients`, `components`, `states`, `utils`, `interfaces`, `mocks`, `i18n`, etc.).
Refer to the layout map in:
* [structure.md#1-directory-structure](../../../tech-specs/react/specs/structure.md#1-directory-structure)

### Step 4: Configure Base Project Files
Configure configurations and versions:
1. **tsconfig.app.json**: Setup TS path alias configurations pointing to `@/*` matching [configurations.md#1-typescript-paths-configuration-tsconfigappjson](../../../tech-specs/react/specs/configurations.md#1-typescript-paths-configuration-tsconfigappjson).
2. **vite.config.ts**: Setup Vite bundler plugins, Vitest configurations, environment loadings, and type checkers matching [configurations.md#2-vite--vitest-configuration-viteconfigts](../../../tech-specs/react/specs/configurations.md#2-vite--vitest-configuration-viteconfigts).
3. **.prettierrc**: Setup code formatter settings matching [configurations.md#4-code-formatting-config-prettierrc](../../../tech-specs/react/specs/configurations.md#4-code-formatting-config-prettierrc).
4. **.gitignore**: Setup exclusions matching [configurations.md#5-version-control-exclusions-gitignore](../../../tech-specs/react/specs/configurations.md#5-version-control-exclusions-gitignore).

### Step 5: Clean Default Assets
Clean up Vite's default templates, empty the contents of `src/index.css`, and replace `src/App.tsx` with a basic Hello World layout matching the specification:
* [configurations.md#3-react-app-cleanup](../../../tech-specs/react/specs/configurations.md#3-react-app-cleanup)

### Step 6: Setup Internationalization (i18n)
Configure namespaces, catalogs, dynamic translations indexes, and the custom reactive translations wrapper component.
Do not duplicate code; copy catalog structures and the reactive provider component implementation directly from:
* [i18n.md](../../../tech-specs/react/specs/i18n.md)

### Step 7: Setup API Mocking (MSW)
1. Initialize the service worker in the public directory:
   ```bash
   npx msw init public --save
   ```
2. Setup the browser worker, Node mock server, and mock data seeds by matching instructions and handlers boilerplates in:
   * [mocks.md](../../../tech-specs/react/specs/mocks.md)
3. Set up the test suite setup helper `src/setupTests.ts` to intercept HTTP requests using MSW during test runs matching:
   * [testing.md#2-integrating-msw-with-vitest](../../../tech-specs/react/specs/testing.md#2-integrating-msw-with-vitest)

### Step 8: Setup Storybook
1. Initialize Storybook:
   ```bash
   npx storybook@latest init --yes
   ```
2. Configure Redux, router, and i18n decorators inside `.storybook/preview.tsx` by implementing the configuration template in:
   * [testing.md#3-minimal-storybook-setup](../../../tech-specs/react/specs/testing.md#3-minimal-storybook-setup)

### Step 9: Configure Store and Hydration Entrypoint
1. Set up the global Redux store configuration `src/store.ts` by combining state reducers matching [state-management.md#configure-the-redux-store-storets](../../../tech-specs/react/specs/state-management.md#configure-the-redux-store-storets).
2. Wire up the bootstrap loader in `src/main.tsx` to conditionally start the MSW browser worker and render the React root wrapped in Redux and i18n providers matching [state-management.md#wrap-the-application-maintsx](../../../tech-specs/react/specs/state-management.md#wrap-the-application-maintsx) and [mocks.md#5-worker--server-configuration](../../../tech-specs/react/specs/mocks.md#5-worker--server-configuration).

