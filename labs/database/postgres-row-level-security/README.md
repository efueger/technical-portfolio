# Postgres Row-Level Security

## Overview

Multi-tenant database architecture demonstrating Row-Level Security (RLS) in Postgres/Supabase to enforce data isolation between organizations at the database level.

## What I Built

A working multi-tenant demo with:

- Three-table schema (organizations, projects, user_org_mapping)
- RLS policies enforcing organization-level data isolation
- API authentication flow with user-specific access tokens
- Verified data isolation through API testing

**Key achievement:** Data isolation enforced by database, not application code - reduces risk of data leaks from application bugs.

## Architecture

### Schema Design

**organizations**

- Represents customer organizations (Acme Corp, Widget Inc)
- Primary entity for tenant isolation

**projects**

- Belongs to an organization via `org_id` foreign key
- Contains tenant-specific data

**user_org_mapping**

- Links authenticated users to their organizations
- Enables users to see only their org's data

### RLS Policies

**organizations table:**

```sql
-- Users can view their org
CREATE POLICY "Users can view their org"
  ON organizations FOR SELECT
  USING (id IN (
    SELECT org_id FROM user_org_mapping WHERE user_id = auth.uid()
  ));
```

**projects table:**

```sql
-- Users can view their org's projects
CREATE POLICY "Users can view their org's projects"
  ON projects FOR SELECT
  USING (org_id IN (
    SELECT org_id FROM user_org_mapping WHERE user_id = auth.uid()
  ));
```

**How it works:**

- `auth.uid()` returns authenticated user's ID from JWT
- Policy checks if user is mapped to the organization
- Postgres automatically adds WHERE clause to all queries
- No application code needed - database enforces isolation

## Testing & Verification

**Authentication flow:**

```bash
# Get access token for user
curl -X POST 'https://[project].supabase.co/auth/v1/token?grant_type=password' \
  -H 'apikey: [anon-key]' \
  -H 'Content-Type: application/json' \
  -d '{"email":"user1@acme.com","password":"[password]"}'
```

**Query with user token:**

```bash
# User sees only their org's projects
curl 'https://[project].supabase.co/rest/v1/projects' \
  -H 'apikey: [anon-key]' \
  -H 'Authorization: Bearer [user-access-token]'
```

**Verified:**

- user1@acme.com sees only Acme projects (3 projects)
- user2@widget.com sees only Widget projects (3 projects)
- No cross-organization data leakage

## Troubleshooting

### Issue: Query returns empty array (no rows)

**Symptom:** Authenticated user queries projects but gets `[]`

**Causes:**

1. User not mapped in `user_org_mapping` table
2. Using anon key instead of user JWT in Authorization header
3. RLS policy too restrictive

**Resolution:**

```sql
-- Verify user is mapped to an organization
SELECT * FROM user_org_mapping WHERE user_id = '[user-uuid]';

-- If missing, add mapping
INSERT INTO user_org_mapping (user_id, org_id) 
VALUES ('[user-uuid]', '[org-uuid]');
```

Common mistake: Creating users in Supabase Auth but forgetting to map them to organizations in `user_org_mapping`.

### Issue: Seeing all rows (RLS not enforcing)

**Cause:** Overly permissive policy or using anon key with public read access

**Resolution:** Remove blanket public policies, ensure queries use user JWT tokens

## Key Learnings

### 1. Database-Level Security

RLS enforces data isolation at query time by Postgres, independent of application code. Benefits:

- Eliminates entire class of data leak vulnerabilities
- No risk from application bugs bypassing access checks
- Single source of truth for authorization

### 2. Performance Considerations

RLS policies add WHERE clauses to every query. For production:

- Index foreign keys (`org_id` in projects table)
- Index mapping table columns (`user_id`, `org_id`)
- Test query plans with `EXPLAIN ANALYZE`

Without indexes, RLS policy requires full table scan for every query.

### 3. User Mapping Critical

Most common issue: User authenticates successfully but sees no data. Root cause: Missing entry in `user_org_mapping`. 

**Production pattern:** Automated user provisioning must include:

1. Create user in auth system
2. Insert mapping to organization
3. Both must succeed or neither (transaction)

## Real-World Applications

**SaaS Multi-Tenancy:** Ensure Customer A never sees Customer B's data, even if application has a bug. Database prevents leaks.

**Compliance Requirements:** HIPAA, SOC 2, GDPR all require data isolation. RLS provides technical control that auditors can verify.

**Customer Support Troubleshooting:** Customer reports "can't see their data." Check RLS policies, verify user mapping, confirm JWT contains correct user_id. More reliable than debugging application-level permissions.

**Migration Safety:** Moving from shared to dedicated infrastructure? RLS policies ensure data stays isolated during transition period.

## Technologies

- **Postgres** - Row-Level Security feature
- **Supabase** - Postgres hosting with built-in authentication
- **PostgREST** - Automatic REST API from Postgres schema
- **JWT** - Token-based authentication with user context

---

## Artifacts

### Screenshots

- **rls-policies-config** - Supabase Policies page showing SELECT policies configured for all three tables
- **data-isolation-tests** - API test results showing different users receiving only their organization's data



---

## Related Labs

- [Kubernetes](../kubernetes/) - Container orchestration and debugging
- [Workflow Automation](../workflow-automation/) - Event-driven automation and API integration
