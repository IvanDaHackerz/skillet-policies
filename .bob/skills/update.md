# Update Skills

Updates installed skills to their latest versions from the Skillet GitHub registry, similar to how npm update works. Use this when you want to keep skills up-to-date with bug fixes, new features, or breaking changes. The updater compares local versions against the registry, automatically updates minor/patch versions, and prompts for confirmation on major version changes.

**Version:** 1.0.0

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
| `skill_name` | string | No | Specific skill to update (e.g., `auth-setup`). If not provided, checks all installed skills |
| `update_all` | boolean | No | Set to `true` to update all outdated skills without individual prompts for minor/patch versions (default: `false`) |
| `check_only` | boolean | No | Set to `true` to only check for updates without installing them (default: `false`) |

---

## Steps

### Step 1: Read skill-index.json from GitHub

Use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json` from the skillet-skills repository.
This file contains the registry of all available skills with their current versions.
Parse the JSON to access the `skills` array with version information.

### Step 2: List Installed Skills

Use the `list_files` tool to list all files in `skillet-skills/skills/` directory.
For each `.md` file found, use the `read_file` tool to read the file and extract the version number.
Look for the line containing `**Version:**` and parse the semantic version (e.g., `1.2.3`).
Build a list of installed skills with their current versions.

### Step 3: Compare Versions

For each installed skill, find its entry in skill-index.json and compare versions:
- Extract the installed version (e.g., `1.2.0`)
- Extract the registry version (e.g., `1.3.0`)
- Determine the version difference type:
  - **MAJOR**: First number changed (1.x.x → 2.x.x) - Breaking changes
  - **MINOR**: Second number changed (1.2.x → 1.3.x) - New features
  - **PATCH**: Third number changed (1.2.3 → 1.2.4) - Bug fixes

Create a list of skills that have updates available with their version change type.

### Step 4: Display Update Summary

Present a formatted summary showing:
- Total number of skills with updates available
- For each outdated skill:
  - Skill name
  - Current version → Latest version
  - Update type indicator (⚠️ MAJOR, ✅ MINOR, ✅ PATCH)

If no updates are available, display "All skills are up to date!" and exit.

### Step 5: Handle Check-Only Mode

If `check_only` is `true`, stop here and display the summary.
Do not proceed with any installations.
Use `attempt_completion` to present the results.

### Step 6: Categorize Updates by Type

Separate the outdated skills into three categories:
- **Major updates**: Require user confirmation (breaking changes)
- **Minor updates**: Auto-update (new features, backward compatible)
- **Patch updates**: Auto-update (bug fixes only)

Display which skills will be updated automatically and which require confirmation.

### Step 7: Auto-Update Minor and Patch Versions

For each skill in the minor and patch categories:
1. Use the MCP tool `read_skill_file` to fetch the latest skill content from GitHub
2. Extract the source path from skill-index.json (e.g., `skills/auth-setup.md`)
3. Use the `write_to_file` tool to save the updated skill to `skillet-skills/skills/{slug}.md`
4. Display progress: `{skill-name}@{version} ✅ Updated`

If any update fails, log the error and continue with remaining updates.

### Step 8: Prompt for Major Version Updates

For each skill with a major version change:
1. Display the skill name and version change
2. If available, show the breaking changes or changelog summary
3. Use the `ask_followup_question` tool to ask: "Update {skill-name} to {version}? (yes/no)"
4. Provide suggestions: "yes", "no", "show changes"

If user responds "yes", proceed with the update using the same process as Step 7.
If user responds "no", skip this skill and continue.
If user responds "show changes", display more detailed changelog if available.

### Step 9: Verify Updated Skills

For each successfully updated skill:
1. Use the `read_file` tool to read the updated skill file
2. Verify the version number matches the expected version
3. Check that all required sections are present (Title, Description, Version, Category, etc.)
4. If validation fails, display a warning but do not roll back

Display a summary of verification results.

### Step 10: Log Activity

Create or append to `logs/activity.log` with the update activity:
- Timestamp of update operation
- List of skills updated with old → new versions
- Any errors or warnings encountered
- User confirmations for major updates

Use the `write_to_file` tool or `insert_content` tool to append to the log file.

### Step 11: Display Final Summary

Present a final summary showing:
- Total skills checked
- Total skills updated
- Breakdown by update type (major/minor/patch)
- Any skills that failed to update
- Location of activity log

---

## Outputs

- Updated skill files in `skillet-skills/skills/` directory
- `logs/activity.log` with update history
- Console output with formatted update summary

---

## Example Usage

### Example 1: Check for Updates Only

**User request:**
> Hey Bob, update --check

**Expected output:**
```
Checking for updates...

3 skills have updates available:

auth-setup        1.0.0 → 2.0.0  ⚠️  MAJOR
error-handling    1.2.0 → 1.3.0  ✅  MINOR
input-validator   1.0.1 → 1.0.2  ✅  PATCH

Run "Hey Bob, update --all" to update all skills.
```

### Example 2: Update Specific Skill

**User request:**
> Hey Bob, update auth-setup

**Expected output:**
```
Checking auth-setup for updates...

auth-setup        1.0.0 → 2.0.0  ⚠️  MAJOR

auth-setup has breaking changes in v2.0.0:
"Step 3 now requires bcrypt rounds 14
 instead of 12. Update your code before
 installing this version."

Update auth-setup to 2.0.0? (yes/no): yes

Updating auth-setup@2.0.0...  ✅ Updated

✅ Successfully updated 1 skill
```

### Example 3: Update All Skills

**User request:**
> Hey Bob, update --all

**Expected output:**
```
Checking for updates...

3 skills have updates available:

auth-setup        1.0.0 → 2.0.0  ⚠️  MAJOR
error-handling    1.2.0 → 1.3.0  ✅  MINOR
input-validator   1.0.1 → 1.0.2  ✅  PATCH

Updating minor and patch versions...
  error-handling@1.3.0    ✅ Updated
  input-validator@1.0.2   ✅ Updated

auth-setup has breaking changes in v2.0.0:
"Step 3 now requires bcrypt rounds 14
 instead of 12. Update your code before
 installing this version."

Update auth-setup to 2.0.0? (yes/no): no

✅ Successfully updated 2 skills
⚠️  Skipped 1 major update

Activity logged to logs/activity.log
```

---

## Notes

- Semantic versioning follows the format MAJOR.MINOR.PATCH (e.g., 2.1.3)
- Minor and patch updates are considered safe and update automatically
- Major updates always require user confirmation due to potential breaking changes
- The `--check` flag is useful for seeing what needs updating without making changes
- Activity logs help track update history and troubleshoot issues
- If a skill update fails, the old version remains in place

## Warnings

> ⚠️ Major version updates may contain breaking changes. Always review the changelog before updating.

> ⚠️ Ensure you have backups of custom modifications to skills before updating.

> ⚠️ The MCP server must be connected and have access to the GitHub repository.

> ⚠️ Skills that fail validation after update should be reviewed manually.

## Related Skills

- Install Skill - Install new skills from the registry
- Uninstall Skill - Remove installed skills
- Search Skills - Find skills in the registry
- Validate Skill - Check skill compliance with standards