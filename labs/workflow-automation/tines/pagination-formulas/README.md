# Pagination & Advanced Formulas

## Overview

Exploration of Tines formula patterns for data transformation and pagination strategies for APIs that return results across multiple pages.

## What I Built

### Part A: Formula Playground

Testing environment for Tines formula patterns including conditionals, date formatting, string manipulation, and loops.

### Part B: Pagination Implementation

Workflow that fetches paginated GitHub issues and processes each issue individually using the Explode pattern.

## Tines Formula Patterns

### Basic Data Access

```
=action_name.body.field              # Access nested field
=items[0].title                      # Array access
=SIZE(items)                         # Array length
```

### Conditionals (in text fields)

```
{{#IF(status == 'active')}}
  User is active
{{#ELSE()}}
  User is inactive
{{/IF}}
```

### Date Formatting

```
=FORMAT_DATE(created_at, '%B %d, %Y')   # January 15, 2024
=FORMAT_DATE(timestamp, '%Y-%m-%d')     # 2024-01-15
```

### String Operations

```
=UPCASE(email)                      # EMILY@EXAMPLE.COM
=DOWNCASE(text)                     # lowercase
=DEFAULT(field, 'N/A')             # Default if empty
```

### Math Operations

```
=MULTIPLY(price, quantity)          # Multiplication
=ADD(total, 5)                      # Addition
=SUBTRACT(count, 1)                 # Subtraction
```

### Loops (in text fields)

```
{{#EACH(items)}}
  {{=item.name}}{{#UNLESS(last)}}, {{/UNLESS}}
{{/EACH}}
```

## Pagination Strategies

### The Problem

APIs limit response sizes (e.g., GitHub returns 30 issues per page). To get all results:

1. Make initial request with `?page=1&per_page=10`
2. Check if more results exist
3. Fetch subsequent pages
4. Continue until complete

### Three Approaches

**1. Fixed Iterations**  
If you know dataset size (e.g., ≤100 items = 10 pages max)

**2. Check for Empty Response**  
Keep fetching until API returns empty array

**3. Use Link Headers**  
GitHub provides `rel="next"` in response headers when more pages exist (most robust)

## The Explode Pattern

**Traditional approach:** Loop through array items sequentially

**Tines approach:** Explode action creates one event per array item

```
API returns: [issue1, issue2, issue3]
       ↓ Explode
Creates: event1(issue1), event2(issue2), event3(issue3)
```

**Why use Explode:**

- Process items in parallel
- Each item can follow different workflow path
- One item failing doesn't stop others
- More efficient than sequential loops

### Implementation

```
Initialize Variables → Fetch Issues (page 1) → Explode Array → Process Each Issue
```

Each issue becomes separate event that can be processed independently.

## Key Learnings

### 1. Explode vs Traditional Loops

Coming from programming background, I initially tried traditional while loops. Tines doesn't use loops - it uses Explode for array processing.

**Benefits:**

- No loop counter management
- Parallel processing
- Cleaner error handling
- More intuitive visual representation

### 2. Formula Debugging

When formulas don't resolve (show as `{{.action.field}}`):

- Check action names match (display name vs slug)
- Verify actions connected on canvas
- Test with simple formula first (`{{.action.body}}`)
- Use autocomplete (type `{{.` and wait)

### 3. Pagination Strategy Selection

**Choose based on:**

- Dataset growth expectations
- API capabilities (provides total_pages? Link headers?)
- Performance requirements
- Error handling needs

For production: Use Link headers when available (most robust)

## Real-World Applications

**Data Migration:** Sync 5,000 users from old API to new platform (100/page = 50 pages)

**Compliance Reporting:** Monthly report of all GitHub PRs with review status

**"Missing Data" Debugging:** Customer says "workflow only processed 30 items but I have 500" - pagination not implemented

## Tines Formula Quick Reference

**In JSON payloads (Plain code tab):**

```
"field": "=action_name.body.field"
"field": "=DEFAULT(field, 'N/A')"
"field": "=UPCASE(text)"
"field": "=FORMAT_DATE(timestamp, '%Y-%m-%d')"
"field": "=SIZE(array)"
"field": "=ADD(num, 5)"
```

**In text fields (for string interpolation):**

```
{{=action_name.body.field}}
{{=DEFAULT(field, 'N/A')}}
{{#IF(condition)}}...{{#ELSE()}}...{{/IF}}
{{#EACH(array)}}{{=item.field}}{{/EACH}}
```

**Common functions:**

- `DEFAULT(value, fallback)` - Default if empty
- `UPCASE(text)` / `DOWNCASE(text)` - Case conversion
- `FORMAT_DATE(date, format)` - Date formatting
- `SIZE(array)` - Array/string length
- `ADD()` / `SUBTRACT()` / `MULTIPLY()` - Math operations
- `CONCAT()` - String concatenation

## Technologies

- **Tines** - Workflow automation platform with formula language
- **Tines Formulas** - Data transformation syntax (not Liquid/Shopify)
- **GitHub API** - Pagination examples
- **Explode Pattern** - Array processing

---

## Artifacts


## Screenshots to Add

- [ ] **formula-playground.png** - Various formula patterns with test data and outputs
- [ ] **pagination-workflow.png** - Workflow showing Initialize → Fetch → Explode → Process
- [ ] **explode-events.png** - Events view showing multiple events from single array

---

## Related Labs

- [GitHub Issue Monitor](../tines/github-issue-monitor/) - Uses formulas for data transformation
- [HMAC Authentication](../hmac-authentication/) - Advanced formula patterns for signatures
