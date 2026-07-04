# Component & Folder Conventions

To maintain a clean, maintainable, and highly discoverable codebase, structure all files (components, hooks, utilities, context providers, HOCs, and repositories) using self-contained folders where the entry file is named `index`.

## Component Folder Layout Example

For a component named `PrimaryButton`:

```text
src/components/PrimaryButton/
├── index.tsx              # Main component implementation and exports
├── index.css              # Component-specific styles (if any)
├── index.test.tsx         # Unit tests (optional; not needed if storybook testing is sufficient)
└── index.stories.tsx      # Isolated Storybook scenarios
```

## Component Structure Guidelines

> [!NOTE]
> If a component contains no custom business logic or complex state transitions and can be verified entirely through visual inspection in Storybook, the `index.test.tsx` file is not required.


### 1. `index.tsx` (Implementation)

```tsx
import React from 'react';
import { Button as MuiButton, ButtonProps as MuiButtonProps } from '@mui/material';

export interface PrimaryButtonProps extends MuiButtonProps {
  label: string;
}

export const PrimaryButton: React.FC<PrimaryButtonProps> = ({
  label,
  variant = 'contained',
  color = 'primary',
  onClick,
  ...props
}) => {
  return (
    <MuiButton variant={variant} color={color} onClick={onClick} {...props}>
      {label}
    </MuiButton>
  );
};
```

### 2. `index.test.tsx` (Unit Tests)

```tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { PrimaryButton } from './index';

describe('PrimaryButton Component', () => {
  it('renders the label correctly', () => {
    render(<PrimaryButton label="Click Me" />);
    expect(screen.getByText('Click Me')).toBeInTheDocument();
  });

  it('fires onClick event handler when clicked', () => {
    const handleClick = vi.fn();
    render(<PrimaryButton label="Click Me" onClick={handleClick} />);
    fireEvent.click(screen.getByText('Click Me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### 3. `index.stories.tsx` (Storybook Stories)

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { PrimaryButton } from './index';

const meta: Meta<typeof PrimaryButton> = {
  title: 'Components/PrimaryButton',
  component: PrimaryButton,
  tags: ['autodocs'],
  argTypes: {
    onClick: { action: 'clicked' },
  },
};

export default meta;
type Story = StoryObj<typeof PrimaryButton>;

export const Primary: Story = {
  args: {
    label: 'Primary Button',
    variant: 'contained',
    color: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    label: 'Secondary Button',
    variant: 'outlined',
    color: 'secondary',
  },
};
```

---

## Custom Hook Folder Layout Example

For a hook named `useToggle`:

```text
src/hooks/useToggle/
├── index.ts               # Hook implementation and exports
└── index.test.ts          # Unit tests using Vitest
```

### Hook Structure Guidelines

### 1. `index.ts` (Implementation)

```typescript
import { useState, useCallback } from 'react';

export interface UseToggleReturn {
  value: boolean;
  toggle: () => void;
  setTrue: () => void;
  setFalse: () => void;
}

export const useToggle = (initialValue = false): UseToggleReturn => {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue((v) => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse };
};
```

### 2. `index.test.ts` (Unit Tests)

```typescript
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useToggle } from './index';

describe('useToggle', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current.value).toBe(false);
  });

  it('should toggle values', () => {
    const { result } = renderHook(() => useToggle(false));
    act(() => {
      result.current.toggle();
    });
    expect(result.current.value).toBe(true);
  });
});
```

---

## General Coding Conventions

1. **Boolean Casting**: Avoid double negation (`!!value`) and prioritize using `Boolean(value)` for casting truthy/falsy expressions to boolean values (e.g., checking if error or helper states exist) to enhance code readability and clarity.
2. **SVG & External Library Icons**: All SVG icons or icons from external libraries MUST be declared inside the `src/assets/icons/` folder, following the nomenclature `{{IconName}}Icon.tsx` (e.g., `HomeIcon.tsx`, `SearchIcon.tsx`).

[Go back to Table of Contents](../README.md)

