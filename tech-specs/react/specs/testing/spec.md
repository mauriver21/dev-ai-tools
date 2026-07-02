# Testing & Storybook Setup

## Testing Strategy
- **Logical Helpers/Hooks/Utilities**: Must have corresponding unit tests (`index.test.ts`).
- **UI Components**: Must have Storybook stories (`index.stories.tsx`). If the component has no state/logic and can be tested visually using Storybook, the `index.test.tsx` file is **optional**. Only add `index.test.tsx` if the component contains logical operations, handlers, or custom interactions requiring assertion validation.

## 1. Vitest & RTL Packages

### Required Packages (pnpm)

```bash
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom @testing-library/user-event
```

### Test Setup Assertion Extension (`src/setupTests.ts`)

Create this file to make all Jest-DOM assertions (like `.toBeInTheDocument()`) globally available in Vitest:

```typescript
import '@testing-library/jest-dom';
```

---

## 2. Integrating MSW with Vitest

For tests that trigger API calls (e.g. Hooks, Repositories, or Components), configure MSW's Node server wrapper to intercept calls during test runs.

### 1. Setup Test Setup File (`src/setupTests.ts`)
Configure Vitest to start the mock server before tests run, reset request handlers between tests (critical for test isolation), and shutdown after all tests finish:

```typescript
import { beforeAll, afterEach, afterAll } from 'vitest';
import { server } from '@/mocks/server';
import '@testing-library/jest-dom';

// Start server before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset handlers after each test (prevents test cross-contamination)
afterEach(() => server.resetHandlers());

// Clean up after all tests are done
afterAll(() => server.close());
```

---

## 3. Minimal Storybook Setup

Storybook renders React components in isolation. All story files (`*.stories.tsx`) SHALL be colocated directly next to the UI component or function they target (e.g., `src/components/Button/index.stories.tsx`).

### Configuration Files

Storybook config files reside in `.storybook/` at the root of the React app workspace.

#### `.storybook/main.ts`

```typescript
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.mdx', '../src/**/*.stories.@(js|jsx|mjs|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

#### `.storybook/preview.tsx`

To support reactive multi-theme selection and language rendering inside Storybook, define custom toolbar selector elements and configure a dynamic Redux store decorator:

```tsx
import React from 'react';
import type { Preview } from '@storybook/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { BrowserRouter } from 'react-router-dom';
import { ThemeProvider } from '../src/components/ThemeProvider';
import { I18nProvider } from '../src/components/I18nProvider';
import { normalizedEntitiesState } from 'react-redux-use-model';
import * as resources from '../src/i18n/translations';

const preview: Preview = {
  parameters: {
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
  },
  globalTypes: {
    theme: {
      name: 'Theme',
      description: 'Global theme for components',
      defaultValue: 'default',
      toolbar: {
        icon: 'paintbrush',
        items: [
          { value: 'default', title: 'Default Blue' },
          { value: 'ocean', title: 'Ocean Teal' },
          { value: 'sunset', title: 'Sunset Orange' },
        ],
        showName: true,
      },
    },
    mode: {
      name: 'Mode',
      description: 'Global color mode for components',
      defaultValue: 'light',
      toolbar: {
        icon: 'circlehollow',
        items: [
          { value: 'light', title: 'Light Mode' },
          { value: 'dark', title: 'Dark Mode' },
        ],
        showName: true,
      },
    },
  },
  decorators: [
    (Story, context) => {
      const activeTheme = context.globals.theme || 'default';
      const activeMode = context.globals.mode || 'light';

      // Re-create the Redux store dynamically reading active toolbar options
      const store = configureStore({
        reducer: {
          normalizedEntitiesState,
          appState: () => ({
            selectedLang: 'en',
            theme: activeTheme,
            mode: activeMode,
          }),
        },
      });

      return (
        <Provider store={store}>
          <I18nProvider resources={resources}>
            <ThemeProvider>
              <BrowserRouter>
                <Story />
              </BrowserRouter>
            </ThemeProvider>
          </I18nProvider>
        </Provider>
      );
    },
  ],
};

export default preview;
```

[Go back to Table of Contents](../README.md)
