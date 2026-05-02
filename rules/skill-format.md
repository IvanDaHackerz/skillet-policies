# Skill Format for Bob (Markdown)

This format is used to generate new reusable skills for Bob. All generated skills must be in Markdown (`.md`) format and follow this structure.

---

## Format Requirements

- **Format**: Markdown
- **File Extension**: `.md`
- **Maximum Length**: 500 lines
- **Encoding**: UTF-8

---

## Markdown Structure

```
# [Skill Name]

[2-3 sentence description]

**Category:** [backend|frontend|devops|qa|shared]
**Roles:** [backend, frontend, fullstack, devops, qa]

---

## Prerequisites

- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `input_name` | string | Yes | What this input is used for |
| `input_name_2` | number | No | What this input is used for |

---

## Steps

### Step 1: [Short Title]

First, [imperative instruction]. [Specific command or tool to use].
[Expected outcome or what to verify].

### Step 2: [Short Title]

Next, [imperative instruction]. [Specific command or tool to use].
[Expected outcome or what to verify].

### Step 3: [Short Title]

Then, [imperative instruction]. [Specific command or tool to use].
[Expected outcome or what to verify].

---

## Outputs

- `path/to/output.ext` — Description of what this file contains and its purpose
- `path/to/output2.ext` — Description of what this file contains and its purpose

---

## Example Usage

**User request:**
> [Concrete example of what the user would ask for]

**Expected output:**
- `file1.js` — Brief description
- `file2.js` — Brief description

---

## Notes

- [Optional: Additional tip or context]
- [Optional: Performance consideration]

## Warnings

> ⚠️ [Important caveat to watch out for]

> ⚠️ [Common pitfall to avoid]

## Related Skills

- [Name of related skill]
- [Name of another related skill]
```

---

## Step Writing Guidelines

Each step MUST be written as **imperative, specific instructions** — not as a high-level workflow overview.

### ❌ Wrong (Workflow Overview Style)

```markdown
### Step 1: Analyze CSS
Analyze the CSS file.

### Step 2: Update Styles
Update styles.
```

### ✅ Correct (Imperative Instruction Style)

```markdown
### Step 1: Read the CSS File

First, run the `read_file` tool on the CSS file at the path provided by the user.
Look for existing color variables, font definitions, and layout patterns.
Note any naming conventions used for CSS classes.

### Step 2: Apply New Theme Colors

Next, use the `apply_diff` tool to update the CSS file.
Replace the old color values with the new theme colors specified by the user.
Ensure all hex codes are lowercase and use shorthand notation where possible.
```

### Key Characteristics of Good Steps

1. **Start with action words** — "First, run...", "Next, use...", "Then, execute...", "Finally, verify..."
2. **Specify exact tools** — "use the `read_file` tool", "run `npm test` with the `execute_command` tool"
3. **Include what to look for** — "Look for existing patterns", "Verify that all tests pass"
4. **Mention expected outcomes** — "This will create a new file", "The output should show no errors"
5. **Be specific about parameters** — "at the path provided by the user", "with the `--recursive` flag"

---

## Description Guidelines

The description (first paragraph) is critical — it helps Bob decide when to trigger this skill.

| | Example |
|-|---------|
| ✅ Clear | "Generates a complete REST API endpoint with authentication, validation, and tests. Use this when adding a new authenticated endpoint following company standards." |
| ❌ Vague | "Helps with API stuff." |

Your description should:
1. **State what the skill does** — be specific about the output
2. **Specify when to use it** — name the trigger scenario
3. **Mention key features** — validation, error handling, test coverage, etc.

---

## Validation Checklist

Before finalizing a generated skill, verify:

- [ ] File is valid Markdown (`.md`)
- [ ] File is under 500 lines
- [ ] Skill name is clear and descriptive
- [ ] Description is 2–3 sentences and helps Bob decide when to trigger it
- [ ] `Category` is one of: `backend`, `frontend`, `devops`, `qa`, `shared`
- [ ] `Roles` contains only valid values: `backend`, `frontend`, `fullstack`, `devops`, `qa`
- [ ] At least 2 prerequisites listed
- [ ] At least 3 steps, each using imperative instructions
- [ ] Each step names the exact tool or command to use
- [ ] Inputs table defines all required inputs with types
- [ ] Outputs list all files that will be created
- [ ] Example usage is concrete and realistic
- [ ] No hardcoded credentials or secrets
- [ ] Follows company coding standards

---

## Complete Example

````markdown
# REST API Endpoint Generator

Generates a complete REST API endpoint with authentication middleware, input validation
using Zod, error handling, and comprehensive tests. Use this when you need to add a new
API endpoint following company security and coding standards. The generated endpoint will
be production-ready with proper error responses and test coverage.

**Category:** backend
**Roles:** backend, fullstack

---

## Prerequisites

- Express.js project with existing route structure in `routes/` directory
- Zod installed as a dependency (`npm install zod`)
- Authentication middleware available at `middleware/auth.js`
- Test framework (Jest or Mocha) configured in `package.json`

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `resource_name` | string | Yes | Name of the resource (e.g., `users`, `orders`) |
| `http_method` | string | Yes | HTTP method: GET, POST, PUT, DELETE, or PATCH |
| `required_fields` | array | Yes | List of required fields and types, e.g. `[{name: 'email', type: 'string'}]` |
| `optional_fields` | array | No | List of optional fields and types |
| `requires_auth` | boolean | No | Whether the endpoint requires authentication (default: `true`) |

---

## Steps

### Step 1: Analyze Existing Routes

First, run the `list_files` tool on the `routes/` directory to analyze existing route files.
Look for naming conventions, file structure patterns, and how routes are organized.
Note the import patterns used for middleware and controllers.

### Step 2: Read Existing Route Files

Next, use the `read_file` tool to read 2–3 existing route files together.
Examine how authentication middleware is applied, how validation schemas are structured,
and how error handling is implemented. This ensures the new file is consistent with the
existing codebase.

### Step 3: Create the Route File

Then, use the `write_to_file` tool to create a new route file at `routes/{resource}.js`.
Include the authentication middleware import, define the Zod validation schema for the
request body, and implement the route handler with `try/catch` error handling.
Follow the patterns observed in Step 2.

### Step 4: Create the Controller File

After that, use the `write_to_file` tool to create a controller file at
`controllers/{resource}Controller.js`. Implement the business logic with proper error
handling, input sanitization, and response formatting. Include JSDoc comments for all functions.

### Step 5: Register the New Route

Next, use the `read_file` tool to check the main router file (usually `app.js` or
`routes/index.js`). Then use the `apply_diff` tool to add the new route import and
registration, maintaining alphabetical order if that is the existing pattern.

### Step 6: Create Tests

Then, use the `write_to_file` tool to create a test file at `tests/{resource}.test.js`.
Include test cases for: successful requests, validation errors, authentication failures,
and edge cases. Use the same test structure as existing test files.

### Step 7: Verify All Tests Pass

Finally, run the `execute_command` tool with `npm test` to verify all tests pass.
If any tests fail, use the `read_file` tool to examine the error output and use
`apply_diff` to fix the issues. Confirm that the new endpoint is working correctly.

---

## Outputs

- `routes/{resource}.js` — Express route file with authentication, validation, and error handling
- `controllers/{resource}Controller.js` — Controller with business logic and proper error handling
- `tests/{resource}.test.js` — Comprehensive unit tests covering success and error cases

---

## Example Usage

**User request:**
> Generate a POST endpoint for creating orders with required fields:
> `customerId` (string), `items` (array), `totalAmount` (number).
> Include authentication and validation.

**Expected output:**
- `routes/orders.js` — POST route with auth middleware and Zod validation
- `controllers/ordersController.js` — `createOrder` function with business logic
- `tests/orders.test.js` — Tests for success, validation errors, and auth failures

---

## Notes

- Always maintain consistency with existing code patterns in the project
- Use Zod for validation to ensure type safety and clear error messages
- Include JSDoc comments for better IDE support and documentation

## Warnings

> ⚠️ Ensure authentication middleware is properly configured before using this skill.

> ⚠️ Test the endpoint manually after generation to verify it works with your database.

> ⚠️ Review generated validation schemas to ensure they match your business requirements.

## Related Skills

- Database Model Generator
- API Documentation Generator
- Integration Test Generator
````

---

## Notes for Skill Generation

1. **Keep it under 500 lines** — be concise but complete
2. **Use imperative steps** — every step tells Bob exactly what to do
3. **Write a helpful description** — this is how Bob decides to use the skill
4. **Be specific about tools** — name exact Bob tools (`read_file`, `apply_diff`, etc.)
5. **Include realistic examples** — show concrete use cases, not abstract ones
6. **Think about prerequisites** — what must exist before this skill can run?
7. **Define clear outputs** — what files or changes will be made?
8. **Use the checklist** — validate before finalizing