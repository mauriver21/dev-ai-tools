---
name: react-generate-translations
description: Generate translation JSON catalogs for target languages and register them within the application following the conventions defined in tech-specs/react/specs/i18n/spec.md.
---

# Skill: Generate Translations

This skill automates the creation of translation JSON catalogs for target languages and registers them within the application. It strictly adheres to the conventions defined in [i18n/spec.md](architecture/react/specs/i18n/spec.md).

---

## 1. Goal

Generate translated JSON catalogs for specific language codes under `src/i18n/translations/{{targetLang}}/` and update both the namespace indexes and global translations bundler.

---

## 2. Inputs Needed

Before executing, the agent must identify or request:

- **targetLanguages**: A list of language codes to translate into (e.g., `es`, `fr`, `de`). The agent **MUST** explicitly ask the user for the desired target languages before proceeding.
- **sourceLanguage**: The language code of the source translations (Optional. Defaults to `en`).
- **namespaces**: The translation namespace keys (e.g., `common`, `glossary`). (Optional. If not specified, the agent should scan and translate all namespaces found in the source language directory).

---

## 3. Execution Instructions

Follow these steps to perform the skill:

### Step 1: Query Desired Languages

Prompt the user in chat (or using a questionnaire tool if available) to specify the desired target languages for translation:

- Ask: "Which languages would you like to generate translations for? (e.g., Spanish 'es', French 'fr', German 'de')"
- Wait for the user's input/selection of target languages before proceeding to Step 2.

### Step 2: Read Source Translation Catalogs

1. Locate the source translation files under:
   ```text
   src/i18n/translations/{{sourceLanguage}}/
   ```
2. Scan the directory to identify all existing namespace files (e.g., `common.json`, `glossary.json`).
3. If specific `namespaces` were requested, filter to only those files. Otherwise, select all detected files.
4. Read and parse each JSON catalog file to extract the source keys and text.

### Step 3: Translate and Create Target Catalogs

For each language code in `targetLanguages` (referred to as `{{targetLang}}`):

1. Create the directory for the target language:
   ```text
   src/i18n/translations/{{targetLang}}/
   ```
2. For each namespace JSON file:
   a. Translate all text values from the source language to `{{targetLang}}` using translation tools or LLM context capabilities.
   b. Ensure that the JSON structure, object keys, and interpolation variables (e.g. `{{name}}` or `{{count}}`) are preserved exactly.
   c. Write the translated content to `src/i18n/translations/{{targetLang}}/{{namespace}}.json`.

### Step 4: Create Language Namespace Index

For each `{{targetLang}}`, create or update the namespace index file `src/i18n/translations/{{targetLang}}/index.ts` to export each translated JSON namespace catalog:

```typescript
export { default as {{namespace1}} } from './{{namespace1}}.json';
export { default as {{namespace2}} } from './{{namespace2}}.json';
```

_(Reference: [i18n/spec.md#2-namespace-bundlers](architecture/react/specs/i18n/spec.md#2-namespace-bundlers))_

### Step 5: Register Target Languages Globally

Update the global namespace aggregator file `src/i18n/translations/index.ts` to import and export the newly created target language namespaces:

```typescript
export * as {{targetLang}} from './{{targetLang}}';
```

_(Reference: [i18n/spec.md#global-namespace-index](architecture/react/specs/i18n/spec.md#global-namespace-index))_

---

## 4. Key Conventions Reminder

> [!IMPORTANT]
> All translated texts must align with the layout in the catalogs. Do not hardcode localized UI texts in code components. Component rendering must fetch keys reactively utilizing the `t` function.
