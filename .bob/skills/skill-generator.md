# Skill Generator

Generates production-ready, reusable skills for Bob that follow company standards for format, security, and coding practices. Use this when you need to create a new skill that developers can use to automate common development tasks. The generated skill will be validated automatically against all company policies and guidelines.

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- Access to the `rules/` directory containing skill-format.md, security-guidelines.md, and coding-standards.md
- Access to the `.bob/skills/` directory containing validate-skill.md
- Understanding of the task or workflow the skill should automate
- Clear definition of inputs, outputs, and steps required

---

## Inputs

| Name                 | Type   | Required | Description                                                                       |
| -------------------- | ------ | -------- | --------------------------------------------------------------------------------- |
| `skill_name`         | string | Yes      | Descriptive name for the skill (e.g., "REST API Endpoint Generator")              |
| `skill_description`  | string | Yes      | 2-3 sentence description of what the skill does and when to use it                |
| `category`           | string | Yes      | One of: backend, frontend, devops, qa, shared                                     |
| `target_roles`       | array  | Yes      | List of roles that will use this skill (backend, frontend, fullstack, devops, qa) |
| `prerequisites`      | array  | Yes      | List of requirements that must exist before the skill can run                     |
| `inputs_definition`  | array  | Yes      | List of input parameters with name, type, required flag, and description          |
| `steps_definition`   | array  | Yes      | List of steps with titles and detailed instructions                               |
| `outputs_definition` | array  | Yes      | List of files or changes that will be created                                     |
| `example_usage`      | object | Yes      | Concrete example with user request and expected output                            |

---

## Steps

### Step 1: Read All Policy Files

First, use the `read_file` tool to read all three policy files together:

- `rules/skill-format.md`
- `rules/security-guidelines.md`
- `rules/coding-standards.md`

Study the format requirements, security checklist, and coding conventions that must be followed.
Note the required sections, validation criteria, and step-writing guidelines.

### Step 2: Analyze User Requirements

Next, review the user's request to understand:

- What problem the skill solves
- What inputs are needed
- What steps are required
- What outputs will be generated

Ask follow-up questions using the `ask_followup_question` tool if any requirements are unclear or missing.
Ensure you have all necessary information before proceeding.

### Step 3: Structure the Skill Content

Then, organize the skill content following the exact structure from `skill-format.md`:

- Title (clear and descriptive)
- Description (2-3 sentences, explains what and when)
- **Version** (semantic versioning: 1.0.0 for new skills)
- **Changes made** (required for versions > 1.0.0, lists specific changes from previous version; omit for v1.0.0)
- Category and Roles
- Prerequisites (at least 2 items)
- Inputs table (with name, type, required, description)
- Steps (minimum 3, using imperative instructions)
- Outputs (specific file paths or changes)
- Example Usage (concrete and realistic)
- Notes (optional tips)
- Warnings (optional caveats)
- Related Skills (optional)

Ensure each step uses imperative language ("First, run...", "Next, use...", "Then, execute...") and specifies exact tools or commands.
For new skills, always start with version **1.0.0** (no "Changes made" section needed).
When updating existing skills to a new version, always include a "Changes made" section listing all modifications.

### Step 4: Apply Security Guidelines

Next, review each step and code example against `security-guidelines.md`:

- Replace any hardcoded credentials with `process.env.VARIABLE_NAME`
- Ensure SQL queries use parameterized queries or ORMs
- Include input validation using Zod or similar libraries
- Add error handling with try/catch blocks
- Verify no sensitive data is logged
- Include authentication and authorization checks where needed

Add security warnings in the Warnings section if the skill involves sensitive operations.

### Step 5: Apply Coding Standards

Then, ensure all code examples follow `coding-standards.md`:

- Use ES6+ syntax (const, let, arrow functions, async/await)
- Follow naming conventions (camelCase for variables, PascalCase for classes)
- Include proper error handling
- Use meaningful variable names
- Add JSDoc comments for complex functions
- Follow RESTful conventions for API endpoints
- Include input validation examples

Verify that generated code examples are production-ready and follow best practices.

### Step 6: Write the Skill File with Version

After that, use the `write_to_file` tool to create the skill file at `.bob/skills/{category}-{skill-name}.md`.
Use kebab-case for the filename (e.g., `backend-rest-api-generator.md`).
Include all sections with complete, meaningful content—no placeholders or TODOs.
**Always include version 1.0.0 for new skills** in the format: `**Version:** 1.0.0`
Place the version line immediately after the description and before Category/Roles.
**For skill updates (version > 1.0.0)**, include a "Changes made" section immediately after the version line with a bulleted list of all modifications.
Ensure the file is under 500 lines and uses UTF-8 encoding.

### Step 7: Validate the Generated Skill

Finally, use the `read_file` tool to load `.bob/skills/validate-skill.md` and follow its validation process:

**Run Check #1: Guideline Compliance (0-100)**

- Evaluate structure & format (25 points)
- Evaluate content quality (35 points)
- Evaluate standards compliance (25 points)
- Evaluate example & documentation (15 points)
- Calculate total score

**Run Check #2: Duplicate Detection (0-100%)**

- Read all existing skills in `.bob/skills/` directory
- Compare purpose, steps, inputs/outputs, and category
- Calculate similarity percentage with most similar skill
- Determine if unique, needs review, or is duplicate

**Run Check #3: Security Review (Pass/Fail)**

- Check for hardcoded credentials
- Check for SQL injection risks
- Check for XSS vulnerabilities
- Check for insecure file operations
- Check for exposed secrets
- Verify input validation is present

**Run Check #4: Completeness Check (Pass/Fail)**

- Verify all required sections are present
- Check for placeholder text or empty sections
- Ensure steps are numbered and actionable
- Verify inputs include types
- Verify outputs are specific

Generate a validation report with scores, issues, and verdict (APPROVED/NEEDS REVIEW/REJECTED).

### Step 8: Update skill-index.json with Version

If the skill is APPROVED, use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json`.
Parse the JSON and add the new skill entry with:

- All standard metadata (name, slug, category, roles, description, etc.)
- **version**: "1.0.0" (for new skills)
- source information (if applicable)
- prerequisites and tags

Then use the MCP tool `update_skill_metadata` to write the updated skill-index.json back to GitHub.
Include a commit message like "Add {skill-name} v1.0.0 to registry".

### Step 9: Handle Validation Results

Then, based on the validation verdict:

**If APPROVED (✅)**:

- Present the skill file path and validation report to the user
- Confirm the skill is ready to use
- Use `attempt_completion` to finalize

**If NEEDS REVIEW (⚠️)**:

- Present the validation report with specific issues
- Ask the user if they want to fix issues now or proceed anyway
- If fixing, use `apply_diff` to make corrections and re-validate
- If proceeding, use `attempt_completion` with warnings

**If REJECTED (❌)**:

- Present the validation report with critical issues
- List required actions to fix each issue
- Use `apply_diff` to fix issues automatically if possible
- Re-run validation after fixes
- Do not complete until validation passes or user explicitly overrides

---

## Outputs

- `.bob/skills/{category}-{skill-name}.md` — Complete skill file following all format and security guidelines
- Validation report showing compliance scores and any issues found

---

## Example Usage

**User request:**

> Create a skill for generating React components with TypeScript, props validation, and unit tests. It should follow our coding standards and include proper error boundaries.

**Expected output:**

- `.bob/skills/frontend-react-component-generator.md` — Complete skill with:
  - Clear description of when to use it
  - Prerequisites (React, TypeScript, Jest setup)
  - Input parameters (component name, props, state requirements)
  - 7 detailed steps using imperative instructions
  - Security considerations (XSS prevention, input sanitization)
  - Example usage with concrete component request
  - Validation report showing 90+ compliance score

---

## Notes

- Always read all three policy files together in Step 1 for efficiency
- Use specific tool names in steps (read_file, apply_diff, execute_command, etc.)
- Include realistic code examples that developers can actually use
- Make steps actionable—tell Bob exactly what to do, not just what to achieve
- Validation is mandatory—never skip Step 7

## Warnings

> ⚠️ Generated skills must pass security validation before being approved. Never include hardcoded credentials or unsafe code patterns.

> ⚠️ Skills with similarity > 70% to existing skills should be reviewed carefully to avoid duplication.

> ⚠️ All steps must use imperative instructions, not workflow descriptions. Review skill-format.md for correct step-writing style.

## Related Skills

- Validate Skill
- Code Review Automation
- Documentation Generator
