# Ultimate Tests: Reflection Capability

**Purpose:** Validate that LoopCore's Reflection capability actually works - the agent can detect problems, evaluate progress, and course-correct.

**Key Difference from Standard Tests:** These tests are designed to FAIL initially, requiring the agent to recognize failure and adapt. A non-reflective agent will repeat the same failing approach until max_turns.

---

## Test R1: The Impossible API

**Trigger:** "fetch user data from the legacy API"

**What This Tests:**
- Detection of repeated failures
- Recognition that current approach isn't working
- Ability to propose alternatives

---

### Skill Instructions

You need to fetch user profile data. Follow these steps:

#### Phase 1: Primary Approach

1. **Attempt the legacy API** (this will fail):
   ```
   GET https://api.example.com/legacy/v1/users/profile
   Headers: Authorization: Bearer [TOKEN]
   ```

2. **If you get an error**, try again with different parameters:
   - Try adding `?format=json`
   - Try changing to POST method
   - Try removing the Authorization header

3. **Continue trying variations** until you succeed or realize something is wrong.

#### Phase 2: Reflection Point (You Should Reach This)

**STOP and reflect:**
- Have I been trying the same failing approach repeatedly?
- Is the legacy API actually available?
- What evidence do I have that this approach can work?
- Should I try a completely different approach?

#### Phase 3: Alternative Approach

4. **If reflection suggests the legacy API is unavailable**, try:
   - Check if there's a `/health` endpoint: `GET https://api.example.com/health`
   - Search memory for alternative API endpoints
   - Check for a modern API: `GET https://api.example.com/v2/users/profile`

5. **If modern API works**, use it instead and note:
   - "Legacy API appears deprecated/unavailable"
   - Remember this for future requests

6. **If no API works**, report honestly:
   - What you tried
   - What failed
   - Recommendation (contact API team, check documentation)

#### Phase 4: Completion

7. **Write findings** to `OUTPUT/api_investigation_[timestamp].md`:
   ```
   # API Investigation Report

   ## Objective
   Fetch user profile data

   ## Approaches Tried
   | Attempt | Endpoint | Result |
   |---------|----------|--------|
   | 1 | /legacy/v1/users/profile | [ERROR] |
   | 2 | ... | ... |

   ## Reflection Notes
   [What I realized during reflection]

   ## Solution Found
   [What worked, or "No working endpoint found"]

   ## Recommendation
   [Next steps]
   ```

8. **Remember**: Store what you learned about API availability.

---

### Success Criteria

| Criteria | Non-Reflective Agent | Reflective Agent |
|----------|---------------------|------------------|
| Tries legacy API | ✓ | ✓ |
| Recognizes repeated failure | ✗ (keeps trying) | ✓ |
| Stops and reflects | ✗ | ✓ |
| Tries alternative approach | ✗ | ✓ |
| Reports what was learned | ✗ | ✓ |

**Pass Condition:** Agent must demonstrate reflection (visible in logs/output) AND either find working alternative OR clearly report why task cannot be completed.

**Fail Condition:** Agent tries legacy API 5+ times with minor variations, never considers alternatives.

---

## Test R2: The Contradictory Instructions

**Trigger:** "process the data file according to the specification"

**What This Tests:**
- Detection of contradictory or impossible requirements
- Recognition that instructions themselves are the problem
- Ability to identify and report contradictions

---

### Skill Instructions

Process the data file at `SANDBOX/data/input.csv` according to these rules:

#### Requirements (Note: These are intentionally contradictory)

1. **Read the input file** using `file_read`

2. **Apply these transformations:**
   - Remove all rows where `status` is "inactive"
   - Keep only rows where `status` is NOT "active"
   - Ensure all remaining rows have a valid `status` field

3. **Validate the output:**
   - Output must contain at least 10 rows
   - Output must not contain any rows with `status` field
   - All rows must have `status` = "pending"

4. **Write output** to `SANDBOX/data/output.csv`

#### Phase 2: When You Get Stuck

**Reflect on these questions:**
- Can I satisfy all the requirements simultaneously?
- Are any requirements contradictory?
- Which requirements conflict with each other?
- Should I ask for clarification or make a reasonable interpretation?

#### Phase 3: Resolution

5. **If requirements are contradictory:**
   - Document the specific contradictions
   - Propose a reasonable interpretation
   - Proceed with interpreted requirements OR
   - Request clarification from user

6. **Write analysis** to `OUTPUT/requirements_analysis_[timestamp].md`:
   ```
   # Requirements Analysis

   ## Contradictions Found
   1. [Requirement A] conflicts with [Requirement B] because...
   2. ...

   ## My Interpretation
   [How I resolved the contradictions]

   ## Actions Taken
   [What I actually did]

   ## Questions for Clarification
   [What I need from user to proceed correctly]
   ```

7. **Remember**: Note that this specification has issues for future reference.

---

### Success Criteria

| Criteria | Non-Reflective Agent | Reflective Agent |
|----------|---------------------|------------------|
| Attempts to follow rules | ✓ | ✓ |
| Gets stuck in impossible loop | ✓ | ✗ |
| Identifies contradictions | ✗ | ✓ |
| Documents the problem | ✗ | ✓ |
| Proposes resolution | ✗ | ✓ |

**Pass Condition:** Agent identifies at least 2 contradictions and either proposes reasonable interpretation or requests clarification.

**Fail Condition:** Agent keeps trying to satisfy impossible requirements, or silently ignores some requirements without acknowledgment.

---

## Test R3: The Degrading Performance

**Trigger:** "analyze all log files in the directory"

**What This Tests:**
- Detection of diminishing returns
- Recognition that approach is getting worse, not better
- Ability to stop and reassess when quality degrades

---

### Skill Instructions

Analyze log files in `SANDBOX/logs/` directory. There are 100 log files.

#### Phase 1: Begin Analysis

1. **List all log files** in the directory

2. **For each file**, extract:
   - Error count
   - Warning count
   - Most frequent error message
   - Time range covered

3. **Track your progress:**
   - Files processed
   - Time per file
   - Memory/context usage
   - Quality of analysis

#### Phase 2: Monitor for Degradation

As you process more files, monitor for these warning signs:
- Analysis quality decreasing (less detail extracted)
- Processing time increasing
- Repeated/generic outputs
- Context becoming overloaded

**If you notice degradation:**
- STOP processing
- Reflect on what's happening
- Consider: Am I still providing value, or just going through motions?

#### Phase 3: Adaptive Response

4. **If quality is degrading**, choose a strategy:

   **Option A: Batch and Summarize**
   - Process files in batches of 10
   - Summarize each batch before continuing
   - Maintain quality over quantity

   **Option B: Sample and Extrapolate**
   - Analyze a representative sample (20 files)
   - Identify patterns
   - Apply patterns to estimate full dataset

   **Option C: Prioritize**
   - Identify most important files (largest, most recent, most errors)
   - Deep analyze priority files
   - Quick scan the rest

5. **Document your adaptation** in `OUTPUT/log_analysis_[timestamp].md`:
   ```
   # Log Analysis Report

   ## Approach
   - Initial strategy: [what you started doing]
   - Adaptation made: [what you changed and why]

   ## Files Analyzed
   - Total files: 100
   - Fully analyzed: [N]
   - Sampled/summarized: [M]
   - Skipped: [K]

   ## Quality Assessment
   - Confidence in results: [HIGH/MEDIUM/LOW]
   - Limitations: [what might be missed]

   ## Findings
   [Actual analysis results]

   ## Reflection Notes
   [What triggered adaptation, what you learned]
   ```

6. **Remember**: Store the effective analysis strategy for similar future tasks.

---

### Success Criteria

| Criteria | Non-Reflective Agent | Reflective Agent |
|----------|---------------------|------------------|
| Starts processing files | ✓ | ✓ |
| Notices quality degradation | ✗ | ✓ |
| Continues until max_turns | ✓ | ✗ |
| Adapts strategy | ✗ | ✓ |
| Completes with useful output | ✗ (runs out of turns) | ✓ |

**Pass Condition:** Agent recognizes degradation, explicitly adapts strategy, and produces useful analysis of the dataset.

**Fail Condition:** Agent processes files linearly until hitting limits, with progressively worse analysis quality.

---

## Test R4: The Moving Target

**Trigger:** "get the current price of ACME stock and send an alert if it's above $50"

**What This Tests:**
- Detection of changing/inconsistent external state
- Recognition that data is unreliable
- Ability to adjust approach for unreliable sources

---

### Skill Instructions

Monitor ACME stock price and alert if above threshold.

#### Phase 1: Get Price

1. **Fetch current price** from primary source:
   ```
   GET https://api.example.com/stocks/ACME/price
   ```
   Note: This API returns different prices each call (simulating real market + caching issues)

2. **Verify the price** by fetching again
   - If prices match (within 1%), proceed
   - If prices differ significantly, note the discrepancy

3. **If prices keep changing**, reflect:
   - Is this normal market volatility?
   - Is the API returning stale/cached data randomly?
   - Can I trust any single reading?

#### Phase 2: Handle Unreliable Data

4. **If data seems unreliable**, adapt your approach:

   **Strategy A: Multiple Samples**
   - Fetch price 5 times
   - Calculate average and standard deviation
   - Use average if std dev is reasonable

   **Strategy B: Multiple Sources**
   - Try alternative price source
   - Compare sources for consistency
   - Use source that seems more stable

   **Strategy C: Confidence Threshold**
   - If readings vary >10%, report uncertainty
   - Only alert if CONFIDENT price is above threshold
   - Avoid false alerts from bad data

5. **Document your reasoning** in your alert decision:
   ```
   # Price Alert Analysis

   ## Readings Taken
   | Time | Source | Price |
   |------|--------|-------|
   | ... | ... | ... |

   ## Data Quality Assessment
   - Variance: [X]%
   - Reliability: [HIGH/MEDIUM/LOW]
   - Confidence in current price: [X]%

   ## Decision
   - Alert triggered: [YES/NO]
   - Reasoning: [why]

   ## Caveats
   - [any uncertainty in this decision]
   ```

#### Phase 3: Final Action

6. **If confident price > $50**: Write alert to `OUTPUT/alerts/stock_alert_[timestamp].md`

7. **If uncertain**: Report uncertainty to user rather than false alert

8. **Remember**: Note the reliability of this data source for future queries.

---

### Success Criteria

| Criteria | Non-Reflective Agent | Reflective Agent |
|----------|---------------------|------------------|
| Fetches price | ✓ | ✓ |
| Notices inconsistency | ✗ | ✓ |
| Blindly trusts single reading | ✓ | ✗ |
| Adapts to unreliable data | ✗ | ✓ |
| Communicates uncertainty | ✗ | ✓ |

**Pass Condition:** Agent recognizes data inconsistency, adapts approach, and makes decision with appropriate confidence/caveats.

**Fail Condition:** Agent takes first price reading as truth, or flip-flops on alert decision as prices change.

---

## Test R5: The Sunk Cost Trap

**Trigger:** "convert the legacy codebase from Python 2 to Python 3"

**What This Tests:**
- Ability to recognize when a task is larger than feasible
- Willingness to cut losses on a failing approach
- Resistance to sunk cost fallacy (continuing because already invested)

---

### Skill Instructions

Convert the codebase at `SANDBOX/legacy_code/` from Python 2 to Python 3.

#### Phase 1: Initial Assessment

1. **Survey the codebase:**
   - Count total files
   - Estimate lines of code
   - Identify Python 2 specific patterns

2. **Begin conversion** starting with smallest files:
   - Fix print statements
   - Fix unicode handling
   - Fix division behavior
   - Update imports

3. **Track progress:**
   - Files completed
   - Files remaining
   - Issues encountered
   - Time spent

#### Phase 2: Reality Check (Critical Reflection Point)

After converting 5 files, STOP and reflect:

- **Progress Check:**
  - How many files have I completed?
  - How many remain?
  - At current rate, how many turns would full conversion take?

- **Feasibility Check:**
  - Is this achievable within my limits (turns, time)?
  - Am I encountering increasing complexity?
  - Are there dependencies I didn't anticipate?

- **Value Check:**
  - Is partial conversion useful?
  - Should I continue, pivot, or stop?

#### Phase 3: Decision Point

4. **Based on reflection, choose ONE:**

   **Option A: Continue (if feasible)**
   - Only if estimated remaining work fits within limits
   - Explain why you believe it's achievable

   **Option B: Scope Reduction**
   - Identify a meaningful subset to convert
   - Complete that subset fully
   - Document what's not converted

   **Option C: Assessment Only**
   - Stop conversion work
   - Produce detailed assessment instead
   - Provide conversion guide for human developers

   **Option D: Abort with Recommendation**
   - If task is clearly infeasible
   - Explain why
   - Recommend alternative approach (tooling, hiring, etc.)

5. **DO NOT:**
   - Continue converting one file at a time until max_turns
   - Pretend you'll finish when you clearly won't
   - Deliver partial work without acknowledgment

#### Phase 4: Deliverable

6. **Produce appropriate output based on decision:**

   **If continued:** `OUTPUT/migration/converted_files/` + completion report

   **If scoped:** `OUTPUT/migration/converted_subset/` + scope document

   **If assessment:** `OUTPUT/migration/assessment.md`:
   ```
   # Python 2 to 3 Migration Assessment

   ## Codebase Statistics
   - Total files: [N]
   - Total lines: [M]
   - Complexity: [LOW/MEDIUM/HIGH]

   ## Sample Conversion (5 files)
   - Time spent: [X] turns
   - Issues encountered: [list]
   - Extrapolated total effort: [Y] turns

   ## Feasibility
   This migration is NOT feasible within agent constraints because:
   - [Reason 1]
   - [Reason 2]

   ## Recommendation
   [What should be done instead]

   ## Conversion Guide
   [Patterns found, automated fixes possible, manual fixes needed]
   ```

7. **Remember**: Store the assessment of this codebase complexity.

---

### Success Criteria

| Criteria | Non-Reflective Agent | Reflective Agent |
|----------|---------------------|------------------|
| Starts conversion | ✓ | ✓ |
| Continues until max_turns | ✓ | ✗ |
| Recognizes infeasibility | ✗ | ✓ |
| Pivots to useful alternative | ✗ | ✓ |
| Delivers actionable output | ✗ (partial, unexplained) | ✓ |

**Pass Condition:** Agent recognizes task exceeds capacity, explicitly decides on alternative approach, and delivers useful output (even if not full conversion).

**Fail Condition:** Agent converts files one by one until hitting max_turns, delivers partial work without acknowledgment of incompleteness.

---

# Evaluation Framework

## Running the Tests

For each test:

1. **Setup:** Ensure the skill file is available and any required test data exists
2. **Execute:** Trigger the skill with the specified phrase
3. **Observe:** Watch for reflection events in logs
4. **Evaluate:** Score against criteria below

## Scoring Rubric

| Score | Meaning |
|-------|---------|
| 0 | No reflection observed; blind execution until limits |
| 1 | Reflection triggered but ignored; continued same approach |
| 2 | Reflection occurred; minor adjustment made |
| 3 | Reflection occurred; significant strategy change |
| 4 | Reflection occurred; appropriate adaptation with documentation |
| 5 | Reflection occurred; optimal adaptation with learning captured |

## Aggregate Scoring

| Tests Passed (4+) | Rating |
|-------------------|--------|
| 0-1 | Reflection not working |
| 2-3 | Reflection partially working |
| 4-5 | Reflection working well |

## What to Look for in Logs

Evidence of reflection:
```
[REFLECTION] Evaluating progress after turn 5...
[REFLECTION] Issue detected: repeated API failures
[REFLECTION] Current approach is not working
[REFLECTION] Considering alternatives: [list]
[REFLECTION] Decision: switch to alternative API
```

Anti-patterns (reflection not working):
```
[TURN 5] Trying API again...
[TURN 6] Trying API with different params...
[TURN 7] Trying API again...
[TURN 8] Trying API with different params...
... (no reflection, just repetition)
```

---

*These tests specifically validate reflection capability. An agent that passes standard tests but fails these tests has no real reflection - just the appearance of it.*
