# GitHub Issue Monitor

## Overview

Event-driven workflow that monitors GitHub issues in real-time, enriches them with API data, transforms using Liquid templates, and sends formatted Slack notifications with error handling.

## What I Built

**Workflow:** GitHub Webhook → Fetch Issue Details → Transform Data → Send to Slack + Error Handler

**Components:**

1. Webhook trigger receiving GitHub issue events
2. HTTP Request to GitHub API for full issue details
3. Event Transform using Liquid templates for data extraction
4. Slack notification with formatted message
5. Error detection and alerting

## Implementation

### Webhook Trigger

Receives GitHub issue events in real-time. Tines provides a unique webhook URL that GitHub calls when issues are created/updated.

### API Integration

```
URL: https://api.github.com/repos/{owner}/{repo}/issues/{number}
Headers:
  Accept: application/vnd.github.v3+json
  User-Agent: Tines-Learning-Lab
```

### Data Transformation (Tines Formulas)

**Event Transform Payload (Plain code tab):**

```json
{
  "issue_number": "=get_issue_details.body.number",
  "title": "=get_issue_details.body.title",
  "author": "=get_issue_details.body.user.login",
  "url": "=get_issue_details.body.html_url",
  "created_at": "=get_issue_details.body.created_at",
  "summary": "=CONCAT('Issue #', get_issue_details.body.number, ': ', get_issue_details.body.title, ' by @', get_issue_details.body.user.login)"
}
```

**Slack Message (text field with interpolation):**

```
🐛 New GitHub Issue #{{=format_issue_summary.body.issue_number}}

Title: {{=format_issue_summary.body.title}}
Author: @{{=format_issue_summary.body.author}}
Created: {{=format_issue_summary.body.created_at}}

View issue: {{=format_issue_summary.body.url}}
```

**Key patterns:**

- **In JSON/Payload:** `"=action_name.body.field"` (equals sign, no curlies)
- **In text fields:** `{{=action_name.body.field}}` (curlies for interpolation)
- **Builder tab:** Click fields to auto-generate correct syntax
- Use `CONCAT()` function for string concatenation

### Error Handling

Configured HTTP action to "emit events on both success and failure", then added Trigger action to detect non-200 status codes and route to error notification.

## Tines vs n8n

**Similar patterns:**

- Webhook triggers
- API calls
- Data transformation
- Conditional routing

**Key difference:**

- **Tines:** Formula language with `=` prefix
- **n8n:** JavaScript functions

**Example comparison:**

```
// Tines (in JSON payload)
"email": "=DEFAULT(api_response.body.user.email, 'No email')"

// Tines (in text field)
Email: {{=DEFAULT(api_response.body.user.email, 'No email')}}

// n8n (JavaScript)
$json.api_response.body.user.email || "No email"
```

Both accomplish the same goal with different syntax.

## Key Learnings

### 1. GitHub API URLs

Web URLs (`github.com/owner/repo/issues/18`) return HTML. API URLs (`api.github.com/repos/owner/repo/issues/18`) return JSON. Always use API endpoints in integrations.

### 2. Formula Resolution

Formulas showing as literal text meant actions weren't properly connected on canvas. Verify data flow between actions.

### 3. Error Detection is Critical

Without explicit error handling, failed API calls silently stop the workflow. Configure actions to emit events on failure and detect errors downstream.

## Real-World Applications

**Automated Triage:** Route issues to different Slack channels based on labels/priority  
**SLA Monitoring:** Alert on-call engineers immediately for high-priority issues  
**Integration Debugging:** Customer's GitHub→Slack integration broke - troubleshoot webhook delivery, API credentials, data transformation

## Technologies

- **Tines** - Workflow automation platform with formula-based transformations
- **GitHub API** - Issue tracking and webhooks
- **Slack API** - Formatted messaging
- **Tines Formulas** - Data transformation and string interpolation

---

## Artifacts

### Screenshots

- [ ] **workflow-canvas.png** - Complete workflow showing all connected actions
- [ ] **events-view.png** - Successful execution with data flowing through actions
- [ ] **slack-notification.png** - Formatted Slack message with issue details

<img width="1414" height="1550" alt="workflow-canvas" src="https://github.com/user-attachments/assets/e11273eb-703f-4248-94ac-783986c5bfbf" />

<img width="2706" height="1708" alt="events-view" src="https://github.com/user-attachments/assets/234edede-1ed5-4a1b-a20b-6970395859d1" />

<img width="2748" height="1516" alt="slack-notification" src="https://github.com/user-attachments/assets/d6d812b0-bd72-4c25-a97f-81a1d049642a" />


---

## Related Labs

- [n8n GitHub Issue Bot](../../n8n/github-issue-bot/) - Similar workflow using JavaScript
- [Pagination & Formulas](../pagination-formulas/) - Advanced Liquid template patterns
