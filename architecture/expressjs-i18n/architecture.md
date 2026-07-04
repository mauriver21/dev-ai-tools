# Internationalization (i18n)

The backend must use **i18next** for all internationalization concerns. Every user-facing message must be translated through the shared `t` function obtained from the current request context.

---

## General Rules

- Use **i18next** as the internationalization library.
- All user-facing text strings **must** be defined in translation catalogs.
- Hardcoded user-facing strings are forbidden in every backend layer.
- Translation catalogs must be generated in **English (`en`)** and **Spanish (`es`)**.
- Translation files must be organized by functionality (namespace) instead of accumulating every translation into a single file.

Example namespaces:

- `common.json`
- `errors.json`
- `glossary.json`
- `validation.json`
- `emails.json`

This organization keeps translation catalogs maintainable as the application grows.

---

# Translation Structure

Translation assets must live under `src/i18n/`.

```text
src/
└── i18n/
    ├── index.ts                 # Shared i18n instance
    └── translations/
        ├── en/
        │   ├── common.json
        │   ├── errors.json
        │   ├── glossary.json
        │   └── index.ts
        ├── es/
        │   ├── common.json
        │   ├── errors.json
        │   ├── glossary.json
        │   └── index.ts
        └── index.ts
```

---

# Namespace Bundlers

Each language exports every namespace through its own `index.ts`.

## English

```typescript
export { default as common } from "./common.json";
export { default as errors } from "./errors.json";
export { default as glossary } from "./glossary.json";
```

## Spanish

```typescript
export { default as common } from "./common.json";
export { default as errors } from "./errors.json";
export { default as glossary } from "./glossary.json";
```

The translation root bundles every language.

```typescript
export * as en from "./en";
export * as es from "./es";
```

---

# Shared i18n Instance

Create a shared i18next instance under `src/i18n/index.ts`.

```typescript
import i18n from "i18next";
import * as resources from "./translations";

i18n.init({
  resources,
  fallbackLng: "en",
  interpolation: {
    escapeValue: false,
  },
});

export { i18n };
```

The application must create **only one** i18next instance.

---

# Request Context

The current request language and translation function must be stored in the request context using `AsyncLocalStorage`.

## RequestContext

```typescript
import { TFunction } from "i18next";

export type RequestContext = {
  lng: string;
  t: TFunction;
};
```

---

## RequestContextProvider

```typescript
import { RequestContext } from "@/interfaces/RequestContext";

export type RequestContextProvider = {
  run(context: RequestContext, callback: () => void): void;
  get(): RequestContext;
};
```

---

# AsyncLocalStorage Singleton

Create a single `AsyncLocalStorage` instance.

```typescript
import { RequestContext } from "@/interfaces/RequestContext";
import { AsyncLocalStorage } from "async_hooks";

export const asyncLocalStorage = new AsyncLocalStorage<RequestContext>();
```

> [!IMPORTANT]
> `AsyncLocalStorage` must be implemented as a singleton. Although the application owns a single storage manager, Node.js automatically creates an isolated store for every request execution context, making the stored request data safe across concurrent requests.

---

# Request Context Provider

Create the request context provider.

```typescript
import { RequestContext } from "@/interfaces/RequestContext";
import { RequestContextProvider } from "@/interfaces/RequestContextProvider";
import { asyncLocalStorage } from "@/singletons/asyncLocalStorage";

export const createRequestContextProvider = (): RequestContextProvider => {
  const run = (context: RequestContext, callback: () => unknown) => {
    asyncLocalStorage.run(context, callback);
  };

  const get = () => {
    const context = asyncLocalStorage.getStore();

    if (!context) {
      throw new Error("Request context is not available.");
    }

    return context;
  };

  return {
    run,
    get,
  };
};
```

The provider exposes two responsibilities:

- `run()` initializes a request context.
- `get()` retrieves the current request context from any backend layer.

---

# i18n Middleware

Initialize the request context before routing.

```typescript
import { Request, Response, NextFunction } from "express";
import i18next from "i18next";
import { createRequestContextProvider } from "@/providers/createRequestContextProvider";

const requestContextProvider = createRequestContextProvider();

export const i18nMiddleware = (
  req: Request,
  _: Response,
  next: NextFunction,
) => {
  const lng = req.headers["accept-language"] ?? "en";

  requestContextProvider.run(
    {
      lng,
      t: i18next.getFixedT(lng),
    },
    next,
  );
};
```

The middleware captures the request language and stores a language-bound translation function (`t`) in the request context.

---

# Middleware Registration

Register the middleware before the API routes.

```typescript
app.use(i18nMiddleware);
```

Every request executed after this middleware automatically has access to its own translation function.

---

# Translation Utility

To simplify translations throughout the application, define a shared `t` utility that delegates translation to the current request context.

Create `src/utils/t/index.ts`:

```typescript
import { createRequestContextProvider } from "@/providers/createRequestContextProvider";
import i18n from "i18next";

const requestContextProvider = createRequestContextProvider();

export const t = (...params: Parameters<typeof i18n.t>) => {
  return requestContextProvider.get().t(...params);
};
```

This utility retrieves the current request's translation function from the request context, allowing translations without repeatedly obtaining the context provider.

## Usage

```typescript
import { t } from "@/utils/t";

throw new Error(t("errors.entityNotFound"));
```

> [!IMPORTANT]
> The `t` utility must only be invoked during the lifetime of an HTTP request after the `i18nMiddleware` has initialized the request context. Invoking it outside an active request (for example, during application startup) will throw a `"Request context is not available."` error.

---

# Translation Usage Rules

Every user-visible message must be translated.

✅ Preferred

```typescript
const { t } = requestContextProvider.get();

throw new Error(t("errors.entityNotFound"));
```

❌ Avoid

```typescript
throw new Error("Entity not found");
```

---

# Translation Key Convention

Translation keys should follow a namespaced hierarchy.

```text
common.save
common.cancel
common.success

errors.entityNotFound
errors.validationFailed
errors.unauthorized

glossary.user
glossary.organization
glossary.project
```

Keys should remain stable over time and never depend on the translated text itself.

---

# Architectural Guidelines

- Controllers, repositories, services, utilities, and middlewares may retrieve the current translator through the request context provider.
- Models should remain focused on persistence concerns and should avoid producing user-facing messages whenever possible.
- Never pass `lng` or `t` through method parameters.
- Never instantiate additional `AsyncLocalStorage` or `i18next` instances.
- The application must rely on a single shared i18next instance and a single shared request context manager.
