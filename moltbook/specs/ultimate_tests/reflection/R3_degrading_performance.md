# Skill: Log File Analysis

**Skill ID:** log_analysis_bulk
**Trigger:** "analyze all log files in the directory", "process all logs"

---

## Instructions

Perform comprehensive analysis of all log files in the logs directory.

### Step 1: Discovery

List all files in `SANDBOX/logs/`:
- Count total files
- Note total size
- Identify file naming pattern

Expected: 100 log files named `app_001.log` through `app_100.log`

### Step 2: Analysis Protocol

For EACH log file, extract:

1. **Error Analysis**
   - Total error count (lines containing "ERROR")
   - Unique error messages
   - Most frequent error
   - Error rate (errors / total lines)

2. **Warning Analysis**
   - Total warning count
   - Warning categories
   - Critical warnings (containing "CRITICAL" or "FATAL")

3. **Timeline Analysis**
   - First timestamp in file
   - Last timestamp in file
   - Duration covered
   - Peak activity periods

4. **Pattern Detection**
   - Recurring sequences
   - Anomalies
   - Correlation with other files

### Step 3: Progress Monitoring

After every 5 files, evaluate:
- Files completed: [N]
- Files remaining: [100-N]
- Quality of recent analyses (detailed? shallow?)
- Am I still extracting meaningful insights?
- Is my analysis getting more superficial?

### Step 4: Reflection Checkpoint

**After processing 10-15 files, STOP and reflect:**

- **Quality Check:** Compare my analysis of file_001 vs file_010. Same depth?
- **Diminishing Returns:** Am I finding new insights or just repetition?
- **Resource Check:** How much capacity do I have left? Can I finish at this depth?
- **Strategic Question:** Should I continue at this depth, or adapt my approach?

### Step 5: Adaptive Strategies

If continuing at current depth is not feasible, choose ONE:

**Strategy A: Batch Summarization**
```
Process files in groups of 10:
- Quick scan each file in batch
- Aggregate statistics for batch
- Deep dive only on outliers
- Produce per-batch summary
```

**Strategy B: Statistical Sampling**
```
Select representative sample:
- First 5 files (early logs)
- Last 5 files (recent logs)
- 5 random files (middle)
- 5 largest files (most data)
Extrapolate patterns to full set
```

**Strategy C: Priority Analysis**
```
Rank files by importance:
- Sort by error count (estimate from file size or filename)
- Deep analyze top 20
- Quick scan next 30
- Skip remaining or metadata only
```

**Strategy D: Parallel Extraction**
```
Single pass extracting only:
- Error counts
- Time ranges
- File sizes
Then deep analyze only anomalies
```

### Step 6: Execute Adapted Strategy

1. Document which strategy you chose and why
2. Execute the strategy
3. Track coverage: what was analyzed deeply vs. lightly vs. skipped

### Step 7: Produce Report

Write to `OUTPUT/log_analysis_{timestamp}.md`:

```markdown
# Log Analysis Report

## Analysis Metadata
- Total files in directory: 100
- Analysis strategy: [FULL / BATCHED / SAMPLED / PRIORITY / PARALLEL]
- Files deeply analyzed: [N]
- Files lightly scanned: [M]
- Files skipped: [K]

## Strategy Adaptation

### Initial Approach
[How I started]

### Reflection Trigger
[What made me reconsider]
- Turn when reflected: [X]
- Files completed at reflection: [Y]
- Issue identified: [description]

### Adapted Approach
[What I changed to]
[Why this was better]

## Aggregate Findings

### Error Summary
| Category | Count | % of Total | Trend |
|----------|-------|------------|-------|
| Total Errors | | | |
| Critical | | | |
| Warning | | | |

### Top 10 Error Messages
1. [error] - [count] occurrences
...

### Timeline
- Earliest log: [timestamp]
- Latest log: [timestamp]
- Peak error periods: [list]

### Anomalies Detected
[Files or patterns that stood out]

## Confidence Assessment
- Confidence in findings: [HIGH/MEDIUM/LOW]
- What might be missed: [limitations]
- Recommended follow-up: [suggestions]

## Per-File Details
[Summary table or reference to detailed files]
```

### Step 8: Memory Update

Remember:
- "Log analysis strategy [X] effective for 100-file batches"
- "SANDBOX/logs/ contains [summary] as of {date}"
- Any anomalies worth tracking

---

## Expected Behavior

**Without Reflection:** Agent processes files 1-by-1, analysis quality degrades around file 20-30, produces shallow/repetitive output for later files, hits max_turns around file 40-50.

**With Reflection:** Agent notices quality degradation around file 10-15, explicitly changes strategy, completes full coverage with appropriate depth distribution, produces useful aggregate analysis.

## Test Data Setup

Create 100 log files with:
- Varying sizes (some 100 lines, some 10000 lines)
- Consistent error patterns (to reward pattern detection)
- Some anomalous files (to reward outlier detection)
- Realistic timestamps spanning a week

Sample log format:
```
2026-01-15 10:23:45.123 INFO [main] Application started
2026-01-15 10:23:46.456 DEBUG [worker-1] Processing request 12345
2026-01-15 10:23:47.789 ERROR [worker-1] NullPointerException at line 42
2026-01-15 10:23:48.012 WARN [main] Memory usage above 80%
```
