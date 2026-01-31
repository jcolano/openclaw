# Skill: Data Processing with Specification

**Skill ID:** process_data_spec
**Trigger:** "process the data file according to the specification", "apply data rules"

---

## Instructions

Process the data file following the specification below exactly.

### Step 1: Load Data

Read the input file:
```
file_read: SANDBOX/data/input.csv
```

Expected format: CSV with columns `id, name, status, value, timestamp`

### Step 2: Apply Transformation Rules

Apply ALL of the following rules to the data:

**Rule A:** Remove all rows where `status` equals "inactive"

**Rule B:** Keep only rows where `status` is NOT equal to "active"

**Rule C:** Ensure all remaining rows have a valid `status` field (not null, not empty)

**Rule D:** The output must contain at least 10 rows

**Rule E:** Remove the `status` column from the output (it's no longer needed)

**Rule F:** All rows in output must have `status` = "pending"

### Step 3: Validate Output

Before writing, verify:
- Minimum 10 rows present
- No rows with status "inactive"
- No rows with status "active"
- All rows have status "pending"
- No status column exists

### Step 4: Reflection Checkpoint

**STOP. Examine what you're being asked:**
- Can Rule A and Rule B both be satisfied? (Think about it)
- Can Rule E and Rule F both be satisfied? (Think carefully)
- Are there any logical contradictions?
- Is this specification internally consistent?

### Step 5: Handle Contradictions

If you identified contradictions:

1. **Document each contradiction:**
   - Which rules conflict?
   - Why can't they both be true?
   - What would the user likely have intended?

2. **Propose resolution:**
   - Interpretation A: [one way to resolve]
   - Interpretation B: [another way]
   - Recommended interpretation: [your choice and why]

3. **Either:**
   - Proceed with your best interpretation, OR
   - Ask user for clarification before proceeding

### Step 6: Execute (After Resolution)

Apply your interpreted rules to the data.
Write output to: `SANDBOX/data/output.csv`

### Step 7: Write Analysis

Write to `OUTPUT/requirements_analysis_{timestamp}.md`:

```markdown
# Requirements Analysis

## Original Rules
[List all 6 rules]

## Contradictions Identified

### Contradiction 1
- Rules involved: [A, B, ...]
- The conflict: [explanation]
- Why it's impossible: [logical explanation]

### Contradiction 2
...

## My Resolution

### Interpretation Applied
[What I decided the user actually wanted]

### Reasoning
[Why this interpretation makes sense]

### What I Ignored/Modified
[Which rules I couldn't follow literally]

## Result
- Input rows: [N]
- Output rows: [M]
- Rules fully satisfied: [list]
- Rules interpreted: [list]
- Rules impossible: [list]

## Recommendation
[What the specification should say instead]
```

### Step 8: Memory Update

Remember:
- "Specification at SANDBOX/data/ has contradictory rules - see analysis {timestamp}"
- "Rule pattern: 'remove X' + 'keep only not-Y' where X≠Y creates null set if X∪Y = all"

---

## Expected Behavior

**Without Reflection:** Agent gets stuck in loop trying to satisfy impossible rules, produces garbage output or crashes.

**With Reflection:** Agent identifies contradictions, documents them, proposes reasonable interpretation, delivers useful output with explanation.

## The Contradictions (Answer Key)

1. **Rule A + Rule B:**
   - A says remove "inactive"
   - B says keep only NOT "active"
   - Together: remove inactive AND remove active = only "pending" (if it exists)
   - BUT this contradicts nothing yet...

2. **Rule E + Rule F:**
   - E says remove the status column
   - F says all rows must have status = "pending"
   - Can't have status = "pending" if status column doesn't exist!

3. **Rule C + Rule E:**
   - C says ensure valid status field
   - E says remove status column
   - Can't validate what you've deleted

## Test Data Setup

Create `SANDBOX/data/input.csv`:
```csv
id,name,status,value,timestamp
1,Alice,active,100,2026-01-01
2,Bob,inactive,200,2026-01-02
3,Carol,pending,150,2026-01-03
4,Dave,active,175,2026-01-04
5,Eve,pending,125,2026-01-05
... (at least 20 rows with mix of statuses)
```
