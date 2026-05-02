# Coding Standards

These are the company's coding conventions that all skills must follow and enforce.

---

## General Principles

### 1. Code Readability
- Write code that is easy to read and understand
- Use meaningful variable and function names
- Keep functions small and focused (single responsibility)
- Add comments for complex logic, not obvious code

### 2. DRY (Don't Repeat Yourself)
- Extract repeated code into reusable functions
- Use constants for repeated values
- Create utility functions for common operations

### 3. KISS (Keep It Simple, Stupid)
- Prefer simple solutions over complex ones
- Avoid premature optimization
- Write code that others can easily maintain

---

## JavaScript / Node.js Standards

### ES6+ Syntax
- **Always use ES6+ features** (const, let, arrow functions, destructuring, etc.)
- **Prefer `const`** over `let`, never use `var`
- **Use arrow functions** for callbacks and short functions
- **Use template literals** for string interpolation
- **Use async/await** over callbacks or raw promises

**Good**:
```javascript
const getUserById = async (id) => {
  const user = await User.findById(id);
  return user;
};
```

**Bad**:
```javascript
var getUserById = function(id, callback) {
  User.findById(id, function(err, user) {
    callback(err, user);
  });
};
```

### Naming Conventions
- **Variables and functions**: camelCase (`getUserData`, `isValid`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRIES`, `API_BASE_URL`)
- **Classes**: PascalCase (`UserController`, `DatabaseService`)
- **Files**: kebab-case (`user-controller.js`, `auth-middleware.js`)
- **React Components**: PascalCase files (`UserCard.jsx`, `LoginForm.jsx`)

### File Organization
```
src/
├── routes/          # API routes
├── controllers/     # Business logic
├── services/        # External services, database
├── middleware/      # Express middleware
├── models/          # Data models
├── utils/           # Utility functions
├── config/          # Configuration files
└── tests/           # Test files
```

---

## Error Handling

### Always Handle Errors Explicitly
- Use try/catch for async operations
- Never swallow errors silently
- Provide meaningful error messages
- Log errors with context

**Good**:
```javascript
const createUser = async (userData) => {
  try {
    const user = await User.create(userData);
    return { success: true, data: user };
  } catch (error) {
    logger.error('Failed to create user:', { error: error.message, userData });
    throw new AppError('User creation failed', 500);
  }
};
```

**Bad**:
```javascript
const createUser = async (userData) => {
  try {
    const user = await User.create(userData);
    return user;
  } catch (error) {
    // Silent failure
  }
};
```

### Custom Error Classes
Use custom error classes for different error types:

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
  }
}
```

---

## API Design

### RESTful Conventions
- **GET** `/api/resources` - List all
- **GET** `/api/resources/:id` - Get one
- **POST** `/api/resources` - Create
- **PUT** `/api/resources/:id` - Update (full)
- **PATCH** `/api/resources/:id` - Update (partial)
- **DELETE** `/api/resources/:id` - Delete

### Response Format
Always return consistent response format:

```javascript
// Success
{
  "success": true,
  "data": { ... }
}

// Error
{
  "success": false,
  "error": {
    "message": "Error description",
    "code": "ERROR_CODE"
  }
}
```

### Status Codes
- **200** - OK (successful GET, PUT, PATCH)
- **201** - Created (successful POST)
- **204** - No Content (successful DELETE)
- **400** - Bad Request (validation error)
- **401** - Unauthorized (authentication required)
- **403** - Forbidden (insufficient permissions)
- **404** - Not Found
- **500** - Internal Server Error

---

## Input Validation

### Always Validate User Input
- Use validation libraries (Zod, Joi, etc.)
- Validate at the route level
- Provide clear validation error messages
- Never trust client-side validation alone

**Example with Zod**:
```javascript
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2).max(50)
});

const validateCreateUser = (req, res, next) => {
  try {
    createUserSchema.parse(req.body);
    next();
  } catch (error) {
    res.status(400).json({
      success: false,
      error: { message: 'Validation failed', details: error.errors }
    });
  }
};
```

---

## Testing Standards

### Test Structure (AAA Pattern)
- **Arrange**: Set up test data and conditions
- **Act**: Execute the code being tested
- **Assert**: Verify the results

```javascript
describe('UserController', () => {
  describe('createUser', () => {
    it('should create a new user with valid data', async () => {
      // Arrange
      const userData = { email: 'test@example.com', password: 'password123' };
      
      // Act
      const result = await createUser(userData);
      
      // Assert
      expect(result.success).toBe(true);
      expect(result.data.email).toBe(userData.email);
    });
  });
});
```

### Test Coverage
- Write tests for happy paths
- Write tests for error cases
- Write tests for edge cases
- Aim for >80% code coverage

### Test Naming
Use descriptive test names that explain what is being tested:

```javascript
it('should return 400 when email is invalid')
it('should create user when all fields are valid')
it('should throw error when database connection fails')
```

---

## Security Best Practices

### Environment Variables
- **Never hardcode** credentials, API keys, or secrets
- Use `.env` files for local development
- Use environment variables in production
- Add `.env` to `.gitignore`

```javascript
// Good
const apiKey = process.env.API_KEY;

// Bad
const apiKey = 'sk-1234567890abcdef';
```

### Authentication & Authorization
- Use JWT tokens for stateless authentication
- Hash passwords with bcrypt (min 10 rounds)
- Implement rate limiting
- Use HTTPS in production

### SQL Injection Prevention
- **Always use parameterized queries**
- Never concatenate user input into SQL strings
- Use ORM/query builders (Sequelize, Prisma, etc.)

```javascript
// Good
const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);

// Bad
const user = await db.query(`SELECT * FROM users WHERE id = ${userId}`);
```

---

## Code Comments

### When to Comment
- **Complex algorithms**: Explain the "why", not the "what"
- **Business logic**: Explain business rules
- **Workarounds**: Explain why a workaround is needed
- **TODOs**: Mark incomplete or temporary code

### When NOT to Comment
- **Obvious code**: Don't comment self-explanatory code
- **Redundant comments**: Don't repeat what the code says

**Good**:
```javascript
// Calculate discount based on customer tier and purchase amount
// Tier 1: 5%, Tier 2: 10%, Tier 3: 15%
const discount = calculateTierDiscount(customer.tier, amount);
```

**Bad**:
```javascript
// Set x to 5
const x = 5;
```

---

## Git Commit Messages

### Format
```
type(scope): subject

body (optional)

footer (optional)
```

### Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, etc.)
- **refactor**: Code refactoring
- **test**: Adding or updating tests
- **chore**: Maintenance tasks

### Examples
```
feat(auth): add JWT token refresh endpoint

fix(users): resolve email validation bug

docs(readme): update installation instructions
```

---

## Performance Guidelines

### Database Queries
- Use indexes for frequently queried fields
- Avoid N+1 queries (use joins or eager loading)
- Paginate large result sets
- Cache frequently accessed data

### API Performance
- Implement caching (Redis, in-memory)
- Use compression (gzip)
- Optimize payload size
- Implement rate limiting

---

## Code Review Checklist

Before submitting code for review, ensure:
- ✅ Code follows naming conventions
- ✅ No hardcoded credentials or secrets
- ✅ Error handling is implemented
- ✅ Input validation is present
- ✅ Tests are written and passing
- ✅ Code is properly formatted
- ✅ Comments explain complex logic
- ✅ No console.log statements in production code
- ✅ Dependencies are up to date
- ✅ Documentation is updated

---

## Tools & Linting

### ESLint Configuration
Use ESLint with Airbnb or Standard style guide:

```json
{
  "extends": ["airbnb-base"],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "prefer-const": "error"
  }
}
```

### Prettier Configuration
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

---

## Summary

These standards ensure:
- **Consistency** across the codebase
- **Maintainability** for future developers
- **Security** against common vulnerabilities
- **Quality** through testing and validation
- **Performance** through best practices

All skills created in Skillet must follow and enforce these standards.