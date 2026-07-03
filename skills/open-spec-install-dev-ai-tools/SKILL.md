---
name: open-spec-install-dev-ai-tools
description: Setup standardized specifications and prompt runbooks (skills) inside the target workspace's .agent/ folder.
license: MIT
compatibility: Requires git and basic bash utilities.
metadata:
  author: dev-ai-tools
  version: "1.0"
---

# Skill: Install Dev AI Tools & Specs

This skill automates cloning the standardized developer AI tools repository, copying its specifications and agent skills into a local `.agent/` directory of the target workspace, and rewriting relative links for local execution.

---

## 1. Goal

Set up the standardized specifications and prompt runbooks (skills) inside the target workspace's `.agent/` folder to enable offline agent alignment.

---

## 2. Execution Runbook

### Step 1: Clone the Repository Temporarily

Clone the `dev-ai-tools` repository into a temporary directory in the workspace:

```bash
git clone https://github.com/mauriver21/dev-ai-tools.git tmp-dev-ai-tools
```

### Step 2: Initialize Target Agent Folders

Since the `.agent/` directory is already pre-created in the target workspace, only initialize the specs resources directory:

```bash
mkdir -p .agent/resources/specs
```

### Step 3: Copy Tech Specs

Copy the `tech-specs` folder from the temporary directory into the target specs directory:

```bash
cp -r tmp-dev-ai-tools/tech-specs .agent/resources/specs/
```

_(This places the `tech-specs` folder structure at `.agent/resources/specs/tech-specs/`)._

### Step 4: Copy and Rename Skills

For each technology category (like `react`), copy its skills into the `.agent/skills/` folder, prefixing them with the stack name (`{{tech}}-{{skillName}}`):

```bash
# Copy react skills
cp -r tmp-dev-ai-tools/skills/react/configure-axios .agent/skills/react-configure-axios
cp -r tmp-dev-ai-tools/skills/react/create-api-client .agent/skills/react-create-api-client
cp -r tmp-dev-ai-tools/skills/react/scaffold-app .agent/skills/react-scaffold-app
cp -r tmp-dev-ai-tools/skills/react/create-mock .agent/skills/react-create-mock
cp -r tmp-dev-ai-tools/skills/react/create-theme .agent/skills/react-create-theme
```

### Step 5: Rewrite Relative Paths in Skills

In every copied skill file (e.g. `.agent/skills/*/SKILL.md`), rewrite all relative links targeting `tech-specs/` to match the new local structure.
Specifically, replace the parent prefix `../../../tech-specs/` with the new relative path `../../resources/specs/tech-specs/`.

#### Replace Rule:

- **Search Pattern**: `../../../tech-specs/`
- **Replacement**: `../../resources/specs/tech-specs/`

_Example replacement result in `.agent/skills/react-configure-axios/SKILL.md`:_

```markdown
[axios.md](../../../tech-specs/react/specs/axios/spec.md) -> [axios.md](../../resources/specs/tech-specs/react/specs/axios/spec.md)
```

### Step 6: Cleanup Temporary Files

Delete the cloned temporary directory:

```bash
rm -rf tmp-dev-ai-tools
```
