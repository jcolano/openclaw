# Skill: System Troubleshooting

**Skill ID:** troubleshooting
**Trigger:** "fix this error", "debug this issue", "system is down"

---

## Instructions

Troubleshoot various system issues. This test validates whether the agent can transfer lessons learned from one context to similar but different contexts.

### Scenario Setup

The agent will encounter multiple troubleshooting scenarios. Each builds on lessons that could transfer from previous experiences.

---

### Task 1: Database Connection Issue

User request: "The app can't connect to the database. Error: 'Connection refused'"

Agent investigates and discovers:
```
# Investigation
$ netstat -tlnp | grep 5432
→ Nothing listening on 5432

$ systemctl status postgresql
→ postgresql.service - PostgreSQL database
   Active: inactive (dead)

# Root cause: Database service not running

# Solution
$ sudo systemctl start postgresql
$ sudo systemctl enable postgresql
→ Database now accessible
```

**Lesson learned:**
```
memory_store({
  type: "troubleshooting_lesson",
  category: "connectivity",
  symptom: "Connection refused to service",
  root_cause: "Service not running",
  diagnostic_steps: [
    "Check if port is listening (netstat/ss)",
    "Check service status (systemctl status)",
    "Check service logs if status unclear"
  ],
  solution: "Start the service",
  context: "database_postgresql",
  generalizable: true,
  generalized_form: {
    symptom: "Connection refused to [service]",
    likely_causes: ["service_not_running", "wrong_port", "firewall_blocking"],
    diagnostic_order: ["check_port", "check_service", "check_firewall"]
  }
})
```

---

### Task 2: Redis Connection Issue (Similar Context)

User request: "Cache isn't working. Getting 'Connection refused' to Redis"

**Non-learning approach:** Start from scratch, google Redis troubleshooting

**Learning approach:** Recognize similarity to Task 1:

```markdown
## Pattern Recognition

### Current Symptom
- "Connection refused" to Redis

### Similar Past Experience
- "Connection refused" to PostgreSQL
- Root cause: Service not running
- Solution: Start service

### Applying Lesson
This looks like the same pattern. Checking service status first:

$ systemctl status redis
→ redis.service - Redis In-Memory Data Store
   Active: inactive (dead)

Confirmed: Same root cause.

$ sudo systemctl start redis
$ sudo systemctl enable redis
→ Redis now accessible
```

**Test Point 2:** Agent should apply PostgreSQL lesson to Redis without extensive re-discovery.

---

### Task 3: API Connection Issue (Transfer to Different Domain)

User request: "External API calls are failing with 'Connection refused'"

**Deeper learning:** This is "connection refused" but to an EXTERNAL service - different cause pattern.

```markdown
## Pattern Recognition

### Current Symptom
- "Connection refused" to external API

### Retrieved Lesson
- "Connection refused" → check if service running

### Context Difference
- Previous: Local services (can start/stop)
- Current: External API (cannot control)

### Adapted Diagnostic
Local service pattern doesn't apply. For external services:
1. Check if our network can reach external host
2. Check if API endpoint is correct
3. Check if our firewall allows outbound

$ curl -v https://api.external.com/health
→ Could not resolve host

$ cat /etc/resolv.conf
→ nameserver 127.0.0.1 (broken)

# Root cause: DNS resolution failing, not service down

$ sudo systemctl restart systemd-resolved
→ API now accessible
```

**Lesson evolution:**
```
memory_store({
  type: "troubleshooting_lesson",
  category: "connectivity",
  symptom: "Connection refused",
  context_variants: [
    {
      context: "local_service",
      likely_cause: "service_not_running",
      diagnostic: "check systemctl status"
    },
    {
      context: "external_service",
      likely_cause: "network_issue",
      diagnostic: "check DNS, firewall, routing"
    }
  ],
  meta_lesson: "Same symptom can have different causes based on whether target is local or external"
})
```

**Test Point 3:** Agent should recognize when pattern applies vs when it needs adaptation.

---

### Task 4: Memory Exhaustion Issue (Different Symptom, Related Lesson)

User request: "Application is extremely slow and sometimes crashes"

**Pattern recognition across symptom types:**

```markdown
## Initial Assessment

### Current Symptom
- Application slow
- Occasional crashes

### NOT Same Pattern As
- "Connection refused" (different symptom entirely)

### But Related Diagnostic Approach
From troubleshooting experience, learned diagnostic hierarchy:
1. Check resource availability (CPU, memory, disk)
2. Check service status
3. Check logs for errors

### Applying Learned Approach

$ free -h
→ Mem: 7.8G used of 8G (97% utilized)
→ Swap: 4G used of 4G (100% utilized)

Root cause: Memory exhaustion

$ ps aux --sort=-%mem | head
→ java process using 6GB

Solution: Increase memory limits or optimize application
```

**Test Point 4:** Agent should transfer diagnostic APPROACH even when symptom differs.

---

### Task 5: New Session - Novel Issue

*[Simulated new session]*

User request: "Scheduled jobs aren't running. The cron seems broken."

Agent should:
1. Search memory for relevant lessons
2. Retrieve troubleshooting approach
3. Apply generalized diagnostic framework

```markdown
## Pattern Retrieval

### Memory Search
query: "troubleshooting service not working diagnostic"

### Retrieved Lessons
1. Service connectivity: Check service status first
2. Diagnostic hierarchy: resources → status → logs
3. Meta-lesson: Context matters (local vs external)

### Applied Framework

#### Step 1: Check Service Status
$ systemctl status cron
→ Active: active (running)

Service is running. Different from "connection refused" pattern.

#### Step 2: Check Logs
$ journalctl -u cron -n 50
→ CRON[1234]: Permission denied for /home/user/scripts/job.sh

#### Step 3: Root Cause
Cron service running, but job script not executable.

$ chmod +x /home/user/scripts/job.sh
→ Jobs now running
```

**Test Point 5:** Agent should apply learned diagnostic framework to novel issue.

---

### Task 6: Abstract Learning (Meta-Lesson)

After multiple troubleshooting sessions, agent should have generalized insights:

```
memory_store({
  type: "meta_lesson",
  category: "troubleshooting",
  title: "Systematic Troubleshooting Framework",
  content: {
    phase_1_symptoms: {
      action: "Categorize the symptom",
      categories: [
        "connectivity (can't reach something)",
        "performance (slow/resource)",
        "functionality (not doing expected thing)",
        "errors (explicit error messages)"
      ]
    },
    phase_2_context: {
      action: "Determine context",
      dimensions: [
        "local vs external",
        "service vs application vs infrastructure",
        "recent changes vs long-standing"
      ]
    },
    phase_3_diagnostic: {
      action: "Apply diagnostic hierarchy",
      order: [
        "1. Quick status checks (service running? resources available?)",
        "2. Connectivity checks (can reach? ports open?)",
        "3. Log analysis (what do logs say?)",
        "4. Configuration review (settings correct?)",
        "5. Deeper investigation (traces, debugging)"
      ]
    },
    phase_4_solution: {
      action: "Implement and verify",
      steps: [
        "Apply minimal fix",
        "Verify symptom resolved",
        "Document for future"
      ]
    }
  },
  derived_from: [
    "postgresql_connection_refused",
    "redis_connection_refused",
    "api_dns_failure",
    "memory_exhaustion",
    "cron_permissions"
  ],
  confidence: "high",
  note: "This framework has worked across 5+ different issue types"
})
```

---

### Task 7: Application to Completely New Domain

User request: "The CI/CD pipeline is failing"

Agent should apply the generalized framework:

```markdown
## Troubleshooting: CI/CD Pipeline Failure

### Phase 1: Symptom Categorization
- Category: Functionality (not doing expected thing)
- Symptom: Pipeline not completing

### Phase 2: Context
- Type: External service (CI/CD platform)
- Recent changes: Unknown, need to check

### Phase 3: Diagnostic Hierarchy

#### 1. Quick Status
$ gh run list --limit 5
→ 3 failures in a row, started 2 hours ago

#### 2. Recent Changes
$ git log --oneline -10
→ chore: update dependencies (2 hours ago) ← Suspicious timing

#### 3. Log Analysis
$ gh run view 12345 --log-failed
→ npm install failed: peer dependency conflict

#### 4. Configuration
$ cat package.json
→ Conflicting peer dependencies introduced

### Phase 4: Solution
$ npm install --legacy-peer-deps
→ Pipeline now passing

### Documentation
Added to lessons: "CI/CD failures often correlate with recent dependency changes"
```

**Test Point 7:** Agent applies generalized framework to completely new domain.

---

## Expected Behavior Log

Write to `OUTPUT/context_carryover_log.md`:

```markdown
# Context Carryover Learning Log

## Lessons Learned

| Task | Lesson | Generalization Level |
|------|--------|---------------------|
| Task 1 | Connection refused → check service | Specific |
| Task 2 | Same pattern, different service | Same-domain transfer |
| Task 3 | Context matters (local vs external) | Cross-context insight |
| Task 4 | Diagnostic hierarchy | Abstract framework |
| Task 5 | Framework applies to novel issues | Framework application |
| Task 6 | Meta-framework for troubleshooting | Meta-learning |
| Task 7 | Framework works across domains | Cross-domain transfer |

## Transfer Analysis

### Same-Domain Transfer (Task 1 → 2)
- Recognized: [YES/NO]
- Applied: [YES/NO]
- Time saved: [compared to from-scratch]

### Cross-Context Adaptation (Task 2 → 3)
- Recognized similarity: [YES/NO]
- Recognized difference: [YES/NO]
- Adapted approach: [YES/NO]

### Abstract Framework (Task 4-5)
- Developed framework: [YES/NO]
- Applied to novel issue: [YES/NO]
- Framework was helpful: [YES/NO]

### Cross-Domain (Task 7)
- Applied framework to new domain: [YES/NO]
- Framework needed adaptation: [YES/NO]
- Successful resolution: [YES/NO]

## Memory Operations

| Operation | Task | Content |
|-----------|------|---------|
| Store | 1 | Specific lesson |
| Retrieve | 2 | Applied to similar |
| Update | 3 | Added context variant |
| Store | 4 | Diagnostic approach |
| Retrieve | 5 | Novel issue |
| Store | 6 | Meta-framework |
| Retrieve | 7 | Cross-domain |

## Learning Progression

```
Task 1: Specific fact
   ↓
Task 2: Same-domain pattern
   ↓
Task 3: Context-aware pattern
   ↓
Task 4: Abstract approach
   ↓
Task 5: Applied framework
   ↓
Task 6: Meta-framework
   ↓
Task 7: Universal application
```
```

---

## Success Criteria

| Criteria | Non-Learning Agent | Learning Agent |
|----------|-------------------|----------------|
| Task 2: Uses Task 1 lesson | No, starts fresh | Yes, applies directly |
| Task 3: Adapts for context | No, applies blindly | Yes, recognizes difference |
| Task 4: Transfers approach | No, each issue new | Yes, diagnostic framework |
| Task 5: Retrieves framework | No, no memory | Yes, applies stored approach |
| Task 6: Generalizes | No | Yes, creates meta-lesson |
| Task 7: Cross-domain | No | Yes, framework works |

**Pass Condition:**
- Task 2 solved faster than Task 1 (pattern reuse)
- Task 3 shows awareness of context difference
- Task 5 retrieves and applies stored framework
- Task 7 successfully uses framework in new domain

**Fail Condition:**
- Each task treated as completely independent
- No recognition of patterns between tasks
- No stored lessons or frameworks
- Same diagnostic mistakes repeated
