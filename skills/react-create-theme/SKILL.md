---
name: react-create-theme
description: Create and register a custom Material UI theme (with light and dark modes) in a React application following the conventions defined in tech-specs/react/specs/styling/spec.md.
---

# Skill: Create React MUI Theme

This skill automates the creation and registration of a custom Material UI theme (with light and dark modes) in a React application. It strictly adheres to the conventions defined in [styling/spec.md](../../../tech-specs/react/specs/styling/spec.md).

---

## 1. Goal
Generate a custom theme module containing design tokens and component overrides under `src/themes/{{themeName}}/` and register it globally.

---

## 2. Inputs Needed
Before executing, the agent must identify:
* **themeName**: The camelCase name representing the theme (e.g., `ocean`, `sunset`).
* **ThemeName**: The PascalCase name representation (e.g., `Ocean`, `Sunset`).
* **paletteColors**: The primary, secondary, text, and background hex colors for light and dark modes. (Optional if `themeImage` is provided).
* **themeImage**: (Optional) Path, name, or content of an image file from which the agent should extract theme details.
* **componentOverrides**: Custom component properties to style (e.g., button corner roundings or custom card shadows).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Extract Theme Details from Image (Optional)
If the user provides a `themeImage`:
1. Use your vision/analysis capabilities to inspect the styling details, colors, and layout of the image.
2. Extract the dominant color tokens (primary, secondary, light/dark text, and light/dark backgrounds).
3. Identify stylistic configurations like corner roundings (border-radius), box shadows, borders, or glassmorphic parameters.
4. Use these extracted parameters as `paletteColors` and `componentOverrides` for the subsequent steps.

### Step 2: Create Theme Directory
Create the target theme module directory:
```text
src/themes/{{themeName}}/
```

### Step 3: Write Light mode Palette
Create `src/themes/{{themeName}}/light.ts` to define the light palette.
Read the structure blueprint directly from:
* [styling/spec.md#1-default-theme-palettes-srcthemesdefaultlightts](../../../tech-specs/react/specs/styling/spec.md#1-default-theme-palettes-srcthemesdefaultlightts)

Replace colors with the custom light palette hex values.

### Step 4: Write Dark mode Palette
Create `src/themes/{{themeName}}/dark.ts` to define the dark palette. Use the same structure format, replacing values with custom dark palette hex values and setting `mode: 'dark'`.

### Step 5: Write Compiler File (`index.ts`)
Create the theme compiler file at `src/themes/{{themeName}}/index.ts`.
Do not duplicate code; implement the theme compiler utilizing the template in:
* [styling/spec.md#2-default-theme-compiler-srcthemesdefaultindexts](../../../tech-specs/react/specs/styling/spec.md#2-default-theme-compiler-srcthemesdefaultindexts)

Ensure you:
1. Import `lightPalette` from `./light` and `darkPalette` from `./dark`.
2. Apply shape configs (e.g. `borderRadius`) and any custom style overrides for `components` (like button shape roundings).
3. Export two compiled themes: `export const {{themeName}}LightTheme = createTheme(...)` and `export const {{themeName}}DarkTheme = createTheme(...)`.

### Step 6: Register Theme Globally
Register the new theme inside the global aggregator file (`src/themes/index.ts`):
1. Import `{{themeName}}LightTheme` and `{{themeName}}DarkTheme` from `./{{themeName}}`.
2. Update the `ThemeType` type definition to include the string literal `'{{themeName}}'`.
3. Add the mapping to the `themes` record:
   ```typescript
   {{themeName}}: {
     light: {{themeName}}LightTheme,
     dark: {{themeName}}DarkTheme,
   },
   ```
