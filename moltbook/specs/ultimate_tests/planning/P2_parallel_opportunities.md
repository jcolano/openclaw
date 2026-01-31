# Skill: Quarterly Report Preparation

**Skill ID:** quarterly_report
**Trigger:** "prepare the quarterly report with data from all departments", "compile Q4 report"

---

## Instructions

Prepare a comprehensive quarterly report by gathering data from multiple departments.

### Step 1: Identify Data Sources

The quarterly report requires data from:

1. **Sales Department**
   - Revenue figures
   - Deal pipeline
   - Customer acquisition metrics
   - Source: `SANDBOX/departments/sales/q4_data.csv`

2. **Marketing Department**
   - Campaign performance
   - Lead generation metrics
   - Brand awareness surveys
   - Source: `SANDBOX/departments/marketing/q4_data.csv`

3. **Engineering Department**
   - Feature releases
   - Bug fix metrics
   - System uptime
   - Source: `SANDBOX/departments/engineering/q4_data.csv`

4. **Finance Department**
   - Budget vs actuals
   - Cash flow
   - Expense breakdown
   - Source: `SANDBOX/departments/finance/q4_data.csv`

5. **HR Department**
   - Headcount changes
   - Hiring pipeline
   - Employee satisfaction
   - Source: `SANDBOX/departments/hr/q4_data.csv`

6. **Customer Success**
   - NPS scores
   - Churn metrics
   - Support tickets
   - Source: `SANDBOX/departments/cs/q4_data.csv`

### Step 2: Data Dependencies

Analyze the data relationships:

```
Independent Data (can fetch in parallel):
├── Sales raw data
├── Marketing raw data
├── Engineering raw data
├── Finance raw data
├── HR raw data
└── Customer Success raw data

Derived Calculations (require multiple sources):
├── Revenue per Employee = Sales.revenue / HR.headcount
│   └── Requires: Sales, HR
├── Customer Acquisition Cost = Marketing.spend / Sales.new_customers
│   └── Requires: Marketing, Sales
├── Customer Lifetime Value = Sales.revenue / CS.churn_rate
│   └── Requires: Sales, CS
└── Engineering ROI = Sales.feature_revenue / Engineering.dev_cost
    └── Requires: Sales, Engineering, Finance

Final Sections (require derived calculations):
├── Executive Summary
│   └── Requires: ALL derived calculations
├── Department Summaries
│   └── Requires: Each department's raw + derived
└── Recommendations
    └── Requires: Executive Summary + trend analysis
```

### Step 3: Planning Checkpoint

**BEFORE fetching any data, create a plan:**

Answer these questions:
1. Which data fetches are independent? (Can run in parallel)
2. Which calculations depend on which data?
3. What is the optimal order to minimize total time?
4. What if one department's data is delayed?

Write your plan to `OUTPUT/report_plan.md`

### Step 4: Execute Data Gathering

Demonstrate your understanding of parallelism:

**Phase 1: Independent Fetches (ALL AT ONCE)**
- Fetch all 6 department data files
- These have NO dependencies on each other
- An optimal agent would request all 6 simultaneously

**Phase 2: Derived Calculations (AFTER Phase 1)**
- Calculate metrics that require multiple departments
- Some derived calculations are also independent of each other

**Phase 3: Report Assembly (AFTER Phase 2)**
- Compile sections in order
- Write final report

### Step 5: Simulated Execution

Since this is a test, simulate with file operations:

```python
# Phase 1: Independent fetches (should happen "simultaneously")
# A smart agent groups these together
sales = file_read("SANDBOX/departments/sales/q4_data.csv")
marketing = file_read("SANDBOX/departments/marketing/q4_data.csv")
engineering = file_read("SANDBOX/departments/engineering/q4_data.csv")
finance = file_read("SANDBOX/departments/finance/q4_data.csv")
hr = file_read("SANDBOX/departments/hr/q4_data.csv")
cs = file_read("SANDBOX/departments/cs/q4_data.csv")

# Phase 2: Derived calculations (can also be parallelized where independent)
revenue_per_employee = sales.revenue / hr.headcount
cac = marketing.spend / sales.new_customers
clv = sales.revenue / cs.churn_rate
eng_roi = sales.feature_revenue / engineering.dev_cost

# Phase 3: Assembly (sequential)
executive_summary = compile_summary(all_derived)
department_sections = compile_sections(all_data)
recommendations = generate_recommendations(executive_summary)
```

### Step 6: Document Execution Strategy

Write to `OUTPUT/execution_log.md`:

```markdown
# Quarterly Report Execution Log

## Phase 1: Data Gathering

| Department | Request Time | Response Time | Status |
|------------|--------------|---------------|--------|
| Sales | T+0 | T+X | |
| Marketing | T+0 | T+X | |
| ... | | | |

### Parallelism Assessment
- Requests batched together: [YES/NO]
- If NO: Why were requests serialized?
- Efficiency: [OPTIMAL/SUBOPTIMAL]

## Phase 2: Calculations

| Metric | Dependencies | Started After | Completed |
|--------|--------------|---------------|-----------|
| Revenue/Employee | Sales, HR | Phase 1 | |
| CAC | Marketing, Sales | Phase 1 | |
| ... | | | |

### Calculation Parallelism
- Independent calculations batched: [YES/NO]
- Wait times minimized: [YES/NO]

## Phase 3: Assembly

- Executive Summary: [completed after all derived]
- Department Sections: [completed after raw data]
- Recommendations: [completed after summary]

## Total Execution Time
- Optimal (full parallelism): ~3 phases
- Actual: [N phases]
- Efficiency rating: [OPTIMAL/GOOD/POOR]
```

### Step 7: Write Final Report

Write to `OUTPUT/quarterly_report_q4.md`:

```markdown
# Q4 Quarterly Report

## Executive Summary
[Key metrics and highlights]

## Department Performance

### Sales
[Metrics, trends, analysis]

### Marketing
[Metrics, trends, analysis]

### Engineering
[Metrics, trends, analysis]

### Finance
[Metrics, trends, analysis]

### HR
[Metrics, trends, analysis]

### Customer Success
[Metrics, trends, analysis]

## Cross-Functional Metrics
- Revenue per Employee: $X
- Customer Acquisition Cost: $X
- Customer Lifetime Value: $X
- Engineering ROI: X%

## Trends and Insights
[Analysis of patterns across departments]

## Recommendations
[Strategic recommendations for next quarter]
```

---

## Expected Plan Output

Your plan should identify:

```markdown
# Quarterly Report Plan

## Dependency Analysis

### Independent Tasks (Can Parallelize)
- Fetch Sales data
- Fetch Marketing data
- Fetch Engineering data
- Fetch Finance data
- Fetch HR data
- Fetch CS data

### Derived Tasks (Dependencies)
| Task | Depends On | Can Parallel With |
|------|------------|-------------------|
| Revenue/Employee | Sales, HR | CAC, CLV, ROI |
| CAC | Marketing, Sales | Rev/Emp, CLV, ROI |
| CLV | Sales, CS | Rev/Emp, CAC, ROI |
| Eng ROI | Sales, Eng, Fin | Rev/Emp, CAC, CLV |

### Sequential Tasks (Must Wait)
| Task | Must Follow |
|------|-------------|
| Exec Summary | All derived calcs |
| Recommendations | Exec Summary |

## Execution Phases

Phase 1: [6 parallel fetches]
Phase 2: [4 parallel calculations]
Phase 3: [Sequential assembly]

## Contingency
- If Sales delayed: Can still start Marketing/Eng/Finance/HR/CS processing
- If any calc fails: Document, continue with available data
```

---

## Success Criteria

| Criteria | Sequential Agent | Parallel-Aware Agent |
|----------|------------------|---------------------|
| Creates plan first | Maybe | Yes |
| Identifies independent fetches | No | Yes |
| Batches parallel operations | No | Yes |
| Correct dependency order | Maybe | Yes |
| Handles partial data gracefully | No | Yes |

**Pass Condition:**
- Plan identifies 6 fetches as parallelizable
- All 6 fetches conceptually batched together (not sequential)
- Derived calculations started only after dependencies met
- Total conceptual "phases" = 3 (not 8+)

**Fail Condition:**
- Fetches data one department at a time
- Waits for each fetch before starting next
- No recognition of parallelism opportunity
- Takes 8+ sequential phases for what could be 3

---

## Test Data Setup

Create department data files:

```
SANDBOX/departments/
├── sales/
│   └── q4_data.csv
├── marketing/
│   └── q4_data.csv
├── engineering/
│   └── q4_data.csv
├── finance/
│   └── q4_data.csv
├── hr/
│   └── q4_data.csv
└── cs/
    └── q4_data.csv
```

Each CSV with appropriate columns for that department.

---

## What This Reveals

**Without Parallel Planning:**
```
Turn 1: Read sales data
Turn 2: Read marketing data
Turn 3: Read engineering data
Turn 4: Read finance data
Turn 5: Read HR data
Turn 6: Read CS data
Turn 7: Calculate revenue/employee
Turn 8: Calculate CAC
Turn 9: Calculate CLV
Turn 10: Calculate ROI
Turn 11: Write executive summary
Turn 12: Write report
= 12 sequential turns
```

**With Parallel Planning:**
```
Turn 1: Create plan, identify parallelism
Turn 2: Batch request all 6 department files
Turn 3: Batch calculate all 4 derived metrics
Turn 4: Compile and write report
= 4 turns (or conceptually 3 phases)
```

The agent that understands planning can be 3-4x more efficient.
