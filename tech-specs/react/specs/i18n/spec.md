# Internationalization (i18n) Architecture & Boilerplate

The i18n setup handles multi-language support by decoupling translation assets (JSON Catalogs) from the UI layer. Translations are loaded dynamically by namespace (e.g., `common`, `glossary`) and context providers synchronize rendering whenever the active language state shifts.

```
Redux Language State
         │ (Triggers language change)
         ▼
┌───────────────────────────┐
│  I18nProvider Wrapper     │  <- translation JSON catalogs
└─────────────┬─────────────┘
              │ (Injects Context via I18nextProvider)
              ▼
┌───────────────────────────┐
│     i18next Engine        │
└─────────────┬─────────────┘
              │ (Translation 't' function)
              ▼
      React UI Components
```

### Key Principles

1. **Namespaced Catalogs**: Translations are divided by functionality (e.g., `common.json` for buttons/alerts, `glossary.json` for domain terms) to prevent single-file bloating.
2. **Reactive Localization**: The app language state is managed globally (via Redux) and connected to the i18n provider so language shifts instantly update the UI.
3. **TypeScript Safety**: Translation resources are structured into namespaces and exported as clean JS namespaces.
4. **Mandatory UI Localization**: All user-facing text strings must be defined in the translation catalogs and translated with the `t` function. Hardcoded text strings in UI components are forbidden.

---

## 1. Directory Layout & Translation Catalogs

Place translation assets under `src/i18n/` structured by language code:

```text
src/i18n/
└── translations/
    ├── en/
    │   ├── common.json
    │   ├── glossary.json
    │   └── index.ts        # Exports English namespaces
    ├── es/
    │   ├── common.json
    │   ├── glossary.json
    │   └── index.ts        # Exports Spanish namespaces
    └── index.ts            # Bundles all languages
```

### English Catalogs (`src/i18n/translations/en/common.json`)

```json
{
  "buttons": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete"
  },
  "messages": {
    "success": "Action completed successfully.",
    "error": "An error occurred. Please try again."
  }
}
```

### Spanish Catalogs (`src/i18n/translations/es/common.json`)

```json
{
  "buttons": {
    "save": "Guardar",
    "cancel": "Cancelar",
    "delete": "Eliminar"
  },
  "messages": {
    "success": "Acción completada con éxito.",
    "error": "Ocurrió un error. Intente nuevamente."
  }
}
```

---

## 2. Namespace Bundlers

Each language folder contains an `index.ts` to export its JSON files, which are then combined at the translations root.

### Language Index Example (`src/i18n/translations/es/index.ts`)

```typescript
export { default as common } from './common.json';
export { default as glossary } from './glossary.json';
```

### Global Namespace Index (`src/i18n/translations/index.ts`)

```typescript
export * as en from './en';
export * as es from './es';
```

---

## 3. Reactive I18nProvider Wrapper

Connect the localization engine to the global Redux state using a custom wrapper component that uses `react-i18next` and `i18next` directly.

### File: `src/components/I18nProvider/index.tsx`

```tsx
import React, { useEffect, useMemo } from 'react';
import { useSelector } from 'react-redux';
import { useAppModel } from '@/models/useAppModel';
import { I18nextProvider } from 'react-i18next';
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

export interface I18nProviderProps {
  children?: React.ReactNode;
  resources: any;
}

export const I18nProvider: React.FC<I18nProviderProps> = ({
  children,
  resources,
}) => {
  const appModel = useAppModel();
  const appState = useSelector(appModel.selectAppState);

  // Initialize the shared i18n instance directly with the translation resources
  const i18nInstance = useMemo(() => {
    i18n.use(initReactI18next).init({
      resources,
      lng: appState.selectedLang || 'en',
      fallbackLng: 'en',
      interpolation: {
        escapeValue: false, // React already escapes values
      },
    });
    return i18n;
  }, []);

  // Sync language selection from Redux store with the i18n instance
  useEffect(() => {
    if (
      appState.selectedLang &&
      i18nInstance.language !== appState.selectedLang
    ) {
      i18nInstance.changeLanguage(appState.selectedLang);
    }
  }, [appState.selectedLang, i18nInstance]);

  return <I18nextProvider i18n={i18nInstance}>{children}</I18nextProvider>;
};
```

---

## 4. Bootstrapping Flow (`main.tsx`)

Import translations, pass them into the custom provider, and hydrate the React DOM.

```tsx
import { createRoot } from 'react-dom/client';
import { App } from './App';
import { Provider } from 'react-redux';
import { I18nProvider } from '@/components/I18nProvider';
import { store } from '@/store';
import * as resources from '@/i18n/translations';

createRoot(document.getElementById('root')!).render(
  <Provider store={store}>
    <I18nProvider resources={resources}>
      <App />
    </I18nProvider>
  </Provider>
);
```

---

## 5. UI Component Usage

Render localized values inside React components using the standard translations hook. All user-facing text must be added in the translation catalogs and written using the `t` function; hardcoded strings in UI components are strictly forbidden.

```tsx
import React from 'react';
import { useTranslation } from 'react-i18next';

export const AlertBox: React.FC = () => {
  const { t } = useTranslation();

  return (
    <div className="alert">
      <p>{t('common:messages.success')}</p>
      <button>{t('common:buttons.save')}</button>
    </div>
  );
};
```

[Go back to Table of Contents](../README.md)
