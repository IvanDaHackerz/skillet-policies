# Skill Format

All skills must follow this structure to ensure consistency and quality.

---

## Required Sections

### Title
Clear, descriptive name that explains what the skill does.

**Example**: "REST API Endpoint Generator"

---

### Description
2-3 sentences explaining what the skill does and when to use it.

**Example**: 
```
Generates a complete REST API endpoint with authentication middleware, 
input validation using Zod, error handling, and tests. Use this when 
you need to add a new API endpoint following company standards.
```

---

### Category
Must be one of: `backend`, `frontend`, `devops`, `qa`, `shared`

**Example**: `backend`

---

### Roles
Comma-separated list of roles that would use this skill.

**Valid roles**: `backend`, `frontend`, `fullstack`, `devops`, `qa`

**Example**: `backend, fullstack`

---

### Prerequisites
List of requirements before using this skill:
- Dependencies that must be installed
- Existing project structure needed
- Configuration files required

**Example**:
```
- Express.js project with existing route structure
- Zod installed as dependency (`npm install zod`)
- Authentication middleware available in `middleware/auth.js`
```

---

### Steps
Numbered list of actions the skill will perform. Each step should be:
- Clear and actionable
- Specific enough to follow
- Include decision points if needed

**Minimum**: 3 steps  
**Recommended**: 5-10 steps

**Example**:
```
1. Analyze existing route files for naming and structure patterns
2. Create a new route file following existing conventions
3. Add authentication middleware import
4. Define Zod validation schema for request body
5. Implement controller with try/catch error handling
6. Add route to the main router
7. Generate unit test file with happy path and error cases
```

---

### Inputs
Information needed from the user to execute this skill.

**Format**: `Field name (type, required/optional): description`

**Example**:
```
- Resource name (string, required): e.g., "users", "orders"
- HTTP method (string, required): GET, POST, PUT, DELETE
- Required fields (array, required): List of field names and their types
- Optional fields (array, optional): List of optional field names and types
```

---

### Outputs
Files or changes the skill will create.

**Format**: `File path: description`

**Example**:
```
- routes/{resource}.js: Express route with authentication and validation
- controllers/{resource}Controller.js: Controller with business logic
- tests/{resource}.test.js: Unit tests with happy path and error cases
```

---

### Example Usage
A concrete example of how to use this skill, showing the user's request and expected outcome.

**Example**:
```
User: "Generate a POST endpoint for creating orders with fields: 
customerId (string, required), items (array, required), 
totalAmount (number, required)"

Expected Output:
- routes/orders.js (with POST route, auth middleware, Zod validation)
- controllers/ordersController.js (with createOrder function)
- tests/orders.test.js (with tests for success and validation errors)
```

---

## Optional Sections

### Notes
Additional information, tips, or context.

### Warnings
Important caveats or things to watch out for.

### Related Skills
Links to other skills that might be useful in combination.

---

## Validation Checklist

Before submitting a skill, ensure:
- ✅ All required sections are present
- ✅ Each section has meaningful content (no placeholders)
- ✅ Steps are clear and actionable (minimum 3)
- ✅ Category is valid
- ✅ Roles are valid
- ✅ Example usage is concrete and helpful
- ✅ No hardcoded credentials or secrets
- ✅ Follows company coding standards

---

## Template

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