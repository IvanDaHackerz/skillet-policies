# Audit Skills

Audits installed skills for security vulnerabilities, conflicts, outdated versions, and policy violations, similar to how npm audit works. Use this when you want to check the health and compliance of your skill ecosystem. The auditor scans local skills, compares against the registry, checks business rules, and generates a comprehensive report with severity ratings and fix recommendations.

**Version:** 1.0.0

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- MCP server `skillet-github` is connected and configured in `.bob/settings/mcp_settings.json`
- Bob must be in a mode that supports MCP tools (not Code mode)
- Access to `skill-index.json` in the skillet-skills repository
- Access to `business-rules/` directory in skillet-policies repository
- Access to developer's workspace `skillet-skills/skills/` directory
- GitHub MCP tools available (read_skill_file, read_policy_file)

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `fix` | boolean | No | Set to `true` to automatically fix CRITICAL and HIGH severity issues (default: `false`) |
| `registry` | boolean | No | Set to `true` to audit the entire master registry instead of local workspace (default: `false`) |

---

## Steps

### Step 1: Determine Audit Scope

If `registry` is `true`, audit the entire skill registry from GitHub.
If `registry` is `false`, audit only the locally installed skills in `skillet-skills/skills/`.

For local audits, use the `list_files` tool to get all `.md` files.
For registry audits, use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json`.

### Step 2: Read Business Rules and Policies

Use the MCP tool `read_policy_file` to read all policy files from the skillet-policies repository:
- `rules/skill-format.md` - Format requirements
- `rules/security-guidelines.md` - Security standards
- `rules/coding-standards.md` - Coding best practices
- `metadata/roles.json` - Valid roles and tier restrictions

Parse these files to understand current compliance requirements.

### Step 3: Scan for Outdated Skills with Known Issues

For each skill, compare the installed version against the registry version.
Check if the registry entry includes a `knownIssues` or `securityAdvisories` field.
If a skill has known security vulnerabilities or critical bugs, flag it as CRITICAL.

Criteria:
- **CRITICAL**: Security vulnerability in current version
- **HIGH**: Critical bug affecting functionality
- **MEDIUM**: Important bug fix available
- **LOW**: Minor improvements available

### Step 4: Check for Skill Conflicts

Analyze all installed skills for conflicts:
- **Duplicate functionality**: Two skills that do the same thing
- **Incompatible versions**: Skills that require different versions of dependencies
- **Overlapping categories**: Skills with identical purposes in the same category
- **Deprecated skills**: Skills marked as deprecated in the registry

For each conflict found, determine severity:
- **HIGH**: Skills that will cause runtime errors together
- **MEDIUM**: Skills with overlapping functionality causing confusion
- **LOW**: Skills that could be consolidated for efficiency

### Step 5: Verify Tier Restrictions

Read the developer's current tier from their profile or settings.
For each installed skill, check if it requires a higher tier than the developer has.

Tier hierarchy (example):
1. Junior Developer
2. Mid-Level Developer
3. Senior Developer
4. Lead Developer
5. Principal Engineer

Flag any skills above the developer's tier:
- **HIGH**: Skill requires tier 2+ levels above current
- **MEDIUM**: Skill requires 1 level above current
- **LOW**: Skill is at the edge of current tier

### Step 6: Check Policy Compliance

For each skill, validate against business rules:
- **Format compliance**: Follows skill-format.md structure
- **Security compliance**: Meets security-guidelines.md requirements
- **Coding standards**: Adheres to coding-standards.md conventions
- **Required sections**: Has all mandatory sections (Version, Category, etc.)
- **Valid roles**: Uses only roles defined in roles.json

Flag violations:
- **CRITICAL**: Security violations (hardcoded credentials, SQL injection risks)
- **HIGH**: Missing required sections or invalid format
- **MEDIUM**: Coding standard violations
- **LOW**: Minor formatting issues

### Step 7: Check for Missing Dependencies

Scan each skill's Prerequisites section for required dependencies.
Check if those dependencies are:
- Installed in the workspace
- Available in the system
- At compatible versions

Flag missing dependencies:
- **HIGH**: Required MCP server not connected
- **MEDIUM**: Required tool not installed
- **LOW**: Optional dependency missing

### Step 8: Calculate Severity Scores

For each issue found, assign a severity level:
- **CRITICAL**: Immediate security risk or system-breaking issue
- **HIGH**: Significant functionality impact or near-term risk
- **MEDIUM**: Moderate impact, should be addressed soon
- **LOW**: Minor issue, informational only

Count the total number of issues at each severity level.

### Step 9: Generate Audit Report

Create a formatted audit report with this structure:

```
Running skill audit...

Scanning [count] installed skills...

Found [count] issues:

CRITICAL
[skill-name]@[version]
[Description of the issue]
Fix: "[exact command to fix]"

HIGH
[skill-name]@[version]
[Description of the issue]
Fix: "[exact command to fix]"

MEDIUM
[skill-name]@[version]
[Description of the issue]
Fix: "[exact command to fix]"

LOW
[skill-name]@[version]
[Description of the issue]
Fix: "[exact command to fix]"

────────────────────────────────
[count] critical  [count] high  [count] medium  [count] low

Run "Hey Bob, audit --fix" to automatically fix critical issues.
```

Group issues by severity level, then by skill name.
Provide exact fix commands for each issue.

### Step 10: Write to Activity Log

Use the `write_to_file` or `insert_content` tool to append the full audit report to `logs/activity.log`.
Include:
- Timestamp of audit
- Audit scope (local or registry)
- Complete list of issues found
- Severity breakdown
- Fix commands executed (if --fix was used)

Format the log entry clearly with separators for easy reading.

### Step 11: Auto-Fix Critical and High Issues (if --fix flag)

If `fix` is `true`, automatically resolve CRITICAL and HIGH severity issues:

**For outdated skills with security issues:**
- Use the update skill to update to the latest version
- Execute: `update [skill-name]`

**For skill conflicts:**
- Uninstall the older or less-maintained skill
- Execute: `uninstall [skill-name]`

**For tier violations:**
- Uninstall skills above the developer's tier
- Execute: `uninstall [skill-name]`
- Notify the developer why the skill was removed

**For policy violations:**
- If fixable automatically (e.g., format issues), apply fixes
- If not fixable, flag for manual review

Display progress for each fix:
```
Fixing critical issues...
  ✅ Updated input-validator@1.0.1 → 1.0.2
  ✅ Uninstalled conflicting-skill@2.0.0
  ⚠️  Manual review needed for policy-violation-skill
```

### Step 12: Display Final Summary

Present a summary showing:
- Total skills audited
- Total issues found by severity
- Issues automatically fixed (if --fix was used)
- Issues requiring manual intervention
- Location of detailed audit log

If issues remain, provide guidance on next steps.
If no issues found, display: "✅ No issues found. All skills are healthy!"

---

## Outputs

- Formatted console output with audit results grouped by severity
- `logs/activity.log` with complete audit report and timestamp
- List of automatically applied fixes (if --fix flag used)
- Recommendations for manual fixes

---

## Example Usage

### Example 1: Basic Audit

**User request:**
> Hey Bob, audit

**Expected output:**
```
Running skill audit...

Scanning 6 installed skills...

Found 3 issues:

CRITICAL
input-validator@1.0.1
Security vulnerability in validation logic.
CVE-2024-1234: Allows bypass of input sanitization.
Fix: "Hey Bob, update input-validator"

MEDIUM
unit-testing@1.0.0
Conflicts with testing-framework@2.0.0
Both cover the same test patterns, causing confusion.
Fix: "Hey Bob, uninstall unit-testing"
     "Hey Bob, install testing-framework"

LOW
error-handling@1.2.0
Minor update available with improvements.
Version 1.2.1 includes better error messages.
Fix: "Hey Bob, update error-handling"

────────────────────────────────
1 critical  0 high  1 medium  1 low

Run "Hey Bob, audit --fix" to automatically fix critical issues.

Full audit report saved to logs/activity.log
```

### Example 2: Audit with Auto-Fix

**User request:**
> Hey Bob, audit --fix

**Expected output:**
```
Running skill audit...

Scanning 6 installed skills...

Found 3 issues:

CRITICAL
input-validator@1.0.1
Security vulnerability in validation logic.
CVE-2024-1234: Allows bypass of input sanitization.

HIGH
auth-setup@1.5.0
Requires Senior Developer tier (current: Mid-Level)
This skill uses advanced security features.

MEDIUM
unit-testing@1.0.0
Conflicts with testing-framework@2.0.0

────────────────────────────────
Fixing critical and high issues...

  ✅ Updated input-validator@1.0.1 → 1.0.2
  ✅ Uninstalled auth-setup@1.5.0 (tier restriction)

────────────────────────────────
1 critical fixed  1 high fixed  1 medium remaining

Remaining issues require manual review.
Run "Hey Bob, audit" to see updated report.

Full audit report saved to logs/activity.log
```

### Example 3: Audit Registry

**User request:**
> Hey Bob, audit --registry

**Expected output:**
```
Running registry audit...

Scanning 24 skills in master registry...

Found 5 issues:

CRITICAL
payment-processor@2.1.0
Security vulnerability in payment handling.
CVE-2024-5678: Potential data exposure.
Fix: Update skill in registry to 2.1.1

HIGH
database-migrator@1.0.0
Missing required security checks in migration logic.
Violates security-guidelines.md section 3.2.
Fix: Add input validation to migration steps

MEDIUM
api-generator@3.0.0
Deprecated coding patterns in generated code.
Uses outdated Express middleware patterns.
Fix: Update to use modern async/await patterns

MEDIUM
test-runner@1.5.0
Conflicts with unit-testing@1.0.0
Both provide similar test execution functionality.
Fix: Consolidate into single testing skill

LOW
logger@2.0.0
Minor formatting issues in documentation.
Missing example usage section.
Fix: Add example usage to skill documentation

────────────────────────────────
1 critical  1 high  2 medium  1 low

Registry audit complete. Issues require maintainer action.

Full audit report saved to logs/activity.log
```

### Example 4: Clean Audit (No Issues)

**User request:**
> Hey Bob, audit

**Expected output:**
```
Running skill audit...

Scanning 6 installed skills...

✅ No issues found. All skills are healthy!

All skills are:
  ✅ Up to date
  ✅ Compatible with each other
  ✅ Within your tier level
  ✅ Compliant with policies
  ✅ Have all dependencies met

────────────────────────────────
0 critical  0 high  0 medium  0 low

Audit report saved to logs/activity.log
```

---

## Notes

- Audits should be run regularly to maintain skill ecosystem health
- The --fix flag only auto-fixes CRITICAL and HIGH issues for safety
- MEDIUM and LOW issues require manual review and decision-making
- Registry audits help maintainers identify systemic issues
- Activity logs provide audit trail for compliance purposes
- Tier restrictions help ensure developers use appropriate skill complexity
- Conflict detection prevents redundant or incompatible skills

## Warnings

> ⚠️ Auto-fix (--fix flag) will uninstall skills that violate tier restrictions without confirmation.

> ⚠️ Security vulnerabilities marked as CRITICAL should be addressed immediately.

> ⚠️ Registry audits require maintainer permissions to fix identified issues.

> ⚠️ Skill conflicts may require manual decision on which skill to keep.

> ⚠️ Always review the activity log after running audit --fix to verify changes.

## Related Skills

- Update Skills - Update skills to fix vulnerabilities
- List Skills - View all installed skills and their status
- Uninstall Skill - Remove problematic or conflicting skills
- Validate Skill - Check individual skill compliance
- Search Skills - Find replacement skills for deprecated ones