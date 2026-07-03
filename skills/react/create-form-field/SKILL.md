# Skill: Create Form Field Component

This skill automates the creation of a standardized, HOC-wrapped form field component in a React application that integrates seamlessly with React Hook Form and includes Storybook interaction tests.

---

## 1. Goal
Generate a reusable form field component (implementation and Storybook play-function test suite) under `src/components/` that complies with the project's form validation standards.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **FieldName**: The PascalCase name of the custom form component (e.g., `TextField`, `SelectField`, `CheckboxField`).
* **MuiComponent**: The Material-UI component being wrapped (e.g., `TextField`, `Select`, `Checkbox`).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Verify HOC Existence
Ensure that the `withReactHookForm` HOC is defined under `src/hocs/withReactHookForm/index.tsx`. If it doesn't exist, read the HOC implementation from:
* [react-hook-form spec.md](../../../tech-specs/react/specs/react-hook-form/spec.md#3-the-withreacthookform-higher-order-component)

### Step 2: Create Folder
Create the target component directory:
```text
src/components/{{FieldName}}/
```

### Step 3: Write Implementation File (`index.tsx`)
Create the implementation file at `src/components/{{FieldName}}/index.tsx` following the wrapper pattern.
Read the reference template directly from section 4 ("Wrapping Base Components") of:
* [react-hook-form spec.md](../../../tech-specs/react/specs/react-hook-form/spec.md#4-wrapping-base-components)

Adapt the template for your specific `{{MuiComponent}}`:
1. Destructure `field`, `fieldState`, `required`, `slotProps` (or equivalent styling properties), and other custom properties.
2. Apply the custom error and helper text logic: calculate `error` using `fieldState?.invalid` and `errorMessage` using `fieldState?.error?.message`.
3. Pass `fullWidth` as default.
4. Disable native browser validation by applying `required` only to label/visual subcomponents (like `slotProps.inputLabel` for input components) instead of letting it propagate to the raw input element.

### Step 4: Write Storybook Interaction Tests (`index.stories.tsx`)
Create `src/components/{{FieldName}}/index.stories.tsx` using Storybook Play functions. This test suite must verify validation behavior.
Read the reference testing suite directly from section 6 ("Testing Form Field Components (Storybook)") of:
* [react-hook-form spec.md](../../../tech-specs/react/specs/react-hook-form/spec.md#6-testing-form-field-components-storybook)

Adapt the test suite stories (`Overview`, `ValidationOnBlur`, `ValidationOnChange`, `ValidateButton`, `SubmitButton`, and `ResetButton`) to target the `{{FieldName}}` component instead of `TextField`.

### Step 5: Export the Component
Export the component from the main components index (`src/components/index.ts`):
```typescript
export * from "./{{FieldName}}";
```
*(If `src/components/index.ts` does not exist, create it).*

---

## 4. Key Conventions

1. **Default Width**: Every form field component must have `fullWidth` enabled by default. Layout size is determined by parent grid/flex containers.
2. **Standard HOC Usage**: Prioritize wrapping UI components with `withReactHookForm` to keep the layout code free of boilerplate.
3. **No Native Required Validation**: Destruct the `required` prop and pass it only to components controlling labels or visual indicators (e.g. `slotProps.inputLabel`). Do not let it propagate to raw input elements to prevent native browser validation popups.
