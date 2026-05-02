# List Skills

Lists all installed skills in the developer's workspace with their versions and update status, similar to how npm list works. Use this when you want to see what skills are installed, check which ones need updates, or filter by category. The lister reads local skills, compares against the registry, and displays a formatted summary grouped by category.

**Version:** 1.0.0

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- MCP server `skillet-github` is connected and configured in `.bob/settings/mcp_settings.json`
- Bob must be in a mode that supports MCP tools (not Code mode)
- Access to `skill-index.json` in the skillet-skills repository
- Access to developer's workspace `skillet-skills/skills/` directory
- GitHub MCP tools available (read_skill_file)

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `outdated_only` | boolean | No | Set to `true` to only show skills that need updates (default: `false`) |
| `category` | string | No | Filter by category: `backend`, `frontend`, `devops`, `qa`, `shared` |

---

## Steps

### Step 1: Read skill-index.json from GitHub

Use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json` from the skillet-skills repository.
This file contains the registry of all available skills with their current versions.
Parse the JSON to access the `skills` array with version information.

### Step 2: List Installed Skills

Use the `list_files` tool to list all files in `skillet-skills/skills/` directory.
For each `.md` file found, use the `read_file` tool to read the first 20 lines and extract:
- Skill name (from the title line)
- Version number (look for `**Version:**` and parse semantic version like `1.2.3`)
- Category (look for `**Category:**`)
- Roles (look for `**Roles:**`)

Build a list of installed skills with their metadata.

### Step 3: Compare with Registry Versions

For each installed skill, find its entry in skill-index.json by matching the slug:
- Extract the installed version (e.g., `1.2.0`)
- Extract the registry version (e.g., `1.3.0`)
- Determine update status:
  - ✅ **Latest**: Installed version matches registry version
  - ⬆️ **Update available**: Registry has a newer version
  - ❓ **Unknown**: Skill not found in registry (custom/local skill)

Store the comparison results for each skill.

### Step 4: Apply Filters

If `outdated_only` is `true`, filter the list to only include skills with updates available.
If `category` is provided, filter the list to only include skills in that category.
Keep track of total counts before and after filtering for the summary.

### Step 5: Group Skills by Category

Organize the filtered skills into groups by category:
- backend
- frontend
- devops
- qa
- shared

Sort skills alphabetically within each category.
Count how many skills are in each category.

### Step 6: Format Output Display

Create a formatted output following this structure:

```
Installed Skills — [developer-name]
Tier: [developer-tier]

[CATEGORY NAME] ([count])
├── [skill-name]@[version]    [status] [latest-version if update available]
├── [skill-name]@[version]    [status] [latest-version if update available]
└── [skill-name]@[version]    [status] [latest-version if update available]

[CATEGORY NAME] ([count])
├── [skill-name]@[version]    [status] [latest-version if update available]
└── [skill-name]@[version]    [status] [latest-version if update available]

────────────────────────────────
Total installed:  [count] skills
Updates needed:   [count] skills
Current tier:     [tier-name]
Unlocks at [next-tier]: [count] more skills

Run "Hey Bob, update --all" to update
```

Use these symbols:
- ✅ for up-to-date skills
- ⬆️ for skills with updates available
- ❓ for unknown/custom skills

Use tree characters (├──, └──) for visual hierarchy.

### Step 7: Determine Developer Tier

Read the developer's tier information from their profile or settings.
Tiers typically include:
- Junior Developer
- Mid-Level Developer
- Senior Developer
- Lead Developer
- Principal Engineer

Calculate how many more skills unlock at the next tier based on tier progression rules.

### Step 8: Display Summary Statistics

At the bottom of the output, display:
- **Total installed**: Count of all installed skills
- **Updates needed**: Count of skills with available updates
- **Current tier**: Developer's current tier level
- **Unlocks at [next tier]**: Number of additional skills available at next tier

If updates are available, include the suggestion: `Run "Hey Bob, update --all" to update`

### Step 9: Handle Empty Results

If no skills are installed, display:
```
No skills installed yet.

Run "Hey Bob, search" to browse available skills
Run "Hey Bob, install [skill-name]" to install a skill
```

If filtering results in no matches, display:
```
No skills found matching your criteria.

Run "Hey Bob, list" to see all installed skills
```

---

## Outputs

- Formatted console output showing installed skills grouped by category
- Update status for each skill
- Summary statistics with tier information
- Helpful suggestions for next actions

---

## Example Usage

### Example 1: List All Skills

**User request:**
> Hey Bob, list

**Expected output:**
```
Installed Skills — john.smith
Tier: Mid-Level Developer

SECURITY (3)
├── auth-setup@2.0.0        ✅ Latest
├── jwt-handling@1.2.0      ✅ Latest
└── input-validator@1.0.1   ⬆️  1.0.2 available

BACKEND (2)
├── api-design@1.1.0        ✅ Latest
└── error-handling@1.2.0    ✅ Latest

TESTING (1)
└── unit-testing@1.0.0      ⬆️  1.1.0 available

────────────────────────────────
Total installed:  6 skills
Updates needed:   2 skills
Current tier:     Mid-Level
Unlocks at Senior: 4 more skills

Run "Hey Bob, update --all" to update
```

### Example 2: List Outdated Skills Only

**User request:**
> Hey Bob, list --outdated

**Expected output:**
```
Installed Skills — john.smith
Tier: Mid-Level Developer

Showing outdated skills only

SECURITY (1)
└── input-validator@1.0.1   ⬆️  1.0.2 available

TESTING (1)
└── unit-testing@1.0.0      ⬆️  1.1.0 available

────────────────────────────────
Total installed:  6 skills
Updates needed:   2 skills
Current tier:     Mid-Level
Unlocks at Senior: 4 more skills

Run "Hey Bob, update --all" to update
```

### Example 3: List Skills by Category

**User request:**
> Hey Bob, list --category security

**Expected output:**
```
Installed Skills — john.smith
Tier: Mid-Level Developer

Showing category: SECURITY

SECURITY (3)
├── auth-setup@2.0.0        ✅ Latest
├── jwt-handling@1.2.0      ✅ Latest
└── input-validator@1.0.1   ⬆️  1.0.2 available

────────────────────────────────
Total installed:  6 skills
Updates needed:   2 skills
Current tier:     Mid-Level
Unlocks at Senior: 4 more skills

Run "Hey Bob, update --all" to update
```

---

## Notes

- Skills are always grouped by category for easier reading
- The tree structure (├──, └──) makes the hierarchy clear
- Update status is shown inline with each skill
- Tier information helps developers understand their progression
- The summary provides quick insights into the skill ecosystem
- Custom/local skills (not in registry) are marked with ❓

## Warnings

> ⚠️ The MCP server must be connected to fetch the latest version information from the registry.

> ⚠️ Skills not found in the registry are marked as "Unknown" and may be custom or deprecated.

> ⚠️ Tier information requires access to developer profile settings.

## Related Skills

- Update Skills - Update installed skills to latest versions
- Install Skill - Install new skills from the registry
- Search Skills - Find skills in the registry
- Uninstall Skill - Remove installed skills