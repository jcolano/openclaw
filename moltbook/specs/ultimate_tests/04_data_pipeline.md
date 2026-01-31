# Ultimate Test 04: Data Pipeline Skill

**Trigger:** "run data pipeline from [SOURCE]" or "etl from [URL/FILE] to [DESTINATION]"

**Expected Duration:** 3-7 minutes depending on data size

---

## Skill Instructions

When user asks to run a data pipeline, follow these steps exactly:

### Phase 1: Source Analysis

1. **Parse request**: Identify:
   - Source: URL, file path, or API endpoint
   - Destination: file, directory, or "analyze only"
   - Format: JSON, CSV, XML, or auto-detect

2. **Create pipeline workspace**:
   - Create `OUTPUT/pipeline_[timestamp]/`
   - Write `manifest.json` with pipeline metadata:
     ```json
     {
       "pipeline_id": "[timestamp]",
       "source": "[SOURCE]",
       "destination": "[DESTINATION]",
       "started_at": "[ISO timestamp]",
       "status": "running"
     }
     ```

3. **Validate source**:
   - IF URL: Check if reachable with HEAD request via `http_call`
   - IF file: Check if exists with `file_read`
   - IF API: Verify endpoint format
   - IF source invalid, STOP and report error with suggestions

### Phase 2: Data Extraction

4. **Fetch data**:
   - IF URL with JSON: `http_call` GET, parse response
   - IF URL with HTML: `web_fetch`, extract structured data
   - IF local file: `file_read`
   - Save raw data to `OUTPUT/pipeline_[timestamp]/raw_data.[ext]`

5. **Detect format** (if not specified):
   - Check first bytes for JSON (`{` or `[`)
   - Check for CSV headers (comma-separated first line)
   - Check for XML (`<?xml` or `<`)
   - IF unknown format, attempt JSON parse first

6. **Parse data**:
   - IF JSON: Parse into records
   - IF CSV: Split by delimiter, use first row as headers
   - IF XML: Extract record elements
   - Count total records, note in manifest

7. **Sample and profile**:
   - Take first 10 records as sample
   - Identify all fields/columns present
   - Detect data types: string, number, date, boolean, null
   - Write profile to `OUTPUT/pipeline_[timestamp]/data_profile.md`:
     ```
     # Data Profile

     ## Overview
     - Total records: [N]
     - Fields: [M]
     - Source format: [FORMAT]

     ## Field Analysis
     | Field | Type | Non-null % | Sample Values |
     |-------|------|------------|---------------|
     | [field1] | string | 98% | "example", "test" |
     ```

### Phase 3: Validation

8. **Check data quality**:
   - NULL values: Count per field, flag if >20% null
   - Duplicates: Check for duplicate records by key fields
   - Outliers: For numeric fields, flag values >3 std dev from mean
   - Format issues: Check dates parse correctly, emails have @, etc.

9. **Create validation report**: Write to `OUTPUT/pipeline_[timestamp]/validation.md`:
   ```
   # Data Validation Report

   ## Summary
   - Records validated: [N]
   - Pass rate: [X]%
   - Critical issues: [Y]

   ## Issues Found
   ### NULL Values
   [List fields with high null rates]

   ### Duplicates
   [Count and sample of duplicates]

   ### Format Errors
   [List of malformed values]

   ## Recommendation
   [PROCEED / PROCEED WITH CAUTION / ABORT]
   ```

10. **Decision point**:
    - IF critical issues > 10% of data: Ask user whether to proceed
    - IF minor issues only: Proceed with warning
    - IF clean data: Proceed normally

### Phase 4: Transformation

11. **Apply standard transformations**:
    - Trim whitespace from strings
    - Normalize dates to ISO format
    - Convert numeric strings to numbers
    - Handle null values (replace with defaults or mark)

12. **FOR EACH transformation rule** (if user specified):
    a. Apply rule to data
    b. Log transformation: "Applied [rule] to [field]: [N] records affected"
    c. IF transformation fails on record, log error and skip record

13. **Create cleaned dataset**:
    - Write to `OUTPUT/pipeline_[timestamp]/cleaned_data.json`
    - Include metadata header with transformation log

### Phase 5: Load

14. **Prepare output**:
    - IF destination is file: Format according to extension
    - IF destination is directory: Split into chunks if large
    - IF destination is "analyze": Skip file write, proceed to analysis

15. **Write output**:
    - Write data to destination
    - Verify write succeeded by reading back first record
    - IF write fails, save to backup location and report

16. **Update manifest**:
    ```json
    {
      "pipeline_id": "[timestamp]",
      "source": "[SOURCE]",
      "destination": "[DESTINATION]",
      "started_at": "[ISO timestamp]",
      "completed_at": "[ISO timestamp]",
      "status": "completed",
      "records_in": [N],
      "records_out": [M],
      "records_failed": [F],
      "duration_seconds": [D]
    }
    ```

### Phase 6: Reporting

17. **Generate summary statistics**:
    - Record counts: input, output, failed
    - Data changes: fields modified, nulls filled
    - Performance: duration, records/second

18. **Remember pipeline run**: Store in memory:
    - "Data pipeline from [SOURCE] on [DATE]: [N] records processed"
    - "Pipeline [timestamp]: [SUCCESS/PARTIAL/FAILED]"

19. **Generate final report**: Write to `OUTPUT/pipeline_[timestamp]/report.md`:
    ```
    # Pipeline Execution Report

    ## Summary
    - Pipeline ID: [timestamp]
    - Duration: [X] seconds
    - Status: [SUCCESS/PARTIAL/FAILED]

    ## Data Flow
    - Records in: [N]
    - Records transformed: [M]
    - Records failed: [F]
    - Records out: [M-F]

    ## Transformations Applied
    1. [Transformation 1]: [N] records
    2. [Transformation 2]: [M] records

    ## Issues Encountered
    [List of errors/warnings]

    ## Output Location
    [Path to output file(s)]
    ```

20. **Report to user**: Provide summary:
    - "Pipeline complete: [N] records processed in [X] seconds"
    - "Output written to: [PATH]"
    - "Issues: [count] - see report for details"

---

## Error Handling

- IF source unreachable: Retry 3 times with 2-second delay, then fail with clear error
- IF data too large (>100MB): Process in chunks, note "Large dataset - chunked processing"
- IF parse error: Log malformed records, continue with valid data
- IF destination write fails: Save to fallback `OUTPUT/pipeline_[timestamp]/output.[ext]`
- IF transformation fails: Skip record, log error, continue pipeline
- IF >50% records fail: STOP pipeline, report critical failure

---

## Success Criteria

- [ ] Source data fetched and raw copy saved
- [ ] Data profiled with field analysis
- [ ] Validation report generated
- [ ] Transformations applied and logged
- [ ] Output written to destination (or fallback)
- [ ] Manifest updated with final status
- [ ] Memory contains pipeline record
- [ ] User received summary with metrics
