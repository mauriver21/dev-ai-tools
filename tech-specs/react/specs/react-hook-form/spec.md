# Form Definition & Validation (react-hook-form & Yup)

Standardize form state handling, error reporting, and schema-based validations in the application by pairing **React Hook Form** with **Yup** schemas.

---

## 1. Directory Structure

All form schemas must be defined in the `src/form-schemas/` folder. This directory hosts validation schemas grouped by form functionality, using custom React hooks to enable live localization during validation.

```text
src/
└── form-schemas/
    ├── useLoginSchema/
    │   └── index.ts        # Login schema definition exporting the useLoginSchema hook
    └── useProfileSchema/
        └── index.ts        # Profile schema definition exporting the useProfileSchema hook
```

---

## 2. Defining a Form Schema

Validation schemas are defined dynamically via custom hooks using the Yup schema builder. By wrapping schemas inside hooks, we can leverage the translation context (`useTranslation`) to dynamically load localized error messages whenever the app language changes.

### Boilerplate Schema Hook: `src/form-schemas/useLoginSchema/index.ts`

```typescript
import * as yup from 'yup';
import { useTranslation } from 'react-i18next';

export const useLoginSchema = () => {
  const { t } = useTranslation();

  return yup.object().shape({
    email: yup
      .string()
      .required(t('login.errors.emailRequired'))
      .email(t('login.errors.invalidEmail')),
    password: yup
      .string()
      .required(t('login.errors.passwordRequired'))
      .min(6, t('login.errors.passwordLength')),
  });
};
```

---

## 3. Implementing Schemas with Material-UI (MUI) Form Components

Integrate React Hook Form with Yup schema resolvers and MUI fields via the React Hook Form `<Controller>` wrapper to handle input binding, error states, and submission lifecycle.

### React Component Implementation Boilerplate

```tsx
import React from 'react';
import { useForm, Controller } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import { TextField, Button, Box, Typography } from '@mui/material';
import { useLoginSchema } from '@/form-schemas/useLoginSchema';

export const LoginForm: React.FC = () => {
  const loginSchema = useLoginSchema();
  
  const {
    control,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    resolver: yupResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  });

  const onSubmit = async (data: any) => {
    try {
      // Execute your API login request here
      console.log('Form Submitted Data:', data);
    } catch (error) {
      console.error('Submission failed', error);
    }
  };

  return (
    <Box 
      component="form" 
      onSubmit={handleSubmit(onSubmit)} 
      noValidate 
      sx={{ mt: 1, width: '100%' }}
    >
      <Typography variant="h5" sx={{ mb: 2 }}>
        Login
      </Typography>

      <Controller
        name="email"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            margin="normal"
            required
            fullWidth
            id="email"
            label="Email Address"
            autoComplete="email"
            autoFocus
            error={!!errors.email}
            helperText={errors.email?.message}
          />
        )}
      />

      <Controller
        name="password"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            margin="normal"
            required
            fullWidth
            name="password"
            label="Password"
            type="password"
            id="password"
            autoComplete="current-password"
            error={!!errors.password}
            helperText={errors.password?.message}
          />
        )}
      />

      <Button
        type="submit"
        fullWidth
        variant="contained"
        sx={{ mt: 3, mb: 2 }}
        disabled={isSubmitting}
      >
        {isSubmitting ? 'Submitting...' : 'Sign In'}
      </Button>
    </Box>
  );
};
```

---

## 4. Key Rules
1. **Dynamic Localization**: Always wrap Yup schema definitions in a React Hook using `useTranslation()` so error labels reactively update on language swaps.
2. **Controller Wrappers**: Always use React Hook Form's `<Controller>` wrapper when integrating with Material UI components to avoid direct state synchronization errors.
3. **Explicit Defaults**: Always provide complete `defaultValues` inside `useForm` configuration to prevent fields from registering as uncontrolled components.

[Go back to Table of Contents](../README.md)
