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
├── hocs/
│   └── withReactHookForm/
│       └── index.tsx       # HOC to automatically inject React Hook Form controllers
└── interfaces/
    └── LoginFormData.ts    # Type definition for the login form data (LoginFormData)
```

---

## 2. Defining a Form Schema

Validation schemas are defined dynamically via custom hooks using the Yup schema builder. By wrapping schemas inside hooks, we can leverage the translation context (`useTranslation`) to dynamically load localized error messages whenever the app language changes.

### Boilerplate Schema Hook: `src/form-schemas/useLoginSchema/index.ts`

```typescript
import * as yup from "yup";
import { useTranslation } from "react-i18next";

export const useLoginSchema = () => {
  const { t } = useTranslation();

  return yup.object().shape({
    email: yup
      .string()
      .required(t("login.errors.emailRequired"))
      .email(t("login.errors.invalidEmail")),
    password: yup
      .string()
      .required(t("login.errors.passwordRequired"))
      .min(6, t("login.errors.passwordLength")),
  });
};
```

### Form Interface Definition: `src/interfaces/LoginFormData.ts`

```typescript
import * as yup from "yup";
import { useLoginSchema } from "@/form-schemas/useLoginSchema";

export type LoginFormData = yup.InferType<ReturnType<typeof useLoginSchema>>;
```

---

## 3. The `withReactHookForm` Higher-Order Component

To eliminate verbose and repetitive `<Controller>` wrapper boilerplate in forms, we use the `withReactHookForm` higher-order component (HOC). It wraps standard UI input components and dynamically registers them with React Hook Form if a `name` is provided. If no `name` is passed, the component falls back to functioning as a standard React component.

### HOC Implementation: `src/hocs/withReactHookForm/index.tsx`

```typescript
import {
  useController,
  UseControllerProps,
  FieldValues,
  ControllerRenderProps,
  ControllerFieldState,
  UseFormStateReturn,
} from 'react-hook-form';

export interface WithReactHookFormProps extends Partial<
  UseControllerProps<any>
> {
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
} from "@mui/material";
import {
  withReactHookForm,
  WithReactHookFormProps,
} from "@/hocs/withReactHookForm";

export type TextFieldProps = MuiTextFieldProps & {
  errorMessage?: string;
  hideErrorMessage?: boolean;
};

export const TextField = withReactHookForm(
  ({
    field,
    fieldState,
    control,
    error: errorProp,
    errorMessage: errorMessageProp,
    hideErrorMessage,
    required,
    slotProps,
    ...rest
  }: TextFieldProps & WithReactHookFormProps) => {
    const error = fieldState?.invalid || errorProp;
    const errorMessage = fieldState?.error?.message || errorMessageProp;

    return (
      <MuiTextField
        {...field}
        slotProps={{
          inputLabel: { required, ...slotProps?.inputLabel },
          ...slotProps,
        }}
        fullWidth
        error={fieldState?.invalid || error}
        helperText={hideErrorMessage ? undefined : errorMessage}
        {...rest}
      />
    );
  },
);
```

---

## 5. Implementing Forms with Wrapped Components

Using components wrapped with `withReactHookForm` yields clean, readable form rendering without nested wrapper elements.

### React Component Implementation Boilerplate

```tsx
import React from "react";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import { useLoginSchema } from "@/form-schemas/useLoginSchema";
import { TextField } from "@/components/TextField";
import { LoginFormData } from "@/interfaces/LoginFormData";

export const LoginForm: React.FC = () => {
  const loginSchema = useLoginSchema();

  const {
    control,
    handleSubmit,
    formState: { isSubmitting },
  } = useForm<LoginFormData>({
    resolver: yupResolver(loginSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      {/* Field automatically registers with React Hook Form using name and control */}
      <TextField name="email" control={control} label="Email Address" />
      <TextField
        name="password"
        control={control}
        label="Password"
        type="password"
      />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Submitting..." : "Sign In"}
      </button>
    </form>
  );
};
```

---

## 6. Testing Form Field Components (Storybook)

To verify the integration between custom form fields (wrapped in `withReactHookForm`) and React Hook Form validation logic, write interaction tests using Storybook's `play` function. Use `@storybook/test` utilities (`within`, `userEvent`, `expect`) to simulate user interactions and assert validation outcomes.

### Storybook Test Suite: `src/components/TextField/index.stories.tsx`

```tsx
import { TextField } from "@/components/TextField";
import { Button, Stack } from "@mui/material";
import { Meta, StoryObj } from "@storybook/react";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";
import { within, userEvent, expect } from "@storybook/test";

const meta: Meta<typeof TextField> = {
  title: "Components/TextField",
  component: TextField,
};

type Story = StoryObj<typeof TextField>;

export default meta;

const schema = yup.object({
  default: yup.string().required("This field is required"),
  helper: yup.string().required("This field is required"),
  email: yup
    .string()
    .email("Please enter a valid email")
    .required("Email is required"),
  required: yup.string().required("This field is required"),
  password: yup.string().required("Password is required"),
});

const fillValues: yup.InferType<typeof schema> = {
  default: "John",
  helper: "Some helper text",
  email: "john@example.com",
  required: "Required value",
  password: "Password123",
};

const defaultValues = {
  default: "",
  helper: "",
  email: "",
  required: "",
  password: "",
};

const render = () => {
  const { control, handleSubmit, trigger, reset } = useForm<
    yup.InferType<typeof schema>
  >({
    resolver: yupResolver(schema),
    mode: "all",
    defaultValues,
  });

  return (
    <Stack
      component="form"
      noValidate
      spacing={3}
      onSubmit={handleSubmit(console.log)}
    >
      <TextField
        name="default"
        control={control}
        label="Default"
        placeholder="Type something..."
      />
      <TextField
        name="helper"
        control={control}
        label="With Helper Text"
        helperText="This helper text provides additional information."
      />
      <TextField name="email" control={control} label="Email" />
      <TextField name="required" control={control} label="Required" required />
      <TextField
        name="password"
        control={control}
        label="Password"
        type="password"
      />
      <Stack direction="row" spacing={2}>
        <Button variant="outlined" type="submit">
          Submit
        </Button>
        <Button variant="outlined" onClick={() => reset(fillValues)}>
          Fill
        </Button>
        <Button variant="outlined" onClick={() => trigger()}>
          Validate
        </Button>
        <Button variant="outlined" onClick={() => reset(defaultValues)}>
          Reset
        </Button>
      </Stack>
    </Stack>
  );
};

export const Overview: Story = {
  render,
};

export const ValidationOnBlur: Story = {
  render,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const field = canvas.getByLabelText("Default");

    await userEvent.click(field);
    await userEvent.tab();

    await expect(
      canvas.getByText("This field is required"),
    ).toBeInTheDocument();
  },
};

export const ValidationOnChange: Story = {
  render,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const email = canvas.getByLabelText("Email");

    await userEvent.type(email, "invalid");
    await userEvent.tab();

    await expect(
      canvas.getByText("Please enter a valid email"),
    ).toBeInTheDocument();

    await userEvent.clear(email);
    await userEvent.type(email, "john@example.com");

    await expect(
      canvas.queryByText("Please enter a valid email"),
    ).not.toBeInTheDocument();
  },
};

export const ValidateButton: Story = {
  render,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    await userEvent.click(canvas.getByRole("button", { name: /validate/i }));

    await expect(canvas.getByText("Password is required")).toBeInTheDocument();

    await expect(canvas.getByText("Email is required")).toBeInTheDocument();
  },
};

export const SubmitButton: Story = {
  render,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    await userEvent.click(canvas.getByRole("button", { name: /submit/i }));

    await expect(canvas.getByText("Password is required")).toBeInTheDocument();

    await expect(canvas.getByText("Email is required")).toBeInTheDocument();
  },
};

export const ResetButton: Story = {
  render,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const email = canvas.getByLabelText("Email");

    await userEvent.type(email, "john@example.com");

    await expect(email).toHaveValue("john@example.com");

    await userEvent.click(canvas.getByRole("button", { name: /reset/i }));

    await expect(email).toHaveValue("");
  },
};

export const FillButton: Story = {
  render,

  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    await userEvent.click(canvas.getByRole("button", { name: /fill/i }));

    await expect(canvas.getByLabelText("Default")).toHaveValue("John");

    await expect(canvas.getByLabelText("With Helper Text")).toHaveValue(
      "Some helper text",
    );

    await expect(canvas.getByLabelText("Email")).toHaveValue(
      "john@example.com",
    );

    await expect(canvas.getByLabelText(/Required/i)).toHaveValue(
      "Required value",
    );

    await expect(canvas.getByLabelText("Password")).toHaveValue("Password123");
  },
};
```

---

## 7. Key Rules

1. **Dynamic Localization**: Always wrap Yup schema definitions in a React Hook using `useTranslation()` so error labels reactively update on language swaps.
2. **Standard HOC Usage**: Prioritize wrapping UI components with `withReactHookForm` to keep the layout code free of boilerplate.
3. **Explicit Defaults**: Always provide complete `defaultValues` inside `useForm` configuration to prevent fields from registering as uncontrolled components.
4. **Fallback to Controller**: Use the raw `<Controller>` component directly only when dealing with complex, one-off, or third-party wrappers that cannot easily be standardized via `withReactHookForm`.
5. **Default Width**: Guarantee `fullWidth` as default on every form field component (e.g., hardcoded inside the wrapped base component's implementation). The parent layout container is responsible for defining the actual size and spacing of the form field.
6. **Disable Browser Validation**: To prevent browser-native 'required' validation popups from overriding custom schema-based validations, destruct `required` and only pass it to `slotProps.inputLabel` (instead of passing it directly to the HTML input). This retains the visual asterisk (\*) indicator without triggering native browser blocking.

[Go back to Table of Contents](../README.md)
