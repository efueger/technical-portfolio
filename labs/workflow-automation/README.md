# Workflow Automation

Hands-on labs demonstrating event-driven automation, API orchestration, and data transformation patterns using modern workflow platforms. These labs show how to build resilient, production-quality integrations across multiple automation tools.

---

## Platforms

### [n8n](./n8n/)

Open-source workflow automation with JavaScript-based transformations and 200+ pre-built integrations.

**Projects:**

- API Health Monitor - Multi-endpoint monitoring with parallel execution
- GitHub Issue Bot - Automated categorization and routing

---

### [Tines](./tines/)

Security/IT operations automation platform using Liquid templates for data transformation.

**Projects:**

- GitHub Issue Monitor - Webhook-driven alerts with error handling
- Pagination & Formulas - Advanced Liquid patterns and array processing
- HMAC Authentication - Cryptographic request signing implementation

---

## Why Multiple Platforms?

### Transferable Patterns

Core concepts remain consistent across platforms:

- **Event-driven triggers** (webhooks, schedules)
- **API orchestration** (parallel calls, error handling)
- **Data transformation** (extracting, formatting, routing)
- **Conditional logic** (IF/ELSE, switches)
- **Error handling** (retry, alerting, graceful degradation)

### Platform Differences

**n8n:**

- JavaScript for custom logic
- Self-hosted, open-source
- Code-accessible when needed

**Tines:**

- Liquid templates (declarative)
- Enterprise security focus
- Visual formula builder

Both accomplish the same integration goals with different approaches.

---

## Real-World Applications

**Security Operations:**  
Alert received → Enrich with threat intel → Categorize → Route to team → Escalate if critical

**Customer Support:**  
Ticket created → Extract customer tier → Check SLA → Route to team → Auto-escalate enterprise customers

**DevOps Monitoring:**  
API failure detected → Check multiple endpoints → Aggregate health → Alert on-call → Create incident

These labs demonstrate the exact same patterns used in production workflows.

---

## Technologies

**Platforms:** n8n, Tines  
**Languages:** JavaScript, Liquid templates  
**APIs:** GitHub, REST endpoints, webhooks  
**Concepts:** Event-driven architecture, parallel execution, error handling, data transformation

---

## Related Labs

- [Kubernetes Troubleshooting](../kubernetes-troubleshooting.md) - Container debugging for workflow deployments
- [Supabase RLS](../supabase-rls-enterprise/) - Database-level security patterns
