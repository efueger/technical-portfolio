# Workflow Automation with n8n

Hands-on workflow automation demos built to understand event-driven systems, API orchestration, and conditional logic - patterns used by modern automation platforms in security operations, DevOps, and customer support.

## Technologies

- **n8n** - Open-source workflow automation platform
- **Docker** - Container runtime for n8n
- **REST APIs** - GitHub API, httpbin for testing
- **JavaScript** - Custom logic and data transformation

## Projects

### 1. API Monitor

Automated monitoring system that checks multiple API endpoints in parallel and routes alerts based on health status.

**What it demonstrates:**

- Scheduled execution (cron-style triggers)
- Parallel API calls
- Data aggregation and analysis
- Conditional branching (healthy vs. unhealthy)
- Error alerting patterns

**Key concepts:**

- Event-driven automation
- Multi-step workflows
- Conditional logic
- API health monitoring

[View details →](./api-monitor/)

---

### 2. GitHub Issue Enrichment Bot

Event-driven workflow that automatically categorizes and routes GitHub issues based on content analysis.

**What it demonstrates:**

- Webhook triggers (event-driven)
- Data extraction and transformation
- Smart categorization using keyword matching
- Dynamic routing based on issue type
- Priority assignment logic

**Key concepts:**

- Webhook-based automation
- Natural language processing (basic)
- Switch/case routing
- Automated triage workflows

[View details →](./github-issue-bot/)

---

## Why These Workflows Matter

### Real-World Applications

**Security Operations:**

Security alert received → Enrich with threat intelligence → Categorize severity → Route to appropriate team → Auto-escalate if critical

**Customer Support:**

Support ticket created → Extract customer tier → Check SLA requirements → Route to correct team → Auto-escalate enterprise customers

**DevOps Incident Response:**

Monitor detects API failure → Check multiple endpoints → Aggregate health status → Alert on-call engineer → Create incident ticket

These demos use the exact same patterns.

---

## Running These Workflows

### Prerequisites

- Docker Desktop installed and running
- n8n container: `docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n`
- Access n8n at: http://localhost:5678

### Import Workflows

1. Open n8n (http://localhost:5678)
2. Click "Add workflow" → "Import from File"
3. Select the `workflow.json` file from each project folder
4. Workflow loads with all nodes configured

### Testing

Each project folder contains specific test instructions and sample payloads.

---

## What I Learned

### Technical Skills

- **Event-driven architecture** - Understanding triggers, webhooks, and scheduled execution
- **API orchestration** - Chaining multiple API calls and transforming data between steps
- **Conditional logic** - Implementing branching, routing, and decision trees
- **Error handling** - Graceful failures and retry patterns
- **Data transformation** - JavaScript for custom logic and data manipulation

### Workflow Automation Patterns

- **Parallel execution** - Running multiple operations simultaneously
- **Data merging** - Combining results from multiple sources
- **Smart routing** - Decision-based workflow paths
- **Alerting strategies** - When and how to notify stakeholders
- **Testing workflows** - Test mode vs. production mode, debugging techniques

### Platform-Specific Insights

Working with n8n taught me:

- How security and operations teams automate incident response
- The importance of reliable webhook handling
- Why categorization and routing are critical
- How to build resilient, production-ready workflows

---

## Interview Talking Points

"I built workflow automation demos using n8n to understand how modern automation platforms work. I created an API monitoring system with parallel execution and conditional alerting, and an event-driven issue triage bot that categorizes and routes based on content analysis.

These workflows demonstrate patterns used in security incident response and operations automation - webhook triggers, API enrichment, intelligent categorization, and automated routing. I understand how to chain multiple API calls, transform data between steps, handle errors gracefully, and build workflows that scale."

---

## Project Structure

```
workflow-automation/
├── README.md                    ← You are here
├── api-monitor/
│   ├── README.md               ← Project-specific details
│   ├── workflow.json           ← Importable n8n workflow
│   ├── screenshot.png          ← Visual of workflow canvas
│   └── LESSONS_LEARNED.md      ← Reflections and insights
└── github-issue-bot/
    ├── README.md
    ├── workflow.json
    ├── screenshot.png
    └── LESSONS_LEARNED.md
```

---

## Next Steps

Potential extensions for these workflows:

- Add Slack/email notifications (real alerting)
- Implement retry logic with exponential backoff
- Add metrics collection and dashboards
- Create more complex categorization (ML-based)
- Build multi-stage approval workflows
- Add database persistence for execution history

---

## Related Labs

- [OAuth Authentication](../oauth-authentication/) - API authentication patterns
- [Docker Troubleshooting](../docker-troubleshooting/) - Container debugging skills

---

## Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Workflow Automation Best Practices](https://docs.n8n.io/workflows/best-practices/)
