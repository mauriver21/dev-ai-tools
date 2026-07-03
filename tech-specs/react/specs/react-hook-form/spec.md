# Form Definition & Validation (react-hook-form & Yup)

Standardize form state handling, error reporting, and schema-based validations in the application by pairing **React Hook Form** with **Yup** schemas and a utility Higher-Order Component (HOC) for clean integration.

---

## 1. Directory Structure

All form schemas must be defined in the `src/form-schemas/` folder. This directory hosts validation schemas grouped by form functionality, using custom React hooks to enable live localization during validation. Custom inputs wrapped to integrate with React Hook Form should be placed in `src/components/`, and the standard HOC is defined in `src/hocs/`.

```text
src/
├── components/
│   └── TextField/
│       └── index.tsx       # Reusable input wrapped with withReactHookForm
├── form-schemas/
│   ├── useLoginSchema/
│   │   └── index.ts        # Login schema definition exporting the useLoginSchema hook
│   └── useProfileSchema/
│       └── index.ts        # Profile schema definition exporting the useProfileSchema hook
└── hocs/
    └── withReactHookForm/
        └── index.tsx       # HOC to automatically inject React Hook Form controllers
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

## 3. The `withReactHookForm` Higher-Order Component

To eliminate verbose and repetitive `<Controller>` wrapper boilerplate in forms, we use the `withReactHookForm` higher-order component (HOC). It wraps standard UI input components and dynamically registers them with React Hook Form if a `name` is provided. If no `name` is passed, the component falls back to functioning as a standard React component.

### HOC Implementation: `src/hocs/withReactHookForm/index.tsx`

```typescript
import React from 'react';
import {
  useController,
  UseControllerProps,
  FieldValues,
  ControllerRenderProps,
  ControllerFieldState,
  UseFormStateReturn,
} from 'react-hook-form';

export interface WithReactHookFormProps {
  field?: ControllerRenderProps<any>;
  fieldState?: ControllerFieldState;
  formState?: UseFormStateReturn<any>;
}

export const withReactHookForm = <TProps extends object>(
  Component: React.ComponentType<TProps & WithReactHookFormProps>
) => {
  type PublicProps = Omit<TProps, keyof WithReactHookFormProps>;

  return <TFieldValues extends FieldValues>(
    props: PublicProps &
      Omit<UseControllerProps<TFieldValues>, 'name'> & {
        name?: UseControllerProps<TFieldValues>['name'];
      }
  ) => {
    const controller =
      props.name !== undefined
        ? useController({ ...props, name: props.name })
        : undefined;

    return <Component {...(props as TProps)} {...controller} />;
  };
};
```

---

## 4. Wrapping Base Components

Every common form input component (e.g., `TextField`, `Select`, `Checkbox`, `DatePicker`) should be wrapped in `withReactHookForm`. The wrapped component handles extracting errors and validation messages from `fieldState` automatically.

### Example Wrapped Component: `src/components/TextField/index.tsx`

```tsx
import {
  TextField as MuiTextField,
  TextFieldProps as MuiTextFieldProps,
} from '@mui/material';
import {
  withReactHookForm,
  WithReactHookFormProps,
} from '@/hocs/withReactHookForm';

export type TextFieldProps = MuiTextFieldProps & {
  errorMessage?: string;
  hideErrorMessage?: boolean;
};

export const TextField = withReactHookForm(
  ({
    field,
    fieldState,
    error: errorProp,
    errorMessage: errorMessageProp,
    hideErrorMessage,
    ...rest
  }: TextFieldProps & WithReactHookFormProps) => {
    const error = fieldState?.invalid || errorProp;
    const errorMessage = fieldState?.error?.message || errorMessageProp;

    return (
      <MuiTextField
        {...field}
        fullWidth
        error={fieldState?.invalid || error}
        helperText={hideErrorMessage ? undefined : errorMessage}
        {...rest}
      />
    );
  }
);
```

---

## 5. Implementing Forms with Wrapped Components

Using components wrapped with `withReactHookForm` yields clean, readable form rendering without nested wrapper elements.

### React Component Implementation Boilerplate

```tsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import { useLoginSchema } from '@/form-schemas/useLoginSchema';
import { TextField } from '@/components/TextField';

export const LoginForm: React.FC = () => {
  const loginSchema = useLoginSchema();
  
  const {
    control,
    handleSubmit,
    formState: { isSubmitting },
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
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Field automatically registers with React Hook Form using name and control */}
      <TextField
        name="email"
        control={control}
        label="Email Address"
      />

      <TextField
        name="password"
        control={control}
        label="Password"
        type="password"
      />

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Sign In'}
      </button>
    </form>
  );
};
```

---

## 6. Key Rules

1. **Dynamic Localization**: Always wrap Yup schema definitions in a React Hook using `useTranslation()` so error labels reactively update on language swaps.
2. **Standard HOC Usage**: Prioritize wrapping UI components with `withReactHookForm` to keep the layout code free of boilerplate.
3. **Explicit Defaults**: Always provide complete `defaultValues` inside `useForm` configuration to prevent fields from registering as uncontrolled components.
4. **Fallback to Controller**: Use the raw `<Controller>` component directly only when dealing with complex, one-off, or third-party wrappers that cannot easily be standardized via `withReactHookForm`.
5. **Default Width**: Guarantee `fullWidth` as default on every form field component (e.g., hardcoded inside the wrapped base component's implementation). The parent layout container is responsible for defining the actual size and spacing of the form field.

[Go back to Table of Contents](../README.md)

