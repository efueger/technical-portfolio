# Pagination & Advanced Formulas

## Overview

Exploration of Tines formula patterns and pagination strategies for APIs that return results across multiple pages, including native pagination and Event Transform loops.

## What I Built

Two workflows demonstrating different approaches to handling paginated data:

**Loop Over Pages**: Native Tines pagination to automatically fetch multiple pages from GitHub API

**Loop Over Results**: Event Transform loop to iterate through array items using `LOOP.value`

## Native Tines Pagination (PAGINATE)

Tines provides built-in pagination in HTTP Request actions that automatically fetches all pages.

### Implementation

```json
{
  "url": "https://api.github.com/repos/microsoft/vscode/issues",
  "method": "get",
  "headers": {
    "Accept": "application/vnd.github.v3+json",
    "User-Agent": "Tines-Lab"
  },
  "query_params": {
    "state": "all",
    "per_page": 10,
    "page": "=PAGINATE.index + 1"
  },
  "pagination": {
    "has_more": "=SIZE(PAGINATE.previous_response.body) = 10",
    "output_as_single_event": true
  }
}
```

### How It Works

1. First request: `PAGINATE.index` = 0, so page = 1
2. After response, evaluates `has_more` formula
3. If true: increments index and fetches next page
4. Repeats until `has_more` returns false
5. Outputs single event with all accumulated results

**Key fields:**

- `PAGINATE.index` - Current iteration (0, 1, 2...)
- `PAGINATE.previous_response.body` - Last API response
- `has_more` - Formula determining if more pages exist
- `output_as_single_event: true` - Combines all pages into one event

## Event Transform Loops (LOOP)

Event Transform actions can loop over arrays using the Loop field.

### Implementation

**Event Transform Configuration:**

Mode: Message only
Loop: `=generate_page_numbers.body` (references array like `[1, 2, 3, 4, 5]`)

**Payload:**

```json
{
  "page": "=LOOP.value",
  "repo": "=initialize_variables.body.repo"
}
```

**Result:** Creates 5 separate events, one for each array element

### Loop Variables

- `LOOP.value` - Current item in the array
- `LOOP.index` - Current position (0, 1, 2...)
- `LOOP.key` - For objects, the current key

## Additional Patterns

**Explode Action**: Alternative to loops for processing array items. Creates one event per array element. Useful when you want to process items in parallel or route them differently based on content.

**CONCAT Function**: Merges multiple arrays together. Syntax: `=CONCAT(array1, array2)`. Used when accumulating results across multiple operations.

## Tines Formula Syntax

### Basic Formulas

```
=action_name.body.field         # Access data
=SIZE(array)                    # Array length
=PAGINATE.index                 # Pagination counter
=LOOP.value                     # Current loop item
```

### Math Operations

```
=field + 1                      # Addition
=field - 1                      # Subtraction
=price * quantity               # Multiplication
```

### Conditionals

```
=SIZE(array) = 10               # Equality
=SIZE(array) > 0                # Greater than
=field != null                  # Not equal
```

### Common Functions

```
=SIZE(array)                    # Array length
=DEFAULT(field, 'fallback')     # Default value
=CONCAT(array1, array2)         # Merge arrays
=UPCASE(text)                   # Uppercase
=DOWNCASE(text)                 # Lowercase
```

## Key Learnings

### 1. Native Pagination vs Event Transform Loops

**Native pagination** (PAGINATE): Best for fetching all pages from an API automatically. Tines handles the iteration and accumulation.

**Event Transform loops** (LOOP): Best for iterating over a known array when you need to perform actions for each item.

### 2. Formula Syntax Rules

- In JSON payloads: use `=` prefix for formulas
- No curly braces in formulas
- Use formula builder (click fields) to insert formula chips
- `SIZE()` for array length (not LENGTH or .length)

### 3. API Behavior Varies

Not all APIs handle pagination parameters the same way. GitHub respects `page` and `per_page` parameters correctly. Always test pagination with the specific API you're working with.

## Real-World Applications

**Data Migration**: Sync 5,000 users from old API to new platform (100/page = 50 pages). Use native pagination to fetch all automatically.

**Compliance Reporting**: Monthly report of all GitHub PRs with review status. Fetch all PRs with pagination, extract relevant fields.

**Debugging "Missing Data"**: Customer reports incomplete sync. Check if pagination is configured and `has_more` formula is correct.

## Technologies

- **Tines** - Workflow automation with native pagination
- **GitHub API** - Paginated REST API
- **Tines Formulas** - Data transformation and loop control
- **PAGINATE object** - Built-in pagination context
- **LOOP object** - Event Transform loop context

---

## Artifacts

### Screenshot

- **pagination-results.png** - Workflows showing both pagination and loop patterns with results

<img width="3004" height="1708" alt="Loops and pagination" src="https://github.com/user-attachments/assets/56a087fa-259f-4619-ac2a-7a2ebc37f6b6" />


---

## Related Labs

- [GitHub Issue Monitor](../github-issue-monitor/) - Uses formulas for data transformation
- [HMAC Authentication](../hmac-authentication/) - Advanced formula patterns for signatures
