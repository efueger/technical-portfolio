# API Health Monitor with Notifications

## Overview

Multi-step API orchestration workflow using n8n that monitors endpoint health, implements error handling, generates aggregated reports, and logs results to Google Sheets for audit trails.

## What I Built

A production-ready monitoring workflow demonstrating API orchestration, conditional alerting, and data persistence.

**Workflow:**

```
Schedule Trigger (every 15 min) → 3 Parallel HTTP Checks → Normalize Results → 
Merge → Conditional Logic → Alert (if unhealthy) / Continue (if healthy) → 
Summary Report → Google Sheets Logging (2 tabs)
```

**APIs monitored:**

- GitHub API
- JSONPlaceholder (public test API)
- Tines webhook (from Lab 1.1)

**Key capabilities:**

- Parallel API calls for efficiency
- Error handling with "Continue on Error"
- Conditional branching based on health status
- Aggregated summary with health percentage
- Dual logging (summary per run + detail per API)

## Implementation

### Parallel HTTP Checks

Three HTTP Request nodes run simultaneously from Manual/Schedule trigger:

**GitHub API:**

```
Method: GET
URL: https://api.github.com/zen
Response Format: Text
Options: Include Response Headers enabled
```

**JSONPlaceholder API:**

```
Method: GET
URL: https://jsonplaceholder.typicode.com/posts/1
Response Format: JSON
```

**Tines Webhook:**

```
Method: POST
URL: <tines-webhook-url>
Body: { "source": "n8n health check", "timestamp": "={{ $now.toISO() }}" }
```

### Result Normalization (Code Nodes)

Each HTTP check followed by Code node that normalizes output to consistent schema:

```javascript
const error = $json.error;

if (error) {
  return {
    json: {
      api: 'GitHub API',
      status: 'error',
      url: 'https://api.github.com/zen',
      is_healthy: false,
      timestamp: new Date().toISOString(),
      message: error.message || 'API request failed'
    }
  };
}

return {
  json: {
    api: 'GitHub API',
    status: 200,
    url: 'https://api.github.com/zen',
    is_healthy: true,
    timestamp: new Date().toISOString(),
    message: 'API responding normally'
  }
};
```

**Why normalize:** Enables consistent downstream processing regardless of API response format or error type.

### Error Handling

HTTP Request nodes configured with:

- **Settings → On Error:** "Continue (using error output)"

This prevents workflow failure when APIs are down. Error items routed to Code nodes with `$json.error` populated.

### Conditional Alerting

**Merge node:** Combines all normalized results into single array

**IF node:** Checks `is_healthy` field

- Expression: `{{ $json.is_healthy === false }}`
- **True branch:** Routes unhealthy APIs to alert
- **False branch:** All healthy, continue to summary

**Alert options:**

- Slack webhook with formatted message blocks
- Email via Gmail node
- Any HTTP endpoint for incident management systems

### Aggregation Summary

Code node (Run Once for All Items) generates health metrics:

```javascript
const items = $input.all();
const total = items.length;
const healthy = items.filter(item => item.json.is_healthy).length;
const unhealthy = total - healthy;

return [{
  json: {
    total_apis_checked: total,
    healthy_count: healthy,
    unhealthy_count: unhealthy,
    health_percentage: Math.round((healthy / total) * 100),
    timestamp: new Date().toISOString(),
    apis: items.map(item => ({
      name: item.json.api,
      status: item.json.is_healthy ? 'healthy' : 'unhealthy',
      url: item.json.url
    }))
  }
}];
```

### Google Sheets Logging

**Summary tab** (one row per workflow run):

- Columns: timestamp, total_apis_checked, healthy_count, unhealthy_count, health_percentage
- Tracks overall health trends over time

**Per API tab** (one row per API per run):

- Columns: timestamp, api_name, status, is_healthy, url, message
- Detailed audit trail for debugging specific API issues

Both use Google Sheets "Append Row" operation with column mapping to expressions.

## Key Learnings

### 1. Error Handling Patterns

**"Continue on Error"** crucial for monitoring workflows. Without it, one failed API stops entire workflow. With it, failures become data points.

**Advantages:**

- Workflow completes even when APIs are down
- Error information captured and logged
- Alerts triggered for actual failures
- No manual intervention needed

### 2. Code Node Modes

**"Run Once for Each Item"** vs **"Run Once for All Items":**

**Each Item:** Process API responses individually (normalization)
**All Items:** Aggregate across all responses (summary generation)

Correct mode selection determines whether you process items individually or as a batch.

### 3. Parallel vs Sequential Execution

Connecting multiple nodes to same trigger creates parallel execution. All HTTP checks run simultaneously, reducing total execution time from 3× sequential to 1× parallel.

**When to use parallel:**

- Independent operations (API health checks)
- Time-sensitive workflows
- No dependencies between operations

**When to use sequential:**

- Output of one step needed as input to next
- Rate limiting concerns
- Ordered processing required

### 4. OAuth for Google Sheets

Initial setup requires:

- Google Cloud project with Sheets API enabled
- OAuth consent screen configuration
- Test user addition (yourself)
- OAuth client credentials (Client ID/Secret)
- n8n credential configuration with OAuth flow

Once configured, credential reusable across all workflows.

## Real-World Applications

**SLA Monitoring**: Track uptime percentage for customer-facing APIs. Alert on-call when health drops below SLA threshold. Historical data in Sheets proves compliance.

**Dependency Mapping**: Monitor all third-party APIs your product depends on. Proactively identify and communicate outages before customers report issues.

**Post-Incident Analysis**: Google Sheets log provides timeline of when APIs became unhealthy, how long they stayed down, and recovery patterns. Useful for root cause analysis and vendor discussions.

**Customer Troubleshooting**: Customer reports intermittent errors. Check "Per API" tab for patterns - is one API consistently slower? Are failures correlated with time of day?

## Technologies

- **n8n** - Workflow automation with visual editor
- **JavaScript** - Custom transformation logic in Code nodes
- **Google Sheets API** - Data persistence and audit trails
- **HTTP APIs** - GitHub, JSONPlaceholder, Tines webhooks
- **OAuth 2.0** - Secure Google Sheets authentication

---

## Artifacts

### Screenshots

- **API-health-monitor.png** - Complete workflow showing all nodes: triggers, HTTP checks, Code nodes, Merge, IF, Alert, Summary, and Google Sheets nodes
- **google-sheets-logs.png** - Both tabs (Summary and Per API) showing logged data with timestamps, health metrics, and API details

<img width="2866" height="1090" alt="API-health-monitor" src="https://github.com/user-attachments/assets/bb260359-3a6e-4553-917e-307aca956e63" />

<img width="2364" height="1810" alt="google-sheet-logs" src="https://github.com/user-attachments/assets/0e09fae4-29f4-4b95-872f-ba7e53f605e6" />


---

## Related Labs

- [GitHub Issue Bot](../workflow-automation/n8n/github-issue-bot/) - n8n webhook handling and data transformation
- [API Monitor](../workflow-automation/n8n/api-monitor/) - Basic n8n HTTP monitoring
- [Tines GitHub Monitor](../workflow-automation/tines/github-issue-monitor/) - Cross-platform workflow comparison
