# Install Skill

Installs skills from the Skillet GitHub registry into a developer's workspace using the GitHub MCP server. Use this when a developer requests to add a new skill to their Bob instance. The installer reads skill-index.json, fetches the skill from GitHub using MCP tools, and installs it locally.

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- MCP server `skillet-github` is connected and configured in `.bob/settings/mcp_settings.json`
- Bob must be in a mode that supports MCP tools (not Code mode)
- Access to `skill-index.json` in the skillet-skills repository
- Access to developer's workspace `skillet-skills/skills/` directory
- GitHub MCP tools available (read_skill_file, write_skill_file)

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `skill_name` | string | Yes | Name or slug of the skill to install (e.g., `frontend-css-to-tailwind-transformer`) |

---

## Steps

### Step 1: Read skill-index.json from GitHub

Use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json` from the skillet-skills repository.
This file contains the registry of all available skills with their metadata and source information.
The file must be accessed via the GitHub MCP server since it may not be available locally.

### Step 2: Find Skill Entry

Search through the `skills` array in skill-index.json to find the entry matching the requested skill.
Match against either the `slug` field or the `name` field (case-insensitive).
Extract the following information from the entry:
- `name`: Display name of the skill
- `slug`: Unique identifier
- `source.type`: Should be "github"
- `source.repo`: GitHub repository (e.g., "IvanDaHackerz/skillet-skills")
- `source.path`: Path to the skill file (e.g., "skills/frontend-css-to-tailwind-transformer.md")
- `source.branch`: Branch to fetch from (default: "main")
- `description`: What the skill does
- `category`: Skill category
- `roles`: Applicable roles

If the skill is not found, stop and display an error message listing available skills.

### Step 3: Switch to MCP-Enabled Mode (if needed)

If Bob is currently in Code mode, you need to inform the user that MCP tools are not available in Code mode.
The user will need to switch to a mode that supports MCP tools or provide an alternative approach.

Code mode limitation: MCP tools like `read_skill_file` are not accessible in Code mode.

### Step 4: Fetch Skill from GitHub using MCP

Use the MCP tool `read_skill_file` to fetch the skill content from the GitHub repository.
- Tool: read_skill_file
- Parameters:
  - path: "{source.path}" (e.g., "skills/frontend-css-to-tailwind-transformer.md")

This will retrieve the file content directly from the skillet-skills GitHub repository.
Store the fetched content for the next step.

**Alternative if MCP is not available:** Fetch the file directly from GitHub using the GitHub API or by reading from a local clone of the repository.

### Step 5: Ensure skillet-skills/skills/ Directory Exists

Check if the `skillet-skills/skills/` directory exists in the current workspace.
The directory should already exist as part of the skillet-skills repository structure.
If it doesn't exist, create it using the `execute_command` tool:
- Windows: `New-Item -ItemType Directory -Force -Path skillet-skills\skills`
- Unix/Mac: `mkdir -p skillet-skills/skills`

### Step 6: Write Skill to Local skillet-skills/skills/

Use the `write_to_file` tool to save the skill content to the developer's local workspace.
Write the file to: `skillet-skills/skills/{slug}.md`
Include the complete file content and accurate line count.

### Step 7: Display Installation Summary

Present a formatted installation summary to the user showing:
- Skill name installed
- Installation location (`skillet-skills/skills/{slug}.md`)
- Brief description of what the skill does
- Category and applicable roles
- Source repository and branch
- Confirmation that the skill is ready to use

Format the output with clear sections and checkmarks for successful steps.

---

## Outputs

- `skillet-skills/skills/{slug}.md` — The installed skill file in the developer's workspace
- Console output — Formatted installation summary with status

---

## Example Usage

**User request:**
> Hey Bob, install frontend-css-to-tailwind-transformer

**Expected flow:**
```
Reading skill-index.json... ✅

Found skill: CSS to Tailwind Transformer
Slug: frontend-css-to-tailwind-transformer
Source: IvanDaHackerz/skillet-skills (main branch)
Path: skills/frontend-css-to-tailwind-transformer.md

Fetching from GitHub via MCP... ✅
Using tool: read_skill_file
Path: skills/frontend-css-to-tailwind-transformer.md

Writing to skillet-skills/skills/frontend-css-to-tailwind-transformer.md... ✅

✅ Successfully installed frontend-css-to-tailwind-transformer

What this skill does:
Transforms traditional CSS styling into Tailwind CSS 
utility classes while preserving design intent and 
responsiveness.

Category: frontend
Roles: frontend, fullstack

The skill is now ready to use!
```

---

## Notes

- Skills are fetched directly from the GitHub repository using the MCP server
- The skill-index.json must contain a `source` object with GitHub repository information
- Installation overwrites existing skill files with the same name
- The `.bob/skills/` directory will be created if it doesn't exist
- Requires the GitHub MCP server to be properly configured and connected

## Warnings

> ⚠️ Ensure the GitHub MCP server (`skillet-github`) is connected before attempting installation.

> ⚠️ Verify the skill exists in skill-index.json before attempting to fetch from GitHub.

> ⚠️ The `skillet-skills/skills/` directory must be writable in the developer's workspace.

> ⚠️ The MCP tool `read_skill_file` requires proper GitHub authentication and repository access.

## Related Skills

- List Skills
- Update Skill
- Remove Skill
- Search Skills