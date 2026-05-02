# Search Skills

Searches the Skillet skill registry to find skills matching keywords, categories, or tags. Use this when a developer wants to discover available skills, find solutions to specific problems, or browse skills by category. The search engine ranks results by relevance, popularity, and recency while filtering by the developer's tier level.

**Category:** shared
**Roles:** backend, frontend, fullstack, devops, qa

---

## Prerequisites

- Access to `skill-index.json` in the skillet-skills repository
- Understanding of the developer's current tier level (Junior, Mid, Senior)
- Access to skill metadata including install counts and ratings

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `keyword` | string | Yes | Search term to match against skill names, descriptions, categories, and tags (e.g., `authentication`, `api`, `testing`) |
| `category` | string | No | Filter by specific category: `backend`, `frontend`, `devops`, `qa`, or `shared` |
| `tier` | string | No | Filter by tier requirement: `junior`, `mid`, or `senior` (defaults to developer's current tier) |
| `limit` | number | No | Maximum number of results to display (default: 5) |

---

## Steps

### Step 1: Read skill-index.json from GitHub

Use the MCP tool `read_skill_file` to fetch `metadata/skill-index.json` from the skillet-skills repository.
This file contains the complete registry of all available skills with their metadata.
The file must be accessed via the GitHub MCP server since it may not be available locally.
Parse the JSON to access the `skills` array.

### Step 2: Extract Search Parameters

Parse the user's search query to identify:
- The main keyword or search term
- Optional category filter (if `--category` flag is present)
- Optional tier filter (if `--tier` flag is present)
- The developer's current tier level from their profile

If no keyword is provided, display an error message with usage examples.

### Step 3: Filter Skills by Tier

Before searching, filter the skills array to only include skills that match or are below the developer's tier level.
Tier hierarchy: Junior < Mid < Senior
- Junior developers see only Junior-tier skills
- Mid developers see Junior and Mid-tier skills
- Senior developers see all skills

If a specific tier filter is provided via `--tier`, use that instead of the developer's tier.

### Step 4: Match Keywords Against Skills

For each skill in the filtered list, calculate a relevance score by matching the keyword against:
- Skill name (highest weight: 40 points)
- Skill description (medium weight: 30 points)
- Skill tags (medium weight: 20 points)
- Skill category (low weight: 10 points)

Use case-insensitive matching and partial word matching.
Calculate a total relevance score for each skill (0-100 points).

### Step 5: Apply Category Filter

If a category filter is specified (e.g., `--category security`), filter the results to only include skills in that category.
Valid categories: `backend`, `frontend`, `devops`, `qa`, `shared`.
If an invalid category is provided, display an error with valid category options.

### Step 6: Rank Results

Sort the filtered and scored skills by:
1. Relevance score (highest priority)
2. Install count (popular skills ranked higher)
3. Rating score (higher ratings ranked higher)
4. Last updated date (most recent ranked higher)

Select the top N results based on the `limit` parameter (default: 5).

### Step 7: Format and Display Results

For each result in the top N, format the output as:
```
{skill-name}@{version}
{one-line description}
⭐ {rating}  📥 {install-count} installs  🔓 {tier-requirement}
→ "Hey Bob, install {skill-slug}"
```

Include a header showing the search query and result count.
Add a blank line between each result for readability.

### Step 8: Handle No Results

If no skills match the search criteria, display a helpful message:
- Confirm what was searched for
- Suggest similar keywords based on common skill categories
- Recommend using `--category` filter to narrow results
- Provide examples of popular skills to explore

Example suggestions:
- "Try searching for: api, database, testing, deployment"
- "Use --category to filter: backend, frontend, devops, qa, shared"

---

## Outputs

- Formatted search results showing top matching skills
- Each result includes: name, version, description, rating, install count, tier requirement, and install command
- Helpful suggestions if no results are found

---

## Example Usage

### Example 1: Basic Keyword Search

**User request:**
> Hey Bob, search authentication

**Expected output:**
```
Searching for "authentication"...

Found 4 results:

auth-setup@2.0.0
Implements secure user authentication
⭐ 4.8  📥 1,847 installs  🔓 Mid+
→ "Hey Bob, install auth-setup"

jwt-handling@1.2.0
Manages JSON Web Token generation
⭐ 4.6  📥 923 installs   🔓 Mid+
→ "Hey Bob, install jwt-handling"

basic-login@1.0.0
Simple login flow for junior devs
⭐ 4.2  📥 445 installs   🔓 Junior+
→ "Hey Bob, install basic-login"

oauth-setup@3.1.0
OAuth2 implementation guide
⭐ 4.9  📥 2,103 installs  🔓 Senior+
→ "Hey Bob, install oauth-setup"
```

### Example 2: Category Filter

**User request:**
> Hey Bob, search --category security

**Expected output:**
```
Searching in category "security"...

Found 3 results:

security-audit@1.5.0
Automated security vulnerability scanning
⭐ 4.7  📥 1,234 installs  🔓 Mid+
→ "Hey Bob, install security-audit"

input-sanitizer@2.1.0
Validates and sanitizes user input
⭐ 4.5  📥 892 installs   🔓 Junior+
→ "Hey Bob, install input-sanitizer"

encryption-helper@1.0.0
Implements data encryption patterns
⭐ 4.8  📥 1,567 installs  🔓 Senior+
→ "Hey Bob, install encryption-helper"
```

### Example 3: Tier Filter

**User request:**
> Hey Bob, search --tier junior

**Expected output:**
```
Searching for Junior-tier skills...

Found 5 results:

basic-login@1.0.0
Simple login flow for junior devs
⭐ 4.2  📥 445 installs   🔓 Junior+
→ "Hey Bob, install basic-login"

html-form-builder@1.3.0
Creates HTML forms with validation
⭐ 4.4  📥 678 installs   🔓 Junior+
→ "Hey Bob, install html-form-builder"

css-helper@2.0.0
Common CSS patterns and utilities
⭐ 4.3  📥 523 installs   🔓 Junior+
→ "Hey Bob, install css-helper"

git-basics@1.1.0
Essential git commands and workflows
⭐ 4.6  📥 891 installs   🔓 Junior+
→ "Hey Bob, install git-basics"

debug-logger@1.0.0
Simple console logging utilities
⭐ 4.1  📥 334 installs   🔓 Junior+
→ "Hey Bob, install debug-logger"
```

### Example 4: No Results Found

**User request:**
> Hey Bob, search quantum-computing

**Expected output:**
```
Searching for "quantum-computing"...

No skills found matching "quantum-computing".

Try searching for:
- api, database, testing, deployment
- authentication, security, validation
- frontend, backend, devops

Or use --category to filter:
Hey Bob, search --category backend
Hey Bob, search --category frontend
Hey Bob, search --category devops
```

---

## Notes

- Search is case-insensitive and supports partial word matching
- Skills above the developer's tier are automatically filtered out
- Results are ranked by relevance first, then popularity
- Install counts and ratings are pulled from skill metadata
- The search engine supports multiple keywords separated by spaces
- Use quotes for exact phrase matching: `search "REST API"`

## Warnings

> ⚠️ Skills above your tier level will not appear in search results. Ask your team lead about tier progression.

> ⚠️ Install counts and ratings are updated periodically and may not reflect real-time data.

> ⚠️ Some skills may have dependencies that also need to be installed. Check the skill's prerequisites before installing.

## Related Skills

- Install Skill - Install skills from search results
- List Skills - View all installed skills
- Uninstall Skill - Remove installed skills
- Skill Generator - Create new skills for the registry