# Theme & Styling Conventions

This document establishes the styling standards for the React application. It restricts raw color usage and details how to implement multi-theme configurations (with light and dark modes) using **Material UI (MUI)**.

---

## 1. Color Palettes & Token Constraints

To ensure design consistency, component flexibility, and perfect light/dark mode rendering, follow these rules:

1. **NO Direct Hex Colors in Components**: Raw hex codes (e.g. `#3f51b5`, `#ffffff`), RGB/HSL values, or raw HTML color names must **NEVER** be hardcoded directly inside JSX elements, styled-components, or inline styles.
2. **Access Theme Tokens Only**: Always reference colors through the active MUI `theme` object.
   - **Incorrect**: `<Box sx={{ backgroundColor: '#2196f3' }} />`
   - **Correct**: `<Box sx={{ backgroundColor: (theme) => theme.palette.primary.main }} />`
3. **Use Semantic Colors**: Choose tokens based on their meaning (e.g. `text.secondary`, `background.default`, `error.main`) rather than their visual appearance.

---

## 2. Multi-Theme Directory Layout

Organize your design tokens and themes inside the `src/assets/` directory:

```text
src/assets/
└── themes/
    ├── default/
    │   ├── light.ts        # Default light palette tokens
    │   ├── dark.ts         # Default dark palette tokens
    │   └── index.ts        # Compiles default theme
    ├── ocean/
    │   ├── light.ts        # Ocean theme light palette tokens
    │   ├── dark.ts         # Ocean theme dark palette tokens
    │   └── index.ts        # Compiles ocean theme
    ├── sunset/
    │   ├── light.ts        # Sunset theme light palette tokens
    │   ├── dark.ts         # Sunset theme dark palette tokens
    │   └── index.ts        # Compiles sunset theme
    ├── win11/
    │   ├── light.ts        # Windows 11 light palette tokens
    │   ├── dark.ts         # Windows 11 dark palette tokens
    │   └── index.ts        # Compiles Windows 11 theme
    ├── tahoe/
    │   ├── light.ts        # macOS Tahoe light palette tokens
    │   ├── dark.ts         # macOS Tahoe dark palette tokens
    │   └── index.ts        # Compiles macOS Tahoe theme
    └── index.ts            # Aggregates and exports all themes
```

---

## 3. Theme Definition Boilerplate

Define separate light and dark configurations for each theme, then compile them using MUI's `createTheme`.

### 1. Default Theme Palettes (`src/assets/themes/default/light.ts`)
```typescript
import { PaletteOptions } from '@mui/material';

export const lightPalette: PaletteOptions = {
  mode: 'light',
  primary: {
    main: '#1976d2',
    light: '#42a5f5',
    dark: '#1565c0',
    contrastText: '#ffffff',
  },
  secondary: {
    main: '#9c27b0',
    light: '#ba68c8',
    dark: '#7b1fa2',
    contrastText: '#ffffff',
  },
  background: {
    default: '#f5f5f5',
    paper: '#ffffff',
  },
  text: {
    primary: 'rgba(0, 0, 0, 0.87)',
    secondary: 'rgba(0, 0, 0, 0.6)',
  },
};
```

### 2. Default Theme Compiler (`src/assets/themes/default/index.ts`)
```typescript
import { createTheme } from '@mui/material/styles';
import { lightPalette } from './light';
import { darkPalette } from './dark'; // Assume matching dark configuration

export const defaultLightTheme = createTheme({
  palette: lightPalette,
  shape: {
    borderRadius: 8,
  },
});

export const defaultDarkTheme = createTheme({
  palette: darkPalette,
  shape: {
    borderRadius: 8,
  },
});
```

### 3. Global Themes Aggregator (`src/assets/themes/index.ts`)
```typescript
import { defaultLightTheme, defaultDarkTheme } from './default';
import { oceanLightTheme, oceanDarkTheme } from './ocean';
import { sunsetLightTheme, sunsetDarkTheme } from './sunset';
import { win11LightTheme, win11DarkTheme } from './win11';
import { tahoeLightTheme, tahoeDarkTheme } from './tahoe';
import { Theme } from '@mui/material/styles';

export type ThemeType = 'default' | 'ocean' | 'sunset' | 'win11' | 'tahoe';
export type ThemeMode = 'light' | 'dark';

export const themes: Record<ThemeType, Record<ThemeMode, Theme>> = {
  default: {
    light: defaultLightTheme,
    dark: defaultDarkTheme,
  },
  ocean: {
    light: oceanLightTheme,
    dark: oceanDarkTheme,
  },
  sunset: {
    light: sunsetLightTheme,
    dark: sunsetDarkTheme,
  },
  win11: {
    light: win11LightTheme,
    dark: win11DarkTheme,
  },
  tahoe: {
    light: tahoeLightTheme,
    dark: tahoeDarkTheme,
  },
};
```

---

## 4. Custom ThemeProvider Component & Redux Integration

To decouple theme selection logic from the application entrypoint, create a custom wrapping `ThemeProvider` component that handles Redux state selections and injects the compiled MUI themes.

### File: `src/components/ThemeProvider/index.tsx`
```tsx
import React from 'react';
import { useSelector } from 'react-redux';
import { ThemeProvider as MuiThemeProvider, CssBaseline } from '@mui/material';
import { useAppModel } from '@/models/useAppModel';
import { themes } from '@/assets/themes';

export interface ThemeProviderProps {
  children?: React.ReactNode;
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const appModel = useAppModel();
  const appState = useSelector(appModel.selectAppState);

  // Retrieve selected theme name and mode from Redux state
  const activeThemeName = appState.theme || 'default';
  const activeMode = appState.mode || 'light';
  
  // Resolve compiled theme configuration
  const currentTheme = themes[activeThemeName][activeMode];

  return (
    <MuiThemeProvider theme={currentTheme}>
      <CssBaseline /> {/* Resets body styles and background based on theme */}
      {children}
    </MuiThemeProvider>
  );
};
```

### App Component Integration (`src/App.tsx`)
Render the custom `ThemeProvider` around your application routes:

```tsx
import React from 'react';
import { ThemeProvider } from '@/components/ThemeProvider';
import { useRoutes } from 'react-router-dom';
import { routes } from '@/routes';

export const App: React.FC = () => {
  const element = useRoutes(routes);

  return (
    <ThemeProvider>
      {element}
    </ThemeProvider>
  );
};
```

---

## 5. Advanced Theme Customization (MUI Component Overrides)

To create high-fidelity themes that override default MUI component aesthetics (like border radius, shadows, typography capitalization, alerts, etc.), use the `components` override section within the MUI `createTheme` function.

```typescript
import { createTheme } from '@mui/material/styles';

export const customTheme = createTheme({
  palette: customPalette,
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none', // Disable forced uppercase
          borderRadius: 4,      // Rounded Fluent-style corners
        },
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          borderRadius: 8,
          boxShadow: '0 4px 12px rgba(0, 0, 0, 0.05)',
        },
      },
    },
  },
});
```
This keeps visual overrides cleanly encapsulated inside theme configuration files instead of cluttering component JSX with custom inline rules.

For macOS Tahoe styling, configure the overrides to support custom rounded corners (12px), smooth active scaling/opacity transitions, and translucent glassmorphism backdrop filters:

```typescript
export const tahoeTheme = createTheme({
  palette: tahoePalette,
  components: {
    MuiCard: {
      styleOverrides: {
        root: {
          borderRadius: 12,
          backdropFilter: 'blur(20px)',
          WebkitBackdropFilter: 'blur(20px)',
          backgroundColor: 'rgba(255, 255, 255, 0.65)',
          border: '1px solid rgba(0, 0, 0, 0.08)',
        },
      },
    },
  },
});
```

[Go back to Table of Contents](./spec.md)
