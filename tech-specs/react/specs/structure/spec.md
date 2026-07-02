# Directory Structure & Dependencies

## 1. Directory Structure

A standardized "Folder-as-Module" layout for the frontend application where every logical unit is self-contained:

```text
src/
├── api-clients/       # Axios API clients for fetching remote backend endpoints
├── assets/            # Static assets like images, fonts, and global CSS
├── components/        # Reusable presentation and layout UI components (PascalCase)
├── constants/         # Application-level constants (e.g. routes, keys)
├── contexts/          # Context providers (PascalCase)
├── hocs/              # High-order components (camelCase starting with with)
├── hooks/             # Custom React hooks (camelCase starting with use)
├── i18n/              # Localization files and translation setup
├── interfaces/        # Shared global TypeScript types and interfaces (one per file)
├── layouts/           # Page structural wrappers (e.g. Auth, Dashboard layouts)
├── mocks/             # Mock Service Worker (MSW) setup for API mocking in development
├── models/            # React-redux-use-model state bindings and business entities
├── pages/             # Route-level page components (views)
├── routes/            # Application route definitions and routing system
├── states/            # Redux store slices (state definitions, actions, reducers)
├── utils/             # Reusable utility functions and helper methods (camelCase)
├── main.tsx           # Application entrypoint (bootstrap and React hydration)
├── setupTests.ts      # Vitest test framework setup and global assertions
├── store.ts           # Redux store setup with optional persistence (redux-persist)
└── vite-env.d.ts      # TypeScript environment variables declaration
```

## 2. Core Dependencies & Packages

| Category                    | Recommended Packages                                                               | Description                                           |
| :-------------------------- | :--------------------------------------------------------------------------------- | :---------------------------------------------------- |
| **Core Framework**          | `react`, `react-dom`                                                               | Standard React web engine                             |
| **Bundling & Transpiling**  | `vite`, `@vitejs/plugin-react`                                                     | Ultra-fast local dev and bundle tooling               |
| **Styling & UI Components** | `@mui/material`, `@emotion/react`, `@emotion/styled`                               | Material-UI v5 components and styling engines         |
| **State Management**        | `@reduxjs/toolkit`, `react-redux`, `redux-persist`, `react-redux-use-model`        | Redux store with schema persistence and Model binding |
| **Routing**                 | `react-router-dom`                                                                 | Browser-based client-side routing                     |
| **Localization**            | `i18next`, `react-i18next`                                                         | Multi-language translation setup                      |
| **Networking & Comms**      | `axios`, `socket.io-client`                                                        | HTTP client and real-time socket events               |
| **Mocking & Development**   | `msw`, `storybook`                                                                 | Local mock APIs for isolated UI building              |
| **Quality & Checking**      | `vitest`, `@testing-library/react`, `@testing-library/jest-dom`, `jsdom`, `eslint` | Testing runner, DOM assertions, and linter            |

---

## 3. General Types and Interfaces Conventions

To ensure clean and modular code separation, follow these rules:

1. **Shared/Global Types**: All data models, API responses, or other shared TypeScript types and interfaces MUST reside inside the `src/interfaces/` directory.
2. **Single-File Principle**: Each file in the `src/interfaces/` directory MUST contain only a single type or interface declaration (e.g. `src/interfaces/User.ts`, `src/interfaces/Item.ts`). Do not combine multiple unrelated types into a single utility types file.
3. **Component-Specific Types**: Props and configurations specific to a single component (e.g. `ButtonProps`, `CardProps`) MUST remain inside that component's file. Do not move component-specific props to the `src/interfaces/` folder.
