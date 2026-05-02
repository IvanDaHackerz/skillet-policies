# Initialize Workspace

Initializes a new Bob Skills workspace with developer profile and tier-appropriate skills, similar to how npm init sets up a new project. Use this when setting up Bob for the first time or resetting a workspace. The initializer collects developer information, creates configuration files, installs appropriate skills, and generates a personalized getting started guide.

**Version:** 1.0.0

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- MCP server `skillet-github` is connected and configured in `.bob/settings/mcp_settings.json`
- Bob must be in a mode that supports MCP tools (not Code mode)
- Access to `skill-index.json` in the skillet-skills repository
- Access to `metadata/roles.json` in the skillet-policies repository
- Write access to workspace directory for creating config files
- GitHub MCP tools available (read_skill_file, read_policy_file)

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `from_sessions` | boolean | No | Set to `true` to analyze exported Bob session files and install only skills that fill knowledge gaps (default: `false`) |
| `tier` | string | No | Specify tier directly to skip questions: `junior`, `mid-level`, `senior`, `lead`, `principal` |

---

## Steps

### Step 1: Check for Existing Workspace

Use the `read_file` tool to check if a workspace config already exists at `.bob/workspace-config.json`.
If it exists, ask the user if they want to:
- Reinitialize (overwrites existing config)
- Update existing config
- Cancel initialization

Use the `ask_followup_question` tool with suggestions: "Reinitialize", "Update", "Cancel"

### Step 2: Collect Developer Information

If `tier` parameter is not provided, use the `ask_followup_question` tool to collect:

**Question 1: What is your name?**
- Suggestions: Leave blank for system username
- Store as `developer.name`

**Question 2: What is your role?**
- Suggestions: "Backend Developer", "Frontend Developer", "Full-Stack Developer", "DevOps Engineer", "QA Engineer"
- Validate against roles in `metadata/roles.json`
- Store as `developer.role`

**Question 3: What is your team?**
- Suggestions: "API Team", "Platform Team", "Product Team", "Infrastructure Team"
- Store as `developer.team`

**Question 4: What is your tier level?**
- Suggestions: "Junior", "Mid-Level", "Senior", "Lead", "Principal"
- Store as `developer.tier`

If `tier` parameter is provided, skip questions and use the provided tier.
Set other fields to defaults or prompt only for name.

### Step 3: Read Tier Configuration

Use the MCP tool `read_policy_file` to fetch `metadata/roles.json` from skillet-policies.
This file contains tier definitions with:
- Skill limits per tier
- Recommended skills for each tier
- Skills that unlock at each tier level

Parse the tier configuration for the developer's specified tier.

### Step 4: Handle Session Analysis (if --from-sessions)

If `from_sessions` is `true`, ask the user to provide their exported Bob session files.
Use the `ask_followup_question` tool: "Please provide the path to your Bob session files directory"

For each session file found:
1. Use the `read_file` tool to read the session content
2. Analyze the conversation to identify:
   - Skills the developer already knows (mentioned concepts, patterns used)
   - Technologies they're familiar with
   - Problem-solving approaches they use
   - Areas where they struggled or asked for help

3. Build a knowledge profile showing:
   - Strong areas (skills they demonstrate mastery in)
   - Weak areas (skills they need help with)
   - Gap areas (skills they haven't encountered)

4. Filter the tier's recommended skills to prioritize:
   - Gap-filling skills (address weak areas)
   - Complementary skills (build on strong areas)
   - Skip skills they already demonstrate mastery in

### Step 5: Create Workspace Configuration

Use the `write_to_file` tool to create `.bob/workspace-config.json` with:

```json
{
  "version": "1.0.0",
  "developer": {
    "name": "John Smith",
    "role": "backend",
    "team": "API Team",
    "tier": "mid-level",
    "initialized": "2026-05-02T21:00:00.000Z"
  },
  "skills": {
    "maxInstalled": 10,
    "autoUpdate": true,
    "preferredCategories": ["backend", "shared"]
  },
  "preferences": {
    "language": "en",
    "verbosity": "normal",
    "autoFix": false
  }
}
```

Adjust `maxInstalled` based on tier level.
Set `preferredCategories` based on role.

### Step 6: Determine Skills to Install

Based on the developer's tier and role, determine which skills to install:

**For Junior tier:**
- 3-5 foundational skills
- Focus on basic patterns and best practices
- Include getting-started guides

**For Mid-Level tier:**
- 5-8 intermediate skills
- Include advanced patterns
- Add productivity tools

**For Senior tier:**
- 8-12 advanced skills
- Include architecture and design skills
- Add mentoring and review tools

**For Lead/Principal tier:**
- 12-15 comprehensive skills
- Include all advanced capabilities
- Add team management and strategy skills

Filter by role to ensure relevant skills (e.g., backend developer gets backend skills).

### Step 7: Install Tier-Appropriate Skills

For each skill identified in Step 6:
1. Use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json`
2. Find the skill entry and get its source path
3. Use the MCP tool `read_skill_file` to fetch the skill content
4. Use the `write_to_file` tool to save to `skillet-skills/skills/{slug}.md`
5. Display progress: `{skill-name}@{version}  ✅`

If any installation fails, log the error but continue with remaining skills.

Display installation progress:
```
Installing Mid-Level skillset:
  auth-setup@2.0.0          ✅
  api-design@1.1.0          ✅
  error-handling@1.2.0      ✅
  jwt-handling@1.2.0        ✅
  input-validator@1.0.2     ✅
```

### Step 8: Generate Getting Started Guide

Create a personalized getting started guide based on role and tier.
Use the `write_to_file` tool to create `.bob/GETTING_STARTED.md`:

```markdown
# Getting Started with Bob Skills

Welcome, [Developer Name]!

## Your Profile
- **Role**: [Role]
- **Team**: [Team]
- **Tier**: [Tier Level]
- **Skills Installed**: [Count]

## Recommended First Steps

1. **Start with [skill-name]**
   This skill covers the patterns your team uses most.
   Run: "Hey Bob, use [skill-name]"

2. **Explore [skill-name]**
   Learn advanced techniques for [area].
   Run: "Hey Bob, use [skill-name]"

3. **Practice with [skill-name]**
   Apply what you've learned to real scenarios.
   Run: "Hey Bob, use [skill-name]"

## Your Skill Ecosystem

### Installed Skills ([count])
[List of installed skills with brief descriptions]

### Unlocks at [Next Tier] ([count] more skills)
[List of skills that unlock at next tier]

## Quick Commands

- `Hey Bob, list` - See all installed skills
- `Hey Bob, search [query]` - Find new skills
- `Hey Bob, update` - Update to latest versions
- `Hey Bob, audit` - Check skill health

## Need Help?

- Review skill documentation in `skillet-skills/skills/`
- Check activity logs in `logs/activity.log`
- Run `Hey Bob, search` to find skills for specific tasks

Happy coding!
```

Customize the content based on the developer's role and tier.

### Step 9: Display Initialization Summary

Present a formatted summary:

```
Initializing Bob Skills workspace...

What is your name? John Smith
What is your role? Backend Developer
What is your team? API Team
What is your tier? Mid-Level

Setting up your workspace...

Installing Mid-Level skillset:
  auth-setup@2.0.0          ✅
  api-design@1.1.0          ✅
  error-handling@1.2.0      ✅
  jwt-handling@1.2.0        ✅
  input-validator@1.0.2     ✅

✅ Workspace ready — 5 skills installed

Getting Started:
Start with api-design.md — it covers
the patterns your team uses most.

Unlocks at Senior tier:
  + security-audit
  + infrastructure-setup
  + architecture-patterns
  + 3 more skills

Run "Hey Bob, list" to see everything installed.
```

### Step 10: Log Initialization Activity

Use the `write_to_file` or `insert_content` tool to append to `logs/activity.log`:

```
[2026-05-02 21:00:00] WORKSPACE INITIALIZED
Developer: John Smith
Role: Backend Developer
Team: API Team
Tier: Mid-Level
Skills Installed: 5
- auth-setup@2.0.0
- api-design@1.1.0
- error-handling@1.2.0
- jwt-handling@1.2.0
- input-validator@1.0.2

Configuration saved to .bob/workspace-config.json
Getting started guide created at .bob/GETTING_STARTED.md
```

### Step 11: Display Next Steps

Show the developer what to do next:
- Location of getting started guide
- First recommended skill to try
- How to see all installed skills
- How to search for more skills
- Link to documentation

Provide clear, actionable next steps based on their role and tier.

---

## Outputs

- `.bob/workspace-config.json` - Developer profile and workspace settings
- `.bob/GETTING_STARTED.md` - Personalized getting started guide
- `skillet-skills/skills/` - Tier-appropriate skills installed
- `logs/activity.log` - Initialization activity record
- Console output with setup summary and next steps

---

## Example Usage

### Example 1: Basic Initialization

**User request:**
> Hey Bob, init

**Expected output:**
```
Initializing Bob Skills workspace...

What is your name? John Smith
What is your role? Backend Developer
What is your team? API Team
What is your tier? Mid-Level

Setting up your workspace...

Installing Mid-Level skillset:
  auth-setup@2.0.0          ✅
  api-design@1.1.0          ✅
  error-handling@1.2.0      ✅
  jwt-handling@1.2.0        ✅
  input-validator@1.0.2     ✅

✅ Workspace ready — 5 skills installed

Getting Started:
Start with api-design.md — it covers
the patterns your team uses most.

Unlocks at Senior tier:
  + security-audit
  + infrastructure-setup
  + architecture-patterns
  + 3 more skills

Run "Hey Bob, list" to see everything installed.

Configuration saved to .bob/workspace-config.json
Getting started guide: .bob/GETTING_STARTED.md
```

### Example 2: Initialize with Tier Specified

**User request:**
> Hey Bob, init --tier senior

**Expected output:**
```
Initializing Bob Skills workspace...

What is your name? Jane Doe

Setting up Senior tier workspace...

Installing Senior skillset:
  auth-setup@2.0.0              ✅
  api-design@1.1.0              ✅
  error-handling@1.2.0          ✅
  security-audit@1.0.0          ✅
  infrastructure-setup@2.1.0    ✅
  architecture-patterns@1.5.0   ✅
  code-review-automation@1.0.0  ✅
  performance-optimization@1.2.0 ✅

✅ Workspace ready — 8 skills installed

Getting Started:
As a Senior developer, start with
architecture-patterns.md to learn
system design approaches.

Unlocks at Lead tier:
  + team-management
  + strategic-planning
  + technical-leadership
  + 4 more skills

Run "Hey Bob, list" to see everything installed.
```

### Example 3: Initialize from Sessions

**User request:**
> Hey Bob, init --from-sessions

**Expected output:**
```
Initializing Bob Skills workspace...

What is your name? Alex Chen
What is your role? Full-Stack Developer
What is your team? Product Team
What is your tier? Mid-Level

Please provide the path to your Bob session files directory:
> ./bob_sessions

Analyzing 12 session files...

Knowledge Profile:
  ✅ Strong: API design, error handling
  ⚠️  Needs work: Testing, security
  ❓ Gaps: Performance optimization, caching

Installing personalized skillset:
  unit-testing@1.1.0           ✅ (fills gap)
  security-basics@1.0.0        ✅ (fills gap)
  input-validator@1.0.2        ✅ (fills gap)
  performance-optimization@1.2.0 ✅ (fills gap)
  caching-strategies@1.0.0     ✅ (fills gap)

Skipped (already proficient):
  - api-design (demonstrated in 8 sessions)
  - error-handling (demonstrated in 6 sessions)

✅ Workspace ready — 5 skills installed

Getting Started:
Based on your sessions, start with
unit-testing.md to strengthen your
testing practices.

Unlocks at Senior tier:
  + advanced-testing
  + security-audit
  + architecture-patterns
  + 3 more skills

Run "Hey Bob, list" to see everything installed.
```

---

## Notes

- Initialization creates a clean workspace with tier-appropriate skills
- The `--from-sessions` flag provides intelligent skill recommendations based on actual usage
- Tier levels determine skill limits and complexity
- Role determines which categories of skills are prioritized
- The getting started guide is personalized to the developer's profile
- Workspace config can be updated later without full reinitialization
- Activity logs track all initialization actions for audit purposes

## Warnings

> ⚠️ Reinitializing will overwrite existing workspace configuration. Back up custom settings first.

> ⚠️ Session analysis requires access to exported Bob session files in JSON format.

> ⚠️ Tier restrictions are enforced - skills above your tier won't be installed.

> ⚠️ The MCP server must be connected to fetch skills from the registry.

> ⚠️ Existing skills will not be removed during initialization - use audit to clean up.

## Related Skills

- List Skills - View all installed skills
- Install Skill - Add individual skills
- Update Skills - Update to latest versions
- Audit Skills - Check workspace health
- Search Skills - Find skills for specific needs