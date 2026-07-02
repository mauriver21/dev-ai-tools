# Routing Architecture (React Router Dom)

This document establishes the guidelines for configuring client-side routing using **React Router Dom**. It details how to set up declarative route objects and bootstrap the router context.

---

## 1. Routing Context Setup (`BrowserRouter`)

To enable routing capabilities across the application, the root application tree must be wrapped in the **`BrowserRouter`** context provider inside the application entrypoint.

### Hydration Entrypoint (`src/main.tsx`)

```tsx
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import { App } from "./App";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
);
```

---

## 2. Declarative Object-Based Routing

Rather than writing inline JSX `<Routes>` and `<Route>` trees inside components, define all application paths dynamically as an array of **`RouteObject`** records.

### 1. Define Routes Array (`src/routes/index.tsx`)

Create your routes folder module and export the array of route objects:

```tsx
import React from "react";
import { RouteObject } from "react-router-dom";
import { MainLayout } from "@/layouts/MainLayout";
import { HomePage } from "@/pages/HomePage";
import { NotFoundPage } from "@/pages/NotFoundPage";

export const routes: RouteObject[] = [
  {
    path: "/",
    element: <MainLayout />,
    children: [
      {
        index: true,
        element: <HomePage />,
      },
      {
        path: "*",
        element: <NotFoundPage />,
      },
    ],
  },
];
```

### 2. Consume Routes Array in App Component (`src/App.tsx`)

Use the **`useRoutes`** hook in `App.tsx` to dynamically mount elements based on the routes configuration:

```tsx
import React from "react";
import { useRoutes } from "react-router-dom";
import { routes } from "@/routes";

export const App: React.FC = () => {
  const element = useRoutes(routes);
  return element;
};
```

### 3. Nested Layout and Child Routes Output (`src/layouts/MainLayout/index.tsx`)

Layout components define structural templates (such as headers, navigation menus, or sidebars) and render nested child routes using the standard `<Outlet />` component:

```tsx
import React from "react";
import { Outlet } from "react-router-dom";
import { Box, Container, AppBar, Toolbar, Typography } from "@mui/material";

export const MainLayout: React.FC = () => {
  return (
    <Box sx={{ display: "flex", flexDirection: "column", minHeight: "100vh" }}>
      <AppBar position="static">
        <Toolbar>
          <Typography variant="h6" component="div">
            My Application
          </Typography>
        </Toolbar>
      </AppBar>
      <Container component="main" sx={{ flexGrow: 1, my: 4 }}>
        <Outlet /> {/* Child routes render here */}
      </Container>
    </Box>
  );
};
```
