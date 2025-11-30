# API Monitor Workflow

Automated monitoring system that checks multiple API endpoints in parallel and routes alerts based on health status.

## Overview

This workflow demonstrates scheduled execution, parallel API calls, and conditional alerting - common patterns in production monitoring systems.

**Execution frequency:** Every 5 minutes (configurable)

**APIs monitored:**

- GitHub API (`https://api.github.com/users/github`)
- httpbin (`https://httpbin.org/get`)

------

## Workflow Architecture

<img width="3014" height="1892" alt="API Monitor - workflow structure" src="https://github.com/user-attachments/assets/bf00f5f8-ae90-45c9-ad8a-2c484d54c1d6" />


### Nodes

1. **Schedule Trigger** - Runs every 5 minutes
2. **Check GitHub API** - HTTP GET request with full response metadata
3. **Check httpbin** - HTTP GET request with full response metadata
4. **Merge** - Combines results from both API checks
5. **Check All Statuses** - JavaScript code to analyze health
6. **IF** - Routes based on health status
7. **Log Alert** (true branch) - Handles unhealthy APIs
8. **Log Success** (false branch) - Confirms all healthy

------

## How It Works

### 1. Scheduled Trigger

Runs every 5 minutes using cron-style scheduling.

**Configuration:**

- Mode: "Every X minutes"
- Value: 5

### 2. Parallel API Checks

Both HTTP requests execute simultaneously (not sequential).

**GitHub API request:**

```
GET https://api.github.com/users/github
Options: Full Response = ON
```

**httpbin request:**

```
GET https://httpbin.org/get
Options: Full Response = ON
```

**Why "Full Response"?**

- Returns `statusCode`, `headers`, `body` metadata
- Without it, only get response body
- Need status code for health checking

### 3. Merge Results

Combines outputs from both HTTP nodes into single dataset.

**Mode:** Append (combines all items)

### 4. Status Analysis

JavaScript code that checks each API response:

```javascript
const results = [];

for (const item of $input.all()) {
  const url = item.json.body?.url || item.json.requestOptions?.uri || 'unknown';
  const status = item.json.statusCode || 0;
  
  results.push({
    url: url,
    status: status,
    healthy: status === 200,
    timestamp: new Date().toISOString()
  });
}

return results.map(r => ({ json: r }));
```

**Output format:**

```json
[
  {
    "url": "https://api.github.com/users/github",
    "status": 200,
    "healthy": true,
    "timestamp": "2025-11-29T20:30:00.000Z"
  },
  {
    "url": "https://httpbin.org/get",
    "status": 200,
    "healthy": true,
    "timestamp": "2025-11-29T20:30:00.000Z"
  }
]
```

### 5. Conditional Routing

IF node checks: `healthy` equals `false`

**True branch (unhealthy):**

- Logs alert with details
- In production: would send to Slack/PagerDuty

**False branch (healthy):**

- Logs success confirmation
- Optional: record metrics

------

## Testing

### Normal Operation (All Healthy)

**What to do:**

1. Open workflow in n8n
2. Ensure both URLs use valid endpoints
3. Click "Execute Workflow"

**Expected result:**

- All nodes show green checkmarks
- IF node takes FALSE branch
- Console shows: "✅ All APIs healthy"

### Failure Scenario

**What to do:**

1. Edit "Check httpbin" node
2. Change URL to: `https://httpbin.org/status/500`
3. Click "Execute Workflow"

**Expected result:**

- IF node takes TRUE branch (red path)
- Console shows: "🚨 ALERT: APIs are down!"
- Lists unhealthy endpoints

**Don't forget to change URL back to `/get` after testing!**

------

## Real-World Production Considerations

### This is a demo. In production you'd add:

**Better alerting:**

- Slack webhook notifications
- PagerDuty integration
- Email alerts
- SMS for critical failures

**Retry logic:**

- Retry failed requests 3 times
- Exponential backoff
- Only alert after multiple failures

**Metrics collection:**

- Store response times
- Track uptime percentage
- Historical trend analysis
- Dashboards (Grafana, etc.)

**More sophisticated health checks:**

- Check response content, not just status code
- Verify API response structure
- Test authentication
- Check rate limit headers

**Configuration management:**

- Store API list in database
- Dynamic endpoint addition
- Per-endpoint alert rules
- Maintenance mode flags

------

## Import Instructions

1. Open n8n at http://localhost:5678
2. Click "Add workflow" → "Import from File"
3. Select `workflow.json`
4. Workflow loads ready to execute

------

## What I Learned

### Technical Concepts

- **Parallel execution** - Both API calls run simultaneously, not sequentially
- **Full Response option** - Critical for getting status codes and headers
- **Data merging** - Combining multiple independent results
- **Conditional branching** - Routing based on computed values
- **JavaScript in workflows** - Custom logic and data transformation

### Operational Patterns

- **Health checking** - More than just "did it return data"
- **Alert fatigue** - Why you don't alert on single failure
- **Observable systems** - Importance of structured logging
- **Error handling** - Graceful degradation when APIs fail

### Interview Value

This workflow demonstrates understanding of:

- Production monitoring patterns
- Event-driven automation
- Conditional logic and routing
- API orchestration
- Error handling strategies

------

## Related Files

- [Main README](https://claude.ai/README.md) - Overview of all workflow automation projects
- [LESSONS_LEARNED.md](https://claude.ai/chat/LESSONS_LEARNED.md) - Detailed reflections
- [workflow.json](https://claude.ai/chat/workflow.json) - Importable n8n workflow
