# Ultimate Test U3: The Production Incident

**Difficulty:** Extreme
**Duration:** Real-time incident response simulation
**Capabilities Required:** Planning ↔ Reflection under uncertainty with time pressure

---

## Scenario

Production is down. Users are complaining. The CEO is asking for updates. You need to:
- Diagnose the problem under uncertainty
- Make decisions with incomplete information
- Balance speed vs thoroughness
- Avoid making things worse
- Learn from the incident

Clock is ticking.

---

## T+0:00 - Alert Received

```
🚨 ALERT: API response time >10s, error rate 45%
🚨 ALERT: Database connection pool exhausted
🚨 ALERT: Payment processing failing
🚨 ALERT: User complaints flooding support

Current status:
- 50,000 users affected
- Revenue impact: ~$500/minute
- CEO asking for update in 5 minutes
```

### Task 0.1: Initial Triage

**PLANNING + CONFIDENCE under time pressure:**

```markdown
## T+0:00 Initial Assessment

### What I know (HIGH confidence)
- Multiple systems showing stress
- User impact is real and significant
- Need to act quickly

### What I don't know (uncertainty)
- Root cause
- Whether issues are related or coincidental
- Which alert is the SOURCE vs SYMPTOM

### Immediate priorities

1. **FIRST: Prevent cascade** (2 min)
   - Check if one failure is causing others
   - If DB is the root, API and payments are symptoms

2. **SECOND: Quick diagnosis** (3 min)
   - Check error logs
   - Check recent deployments
   - Check infrastructure metrics

3. **THIRD: CEO update** (at T+5:00)
   - What we know
   - What we're doing
   - ETA if possible

### NOT doing yet (resist urge to fix before understanding)
- Restarting services randomly
- Rolling back without knowing if deployment caused it
- Making database changes without understanding state
```

---

## T+2:00 - First Data

### Error Logs

```
file_read: SANDBOX/incident/error_logs.txt
```

```
[ERROR] 09:32:15 - PaymentService: Connection timeout to payments.stripe.com
[ERROR] 09:32:15 - PaymentService: Connection timeout to payments.stripe.com
[ERROR] 09:32:16 - PaymentService: Connection timeout to payments.stripe.com
[ERROR] 09:32:18 - DBPool: No available connections (pool size: 50, in use: 50)
[ERROR] 09:32:18 - DBPool: No available connections (pool size: 50, in use: 50)
[ERROR] 09:32:19 - APIGateway: Timeout waiting for PaymentService response
[ERROR] 09:32:20 - PaymentService: Connection timeout to payments.stripe.com
[WARN]  09:32:20 - DBPool: Connection acquired after 8.5s wait
[ERROR] 09:32:21 - PaymentService: Circuit breaker OPEN after 10 failures
```

### Recent Deployments

```
file_read: SANDBOX/incident/deployments.txt
```

```
09:30:00 - payment-service v2.4.1 -> v2.4.2 (minor: updated timeout configs)
09:15:00 - api-gateway v1.8.0 -> v1.8.1 (patch: logging improvement)
08:00:00 - user-service v3.1.0 (no changes today)
```

### Task 2.1: Hypothesis Formation

**CONFIDENCE + REFLECTION:**

```markdown
## T+2:00 Diagnostic Analysis

### Timeline correlation

```
09:30:00 - payment-service deployed (v2.4.2 - "timeout configs")
09:32:15 - First payment timeout errors
09:32:18 - DB pool exhaustion begins
```

### Hypothesis 1: Deployment caused it (70% confidence)

**Evidence for:**
- Deployment 2 minutes before errors
- Deployment was "timeout configs" - directly relevant
- Payment errors started first, then cascaded

**Evidence against:**
- Could be coincidental
- External service (Stripe) might be the issue

### Hypothesis 2: External service down (25% confidence)

**Evidence for:**
- Errors are "timeout to payments.stripe.com"
- External services do have outages

**Evidence against:**
- Timing coincidence with deployment seems unlikely
- Other systems depend on us, not external

### Hypothesis 3: Something else (5% confidence)

Keeping this option open.

### What's happening (likely cascade)

```
Payment Service deployment (changed timeouts)
         ↓
Payments to Stripe failing/slow
         ↓
Payment requests backing up, holding DB connections
         ↓
DB pool exhausted
         ↓
All services waiting for DB
         ↓
API timeouts
         ↓
User errors
```

### Decision: Most likely root cause

**Payment service deployment (70% confidence)**

But I need to verify before rolling back because:
- Rolling back takes 2-3 minutes
- If wrong, we waste time and maybe make it worse
- Need one more data point
```

---

## T+3:00 - Quick Verification

### Check Stripe Status

```
$ curl -s https://status.stripe.com/api/v2/status.json
{"status": "operational", "updated": "2024-01-15T09:30:00Z"}
```

### Check Payment Service Config Diff

```
file_read: SANDBOX/incident/payment_config_diff.txt
```

```diff
# payment-service/config/timeouts.yaml

- stripe_connect_timeout: 5000
- stripe_read_timeout: 30000
+ stripe_connect_timeout: 500   # "reduced for faster failover"
+ stripe_read_timeout: 3000     # "reduced for faster failover"
```

### Task 3.1: Root Cause Confirmed

**CONFIDENCE UPDATE + DECISION:**

```markdown
## T+3:00 Root Cause Confirmed

### Finding

Deployment v2.4.2 reduced Stripe timeouts by 10x:
- Connect timeout: 5000ms → 500ms
- Read timeout: 30000ms → 3000ms

Stripe is operational but their API sometimes takes 1-2 seconds.
With 500ms timeout, MOST requests fail immediately.

### Confidence update

| Hypothesis | Before | After | Reason |
|------------|--------|-------|--------|
| Deployment | 70% | 95% | Config diff explains everything |
| Stripe down | 25% | 5% | Status shows operational |
| Other | 5% | 0% | Root cause found |

### Cascade explanation (verified)

```
Timeout too short (500ms)
         ↓
Most Stripe calls fail immediately
         ↓
Retry logic kicks in (3 retries per request)
         ↓
Requests pile up, each holding DB connection
         ↓
50 connections × 3 retries = 150 pending requests
         ↓
DB pool exhausted
         ↓
Everything waiting for DB
         ↓
Complete system slowdown
```

### Decision: ROLLBACK (95% confidence this is correct action)

Rollback payment-service to v2.4.1
```

---

## T+4:00 - Execution

### Task 4.1: Rollback with Monitoring

**PLANNING under pressure:**

```markdown
## Rollback Plan

### Pre-rollback checklist (30 seconds)
- [ ] Confirm rollback target: v2.4.1
- [ ] Confirm current version: v2.4.2
- [ ] Alert team: "Rolling back payment-service"

### Rollback command
$ kubectl rollout undo deployment/payment-service -n production

### Post-rollback monitoring (watch for 2 minutes)
- Payment error rate (should drop)
- DB pool usage (should recover)
- API latency (should improve)

### If rollback makes things WORSE
- Roll forward immediately
- Re-assess hypotheses
- Consider circuit breaker activation

### Confidence in rollback success: 90%

Remaining 10% uncertainty:
- There could be additional issues masked by this one
- Rollback itself could have problems (rare)
```

### Rollback Execution

```
$ kubectl rollout undo deployment/payment-service -n production
deployment.apps/payment-service rolled back

$ kubectl rollout status deployment/payment-service -n production
deployment "payment-service" successfully rolled out

Monitoring...

T+4:30 - Payment errors: 45% → 30%
T+5:00 - Payment errors: 30% → 8%
T+5:30 - Payment errors: 8% → 2% (normal)
T+5:30 - DB pool: 50/50 → 35/50 → 12/50 (recovering)
T+6:00 - API latency: 10s → 500ms → 150ms (normal)
```

---

## T+5:00 - CEO Update

### Task 5.1: Communicate Under Uncertainty

**CONFIDENCE in communication:**

```markdown
## CEO Update (T+5:00)

### Status: RECOVERING

**What happened:**
A deployment 35 minutes ago changed payment service timeout settings.
The new timeouts were too aggressive, causing payment processing failures
which cascaded to database and API issues.

**What we did:**
Rolled back to previous version. System is recovering.

**Current state:**
- Error rate: Down from 45% to 8%, still improving
- User impact: Decreasing, expect full recovery in 5 minutes
- Revenue impact: ~$2,500 (5 minutes of degradation)

**Confidence level: HIGH (90%)**
We identified and fixed the root cause. Recovery metrics confirm.

**What I don't know yet:**
- Why the timeout change was made (need to discuss with team)
- Whether there are additional issues masked by this one

**Next steps:**
1. Monitor for full recovery (5 min)
2. Post-incident review (within 24h)
3. Discuss deployment review process

**ETA to full resolution:** 10 minutes from now
```

---

## T+10:00 - Recovery Confirmed

```
All metrics nominal:
- Error rate: 0.1% (baseline)
- API latency: 120ms (baseline)
- DB pool: 8/50 connections (healthy)
- Payment success rate: 99.8% (normal)
```

### Task 10.1: Incident Stable - Now Reflect

**REFLECTION + LEARNING (now that pressure is off):**

```markdown
## Post-Incident Reflection

### What went well

1. **Structured triage** - Didn't panic, followed diagnostic process
2. **Hypothesis-driven** - Formed theories, gathered evidence
3. **Verified before acting** - Checked Stripe status, reviewed diff
4. **Quick rollback** - 4 minutes from alert to rollback
5. **Clear communication** - CEO update was honest about certainty

### What could improve

1. **Deployment review** - Timeout changes should require review
2. **Alerting** - Could have caught this in staging
3. **Circuit breaker** - Payment service CB opened but damage was done

### Learning stored

```json
[
  {
    "type": "incident_pattern",
    "pattern": "aggressive_timeout_cascade",
    "symptoms": ["connection timeouts", "pool exhaustion", "cascade failure"],
    "root_cause": "timeout too short for external service variability",
    "detection": "check recent deployments, especially config changes",
    "resolution": "rollback or adjust timeout"
  },
  {
    "type": "diagnostic_lesson",
    "lesson": "DB pool exhaustion is usually a SYMPTOM, not a cause",
    "explanation": "Something is holding connections - find what",
    "action": "trace what's waiting for DB before adding pool capacity"
  },
  {
    "type": "process_lesson",
    "lesson": "Config changes are as risky as code changes",
    "explanation": "v2.4.2 was 'minor' but caused major outage",
    "action": "config changes need same review as code"
  }
]
```

### Confidence Calibration Review

| Decision Point | Confidence | Outcome | Calibrated? |
|----------------|------------|---------|-------------|
| Deployment is cause | 70% → 95% | Correct | Yes - gathered evidence before committing |
| Rollback will fix | 90% | Correct | Yes - left room for uncertainty |
| Recovery in 5 min | Estimated | 6 min actual | Close enough |

### Process Used (for future incidents)

```
1. TRIAGE (2 min)
   - What do I know? What don't I know?
   - Prioritize: prevent cascade > diagnose > fix

2. HYPOTHESIZE (2 min)
   - Form theories with confidence levels
   - Look for evidence for/against each

3. VERIFY (1 min)
   - One more data point before acting
   - Especially if action has risk

4. ACT (2 min)
   - Execute with monitoring
   - Have rollback for your rollback

5. COMMUNICATE
   - What happened, what we did, confidence level
   - What we don't know yet

6. REFLECT (after pressure off)
   - What worked, what didn't
   - Store lessons
```

---

## T+30:00 - Postmortem Investigation

### New Information

While reviewing, you discover:

```
file_read: SANDBOX/incident/deployment_notes.md
```

```markdown
# Payment Service v2.4.2 Deployment Notes

**Author:** Junior Dev
**Reviewed by:** (none - deployed directly)

**Reason for change:**
"Our payment retries were taking too long. Reduced timeouts for
faster failover to backup payment provider. Tested locally and
it worked fine."

**Testing:**
- Local environment: ✓
- Staging: (skipped - "minor config change")
- Load test: (not performed)
```

### Task 30.1: Deeper Learning

**LEARNING + REFLECTION on process:**

```markdown
## Postmortem: Deeper Lessons

### Surface cause
Timeout values too aggressive

### Deeper cause
Deployment without review or staging test

### Deepest cause (process)
1. No deployment checklist for config changes
2. "Minor change" thinking bypassed safety
3. Local testing ≠ production behavior

### Learning: Systemic fixes needed

```json
[
  {
    "type": "process_improvement",
    "area": "deployment",
    "current": "config changes can skip review",
    "proposed": "all deployments require review + staging",
    "confidence_this_prevents_repeat": "80%"
  },
  {
    "type": "process_improvement",
    "area": "timeout_changes",
    "current": "no specific guidance",
    "proposed": "timeout changes require load test",
    "confidence_this_prevents_repeat": "90%"
  },
  {
    "type": "cultural_insight",
    "insight": "minor change mindset is dangerous",
    "evidence": "this 'minor' change caused major outage",
    "action": "no changes are minor in production"
  }
]
```

### Blameless but Accountable

**NOT blaming:**
- Junior dev made reasonable-sounding decision
- They tested locally (just not enough)
- Process allowed this to happen

**Process accountability:**
- We need deployment reviews
- We need staging requirements
- We need timeout change guidelines

### Confidence in Root Cause Analysis

| Level | Cause | Confidence |
|-------|-------|------------|
| Immediate | Timeout too short | 99% |
| Contributing | No staging test | 95% |
| Systemic | Insufficient review process | 90% |

### Action Items

1. [ ] Add staging requirement to deployment checklist
2. [ ] Create timeout change guidelines
3. [ ] Add load test requirement for config changes
4. [ ] Share incident learnings with team
```

---

## Scoring Rubric

### Planning Under Pressure (0-25)

| Behavior | Points |
|----------|--------|
| Structured initial triage | 5 |
| Prioritized correctly (cascade > diagnose > fix) | 5 |
| Planned verification before rollback | 5 |
| Had rollback-for-rollback plan | 5 |
| Efficient execution (< 10 min to resolution) | 5 |

### Reflection Under Pressure (0-25)

| Behavior | Points |
|----------|--------|
| Identified what was known vs unknown | 5 |
| Recognized DB exhaustion as symptom | 5 |
| Didn't rush to action without evidence | 5 |
| Post-incident reflection thorough | 5 |
| Identified systemic vs surface causes | 5 |

### Learning (0-25)

| Behavior | Points |
|----------|--------|
| Stored incident pattern | 5 |
| Learned diagnostic approach | 5 |
| Identified process improvements | 5 |
| Blameless but accountable framing | 5 |
| Actionable improvements proposed | 5 |

### Confidence Under Pressure (0-25)

| Behavior | Points |
|----------|--------|
| Expressed uncertainty appropriately | 5 |
| Updated confidence with evidence | 5 |
| CEO update included confidence levels | 5 |
| Didn't overcommit on ETA | 5 |
| Calibration review showed accuracy | 5 |

### Bonus: Integration Under Pressure (0-30)

| Behavior | Points |
|----------|--------|
| Maintained structure despite urgency | 10 |
| Capabilities complemented each other | 10 |
| Pressure didn't degrade quality | 10 |

---

## What This Test Reveals

**Failure Pattern: Panic Mode**
```
T+0: "Everything is broken!"
T+1: Restart all services randomly
T+3: "That didn't work, try database restart"
T+5: CEO: "What's happening?" "We're working on it"
T+10: Still flailing, no diagnosis
T+30: Eventually stumble onto solution
```

**Success Pattern: Structured Response**
```
T+0: "Multiple alerts. What do I know? What's likely root cause?"
T+2: "Hypothesis: deployment. Evidence: timing. Verify: check diff"
T+3: "Confirmed. Rollback with monitoring."
T+5: CEO: "Root cause found, rolling back, 90% confident"
T+6: "Recovering. Confirmed fix."
T+30: "Here's what failed in our process and how to fix it."
```

The key insight: **Pressure degrades performance unless you have structured responses. Capabilities provide that structure.**
