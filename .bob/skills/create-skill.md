# Create Skill

## When to Use
When a developer wants to create a new reusable skill for the team to add to the Skillet library.

---

## Steps

### 1. Understand the Requirement
Ask the developer:
- What should this skill do?
- What problem does it solve?
- What inputs does it need?
- What outputs should it produce?
- Which role(s) would use this skill?

### 2. Read Required Documentation
Before creating the skill, read these files from the **skillet-policies** repo to understand the standards:
- `../skillet-policies/rules/skill-format.md` - Required structure and sections
- `../skillet-policies/rules/coding-standards.md` - Company coding conventions
- `../skillet-policies/rules/security-guidelines.md` - Security best practices

**IMPORTANT**: These files are in the read-only policies repo and cannot be modified.

### 3. Check for Duplicate Skills
- Read ALL existing skills in the `../skillet-skills/skills/` directory
- Check if a similar skill already exists
- If found, ask the developer if they want to:
  - Use the existing skill
  - Enhance the existing skill
  - Create a new skill anyway (with justification)

### 4. Determine Category and Roles
Based on the skill's purpose, determine:
- **Category**: One of `backend`, `frontend`, `devops`, `qa`, `shared`
- **Roles**: Comma-separated list from `backend`, `frontend`, `fullstack`, `devops`, `qa`

Reference `../skillet-policies/metadata/roles.json` for valid categories and roles.

### 5. Generate Structured Skill Markdown
Create a complete skill file following the EXACT format from `skill-format.md`:

**IMPORTANT**: Use the template from `../skillet-policies/rules/skill-format.md` (lines 156-189).

The skill MUST include these sections in this order:

1. **Title** (H1): Clear, descriptive name
2. **Description**: 2-3 sentences explaining what it does and when to use it
3. **Category**: One of: backend, frontend, devops, qa, shared
4. **Roles**: Comma-separated list (backend, frontend, fullstack, devops, qa)
5. **Prerequisites**: List of requirements (dependencies, project structure, config files)
6. **Steps**: Numbered list of actions (minimum 3, recommended 5-10)
7. **Inputs**: Information needed from user (format: "Field name (type, required/optional): description")
8. **Outputs**: Files or changes created (format: "File path: description")
9. **Example Usage**: Concrete example showing user request and expected outcome
10. **Notes** (optional): Additional information, tips, or context

**Template to follow**:
```markdown
# [Skill Title]

## Description
[2-3 sentences]

## Category
[backend/frontend/devops/qa/shared]

## Roles
[comma-separated list]

## Prerequisites
- [Requirement 1]
- [Requirement 2]

## Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Inputs
- [Input 1] (type, required/optional): description
- [Input 2] (type, required/optional): description

## Outputs
- [File path 1]: description
- [File path 2]: description

## Example Usage
[Concrete example]

## Notes (optional)
[Additional information]
```

### 6. Ensure Quality Standards
Verify the skill meets these criteria:
- ✅ All required sections are present
- ✅ Description is clear and concise (2-3 sentences)
- ✅ Category is valid
- ✅ Roles are valid
- ✅ Prerequisites are realistic and specific
- ✅ Steps are clear, actionable, and numbered (minimum 3)
- ✅ Inputs are well-defined with types
- ✅ Outputs are specific (file paths or changes)
- ✅ Example usage is concrete and helpful
- ✅ No hardcoded credentials or secrets
- ✅ Follows security guidelines
- ✅ Follows coding standards

### 7. Automatically Validate the Skill
**IMPORTANT**: After generating the skill, automatically run the `validate-skill` meta-skill on it.

Do NOT skip this step. Validation is mandatory.

### 8. Show Validation Report
Present the validation report to the developer with:
- Guideline Compliance score (0-100)
- Duplicate Detection results (similarity %)
- Security Review status (Pass/Fail)
- Completeness Check status (Pass/Fail)
- Overall verdict (Approved/Needs Review/Rejected)

### 9. Handle Validation Results

**If APPROVED (all checks passed)**:
- Congratulate the developer
- Save the skill to `../skillet-skills/skills/{category}-{skill-name}.md`
- Provide next steps for committing to GitHub

**If NEEDS REVIEW (some issues found)**:
- Explain the issues clearly
- Ask if the developer wants to:
  - Fix the issues now
  - Save anyway and fix later
  - Cancel skill creation

**If REJECTED (critical issues)**:
- Explain why it was rejected
- List all issues that must be fixed
- Ask if the developer wants to:
  - Revise and try again
  - Cancel skill creation

### 10. Save the Skill
If the developer approves (and validation passed or they chose to save anyway):
- Save to `../skillet-skills/skills/{category}-{skill-name}.md`
- Use kebab-case for the filename with category prefix (e.g., `backend-rest-api-endpoint-generator.md`)
- Confirm the file was saved successfully

**IMPORTANT**: Skills are saved to the **skillet-skills** repo, NOT the policies repo.

### 11. Provide Git Instructions
After saving, provide clear instructions:

```
✅ Skill created successfully!

Next steps to add it to the Skillet library:

1. Commit the skill:
   cd skillet-skills
   git add skills/{category}-{skill-name}.md
   git commit -m "feat: add {category}-{skill-name} skill"

2. Push to GitHub:
   git push origin main

   OR create a branch for review:
   git checkout -b skill/{skill-name}
   git push origin skill/{skill-name}
   # Then create a Pull Request on GitHub

3. Other team members can pull the update:
   git pull origin main
```

---

## Important Notes

### Validation is Mandatory
- **NEVER** skip the validation step
- Always run `validate-skill` after generating the skill
- Show the full validation report to the developer

### Quality Over Speed
- Take time to create a well-structured skill
- Ask clarifying questions if needed
- Ensure all sections have meaningful content

### Security First
- Never include hardcoded credentials
- Always use environment variables for sensitive data
- Follow security guidelines strictly

### Be Helpful
- Provide clear explanations
- Offer suggestions for improvement
- Guide the developer through the process

---

## Example Interaction

```
Developer: "Hey Bob, I want to create a skill for generating REST API 
endpoints with authentication and validation."

Bob: "Great! Let me help you create that skill. I need some information:

1. What inputs should this skill need from the developer?
2. What files should it generate?
3. Should it work with any specific framework or library?
4. Which roles would use this skill?"

Developer: "It should work with Express.js, use Zod for validation, 
and generate route, controller, and test files. Backend and fullstack 
developers would use it."

Bob: [Reads rules files]
     [Checks for existing similar skills]
     [Generates structured skill markdown]
     [Automatically validates the skill]
     
Bob: "I've created the REST API Endpoint Generator skill. Here's the 
validation report:

🤖 Skillet Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Guideline Compliance: 95/100 ✅
🔄 Duplicate Check: 15% ✅ (Unique)
🔒 Security Review: PASSED ✅
📝 Completeness: PASSED ✅

Verdict: ✅ APPROVED

The skill is ready to be added to the library! Should I save it?"

Developer: "Yes, save it"

Bob: [Saves to skills/backend-rest-api-endpoint-generator.md]
     
Bob: "✅ Skill saved successfully!

Next steps:
1. cd skillet-skills
2. git add skills/backend-rest-api-endpoint-generator.md
3. git commit -m 'feat: add backend-rest-api-endpoint-generator skill'
4. git push origin main"
```

---

## Validation Checklist

Before considering a skill complete, verify:
- ✅ All required sections present
- ✅ Validation was run automatically
- ✅ Validation report was shown to developer
- ✅ Developer approved the skill
- ✅ File was saved to correct location
- ✅ Git instructions were provided

---

## Common Mistakes to Avoid

❌ **Don't skip validation** - It's mandatory
❌ **Don't use placeholder content** - All sections must be meaningful
❌ **Don't forget to check for duplicates** - Avoid redundancy
❌ **Don't include secrets** - Use environment variables
❌ **Don't save without approval** - Always get developer confirmation

---

## Success Criteria

A skill creation is successful when:
1. ✅ Skill is well-structured and complete
2. ✅ Validation passed (or developer acknowledged issues)
3. ✅ File is saved to correct location
4. ✅ Developer knows how to commit to GitHub
5. ✅ Skill follows all company standards