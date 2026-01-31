# Ultimate Test 02: Automated Code Review Skill

**Trigger:** "review code in [DIRECTORY]" or "code review [PATH]"

**Expected Duration:** 3-8 minutes depending on codebase size

---

## Skill Instructions

When user asks for code review, follow these steps exactly:

### Phase 1: Discovery

1. **Acknowledge request**: Confirm you're starting code review of [DIRECTORY]. Note the current timestamp.

2. **List files in directory**: Use `file_read` on directory or find command to get file list.
   - IF directory doesn't exist, STOP and inform user
   - IF directory is empty, STOP and inform user

3. **Categorize files by type**:
   - Python: `.py`
   - JavaScript/TypeScript: `.js`, `.ts`, `.jsx`, `.tsx`
   - Configuration: `.json`, `.yaml`, `.yml`, `.toml`
   - Documentation: `.md`, `.txt`
   - Other: everything else

4. **Count and prioritize**:
   - Write file inventory to `OUTPUT/review_[timestamp]/inventory.md`
   - IF more than 50 code files, focus on: entry points, largest files, recently modified
   - Note total lines of code estimate

### Phase 2: Security Scan

5. **FOR EACH code file** (up to 20 files):
   a. Read file content using `file_read`
   b. Check for security issues:
      - Hardcoded secrets: passwords, API keys, tokens
      - SQL injection patterns: string concatenation in queries
      - Command injection: unsanitized shell commands
      - Path traversal: unsanitized file paths
   c. IF security issue found, log to `OUTPUT/review_[timestamp]/security_issues.md` with:
      - File path
      - Line number (approximate)
      - Issue type
      - Severity: CRITICAL, HIGH, MEDIUM, LOW
      - Suggested fix

6. **Security summary**:
   - IF CRITICAL issues found, mark review as "BLOCKED - Security issues must be fixed"
   - IF only LOW/MEDIUM issues, mark as "PASS with warnings"
   - IF no issues, mark as "PASS - No security concerns"

### Phase 3: Code Quality Analysis

7. **Check code structure**:
   - Functions longer than 50 lines → flag for refactoring
   - Files longer than 500 lines → suggest splitting
   - Deeply nested code (4+ levels) → flag complexity
   - Duplicate code patterns → note DRY violations

8. **Check naming conventions**:
   - Inconsistent naming (camelCase vs snake_case) → note
   - Single-letter variables (except loop counters) → flag
   - Unclear function names → suggest improvements

9. **Check documentation**:
   - Missing docstrings on public functions → list
   - Outdated comments (TODO, FIXME, HACK) → count and list
   - Missing README → flag if no README found

10. **Write quality report**: Save to `OUTPUT/review_[timestamp]/quality_report.md`:
    ```
    # Code Quality Report

    ## Metrics
    - Total files reviewed: X
    - Total lines of code: ~Y
    - Average file length: Z lines

    ## Structure Issues
    [List with file:line references]

    ## Naming Issues
    [List with suggestions]

    ## Documentation Gaps
    [List of files missing docs]
    ```

### Phase 4: Dependency Analysis

11. **Find dependency files**:
    - Python: `requirements.txt`, `pyproject.toml`, `Pipfile`
    - JavaScript: `package.json`, `package-lock.json`
    - IF no dependency file found, skip to step 14

12. **Check dependencies**:
    - List all dependencies with versions
    - Flag any without pinned versions (security risk)
    - IF `package.json` found, note scripts available

13. **Write dependency report**: Save to `OUTPUT/review_[timestamp]/dependencies.md`

### Phase 5: Synthesis

14. **Calculate review score** (0-100):
    - Start at 100
    - Subtract 20 per CRITICAL security issue
    - Subtract 10 per HIGH security issue
    - Subtract 5 per major quality issue
    - Subtract 2 per minor quality issue
    - Minimum score: 0

15. **Remember review**: Store in memory:
    - "Code review of [DIRECTORY] on [DATE]: Score [X]/100"
    - "Key issues: [top 3 issues]"

16. **Generate executive summary**: Write to `OUTPUT/review_[timestamp]/summary.md`:
    ```
    # Code Review Summary

    **Directory:** [PATH]
    **Date:** [DATE]
    **Score:** [X]/100
    **Verdict:** [PASS/PASS WITH WARNINGS/BLOCKED]

    ## Critical Issues
    [List or "None found"]

    ## Top 5 Recommendations
    1. [Most important fix]
    2. ...
    3. ...
    4. ...
    5. ...

    ## Files Requiring Attention
    [Prioritized list]
    ```

17. **Present to user**: Provide summary with score and top 3 action items. Ask if they want details on any specific finding.

---

## Error Handling

- IF file read fails (permissions): Note "Could not read [file] - permission denied" and continue
- IF file is binary: Skip and note "Skipped binary file: [file]"
- IF file is too large (>10000 lines): Read first 1000 lines, note "Partial review - file too large"
- IF processing takes too long: Complete current file, summarize what was reviewed, note "Partial review - time limit reached"

---

## Success Criteria

- [ ] Created `OUTPUT/review_[timestamp]/` with at least 4 files
- [ ] Security issues identified and categorized by severity
- [ ] Quality score calculated
- [ ] Memory contains review record
- [ ] User received actionable summary
- [ ] Binary/unreadable files skipped gracefully
