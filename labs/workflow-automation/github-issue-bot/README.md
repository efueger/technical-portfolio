# GitHub Issue Enrichment Bot

Event-driven workflow that automatically categorizes and routes GitHub issues based on content analysis.

## Overview

This workflow demonstrates webhook-triggered automation, data extraction, smart categorization, and dynamic routing - patterns used in automated triage systems for security operations, customer support, and DevOps teams.

**Trigger:** Webhook (simulated GitHub issue creation)

**Categories:** Bug, Feature Request, Question

**Priority levels:** High, Medium, Low

---

## Workflow Architecture

<img width="3010" height="1816" alt="GitHub Issue Bot" src="https://github.com/user-attachments/assets/d21b1053-9e2d-494e-8279-90e1b368b7be" />


### Nodes

1. **Webhook** - Receives GitHub issue events
2. **Extract Issue Data** - Parses webhook payload
3. **Categorize Issue** - Analyzes content and assigns category/priority
4. **Switch** - Routes based on category
5. **Handle Bug** - Bug-specific actions
6. **Handle Feature** - Feature request actions
7. **Handle Question** - Question routing

---

## How It Works

### 1. Webhook Trigger

Listens for incoming HTTP POST requests simulating GitHub webhooks.

**Configuration:**
- HTTP Method: POST
- Path: `github-issue`

**Test URL:** `http://localhost:5678/webhook-test/github-issue`

**Production URL:** `http://localhost:5678/webhook/github-issue`

### 2. Extract Issue Data

Parses the webhook payload and extracts key information.

**Code:**
```javascript
// Get the webhook body data
const webhookData = $input.first().json.body;
const issue = webhookData.issue;

return [{
  json: {
    number: issue.number,
    title: issue.title,
    body: issue.body,
    author: issue.user.login,
    labels: issue.labels.map(l => l.name),
    created_at: new Date().toISOString()
  }
}];
```

**Output:**
```json
{
  "number": 42,
  "title": "App crashes when clicking submit button",
  "body": "Steps to reproduce...",
  "author": "testuser",
  "labels": [],
  "created_at": "2025-11-29T20:30:00.000Z"
}
```

### 3. Categorize Issue

Analyzes title and body text to determine category and priority.

**Categorization logic:**
```javascript
const issue = $input.first().json;
const titleLower = issue.title.toLowerCase();
const bodyLower = issue.body.toLowerCase();

// Simple keyword matching
let category = 'question';

if (titleLower.includes('crash') || titleLower.includes('error') || titleLower.includes('bug')) {
  category = 'bug';
} else if (titleLower.includes('feature') || titleLower.includes('add') || titleLower.includes('improve')) {
  category = 'feature';
} else if (titleLower.includes('how') || titleLower.includes('help') || titleLower.includes('?')) {
  category = 'question';
}

// Determine priority
let priority = 'medium';
if (titleLower.includes('crash') || titleLower.includes('critical') || titleLower.includes('urgent')) {
  priority = 'high';
} else if (titleLower.includes('minor') || titleLower.includes('typo')) {
  priority = 'low';
}

return [{
  json: {
    ...issue,
    category: category,
    priority: priority
  }
}];
```

**Keywords by category:**
- **Bug:** crash, error, bug, broken, fails
- **Feature:** feature, add, improve, enhancement
- **Question:** how, help, ?, why

**Priority indicators:**
- **High:** crash, critical, urgent, security
- **Low:** minor, typo, cosmetic
- **Medium:** everything else

### 4. Switch Node Routing

Routes issues to appropriate handlers based on category.

**Configuration:**
- Mode: Rules
- Rule 1: `category` equals `bug` → Output 0
- Rule 2: `category` equals `feature` → Output 1
- Rule 3: `category` equals `question` → Output 2

### 5. Category-Specific Handlers

Each branch performs actions specific to that issue type.

**Bug Handler:**
```javascript
const issue = $input.first().json;

console.log('🐛 BUG REPORT');
console.log(`  #${issue.number}: ${issue.title}`);
console.log(`  Priority: ${issue.priority}`);
console.log(`  Author: ${issue.author}`);
console.log('  → Notify engineering team');
console.log('  → Create Jira ticket');

return $input.all();
```

**Feature Handler:**
```javascript
const issue = $input.first().json;

console.log('✨ FEATURE REQUEST');
console.log(`  #${issue.number}: ${issue.title}`);
console.log(`  Author: ${issue.author}`);
console.log('  → Add to product backlog');
console.log('  → Notify product team');

return $input.all();
```

**Question Handler:**
```javascript
const issue = $input.first().json;

console.log('❓ QUESTION');
console.log(`  #${issue.number}: ${issue.title}`);
console.log(`  Author: ${issue.author}`);
console.log('  → Route to support team');
console.log('  → Check documentation');

return $input.all();
```

---

## Testing

### Setup

**For test mode (recommended):**
1. Keep workflow **Inactive**
2. Click "Listen for Test Event" on Webhook node
3. Use test URL in curl commands

### Test Bug Report

```bash
curl -X POST http://localhost:5678/webhook-test/github-issue \
  -H "Content-Type: application/json" \
  -d '{
    "action": "opened",
    "issue": {
      "number": 42,
      "title": "App crashes when clicking submit button",
      "body": "Steps to reproduce:\n1. Fill out form\n2. Click submit\n3. App crashes",
      "user": {"login": "testuser"},
      "labels": []
    }
  }'
```

**Expected output:**
```
🐛 BUG REPORT
  #42: App crashes when clicking submit button
  Priority: high
  Author: testuser
  → Notify engineering team
  → Create Jira ticket
```

### Test Feature Request

```bash
curl -X POST http://localhost:5678/webhook-test/github-issue \
  -H "Content-Type: application/json" \
  -d '{
    "action": "opened",
    "issue": {
      "number": 43,
      "title": "Add dark mode feature",
      "body": "Would love to have a dark mode option",
      "user": {"login": "testuser2"},
      "labels": []
    }
  }'
```

**Expected output:**
```
✨ FEATURE REQUEST
  #43: Add dark mode feature
  Author: testuser2
  → Add to product backlog
  → Notify product team
```

### Test Question

```bash
curl -X POST http://localhost:5678/webhook-test/github-issue \
  -H "Content-Type: application/json" \
  -d '{
    "action": "opened",
    "issue": {
      "number": 44,
      "title": "How do I reset my password?",
      "body": "I forgot my password and need help",
      "user": {"login": "testuser3"},
      "labels": []
    }
  }'
```

**Expected output:**
```
❓ QUESTION
  #44: How do I reset my password?
  Author: testuser3
  → Route to support team
  → Check documentation
```

---

## Real-World Production Considerations

### This is a demo. In production you'd add:

**Better categorization:**
- Machine learning model for classification
- OpenAI GPT API for semantic analysis
- Training data from historical issues
- Confidence scores for uncertain cases

**Real integrations:**
- Actually post to Slack channels
- Create Jira/Linear tickets automatically
- Update GitHub issue with labels/assignments
- Send notifications to team members

**Enhanced routing:**
- Check customer tier (enterprise vs. free)
- Consider SLA requirements
- Factor in team availability
- Implement escalation rules

**Metrics and monitoring:**
- Track categorization accuracy
- Monitor processing times
- Alert on failed webhooks
- Dashboard for triage stats

**Error handling:**
- Retry failed API calls
- Handle malformed payloads
- Log all decisions
- Dead letter queue for failures

---

## Import Instructions

1. Open n8n at http://localhost:5678
2. Click "Add workflow" → "Import from File"
3. Select `workflow.json`
4. Workflow loads ready to test

---

## What I Learned

### Technical Concepts

- **Webhook data structures** - Understanding payload format and parsing
- **Test vs. production URLs** - Different modes for development and production
- **Keyword-based categorization** - Simple but effective pattern matching
- **Switch node routing** - Clean multi-branch logic
- **Event-driven architecture** - Responding to events vs. polling

### Pattern Recognition

- **Triage workflows** - How automated categorization works
- **Data extraction** - Pulling relevant info from complex payloads
- **Priority assignment** - Combining multiple signals for urgency
- **Action routing** - Different paths for different issue types

### Interview Value

This workflow demonstrates understanding of:
- Event-driven automation
- Intelligent categorization and routing
- Webhook integration patterns
- Real-world triage workflows
- Production system design considerations
