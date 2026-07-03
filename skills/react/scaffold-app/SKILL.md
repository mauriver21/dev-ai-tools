# Skill: Scaffold React Application

This skill automates the scaffolding and bootstrapping of a complete, premium React TypeScript application. It orchestrates project initialization and configures the codebase to conform strictly to the specifications defined in the repository.

---

## 1. Goal
Initialize a standard, production-ready React web application in a target directory with all necessary configurations and architectural layers.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **projectName**: The directory name to initialize the app in, or `./` for the current workspace.
* **packageManager**: The package manager to use. The agent **MUST** explicitly ask the user to choose between **pnpm** (prioritized & recommended) or **npm** before installing dependencies.

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
* **Production Packages**: `@mui/material`, `@emotion/react`, `@emotion/styled`, `@reduxjs/toolkit`, `react-redux`, `redux-persist`, `react-redux-use-model`, `react-router-dom`, `axios`, `i18next`, `react-i18next`, `uuid`.
* **Dev Packages**: `vitest`, `@testing-library/react`, `@testing-library/jest-dom`, `@testing-library/user-event`, `jsdom`, `msw`, `vite-tsconfig-paths`, `vite-plugin-environment`, `vite-plugin-checker`, `dotenv`, `eslint`, `prettier`, `@storybook/react-vite`.

Refer to the recommended package catalog in:
* [structure/spec.md#2-core-dependencies--packages](../../../tech-specs/react/specs/structure/spec.md#2-core-dependencies--packages)

### Step 3: Setup Directory Layout
Scaffold the standard "Folder-as-Module" directory structure under `src/` (creating directories like `api-clients`, `components`, `states`, `utils`, `interfaces`, `mocks`, `i18n`, etc.).
Refer to the layout map in:
* [structure/spec.md#1-directory-structure](../../../tech-specs/react/specs/structure/spec.md#1-directory-structure)

### Step 4: Configure Base Project Files
Configure configurations and versions:
1. **tsconfig.app.json**: Setup TS path alias configurations pointing to `@/*` matching [configurations/spec.md#1-typescript-paths-configuration-tsconfigappjson](../../../tech-specs/react/specs/configurations/spec.md#1-typescript-paths-configuration-tsconfigappjson).
2. **vite.config.ts**: Setup Vite bundler plugins, Vitest configurations, environment loadings, and type checkers matching [configurations/spec.md#2-vite--vitest-configuration-viteconfigts](../../../tech-specs/react/specs/configurations/spec.md#2-vite--vitest-configuration-viteconfigts).
3. **.prettierrc**: Setup code formatter settings matching [configurations/spec.md#4-code-formatting-config-prettierrc](../../../tech-specs/react/specs/configurations/spec.md#4-code-formatting-config-prettierrc).
4. **.gitignore**: Setup exclusions matching [configurations/spec.md#5-version-control-exclusions-gitignore](../../../tech-specs/react/specs/configurations/spec.md#5-version-control-exclusions-gitignore).

### Step 5: Clean Default Assets
Clean up Vite's default templates, empty the contents of `src/index.css`, and replace `src/App.tsx` with a refined Hello World layout that implements the theme selector and mode toggle switch layout illustrated in the wireframe:
* **Wireframe Layout Source**: [hello-world-layout.png](./resources/images/hello-world-layout.png)
* **Code Specification**: [configurations/spec.md#3-react-app-cleanup](../../../tech-specs/react/specs/configurations/spec.md#3-react-app-cleanup)

Ensure that:
1. The top header/app bar contains a Dropdown menu (`Select`) to select the active theme (e.g. `'default'`, `'ocean'`, `'sunset'`) and a Switch toggle to alternate between light and dark mode.
2. Changes to the theme select and mode toggle are dispatched to Redux state using the standard `useAppModel()` hook to update the global theme.
3. The main page area renders a centered Card element displaying "Hello World!" in a prominent Typography block.

### Step 6: Setup Internationalization (i18n)
Configure namespaces, catalogs, dynamic translations indexes, and the custom reactive translations wrapper component.
Do not duplicate code; copy catalog structures and the reactive provider component implementation directly from:
* [i18n/spec.md](../../../tech-specs/react/specs/i18n/spec.md)

### Step 7: Setup API Mocking (MSW)
1. Initialize the service worker in the public directory:
   ```bash
   npx msw init public --save
   ```
2. Setup the browser worker, Node mock server, and mock data seeds by matching instructions and handlers boilerplates in:
   * [mocks/spec.md](../../../tech-specs/react/specs/mocks/spec.md)
3. Set up the test suite setup helper `src/setupTests.ts` to intercept HTTP requests using MSW during test runs matching:
   * [testing/spec.md#2-integrating-msw-with-vitest](../../../tech-specs/react/specs/testing/spec.md#2-integrating-msw-with-vitest)

### Step 8: Setup Storybook
1. Initialize Storybook:
   ```bash
   npx storybook@latest init --yes
   ```
2. Configure Redux, router, and i18n decorators inside `.storybook/preview.tsx` by implementing the configuration template in:
   * [testing/spec.md#3-minimal-storybook-setup](../../../tech-specs/react/specs/testing/spec.md#3-minimal-storybook-setup)

### Step 9: Configure Store and Hydration Entrypoint
1. Set up the global Redux store configuration `src/store.ts` by combining state reducers matching [state-management/spec.md#configure-the-redux-store-storets](../../../tech-specs/react/specs/state-management/spec.md#configure-the-redux-store-storets).
2. Wire up the bootstrap loader in `src/main.tsx` to conditionally start the MSW browser worker and render the React root wrapped in Redux and i18n providers matching [state-management/spec.md#wrap-the-application-maintsx](../../../tech-specs/react/specs/state-management/spec.md#wrap-the-application-maintsx) and [mocks/spec.md#5-worker--server-configuration](../../../tech-specs/react/specs/mocks/spec.md#5-worker--server-configuration).

### Step 10: Verify the Application is Running

Verify that the scaffolded application compiles and runs successfully:
1. Run the local development server (e.g., `pnpm dev` or `npm run dev`).
2. Verify that there are no terminal compilation errors or runtime exceptions in the browser.
3. If the application fails to run or compile, investigate the logs and apply the necessary fixes. Typical solutions include:
   * Resolving alias path resolutions in `tsconfig.app.json` by adding the `./` prefix (e.g. `"@/routes/*": ["./src/routes/*"]` instead of `"src/routes/*"`).
   * Checking that all state reducers are imported and combined correctly.
   * Checking that relative paths in imports match target structures exactly.
4. Run the test suite (e.g. `pnpm test` or `npm run test`) and Storybook (e.g. `pnpm storybook` or `npm run storybook`) to confirm the entire environment is fully healthy.

---

## 4. Key Conventions & Specifications Reminder
> [!IMPORTANT]
> The scaffolded application and all of its modules, components, hooks, utilities, context providers, state stores, and test suites must strictly adhere to the technical specifications defined in the workspace:
>
> * **[Directory Structure & Dependencies](../../../tech-specs/react/specs/structure/spec.md)**
> * **[Component & Folder Conventions](../../../tech-specs/react/specs/conventions/spec.md)**
> * **[Configuration Files](../../../tech-specs/react/specs/configurations/spec.md)**
> * **[Axios Configuration](../../../tech-specs/react/specs/axios/spec.md)**
> * **[API Clients](../../../tech-specs/react/specs/api-clients/spec.md)**
> * **[React Redux Use Model](../../../tech-specs/react/specs/state-management/spec.md)**
> * **[API Mocking (MSW)](../../../tech-specs/react/specs/mocks/spec.md)**
> * **[Routing Specification](../../../tech-specs/react/specs/routing/spec.md)**
> * **[Internationalization (i18n)](../../../tech-specs/react/specs/i18n/spec.md)**
> * **[Styling Specification](../../../tech-specs/react/specs/styling/spec.md)**
> * **[Testing & Storybook Setup](../../../tech-specs/react/specs/testing/spec.md)**
> * **[Form Definition & Validation](../../../tech-specs/react/specs/react-hook-form/spec.md)**


