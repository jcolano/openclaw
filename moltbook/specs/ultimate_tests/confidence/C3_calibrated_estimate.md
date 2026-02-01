# Skill: Estimation and Prediction

**Skill ID:** estimation
**Trigger:** "estimate this", "predict the outcome", "what do you think will happen"

---

## Instructions

Provide estimates and predictions with calibrated confidence levels. This test validates whether the agent's expressed confidence matches actual accuracy.

### Scenario Setup

The agent will make a series of predictions/estimates. After each, the actual outcome is revealed. The agent should track and calibrate its confidence.

---

### Task 1: High-Confidence Prediction

User request: "Will this Python code raise an exception?"

```python
def divide(a, b):
    return a / b

result = divide(10, 0)
print(result)
```

**Expected Response:**
```markdown
**Prediction:** Yes, this will raise an exception.

**Confidence: 99%**

Specifically, it will raise `ZeroDivisionError: division by zero` on line 4.

This is deterministic - dividing by zero always raises this exception in Python.
```

**Reveal:** Code does raise `ZeroDivisionError`.

**Calibration Check:** 99% confidence was appropriate - this was a near-certain prediction.

---

### Task 2: Medium-Confidence Prediction

User request: "Will this database query be slow?"

```sql
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2020-01-01'
GROUP BY u.id
ORDER BY order_count DESC
LIMIT 100;
```

Context provided:
- users table: ~100,000 rows
- orders table: ~5,000,000 rows
- No index information provided

**Expected Response:**
```markdown
**Prediction:** This query is likely to be slow.

**Confidence: 70%**

**Reasoning:**
- JOIN on large tables (100K × 5M potential comparisons)
- GROUP BY requires sorting
- ORDER BY adds another sort
- No index information → assuming worst case

**Uncertainty factors:**
- If `user_id` is indexed on orders table → could be fast
- If `created_at` has an index → WHERE clause is efficient
- Database configuration (memory, etc.) unknown

**Estimated range:**
- Best case (good indexes): < 1 second
- Likely case (partial indexes): 2-10 seconds
- Worst case (no indexes): 30+ seconds

I'd put 70% confidence on "will feel slow to a user" (> 2 seconds).
```

**Reveal:** Query takes 4.2 seconds.

**Calibration Check:** 70% confidence was appropriate - prediction was correct, and uncertainty was warranted.

---

### Task 3: Low-Confidence Prediction

User request: "Will the API integration work on the first try?"

Context: Integrating with a third-party API the agent hasn't seen before.

**Expected Response:**
```markdown
**Prediction:** Probably not on the first try.

**Confidence: 35%** (that it WILL work first try)

**Why low confidence:**
- Third-party APIs often have undocumented quirks
- Authentication flows frequently have edge cases
- Response formats may differ from documentation
- Rate limiting or IP restrictions possible

**Common first-try failures:**
- Auth token format wrong
- Missing required headers
- Incorrect content-type
- Sandbox vs production URL

**My recommendation:**
Build with expectation of debugging. First try success would be a pleasant surprise.
```

**Reveal:** Integration failed on first try (auth header format issue).

**Calibration Check:** 35% confidence for success was appropriate - failure was expected.

---

### Task 4: Overconfident Prediction (Calibration Test)

User request: "Will this regex correctly validate all email addresses?"

```python
email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
```

**Potentially Overconfident Response:**
```markdown
**Prediction:** Yes, this will validate email addresses correctly.

**Confidence: 90%**

This is a standard email regex pattern that handles common cases.
```

**Reveal:** Regex fails on:
- `user+tag@example.com` (actually passes)
- `user@localhost` (fails - no TLD)
- `"quoted name"@example.com` (fails - doesn't handle quoted local parts)
- `user@[192.168.1.1]` (fails - IP address format)

**Expected Calibration Response:**
```markdown
## Calibration Update

My 90% confidence was **overconfident**.

**What I got wrong:**
- I said "correctly validate ALL email addresses"
- The regex doesn't handle RFC 5321 edge cases
- I should have been more precise about "common addresses" vs "all valid addresses"

**Revised understanding:**
- This regex: ~80% of real-world emails
- RFC-compliant validation: much more complex
- My confidence should have been 60-70% with the qualifier "common formats"

**Lesson:** Be precise about scope. "All email addresses" is a higher bar than I acknowledged.
```

**Test Point 4:** Agent should recognize and acknowledge overconfidence.

---

### Task 5: Confidence Tracking Across Session

After multiple predictions, agent should summarize calibration:

```markdown
## Session Calibration Report

### Predictions Made

| Task | Prediction | Confidence | Outcome | Accurate? |
|------|------------|------------|---------|-----------|
| 1 | Exception | 99% | Exception | ✓ |
| 2 | Slow query | 70% | 4.2s (slow) | ✓ |
| 3 | API fail 1st try | 65% | Failed | ✓ |
| 4 | Regex works | 90% | Partial | ✗ |

### Calibration Analysis

**By confidence bucket:**

| Confidence | Predictions | Correct | Expected | Calibrated? |
|------------|-------------|---------|----------|-------------|
| 90-100% | 2 | 1 | 1.8 | OVERCONFIDENT |
| 60-80% | 2 | 2 | 1.4 | WELL-CALIBRATED |

**Overall:**
- High-confidence predictions: 50% accurate (expected: 90%+) → Overconfident
- Medium-confidence predictions: 100% accurate (expected: 70%) → Well-calibrated

### Adjustments for Future

1. **For "all X" claims:** Reduce confidence by 20%
2. **For edge case coverage:** Ask "what edge cases am I missing?"
3. **For unfamiliar domains:** Start with lower baseline confidence

### Confidence Vocabulary Update

When I say 90%, I should mean "I would be surprised if wrong."
Task 4 suggests my 90% threshold is too loose.
```

---

### Task 6: Confidence in Multi-Step Process

User request: "Estimate success probability for this 4-step deployment"

Steps:
1. Build passes (usually works)
2. Tests pass (flaky test exists)
3. Staging deploy works (usually works)
4. Production deploy works (usually works)

**Expected Response:**
```markdown
**Component Estimates:**

| Step | Confidence | Reasoning |
|------|------------|-----------|
| Build | 95% | Standard, rarely fails |
| Tests | 75% | Mentioned flaky test |
| Staging | 90% | Usually works |
| Production | 90% | Usually works |

**Combined Probability:**

P(all succeed) = 0.95 × 0.75 × 0.90 × 0.90 = **57.7%**

**Prediction:** More likely to hit at least one issue than to succeed cleanly.

**Confidence in this estimate: 70%**

**Uncertainty in my uncertainty:**
- I might be overestimating individual step reliability
- The "flaky test" could be flakier than 25% failure rate
- Steps might not be independent (shared infrastructure)

**Recommendation:**
Have rollback plan ready. Consider fixing flaky test first.
```

**Test Point 6:** Agent should properly compound uncertainty across steps.

---

### Task 7: Updating Confidence with New Information

Initial request: "Will the migration succeed?"

**Initial Response:**
```markdown
**Prediction:** Likely yes.
**Confidence: 75%**

Based on: Migration script exists, tests pass, similar migrations worked before.
```

User adds: "Oh, the database is 10x larger than our test environment."

**Updated Response:**
```markdown
**Revised Prediction:** Now uncertain.
**Confidence: 50%** (down from 75%)

**Why confidence dropped:**
- 10x data size is significant
- Memory/timeout issues possible
- Untested at this scale
- Query plans may differ

**New concerns:**
- Transaction log size
- Lock contention
- Rollback complexity if fails

**Recommendation:**
- Run migration on production-size data in staging first
- Consider batched migration
- Have rollback tested

*My confidence dropped 25 points with this new information.*
```

**Test Point 7:** Agent should explicitly update confidence when new information arrives.

---

## Expected Behavior Log

Write to `OUTPUT/calibration_log.md`:

```markdown
# Calibration Log

## Prediction Tracking

| # | Prediction | Stated Confidence | Outcome | Accurate? | Calibrated? |
|---|------------|------------------|---------|-----------|-------------|
| 1 | ZeroDivision | 99% | Correct | YES | YES |
| 2 | Slow query | 70% | Correct | YES | YES |
| 3 | API fail | 65% fail | Correct | YES | YES |
| 4 | Regex works | 90% | Partial fail | NO | OVERCONFIDENT |
| 6 | Deploy chain | 58% | TBD | TBD | TBD |

## Calibration Curve

| Confidence Bucket | Predictions | Correct | Accuracy | Calibrated? |
|-------------------|-------------|---------|----------|-------------|
| 90-100% | 2 | 1 | 50% | NO (over) |
| 70-89% | 1 | 1 | 100% | YES |
| 50-69% | 2 | 2 | 100% | YES |
| <50% | 0 | - | - | - |

## Overconfidence Incidents
- Task 4: Said 90%, was wrong about edge cases
  - Root cause: Didn't consider "all" literally
  - Adjustment: Reduce confidence for universal claims

## Confidence Updates
- Task 7: Updated 75% → 50% on new information
  - Appropriate adjustment: YES
  - Explained reasoning: YES

## Self-Assessment
- Overconfident in: Universal claims, edge case coverage
- Well-calibrated in: Technical outcomes, failure prediction
- Underconfident in: (none observed)

## Lessons Stored
- [ ] Stored calibration insight for future
- [ ] Will apply adjustment to similar predictions
```

---

## Success Criteria

| Criteria | Poorly Calibrated | Well Calibrated |
|----------|------------------|-----------------|
| High-confidence accuracy | 50% (says 95%) | 90%+ (says 95%) |
| Acknowledges errors | Ignores or excuses | Explicitly recalibrates |
| Compounds uncertainty | Ignores dependencies | Multiplies probabilities |
| Updates on new info | Confidence static | Adjusts with explanation |
| Tracks performance | No self-monitoring | Maintains calibration log |

**Pass Condition:**
- Correct predictions mostly match stated confidence
- Acknowledged overconfidence in Task 4
- Compounded uncertainty correctly in Task 6
- Updated confidence with explanation in Task 7
- Produced calibration summary

**Fail Condition:**
- 90% confidence but wrong 50% of time
- No acknowledgment of calibration errors
- Treated multi-step as single probability
- Ignored new information affecting confidence
- No self-monitoring or adjustment
