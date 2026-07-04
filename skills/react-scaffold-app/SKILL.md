---
name: react-scaffold-app
description: Scaffold and bootstrap a complete React TypeScript application and configure the codebase to conform to the specifications defined in the repository.
---

# Skill: Scaffold React Application

This skill automates the scaffolding and bootstrapping of a complete, premium React TypeScript application. It orchestrates project initialization and configures the codebase to conform strictly to the specifications defined in the repository.

---

## 1. Goal

Initialize a standard, production-ready React web application in a target directory with all necessary configurations and architectural layers.

---

## 2. Inputs Needed

Before executing, the agent must identify:

- **projectName**: The directory name to initialize the app in, or `./` for the current workspace.
- **packageManager**: The package manager to use. The agent **MUST** explicitly ask the user to choose between **pnpm** (prioritized & recommended) or **npm** before installing dependencies.

---

## 3. Scaffolding Runbook

### Step 1: Query Package Manager Preference

First, prompt the user in chat (or using a questionnaire tool if available) to select their package manager of choice:

- **Option 1**: `pnpm` (Recommended / Prioritized)
- **Option 2**: `npm`

Once the user selects the package manager, proceed with initialization.

### Step 2: Initialize Vite Project

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

- **Production Packages**: `@mui/material`, `@emotion/react`, `@emotion/styled`, `@reduxjs/toolkit`, `react-redux`, `redux-persist`, `react-redux-use-model`, `react-router-dom`, `axios`, `i18next`, `react-i18next`, `uuid`.
- **Dev Packages**: `vitest`, `@testing-library/react`, `@testing-library/jest-dom`, `@testing-library/user-event`, `jsdom`, `msw`, `vite-tsconfig-paths`, `vite-plugin-environment`, `vite-plugin-checker`, `dotenv`, `eslint`, `prettier`, `@storybook/react-vite`.

Refer to the recommended package catalog in:

- [architecture.md#2-core-dependencies--packages](architecture/react-structure/architecture.md#2-core-dependencies--packages)

### Step 3: Setup Directory Layout

Scaffold the standard "Folder-as-Module" directory structure under `src/` (creating directories like `api-clients`, `components`, `states`, `utils`, `interfaces`, `mocks`, `i18n`, etc.).
Refer to the layout map in:

- [architecture.md#1-directory-structure](architecture/react-structure/architecture.md#1-directory-structure)

### Step 4: Configure Base Project Files

Configure configurations and versions:

1. **tsconfig.app.json**: Setup TS path alias configurations pointing to `@/*` matching [architecture.md#1-typescript-paths-configuration-tsconfigappjson](architecture/react-configurations/architecture.md#1-typescript-paths-configuration-tsconfigappjson).
2. **vite.config.ts**: Setup Vite bundler plugins, Vitest configurations, environment loadings, and type checkers matching [architecture.md#2-vite--vitest-configuration-viteconfigts](architecture/react-configurations/architecture.md#2-vite--vitest-configuration-viteconfigts).
3. **.prettierrc**: Setup code formatter settings matching [architecture.md#4-code-formatting-config-prettierrc](architecture/react-configurations/architecture.md#4-code-formatting-config-prettierrc).
4. **.gitignore**: Setup exclusions matching [architecture.md#5-version-control-exclusions-gitignore](architecture/react-configurations/architecture.md#5-version-control-exclusions-gitignore).

### Step 5: Clean Default Assets

Clean up Vite's default templates, empty the contents of `src/index.css`, and replace `src/App.tsx` with a refined Hello World layout that implements the theme selector and mode toggle switch layout illustrated in the wireframe:

- **Wireframe Layout Source**: [hello-world-layout.png](./resources/images/hello-world-layout.png)
- **Code Specification**: [architecture.md#3-react-app-cleanup](architecture/react-configurations/architecture.md#3-react-app-cleanup)

Ensure that:

1. The top header/app bar contains a Dropdown menu (`Select`) to select the active theme (e.g. `'default'`, `'ocean'`, `'sunset'`), a Dropdown menu (`Select`) to select the active language (e.g. `'en'`, `'es'`), and a Switch toggle to alternate between light and dark mode.
2. Changes to the theme select, language select, and mode toggle are dispatched to Redux state using the standard `useAppModel()` hook.
3. The main page area renders a centered Card element displaying "Hello World!" in a prominent Typography block.

### Step 6: Setup Internationalization (i18n)

Configure namespaces, catalogs, dynamic translations indexes, and the custom reactive translations wrapper component.
Do not duplicate code; copy catalog structures and the reactive provider component implementation directly from:

- [architecture.md](architecture/react-i18n/architecture.md)

### Step 7: Setup API Mocking (MSW)

1. Initialize the service worker in the public directory:
   ```bash
   npx msw init public --save
   ```
2. Setup the browser worker, Node mock server, and mock data seeds by matching instructions and handlers boilerplates in:
   - [architecture.md](architecture/react-mocks/architecture.md)
3. Set up the test suite setup helper `src/setupTests.ts` to intercept HTTP requests using MSW during test runs matching:
   - [architecture.md#2-integrating-msw-with-vitest](architecture/react-testing/architecture.md#2-integrating-msw-with-vitest)

### Step 8: Setup Storybook

1. Initialize Storybook:
   ```bash
   npx storybook@latest init --yes
   ```
2. Configure Redux, router, and i18n decorators inside `.storybook/preview.tsx` by implementing the configuration template in:
   - [architecture.md#3-minimal-storybook-setup](architecture/react-testing/architecture.md#3-minimal-storybook-setup)

### Step 9: Configure Store and Hydration Entrypoint

1. Set up the global Redux store configuration `src/store.ts` by combining state reducers matching [architecture.md#configure-the-redux-store-storets](architecture/react-state-management/architecture.md#configure-the-redux-store-storets).
2. Wire up the bootstrap loader in `src/main.tsx` to conditionally start the MSW browser worker and render the React root wrapped in Redux and i18n providers matching [architecture.md#wrap-the-application-maintsx](architecture/react-state-management/architecture.md#wrap-the-application-maintsx) and [architecture.md#5-worker--server-configuration](architecture/react-mocks/architecture.md#5-worker--server-configuration).

### Step 10: Verify the Application is Running

Verify that the scaffolded application compiles and runs successfully:

1. Run the local development server (e.g., `pnpm dev` or `npm run dev`).
2. Verify that there are no terminal compilation errors or runtime exceptions in the browser.
3. If the application fails to run or compile, investigate the logs and apply the necessary fixes. Typical solutions include:
   - Resolving alias path resolutions in `tsconfig.app.json` by adding the `./` prefix (e.g. `"@/routes/*": ["./src/routes/*"]` instead of `"src/routes/*"`).
   - Checking that all state reducers are imported and combined correctly.
   - Checking that relative paths in imports match target structures exactly.
4. Run the test suite (e.g. `pnpm test` or `npm run test`) and Storybook (e.g. `pnpm storybook` or `npm run storybook`) to confirm the entire environment is fully healthy.

---

## 4. Key Conventions & Specifications Reminder

> [!IMPORTANT]
> The scaffolded application and all of its modules, components, hooks, utilities, context providers, state stores, and test suites must strictly adhere to the technical specifications defined in the workspace:
>
> - **[Directory Structure & Dependencies](architecture/react-structure/architecture.md)**
> - **[Component & Folder Conventions](architecture/react-conventions/architecture.md)**
> - **[Configuration Files](architecture/react-configurations/architecture.md)**
> - **[Axios Configuration](architecture/react-axios/architecture.md)**
> - **[API Clients](architecture/react-api-clients/architecture.md)**
> - **[React Redux Use Model](architecture/react-state-management/architecture.md)**
> - **[API Mocking (MSW)](architecture/react-mocks/architecture.md)**
> - **[Routing Specification](architecture/react-routing/architecture.md)**
> - **[Internationalization (i18n)](architecture/react-i18n/architecture.md)**
> - **[Styling Specification](architecture/react-styling/architecture.md)**
> - **[Testing & Storybook Setup](architecture/react-testing/architecture.md)**
> - **[Form Definition & Validation](architecture/react-hook-form/architecture.md)**
