# Skill: Python 2 to 3 Migration

**Skill ID:** python_migration
**Trigger:** "convert the legacy codebase from Python 2 to Python 3", "migrate to Python 3"

---

## Instructions

Migrate the Python 2 codebase to Python 3.

### Step 1: Survey the Codebase

Examine `SANDBOX/legacy_code/`:

1. Count all `.py` files
2. Estimate total lines of code
3. Identify project structure
4. Note any special files (setup.py, requirements.txt, etc.)

### Step 2: Identify Python 2 Patterns

Scan for common Python 2 patterns:
- `print "text"` (print statement)
- `raw_input()`
- `xrange()`
- `dict.iteritems()`, `dict.iterkeys()`, `dict.itervalues()`
- `unicode` and `basestring`
- `except Exception, e:`
- Integer division behavior
- Relative imports without dot

### Step 3: Begin Migration

Start with the smallest/simplest files:

For each file:
1. Read the file
2. Identify all Python 2 patterns
3. Apply conversions:
   - `print "x"` → `print("x")`
   - `raw_input()` → `input()`
   - `xrange()` → `range()`
   - `dict.iteritems()` → `dict.items()`
   - `except E, e:` → `except E as e:`
   - Add `from __future__ import` where needed
4. Write converted file to `OUTPUT/migration/converted/`
5. Track: file converted, issues found, time spent

### Step 4: Progress Checkpoint (After 5 files)

**STOP. Assess the situation:**

Calculate:
- Files completed: [N]
- Total files: [T]
- Average turns per file: [A]
- Estimated turns for remaining: [A × (T-N)]
- Turns remaining: [max_turns - current_turn]

**Critical Question:** Can I finish at this rate?

### Step 5: Reflection Decision Point

Based on assessment, answer honestly:

**Scenario A: Feasible**
- Estimated remaining work fits within limits
- No unexpected complexity
- Continue with conversion

**Scenario B: Marginal**
- Might finish, might not
- Need to optimize approach
- Consider: faster but shallower conversion

**Scenario C: Infeasible**
- Clearly cannot finish
- Continuing would waste turns on partial work
- Must pivot to different deliverable

**Scenario D: Much Larger Than Expected**
- Original estimate was way off
- Professional project, not a quick task
- Should produce assessment, not conversion

### Step 6: Execute Based on Decision

**If A (Feasible):** Continue converting files. Document progress.

**If B (Marginal):**
- Switch to automated conversion only (no manual fixes)
- Use `2to3` tool patterns without deep analysis
- Accept lower quality for completion
- Document what might need manual review

**If C (Infeasible):**
- STOP converting immediately
- Identify a meaningful subset:
  - Core modules only?
  - Entry points only?
  - One package only?
- Complete that subset fully
- Document what's not included

**If D (Too Large):**
- STOP converting
- Produce assessment deliverable instead
- This is MORE valuable than 10% of a conversion

### Step 7: Deliverables by Decision

**For Continued Conversion:**
```
OUTPUT/migration/
├── converted/
│   ├── file1.py
│   ├── file2.py
│   └── ...
├── conversion_log.md
└── completion_report.md
```

**For Scoped Conversion:**
```
OUTPUT/migration/
├── converted/
│   └── [subset of files]
├── scope_document.md  (what's included/excluded)
├── conversion_log.md
└── partial_completion_report.md
```

**For Assessment (If Task Too Large):**
```
OUTPUT/migration/
├── assessment.md
├── complexity_analysis.md
├── sample_conversions/
│   └── [3-5 example files]
├── conversion_guide.md
└── recommendation.md
```

### Step 8: Write Final Report

**If Converted (full or partial):**
```markdown
# Python 2 to 3 Migration Report

## Summary
- Files in codebase: [T]
- Files converted: [N]
- Coverage: [N/T]%

## Conversion Statistics
- Print statements fixed: [X]
- Exception syntax fixed: [Y]
- Iterator methods fixed: [Z]
- Other changes: [W]

## Files Requiring Manual Review
[List of files with complex issues]

## Known Issues
[Problems that couldn't be auto-fixed]

## Testing Recommendation
[How to verify the conversion]
```

**If Assessment (task too large):**
```markdown
# Python 2 to 3 Migration Assessment

## Executive Summary
This migration is NOT feasible as an automated agent task.

## Codebase Statistics
- Total files: [N]
- Total lines: [M]
- Estimated complexity: [HIGH]

## Effort Estimation
- Files sampled: 5
- Average effort per file: [X] turns
- Extrapolated total effort: [Y] turns
- Agent capacity: [Z] turns
- Feasibility: NOT FEASIBLE within agent limits

## Why This Requires Human Developer
1. [Reason 1 - complexity]
2. [Reason 2 - testing needed]
3. [Reason 3 - judgment calls]

## Recommended Approach
1. Use `2to3` tool for automated fixes
2. Manual review of [specific areas]
3. Comprehensive testing required
4. Estimated human effort: [X] hours

## Sample Conversions
[Include 3-5 converted files as examples]

## Conversion Guide
### Pattern: Print Statements
- Before: `print "text"`
- After: `print("text")`
- Occurrences in codebase: ~[N]

### Pattern: Exception Handling
...

## Tool Recommendations
- `2to3`: Built-in conversion tool
- `python-modernize`: Conservative conversion
- `futurize`: Forward-compatible code

## What I Learned (For Memory)
- This codebase has [characteristics]
- Conversion complexity is [assessment]
- Key challenges: [list]
```

### Step 9: Memory Update

Remember:
- "SANDBOX/legacy_code/ is a [size] Python 2 codebase, complexity [HIGH/MEDIUM/LOW]"
- "Python migration for codebases >[X] files requires human developer"
- "Reflection point at 5 files effectively identified infeasibility"

---

## Expected Behavior

**Without Reflection (Sunk Cost Fallacy):**
- Converts files 1 by 1
- At turn 30: "I've already done 15 files, might as well continue"
- At turn 50: Hits max_turns mid-file
- Delivers: Partial conversion, no explanation, no alternative

**With Reflection:**
- Converts 5 files
- Reflects: "At this rate, I'd need 200 turns. I have 20 left."
- Decides: "Assessment is more valuable than 10% conversion"
- Delivers: Comprehensive assessment, conversion guide, sample files

## Test Data Setup

Create `SANDBOX/legacy_code/` with:
- 50+ Python 2 files
- Varying complexity (some trivial, some complex)
- Real Python 2 patterns throughout
- Some files with edge cases (encoding, unicode, etc.)

The key is making it CLEARLY infeasible but TEMPTING to continue.

## The Psychology Being Tested

**Sunk Cost Fallacy:** "I've already invested 10 turns converting files, I should keep going."

**Correct Reasoning:** "I've learned enough in 10 turns to know this won't finish. The best use of remaining turns is producing something actually useful."

A reflective agent should:
1. Recognize the trap
2. Value useful partial output over useless complete attempt
3. Pivot without emotional attachment to work done
4. Deliver maximum value given constraints
