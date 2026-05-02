# Custom @ Command - Automated Skill Creation Pipeline

Automates the complete skill creation and validation pipeline by chaining together the skill-generator and validate-skill workflows. This Context Mention handles everything from initial skill generation through final validation, ensuring all company standards are met before the skill is ready for use.

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Trigger

Invoke this Context Mention by typing:
```
@custom-at-command execute [Skill Name]
```

Example:
```
@custom-at-command execute REST API Endpoint Generator
```

---

## Prerequisites

- Access to `skillet-policies/.bob/skills/skill-generator.md`
- Access to `skillet-policies/.bob/skills/validate-skill.md`
- Access to the `rules/` directory (skill-format.md, security-guidelines.md, coding-standards.md)
- Access to the `skillet-skills/skills/` directory for duplicate detection
- Clear understanding of the skill requirements from the user

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `skill_name` | string | Yes | The name of the skill to create (provided in the Context Mention) |
| `skill_requirements` | object | Yes | User's description of what the skill should do (gathered through conversation) |

---

## Steps

### Step 1: Load Skill Generator Workflow

First, use the `read_file` tool to load the skill-generator workflow:
```
skillet-policies/.bob/skills/skill-generator.md
```

Study the complete workflow to understand all 8 steps that must be executed.

### Step 2: Execute Skill Generation (Steps 1-6 from skill-generator.md)

Then, execute the skill generation workflow exactly as defined in skill-generator.md:

**Step 2.1: Read All Policy Files**
- Use `read_file` to read all three policy files together:
  - `skillet-policies/rules/skill-format.md`
  - `skillet-policies/rules/security-guidelines.md`
  - `skillet-policies/rules/coding-standards.md`

**Step 2.2: Analyze User Requirements**
- Review the user's request for the skill
- Ask follow-up questions using `ask_followup_question` if requirements are unclear
- Gather all necessary information about inputs, steps, and outputs

**Step 2.3: Structure the Skill Content**
- Organize content following the exact structure from skill-format.md
- Include all required sections: Title, Description, Category, Roles, Prerequisites, Inputs, Steps, Outputs, Example Usage, Notes, Warnings, Related Skills
- Ensure steps use imperative language with specific tool names

**Step 2.4: Apply Security Guidelines**
- Review each step against security-guidelines.md
- Replace hardcoded credentials with environment variables
- Ensure SQL queries use parameterized queries
- Include input validation
- Add error handling
- Verify no sensitive data is logged

**Step 2.5: Apply Coding Standards**
- Ensure code examples follow coding-standards.md
- Use ES6+ syntax
- Follow naming conventions
- Include proper error handling
- Add JSDoc comments for complex functions

**Step 2.6: Write the Skill File**
- Use `write_to_file` to create the skill at `skillet-skills/skills/{category}-{skill-name}.md`
- Use kebab-case for filename
- Include all sections with complete content
- Ensure file is under 500 lines

### Step 3: Load Validation Workflow

Next, use the `read_file` tool to load the validation workflow:
```
skillet-policies/.bob/skills/validate-skill.md
```

Study the complete validation process with all 4 checks.

### Step 4: Execute Validation (Steps 1-7 from validate-skill.md)

After that, execute the validation workflow exactly as defined in validate-skill.md:

**Step 4.1: Read the Skill Content**
- Use `read_file` to load the newly created skill file
- Parse all sections for analysis

**Step 4.2: Run Check #1 - Guideline Compliance (0-100)**
- Evaluate Structure & Format (25 points)
- Evaluate Content Quality (35 points)
- Evaluate Standards Compliance (25 points)
- Evaluate Example & Documentation (15 points)
- Calculate total score and determine level (Excellent/Good/Fair/Poor)

**Step 4.3: Run Check #2 - Duplicate Detection (0-100%)**
- Use `list_files` to get all existing skills in `skillet-skills/skills/`
- Use `read_file` to read existing skills for comparison
- Calculate similarity based on:
  - Purpose Similarity (40%)
  - Steps Similarity (30%)
  - Inputs/Outputs Similarity (20%)
  - Category Match (10%)
- Identify most similar skill and similarity percentage
- Determine recommendation (Unique/Review/Reject)

**Step 4.4: Run Check #3 - Security Review (Pass/Fail)**
- Check for hardcoded credentials
- Check for SQL injection risks
- Check for XSS vulnerabilities
- Check for insecure file operations
- Check for exposed secrets
- Verify input validation is present
- Determine Pass/Fail status

**Step 4.5: Run Check #4 - Completeness Check (Pass/Fail)**
- Verify all required sections are present
- Check for placeholder text or empty sections
- Ensure steps are numbered and actionable
- Verify inputs include types
- Verify outputs are specific
- Determine Pass/Fail status

**Step 4.6: Generate Validation Report**
- Compile all results into structured report format
- Calculate verdict based on logic:
  - ✅ APPROVED: Compliance >= 70, Duplicate < 70%, Security PASSED, Completeness PASSED
  - ⚠️ NEEDS REVIEW: Compliance 50-69, OR Duplicate 50-89%, OR minor security issues
  - ❌ REJECTED: Compliance < 50, OR Duplicate >= 90%, OR Security FAILED, OR Completeness FAILED

**Step 4.7: Provide Recommendations**
- Based on verdict, provide specific guidance
- List concrete issues and actionable steps

### Step 5: Handle Validation Results

Finally, based on the validation verdict from Step 4:

**If APPROVED (✅)**:
- Present the complete validation report to the user
- Confirm the skill file path: `skillet-skills/skills/{category}-{skill-name}.md`
- Indicate the skill is ready to use
- Use `attempt_completion` with success message

**If NEEDS REVIEW (⚠️)**:
- Present the validation report with all identified issues
- Ask the user: "Would you like me to fix these issues now, or proceed anyway?"
- If user wants fixes:
  - Use `apply_diff` to make corrections to the skill file
  - Re-run validation (repeat Step 4)
  - Present updated results
- If user wants to proceed:
  - Use `attempt_completion` with warning message

**If REJECTED (❌)**:
- Present the validation report with all critical issues
- List required actions to fix each issue
- Attempt automatic fixes using `apply_diff` where possible
- Re-run validation after fixes (repeat Step 4)
- Do not use `attempt_completion` until validation passes or user explicitly overrides

---

## Outputs

- `skillet-skills/skills/{category}-{skill-name}.md` — Complete, validated skill file
- Validation Report — Detailed report showing:
  - Guideline Compliance Score (0-100)
  - Duplicate Detection Result (0-100% similarity)
  - Security Review Status (Pass/Fail)
  - Completeness Check Status (Pass/Fail)
  - Final Verdict (APPROVED/NEEDS REVIEW/REJECTED)
  - Specific issues and recommendations

---

## Example Usage

**User Context Mention:**
```
@custom-at-command execute React Component Generator
```

**Follow-up conversation:**
```
Bob: What should this skill do? Please describe the requirements.

User: Create a skill for generating React components with TypeScript, 
props validation, and unit tests. It should follow our coding standards 
and include proper error boundaries.

Bob: [Executes skill-generator workflow Steps 1-6]
     [Creates skillet-skills/skills/frontend-react-component-generator.md]
     [Executes validate-skill workflow Steps 1-7]
```

**Expected output:**
```
🤖 Skillet Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skill: React Component Generator
Category: frontend
File: skillet-skills/skills/frontend-react-component-generator.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 Guideline Compliance: 92/100 ✅
   Score: 92
   Level: Excellent
   Issues found: None

🔄 Duplicate Check: 35% ✅
   Most similar: React Hook Generator (35%)
   Recommendation: Unique - Safe to add
   Overlapping features: None significant

🔒 Security Review: PASSED ✅
   Issues found: None

📝 Completeness: PASSED ✅
   Missing or incomplete: None

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Verdict: ✅ APPROVED

This skill meets all quality standards and is ready to be added 
to the Skillet library.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Skill created successfully at:
   skillet-skills/skills/frontend-react-component-generator.md

The skill is ready to use!
```

---

## Notes

- This Context Mention is a wrapper that chains two existing workflows together
- Do not modify the logic in skill-generator.md or validate-skill.md
- All 8 steps from skill-generator must be executed in order
- All 7 steps from validate-skill must be executed in order
- Validation is mandatory and cannot be skipped
- The Context Mention should feel seamless to the user—they only see the final result
- If validation fails, attempt automatic fixes before asking the user

---

## Warnings

> ⚠️ This Context Mention executes two complex workflows sequentially. Do not skip any steps from either workflow.

> ⚠️ Always wait for user confirmation after creating the skill file before starting validation.

> ⚠️ Never approve a skill that fails security validation, even if the user requests it.

> ⚠️ If duplicate detection shows >70% similarity, always inform the user and get their decision before proceeding.

---

## Related Skills

- Skill Generator (executed in Step 2)
- Validate Skill (executed in Step 4)
- Code Review Automation
- Documentation Generator