# Skill: Refactor Unstaged Files

This skill automates the refactoring of modified, unstaged files in the workspace to ensure they strictly adhere to all React technical specifications and architectural conventions.

---

## 1. Goal
Identify all files currently modified but not yet staged in the git repository, and refactor them to conform to the project's folder layout, coding style, type declarations, and naming conventions.

---

## 2. Inputs Needed
Before executing, the agent must run git commands to identify the target files. No manual parameters are required, but the agent should focus on the following:
* **Unstaged Files**: Files listed as modified under the `src/` directory from running `git status --porcelain` or `git diff --name-only`. Files that belong to the `.agent/` folder MUST be ignored.

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Detect Unstaged Files
Run the following command to get the list of modified/unstaged files:
```bash
git diff --name-only
```
Filter the output to target only source files (e.g., `.ts`, `.tsx`, `.js`, `.jsx`, `.css` files located within the `src/` directory).

> [!IMPORTANT]
> Do NOT take into account any files that belong to the `.agent/` folder.

### Step 2: Analyze and Refactor Each File
For each identified file, apply the relevant technical specifications:

1. **Folder-as-Module Layout**:
   * If the file is a component (e.g., `src/components/MyComponent.tsx`), refactor it into a self-contained folder structure:
     ```text
     src/components/MyComponent/
     ├── index.tsx              # Main component implementation and exports
     ├── index.css              # Component-specific styles (if any)
     ├── index.test.tsx         # Unit tests (using Vitest)
     └── index.stories.tsx      # Isolated Storybook scenarios
     ```
   * If the file is a custom hook (e.g., `src/hooks/useMyHook.ts`), refactor it into:
     ```text
     src/hooks/useMyHook/
     ├── index.ts               # Hook implementation and exports
     └── index.test.ts          # Unit tests (using Vitest)
     ```
   * References: [conventions/spec.md](../../../tech-specs/react/specs/conventions/spec.md) and [structure/spec.md](../../../tech-specs/react/specs/structure/spec.md).

2. **Types and Interfaces Separation**:
   * Inspect all custom type and interface declarations in the files.
   * Apart from **component-specific props** (which must stay inside the component's file), extract every shared type/interface into a standalone file inside `src/interfaces/`.
   * Follow the **Single-File Principle**: each file under `src/interfaces/` must contain only one type or interface declaration (e.g. `src/interfaces/User.ts`).
   * Reference: [structure/spec.md](../../../tech-specs/react/specs/structure/spec.md#3-general-types-and-interfaces-conventions).

3. **SVG & External Library Icons**:
   * Check if the unstaged files contain inline SVGs or custom icons imported from external packages.
   * Declare/move them into the `src/assets/icons/` directory using the `{{IconName}}Icon.tsx` nomenclature (e.g., `HomeIcon.tsx`, `SearchIcon.tsx`).
   * Reference: [conventions/spec.md](../../../tech-specs/react/specs/conventions/spec.md#general-coding-conventions) and [structure/spec.md](../../../tech-specs/react/specs/structure/spec.md).

4. **Boolean Casting**:
   * Replace any usage of double negations (`!!value`) with explicit casting using the `Boolean(value)` function.
   * Reference: [conventions/spec.md](../../../tech-specs/react/specs/conventions/spec.md#general-coding-conventions).

5. **Axios & API Client Specifications**:
   * Check if files declare HTTP clients or API request hooks.
   * Standardize Axios configurations using the client factory structure under `src/utils/{{clientName}}Axios/` as per [axios/spec.md](../../../tech-specs/react/specs/axios/spec.md).
   * Ensure custom API client hooks conform to the pagination, response wrapper, and MSW unit test mocks described in [api-clients/spec.md](../../../tech-specs/react/specs/api-clients/spec.md) and [mocks/spec.md](../../../tech-specs/react/specs/mocks/spec.md).

6. **Forms & Validation**:
   * Ensure form implementations combine `react-hook-form` and validation schemas (e.g., Yup) under `src/form-schemas/` following camelCase hooks naming.
   * Reference: [react-hook-form/spec.md](../../../tech-specs/react/specs/react-hook-form/spec.md).

7. **Styling and Design System**:
   * Ensure any custom style rules or Material-UI component overrides align with styling palettes and the design system in [styling/spec.md](../../../tech-specs/react/specs/styling/spec.md).

### Step 3: Run Quality Checks
Verify the code builds correctly and all checks pass. Run the following in the project root:
1. **Type Check**:
   ```bash
   npx tsc --noEmit
   ```
2. **Linter Check**:
   ```bash
   npm run lint  # or pnpm lint / yarn lint depending on the project
   ```
3. **Run Test Suite**:
   ```bash
   npm run test  # or pnpm test / yarn test
   ```

### Step 4: Verify with Git Diff
Run `git diff` to review all refactored changes and confirm they align perfectly with the React technical specifications.
