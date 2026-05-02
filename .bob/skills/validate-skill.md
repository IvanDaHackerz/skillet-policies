# Validate Skill

## When to Use
When a skill needs to be checked against company standards before being added to the Skillet library. This is a **REQUIRED** step after creating any new skill.

---

## Steps

### 1. Read the Skill Content
- Load the skill markdown file to validate
- Parse all sections
- Store the content for analysis

### 2. Run Check #1: Guideline Compliance (Score: 0-100)

Read ALL files in the `../skillet-policies/rules/` directory:
- `../skillet-policies/rules/coding-standards.md`
- `../skillet-policies/rules/security-guidelines.md`
- `../skillet-policies/rules/skill-format.md`

**IMPORTANT**: These are immutable policy files from the read-only policies repo.

**Evaluate the skill against these criteria**:

#### Structure & Format (25 points)
- ✅ Follows the required format from `skill-format.md` (10 pts)
- ✅ All required sections are present (10 pts)
- ✅ Sections are in the correct order (5 pts)

#### Content Quality (35 points)
- ✅ Description is clear and concise (2-3 sentences) (10 pts)
- ✅ Steps are clear, actionable, and specific (15 pts)
- ✅ Prerequisites are realistic and well-defined (5 pts)
- ✅ Outputs are specific (file paths or changes) (5 pts)

#### Standards Compliance (25 points)
- ✅ Follows coding standards from `coding-standards.md` (15 pts)
- ✅ Uses proper naming conventions (5 pts)
- ✅ Includes error handling guidance (5 pts)

#### Example & Documentation (15 points)
- ✅ Example usage is concrete and helpful (10 pts)
- ✅ Inputs are well-defined with types (5 pts)

**Scoring Levels**:
- **90-100**: Excellent - Follows all guidelines perfectly
- **70-89**: Good - Minor improvements needed
- **50-69**: Fair - Several issues to address
- **Below 50**: Poor - Major revisions required

**Output**: Score (0-100) and list of issues found

---

### 3. Run Check #2: Duplicate Detection (Similarity: 0-100%)

Read ALL existing skills in the `../skillet-skills/skills/` directory.

**For each existing skill, calculate similarity based on**:

#### Purpose Similarity (40%)
- Compare descriptions
- Check if they solve the same problem
- Look for overlapping use cases

#### Steps Similarity (30%)
- Compare the steps/actions
- Check for similar workflows
- Look for identical processes

#### Inputs/Outputs Similarity (20%)
- Compare required inputs
- Compare generated outputs
- Check for matching file patterns

#### Category Match (10%)
- Same category = higher similarity
- Different category = lower similarity

**Calculate overall similarity percentage** for each existing skill.

**Find the most similar skill** and its similarity percentage.

**Similarity Levels**:
- **90-100%**: Duplicate - Likely redundant, reject or merge
- **70-89%**: High overlap - Review carefully, may need consolidation
- **50-69%**: Some overlap - Skills may coexist but review recommended
- **Below 50%**: Unique - Safe to add as new skill

**Output**: 
- Most similar skill name and similarity percentage
- Recommendation (Unique/Review/Reject)
- List of overlapping features

---

### 4. Run Check #3: Security Review (Pass/Fail)

Scan the skill content for security issues:

#### Check for Hardcoded Credentials
- ❌ API keys, tokens, passwords in examples
- ❌ Database connection strings with credentials
- ❌ Secret keys or encryption keys
- ✅ Should use `process.env.VARIABLE_NAME` instead

#### Check for SQL Injection Risks
- ❌ String concatenation in SQL queries
- ❌ Direct user input in queries
- ✅ Should use parameterized queries or ORM

#### Check for XSS Vulnerabilities
- ❌ Unescaped user input in HTML
- ❌ `dangerouslySetInnerHTML` without sanitization
- ❌ `eval()` or similar dangerous functions
- ✅ Should sanitize and escape user input

#### Check for Insecure File Operations
- ❌ Path traversal vulnerabilities
- ❌ Unrestricted file uploads
- ❌ Insecure file permissions
- ✅ Should validate and sanitize file paths

#### Check for Exposed Secrets
- ❌ Hardcoded JWT secrets
- ❌ Hardcoded encryption keys
- ❌ Hardcoded API endpoints with credentials
- ✅ Should use environment variables

#### Check for Input Validation
- ✅ Skill should mention input validation
- ✅ Should use validation libraries (Zod, Joi, etc.)
- ✅ Should validate user input before processing

**Security Checks Checklist**:
- ✅ No hardcoded passwords/tokens
- ✅ No SQL string concatenation
- ✅ No eval() or dangerous functions
- ✅ Proper error handling (no stack traces exposed)
- ✅ Input validation present
- ✅ Uses environment variables for sensitive data

**Result**: PASSED or FAILED
- **PASSED**: All security checks passed
- **FAILED**: One or more security issues found

**Output**:
- Pass/Fail status
- List of security issues found (if any)
- Recommendations for fixes

---

### 5. Run Check #4: Completeness Check (Pass/Fail)

Verify ALL required sections from `../skillet-policies/rules/skill-format.md` are present and meaningful:

#### Required Sections Checklist
- ✅ **Title**: Present and descriptive
- ✅ **Description**: Present, 2-3 sentences, meaningful
- ✅ **Version**: Present and follows semantic versioning (e.g., 1.0.0, 1.2.3)
- ✅ **Changes made**: Present for versions > 1.0.0, lists specific changes from previous version (not required for v1.0.0)
- ✅ **Category**: Present and valid (backend/frontend/devops/qa/shared)
- ✅ **Roles**: Present and valid (from roles.json)
- ✅ **Prerequisites**: Present with at least 1 item, not placeholder
- ✅ **Steps**: Present with minimum 3 steps, clear and actionable
- ✅ **Inputs**: Present with at least 1 input, includes type
- ✅ **Outputs**: Present with at least 1 output, specific
- ✅ **Example Usage**: Present and concrete (not placeholder)

#### Content Quality Checks
- ✅ No placeholder text (e.g., "[TODO]", "[Fill this in]")
- ✅ No empty sections
- ✅ Steps are numbered and actionable
- ✅ Inputs include types (string, number, array, etc.)
- ✅ Outputs are specific (file paths or changes)

**Result**: PASSED or FAILED
- **PASSED**: All required sections present with meaningful content
- **FAILED**: Missing sections or placeholder content

**Output**:
- Pass/Fail status
- List of missing or incomplete sections
- Specific issues to fix

---

### 6. Generate Validation Report

Compile all results into a structured report:

```
🤖 Skillet Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skill: [Skill Name]
Category: [Category]
File: ../skillet-skills/skills/[category]-[skill-name].md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 Guideline Compliance: [Score]/100 [✅/⚠️/❌]
   Score: [Score]
   Level: [Excellent/Good/Fair/Poor]
   Issues found:
   [List of issues, or "None" if score >= 90]

🔄 Duplicate Check: [Similarity]% [✅/⚠️/❌]
   Most similar: [Skill Name] ([Similarity]%)
   Recommendation: [Unique/Review/Reject]
   Overlapping features:
   [List of overlaps, or "None" if similarity < 50%]

🔒 Security Review: [PASSED/FAILED] [✅/❌]
   Issues found:
   [List of security issues, or "None" if passed]

📝 Completeness: [PASSED/FAILED] [✅/❌]
   Missing or incomplete:
   [List of issues, or "All sections complete" if passed]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Verdict: [✅ APPROVED / ⚠️ NEEDS REVIEW / ❌ REJECTED]

[Detailed explanation of verdict]
[Specific actions needed if not approved]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Verdict Logic**:
- **✅ APPROVED**: 
  - Guideline Compliance >= 70
  - Duplicate Check < 70%
  - Security Review = PASSED
  - Completeness = PASSED

- **⚠️ NEEDS REVIEW**:
  - Guideline Compliance 50-69, OR
  - Duplicate Check 50-89%, OR
  - Minor security issues that can be fixed

- **❌ REJECTED**:
  - Guideline Compliance < 50, OR
  - Duplicate Check >= 90%, OR
  - Security Review = FAILED (critical issues), OR
  - Completeness = FAILED

---

### 7. Provide Recommendations

Based on the verdict, provide specific guidance:

#### If APPROVED ✅
```
Great work! This skill meets all quality standards and is ready to be 
added to the Skillet library.

Next steps:
1. Save the skill to the repository
2. Commit and push to GitHub
3. Other team members can start using it
```

#### If NEEDS REVIEW ⚠️
```
This skill has some issues that should be addressed:

[List specific issues]

Recommendations:
[Specific suggestions for improvement]

You can:
1. Fix the issues now and re-validate
2. Save anyway and fix later (not recommended)
3. Cancel and revise
```

#### If REJECTED ❌
```
This skill has critical issues that must be fixed before it can be added:

[List all critical issues]

Required actions:
[Specific steps to fix each issue]

Please revise the skill and run validation again.
```

---

## Important Notes

### Validation is Objective
- Use clear criteria for each check
- Provide specific scores and percentages
- List concrete issues, not vague feedback

### Be Thorough
- Check every section carefully
- Don't skip any validation steps
- Compare against ALL existing skills for duplicates

### Be Helpful
- Explain why something failed
- Provide specific recommendations
- Offer examples of correct implementations

### Security is Critical
- Never approve skills with security issues
- Be strict about hardcoded credentials
- Verify input validation is mentioned

---

## Example Validation Report

```
🤖 Skillet Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skill: REST API Endpoint Generator
Category: backend
File: ../skillet-skills/skills/backend-rest-api-endpoint-generator.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 Guideline Compliance: 95/100 ✅
   Score: 95
   Level: Excellent
   Issues found:
   - Minor: Could add more detail in step 4 (5 pts deducted)

🔄 Duplicate Check: 72% ⚠️
   Most similar: Express Route Builder (72%)
   Recommendation: Review - High overlap detected
   Overlapping features:
   - Both generate Express routes
   - Both include authentication
   - This skill adds: Zod validation, tests

🔒 Security Review: PASSED ✅
   Issues found: None
   - Uses environment variables for secrets
   - Includes input validation with Zod
   - No hardcoded credentials

📝 Completeness: PASSED ✅
   Missing or incomplete: None
   - All required sections present
   - All sections have meaningful content
   - 7 clear, actionable steps

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Verdict: ⚠️ NEEDS REVIEW

This skill is well-structured and secure, but has 72% similarity 
with the existing "Express Route Builder" skill. 

Recommendation: Review both skills to determine if:
1. This should replace the old skill (it's more complete)
2. Both should coexist (different use cases)
3. Features should be merged

The skill is technically sound and can be approved if the 
maintainer decides it adds sufficient value over the existing skill.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Validation Checklist

Before completing validation, verify:
- ✅ All 4 checks were run
- ✅ Scores/percentages were calculated
- ✅ Issues were listed specifically
- ✅ Verdict was determined correctly
- ✅ Recommendations were provided
- ✅ Report was formatted clearly

---

## Common Validation Scenarios

### Scenario 1: Perfect Skill
- Guideline: 95/100
- Duplicate: 15%
- Security: PASSED
- Completeness: PASSED
- **Verdict**: ✅ APPROVED

### Scenario 2: Good Skill with Minor Issues
- Guideline: 75/100
- Duplicate: 45%
- Security: PASSED
- Completeness: PASSED
- **Verdict**: ✅ APPROVED (with suggestions)

### Scenario 3: Skill Needs Work
- Guideline: 60/100
- Duplicate: 80%
- Security: PASSED
- Completeness: PASSED
- **Verdict**: ⚠️ NEEDS REVIEW (high duplication)

### Scenario 4: Critical Issues
- Guideline: 85/100
- Duplicate: 30%
- Security: FAILED (hardcoded API key)
- Completeness: PASSED
- **Verdict**: ❌ REJECTED (security issue)

---

## Success Criteria

Validation is successful when:
1. ✅ All 4 checks completed
2. ✅ Clear verdict provided
3. ✅ Specific issues identified
4. ✅ Actionable recommendations given
5. ✅ Developer understands next steps