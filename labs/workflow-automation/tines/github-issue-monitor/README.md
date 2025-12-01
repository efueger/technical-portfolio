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

### Data Transformation (Liquid Templates)

```liquid
{
  "issue_number": "{{.get_issue_details.body.number}}",
  "title": "{{.get_issue_details.body.title}}",
  "author": "{{.get_issue_details.body.user.login}}",
  "url": "{{.get_issue_details.body.html_url}}",
  "created_at": "{{.get_issue_details.body.created_at}}",
  "summary": "Issue #{{.get_issue_details.body.number}}: {{.get_issue_details.body.title}} by @{{.get_issue_details.body.user.login}}"
}
```

**Key patterns:**

- `{{.action_name.body.field}}` - Access nested JSON data
- Formula chaining for summary generation
- Clean data structure for downstream actions

### Error Handling

Configured HTTP action to "emit events on both success and failure", then added Trigger action to detect non-200 status codes and route to error notification.

## Tines vs n8n

**Similar patterns:**

- Webhook triggers
- API calls
- Data transformation
- Conditional routing

**Key difference:**

- **Tines:** Liquid templates (declarative, JSON-based)
- **n8n:** JavaScript functions (imperative, code-based)

**Example comparison:**

```liquid
// Tines (Liquid)
{{.api_response.body.user.email | default: "No email"}}

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

- **Tines** - Workflow automation platform
- **GitHub API** - Issue tracking and webhooks
- **Slack API** - Formatted messaging
- **Liquid Templates** - Shopify's template language

---

## Artifacts

### Screenshots

- [x] **workflow-canvas.png** - Complete workflow showing all connected actions
- [x] **events-view.png** - Successful execution with data flowing through actions
- [x] **slack-notification.png** - Formatted Slack message with issue details

<img width="1414" height="1550" alt="workflow-canvas" src="https://github.com/user-attachments/assets/26cee7cd-a4a5-4f82-be64-d1779cfba100" />


<img width="2706" height="1708" alt="events-view" src="https://github.com/user-attachments/assets/3c6852fb-9ee8-4434-a22d-ef06441b362b" />


<img width="2748" height="1516" alt="slack-notification" src="https://github.com/user-attachments/assets/388de7f3-c803-4fb9-9fb6-821a2e6fb8af" />



---

## Related Labs

- [n8n GitHub Issue Bot](../../n8n/github-issue-bot/) - Similar workflow using JavaScript
- [Pagination & Formulas](../tines/pagination-formulas/) - Advanced Liquid template patterns
