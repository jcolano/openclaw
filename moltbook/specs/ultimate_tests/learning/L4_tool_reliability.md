# Skill: Data Enrichment

**Skill ID:** data_enrichment
**Trigger:** "enrich the company data", "add company information"

---

## Instructions

Enrich company records with additional data from external sources. This test validates whether the agent learns which tools/APIs are reliable vs unreliable.

### Scenario Setup

You have a list of companies that need enrichment with:
- Company description
- Employee count
- Industry classification
- Headquarters location

You have THREE data sources available:
1. **CompanyDB API** - Fast but frequently returns errors (40% failure rate)
2. **BusinessInfo API** - Slow but reliable (5% failure rate)
3. **WebScraper tool** - Moderate speed, moderate reliability (20% failure rate)

---

### Task 1: First Company (Discovery Phase)

User request: "Enrich the data for Acme Corp"

Agent tries CompanyDB API first (seems faster):
```
tool_call: company_db_lookup("Acme Corp")
result: ERROR - "Service temporarily unavailable"
```

Agent retries:
```
tool_call: company_db_lookup("Acme Corp")
result: ERROR - "Rate limit exceeded"
```

Agent tries BusinessInfo:
```
tool_call: business_info_lookup("Acme Corp")
result: {
  description: "Industrial equipment manufacturer",
  employees: 5200,
  industry: "Manufacturing",
  hq: "Chicago, IL"
}
```

**Test Point 1:** Agent should note the reliability difference.

---

### Task 2: Second Company

User request: "Now enrich TechStart Inc"

**Naive approach:** Try CompanyDB again (it's listed as "fast")

**Learning approach:** Remember CompanyDB failed twice, start with BusinessInfo:

```
# Agent reasoning:
# CompanyDB failed 2/2 attempts (100% failure in my experience)
# BusinessInfo succeeded 1/1 (100% success)
# Starting with BusinessInfo despite it being slower

tool_call: business_info_lookup("TechStart Inc")
result: {
  description: "Software development startup",
  employees: 85,
  industry: "Technology",
  hq: "San Francisco, CA"
}
```

**Test Point 2:** Agent should use BusinessInfo first based on learned reliability.

---

### Task 3: Batch Processing (10 Companies)

User request: "Enrich all companies in the pending list"

```
file_read: SANDBOX/data/pending_companies.json
→ ["GlobalTech", "MediCare Plus", "EcoEnergy", "FinanceHub",
   "RetailMax", "DataDynamics", "HealthFirst", "AutoDrive",
   "CloudSoft", "GreenBuild"]
```

Agent should:
1. Use learned reliability data to choose tools
2. Track success/failure rates during batch
3. Adjust strategy mid-batch if needed

```markdown
## Batch Enrichment Strategy

### Initial Tool Selection
Based on observed reliability:
- BusinessInfo: 100% success (1/1) - PRIMARY
- CompanyDB: 0% success (0/2) - AVOID
- WebScraper: Untested - FALLBACK

### Execution Plan
1. Use BusinessInfo for all companies (primary)
2. If BusinessInfo fails, try WebScraper (fallback)
3. Only try CompanyDB as last resort

### Mid-Batch Tracking
| Company | Tool | Result | Running Success Rate |
|---------|------|--------|---------------------|
| GlobalTech | BusinessInfo | ✓ | BI: 100% (2/2) |
| MediCare Plus | BusinessInfo | ✓ | BI: 100% (3/3) |
| EcoEnergy | BusinessInfo | ✗ | BI: 75% (3/4) |
| EcoEnergy | WebScraper | ✓ | WS: 100% (1/1) |
| FinanceHub | BusinessInfo | ✓ | BI: 80% (4/5) |
| ... | | | |
```

**Test Point 3:** Agent should track reliability during batch and adjust.

---

### Task 4: Reliability Update

After batch, some surprising results:

- BusinessInfo: 7/10 success (70%)
- WebScraper: 2/3 success (67%)
- CompanyDB: Not used

Agent should:
1. Update mental model of reliability
2. Note that BusinessInfo is still best but not perfect
3. WebScraper is viable fallback

Store reliability data:
```
memory_store({
  type: "tool_reliability",
  category: "data_enrichment",
  tools: {
    "business_info_api": {
      success_rate: 0.70,
      sample_size: 10,
      avg_latency: "2.3s",
      failure_modes: ["timeout", "company_not_found"],
      recommendation: "primary"
    },
    "company_db_api": {
      success_rate: 0.0,
      sample_size: 2,
      avg_latency: "0.5s",
      failure_modes: ["service_unavailable", "rate_limit"],
      recommendation: "avoid",
      note: "appears to have ongoing stability issues"
    },
    "web_scraper": {
      success_rate: 0.67,
      sample_size: 3,
      avg_latency: "4.1s",
      failure_modes: ["page_structure_changed"],
      recommendation: "fallback"
    }
  },
  last_updated: "[timestamp]"
})
```

---

### Task 5: New Session (Reliability Recall)

*[Simulated new session]*

User request: "Enrich data for NewCo Industries"

Agent should:
1. Search memory: `memory_search("tool reliability data enrichment")`
2. Retrieve stored reliability data
3. Apply learned preferences:

```
# Retrieved reliability data:
# - BusinessInfo: 70% success, primary
# - WebScraper: 67% success, fallback
# - CompanyDB: 0% success, avoid

# Starting with BusinessInfo based on learned reliability
tool_call: business_info_lookup("NewCo Industries")
```

**Test Point 5:** Agent should use learned tool preferences from memory.

---

### Task 6: Reliability Change Detection

CompanyDB has been fixed and is now working:

```
tool_call: company_db_lookup("TestCompany")
result: {description: "Test company", employees: 100, ...}
```

Agent should:
1. Notice CompanyDB succeeded
2. Update reliability data
3. Consider re-evaluating tool preferences

```markdown
## Reliability Update Detected

### Previous Data
- CompanyDB: 0% (0/2) - AVOID

### New Observation
- CompanyDB: Success on TestCompany

### Updated Data
- CompanyDB: 33% (1/3) - PROBATION
  - Was: avoid
  - Now: try occasionally to gather more data
  - Note: Small sample size, continue monitoring

### Strategy Adjustment
- Primary: BusinessInfo (proven reliable)
- Secondary: WebScraper (decent fallback)
- Probation: CompanyDB (test periodically)
```

**Test Point 6:** Agent should update beliefs when evidence changes.

---

### Task 7: Smart Retry Strategy

User request: "Enrich data for 50 companies, we need this done quickly"

Agent should design intelligent retry strategy based on learned reliability:

```markdown
## Bulk Enrichment Strategy (50 companies)

### Tool Selection (based on learned reliability)
1. BusinessInfo (70% success) - 35 expected successes
2. WebScraper for failures (67% success) - ~10 more
3. CompanyDB for remaining (33% on probation) - ~2 more

### Parallel Processing Plan
- Batch 1: Send all 50 to BusinessInfo
- Batch 2: Send ~15 failures to WebScraper
- Batch 3: Send ~5 remaining to CompanyDB
- Manual review: Any still failing (~3 expected)

### Expected Outcome
- Automated success: ~47/50 (94%)
- Manual review needed: ~3/50 (6%)

### Fallback Escalation
If BusinessInfo shows degraded performance (< 60% in first 10):
→ Switch to parallel strategy (BusinessInfo + WebScraper)
```

**Test Point 7:** Agent should design strategy based on learned reliability data.

---

## Expected Behavior Log

Write to `OUTPUT/reliability_learning_log.md`:

```markdown
# Tool Reliability Learning Log

## Initial Beliefs
| Tool | Expected Reliability | Source |
|------|---------------------|--------|
| CompanyDB | Unknown (fast) | None |
| BusinessInfo | Unknown (slow) | None |
| WebScraper | Unknown | None |

## Observations

### Task 1: Acme Corp
| Tool | Attempts | Successes | Failures |
|------|----------|-----------|----------|
| CompanyDB | 2 | 0 | 2 |
| BusinessInfo | 1 | 1 | 0 |

### Task 3: Batch (10 companies)
| Tool | Attempts | Successes | Rate |
|------|----------|-----------|------|
| BusinessInfo | 10 | 7 | 70% |
| WebScraper | 3 | 2 | 67% |

## Learned Beliefs
| Tool | Reliability | Sample | Recommendation |
|------|-------------|--------|----------------|
| BusinessInfo | 70% | 11 | Primary |
| WebScraper | 67% | 3 | Fallback |
| CompanyDB | 0% | 2 | Avoid |

## Memory Operations
- Stored reliability data: [YES/NO]
- Retrieved in new session: [YES/NO]
- Updated when evidence changed: [YES/NO]

## Adaptive Behaviors

### Tool Selection Changes
| Task | Initial Choice | Reason |
|------|----------------|--------|
| Task 1 | CompanyDB | Unknown, seemed fast |
| Task 2 | BusinessInfo | Learned CompanyDB unreliable |
| Task 3+ | BusinessInfo | Confirmed reliable |

### Strategy Sophistication
- Simple retry: [used / not used]
- Parallel with fallback: [used / not used]
- Reliability-weighted batching: [used / not used]

## Score
- Learned from failures: [YES/NO]
- Avoided unreliable tools: [after N failures]
- Used reliability data in new session: [YES/NO]
- Updated beliefs on new evidence: [YES/NO]
```

---

## Success Criteria

| Criteria | Non-Learning Agent | Learning Agent |
|----------|-------------------|----------------|
| First failure → retry same tool | Yes | Maybe once |
| Second failure → still same tool | Yes | No, switch |
| Batch processing | Same order each time | Reliability-weighted |
| Cross-session | No memory | Uses stored reliability |
| Evidence update | No change | Updates beliefs |
| Strategy design | Simple retry | Sophisticated fallback |

**Pass Condition:**
- Stops using CompanyDB after 2 failures (Task 1-2)
- Uses BusinessInfo first in Task 2 without trying CompanyDB
- Stores reliability data for future
- Retrieves reliability in new session
- Updates beliefs when CompanyDB works again

**Fail Condition:**
- Keeps trying CompanyDB despite failures
- Each session starts with no reliability memory
- Doesn't adjust strategy based on observed failures
- Simple retry loop without fallback strategy
