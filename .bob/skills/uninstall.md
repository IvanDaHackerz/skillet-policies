# Uninstall Skill

Safely removes skills from a developer's workspace with dependency checking and cleanup options. Use this when a developer wants to remove an installed skill from their Bob instance. The uninstaller checks for dependencies, warns about impacts, and optionally removes orphaned dependencies.

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- Access to developer's workspace `skillet-skills/skills/` directory
- Access to `skill-index.json` to check skill metadata and dependencies
- Ability to create `.deprecated/` folder for backup
- File system write permissions

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `skill_name` | string | Yes | Name or slug of the skill to uninstall (e.g., `auth-setup`, `frontend-css-to-tailwind-transformer`) |
| `version` | string | Optional | Specific version to uninstall (defaults to currently installed version) |
| `force` | boolean | Optional | Skip dependency warnings and force removal (default: false) |
| `keep_backup` | boolean | Optional | Keep backup in .deprecated folder (default: true) |

---

## Steps

### Step 1: Locate Skill in Workspace

Search for the skill file in the developer's workspace at `skillet-skills/skills/{slug}.md`.
If the skill file doesn't exist, inform the user that the skill is not installed and exit.

### Step 2: Read Skill Metadata

Parse the skill file to extract:
- Skill name
- Version (if specified in metadata)
- Category
- Dependencies (if any)

Also check `skill-index.json` for additional metadata about the skill.

### Step 3: Check for Dependent Skills

Scan all installed skills in `skillet-skills/skills/` directory to identify any skills that depend on the skill being uninstalled.

For each installed skill:
- Read its prerequisites or dependencies section
- Check if it references the skill being uninstalled
- Build a list of dependent skills

### Step 4: Display Dependency Warning (if applicable)

If dependent skills are found, display a warning message:

```
⚠️  Warning: {count} skill(s) depend on this:
    - {skill-1} depends on {skill-name}
    - {skill-2} depends on {skill-name}
    ...

Continue? (yes/no):
```

Use the `ask_followup_question` tool to prompt the user for confirmation.
- If user responds "no" or cancels, abort the uninstallation
- If user responds "yes", proceed to next step
- If `force` parameter is true, skip this warning

### Step 5: Create Backup (if enabled)

If `keep_backup` is true (default):
1. Create `.deprecated/` directory if it doesn't exist
2. Move the skill file to `.deprecated/{slug}.md.{timestamp}`
3. Confirm backup creation with checkmark

If `keep_backup` is false:
1. Delete the skill file directly
2. Confirm deletion with checkmark

### Step 6: Identify Orphaned Dependencies

After removing the skill, scan for orphaned dependencies:
- Check if any skills that were dependencies of the removed skill are no longer needed
- A dependency is orphaned if no other installed skill depends on it
- Build a list of orphaned dependencies

### Step 7: Prompt for Orphaned Dependency Removal

If orphaned dependencies are found, ask the user:

```
Remove unused dependencies? (yes/no):
```

Use the `ask_followup_question` tool with suggestions:
- "yes" - Remove all orphaned dependencies
- "no" - Keep orphaned dependencies
- "show list" - Display list of orphaned dependencies before deciding

If user confirms, remove each orphaned dependency and display progress.

### Step 8: Display Uninstallation Summary

Present a formatted summary showing:
- Skill name and version uninstalled
- Backup location (if created)
- List of removed orphaned dependencies (if any)
- Confirmation that uninstallation is complete

---

## Outputs

- Removed skill file from `skillet-skills/skills/`
- Backup file in `.deprecated/` folder (if enabled)
- Console output with formatted uninstallation summary
- List of removed orphaned dependencies (if any)

---

## Example Usage

### Example 1: Basic Uninstallation with Dependencies

**User request:**
> bob uninstall auth-setup

**Expected flow:**
```
Uninstalling auth-setup@2.1.0...

⚠️  Warning: 2 skills depend on this:
    - api-design depends on auth-setup
    - user-management depends on auth-setup

Continue? (yes/no): yes

Removing auth-setup@2.1.0...     ✅
Moving to .deprecated folder...  ✅

Remove unused dependencies? (yes/no): yes
Removing input-validator@1.0.0   ✅

✅ Successfully uninstalled auth-setup
```

### Example 2: Uninstallation without Dependencies

**User request:**
> bob uninstall frontend-css-to-tailwind-transformer

**Expected flow:**
```
Uninstalling frontend-css-to-tailwind-transformer...

No dependent skills found.

Removing frontend-css-to-tailwind-transformer...  ✅
Moving to .deprecated folder...                   ✅

No orphaned dependencies found.

✅ Successfully uninstalled frontend-css-to-tailwind-transformer

Backup location: .deprecated/frontend-css-to-tailwind-transformer.md.2026-05-02T19-08-00
```

### Example 3: Force Uninstallation

**User request:**
> bob uninstall auth-setup --force

**Expected flow:**
```
Uninstalling auth-setup@2.1.0... (forced)

Removing auth-setup@2.1.0...     ✅
Moving to .deprecated folder...  ✅

Remove unused dependencies? (yes/no): no

✅ Successfully uninstalled auth-setup (forced)

⚠️  Warning: 2 skills may be affected:
    - api-design
    - user-management
```

### Example 4: Uninstallation without Backup

**User request:**
> bob uninstall old-skill --no-backup

**Expected flow:**
```
Uninstalling old-skill...

No dependent skills found.

Permanently deleting old-skill...  ✅

⚠️  No backup created (--no-backup flag used)

✅ Successfully uninstalled old-skill
```

---

## Notes

### Dependency Detection
- Parse Prerequisites section for skill names and explicit dependency declarations
- Check Related Skills section for required skills
- Scan for skill references using patterns like "requires {skill-name}" or "depends on {skill-name}"
- Warn about both hard dependencies (required) and soft dependencies (recommended)

### Best Practices
- Always create backups by default unless explicitly disabled
- Provide clear warnings about dependent skills
- Use timestamps in backup filenames to avoid conflicts
- Organize backups in `.deprecated/` folder for easy cleanup
- Log uninstallation actions for audit trail

### Advanced Options
- Use `--force` flag to skip dependency warnings (use with caution)
- Use `--no-backup` to permanently delete without backup
- Use `--keep-dependencies` to skip orphaned dependency cleanup
- Batch uninstall multiple skills: `bob uninstall skill-1 skill-2 skill-3`

---

## Warnings

> ⚠️ Always review dependent skills before uninstalling to avoid breaking functionality.

> ⚠️ Force uninstallation bypasses safety checks and may break dependent skills.

> ⚠️ Backup files in `.deprecated/` folder should be cleaned up periodically.

---

## Related Skills

- Install Skill - Install skills from the Skillet registry
- Update Skill - Update installed skills to newer versions
- List Skills - View all installed skills and their dependencies
- Restore Skill - Restore a skill from backup
