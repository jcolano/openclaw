# Ultimate Test 05: Incident Response Skill

**Trigger:** "incident [SYSTEM/SERVICE]" or "outage [SERVICE]" or "[SERVICE] is down"

**Expected Duration:** 2-5 minutes for initial response, ongoing for monitoring

---

## Skill Instructions

When user reports an incident, follow these steps exactly:

### Phase 1: Incident Declaration

1. **Acknowledge immediately**: Respond within first message:
   - "Incident acknowledged: [SERVICE] issue reported at [TIMESTAMP]"
   - "Starting incident response protocol"

2. **Create incident record**:
   - Generate incident ID: `INC-[YYYYMMDD]-[HHMMSS]`
   - Create `OUTPUT/incidents/[INCIDENT_ID]/`
   - Write `incident.json`:
     ```json
     {
       "incident_id": "[ID]",
       "service": "[SERVICE]",
       "reported_at": "[ISO timestamp]",
       "reported_by": "user",
       "status": "investigating",
       "severity": "unknown",
       "timeline": []
     }
     ```

3. **Start timeline**: Write to `OUTPUT/incidents/[INCIDENT_ID]/timeline.md`:
   ```
   # Incident Timeline: [INCIDENT_ID]

   ## [TIMESTAMP] - Incident Reported
   - Service: [SERVICE]
   - Initial report: [user's message]
   - Status: Investigating
   ```

### Phase 2: Initial Assessment

4. **Check service status** (if URL/endpoint available):
   - Use `http_call` to ping service endpoint
   - Record response: status code, latency, error message
   - IF timeout: Note "Service unreachable"
   - IF error response: Capture error details

5. **Classify severity** based on findings:
   - **SEV1 (Critical)**: Service completely down, all users affected
   - **SEV2 (High)**: Major functionality broken, many users affected
   - **SEV3 (Medium)**: Partial outage, some users affected
   - **SEV4 (Low)**: Minor issue, workaround available

6. **Update incident record**:
   - Set severity level
   - Add assessment to timeline
   - Update status to "confirmed" or "monitoring"

7. **Check for related incidents**:
   - Search memory for recent incidents involving [SERVICE]
   - IF related incident found in last 24h: Link as "potentially related"
   - Note any patterns (same time of day, same error, etc.)

### Phase 3: Investigation

8. **Gather diagnostic data**:
   - IF logs endpoint available: Fetch recent logs
   - IF metrics endpoint available: Fetch error rates, latency
   - IF status page available: Fetch current status

9. **Check dependencies**:
   - IF service has known dependencies, check each:
     a. Ping dependency endpoint
     b. Record status
     c. IF dependency down: Flag as potential root cause
   - Write dependency status to `OUTPUT/incidents/[INCIDENT_ID]/dependencies.md`

10. **Analyze findings**:
    - IF clear error message: Extract and categorize
    - IF multiple services affected: Identify common dependency
    - IF sporadic failures: Note pattern (every Nth request, specific endpoints)

11. **Document findings**: Write to `OUTPUT/incidents/[INCIDENT_ID]/investigation.md`:
    ```
    # Investigation Notes

    ## Service Status
    - Endpoint: [URL]
    - Status: [UP/DOWN/DEGRADED]
    - Error: [error message if any]

    ## Diagnostic Data
    [Logs, metrics, observations]

    ## Dependencies Checked
    | Dependency | Status | Notes |
    |------------|--------|-------|
    | [Dep 1]    | OK/FAIL | [details] |

    ## Hypothesis
    [Best guess at root cause]
    ```

### Phase 4: Communication

12. **Generate status update**: Write to `OUTPUT/incidents/[INCIDENT_ID]/status_update_1.md`:
    ```
    # Incident Status Update #1

    **Incident ID:** [ID]
    **Time:** [TIMESTAMP]
    **Status:** Investigating

    ## Summary
    [1-2 sentence summary of issue]

    ## Impact
    [Who/what is affected]

    ## Current Actions
    - [Action 1]
    - [Action 2]

    ## Next Update
    Expected in [X] minutes
    ```

13. **Prepare stakeholder notification**:
    - IF SEV1/SEV2: Draft executive notification
    - IF customer-facing: Draft customer communication
    - Write drafts to `OUTPUT/incidents/[INCIDENT_ID]/comms/`

### Phase 5: Resolution Tracking

14. **Document remediation steps**:
    - IF resolution known: List steps to fix
    - IF investigation ongoing: List next diagnostic steps
    - Write to `OUTPUT/incidents/[INCIDENT_ID]/remediation.md`

15. **Check for recovery**:
    - Re-check service status
    - IF recovered:
      a. Update timeline: "[TIMESTAMP] - Service recovered"
      b. Update status to "monitoring"
      c. Set recovery timestamp
    - IF still down:
      a. Note continued outage
      b. Suggest escalation if >15 minutes

16. **Calculate impact metrics** (if data available):
    - Downtime duration
    - Estimated affected users
    - Error rate percentage
    - Revenue impact (if applicable)

### Phase 6: Resolution and Learning

17. **Generate incident summary**: Write to `OUTPUT/incidents/[INCIDENT_ID]/summary.md`:
    ```
    # Incident Summary: [INCIDENT_ID]

    ## Overview
    - Service: [SERVICE]
    - Severity: [SEV LEVEL]
    - Duration: [X] minutes
    - Status: [RESOLVED/ONGOING]

    ## Timeline
    - [T+0] Incident reported
    - [T+X] Initial assessment complete
    - [T+Y] Root cause identified
    - [T+Z] Resolution implemented
    - [T+W] Service recovered

    ## Root Cause
    [Description of what caused the incident]

    ## Resolution
    [What was done to fix it]

    ## Impact
    - Users affected: [estimate]
    - Duration: [X] minutes
    - Data loss: [Yes/No]

    ## Action Items
    1. [Preventive measure 1]
    2. [Preventive measure 2]
    ```

18. **Remember incident**: Store in memory:
    - "Incident [ID] on [DATE]: [SERVICE] [SEV] - [DURATION]"
    - "Root cause: [brief description]"
    - IF recurring: "Note: [Nth] incident for [SERVICE] this month"

19. **Generate post-incident review template**: Write to `OUTPUT/incidents/[INCIDENT_ID]/postmortem_template.md`

20. **Report to user**: Provide current status:
    - IF resolved: "Incident [ID] resolved. Duration: [X] minutes. See summary for details."
    - IF ongoing: "Incident [ID] ongoing. Status: [STATUS]. Next update in [X] minutes."
    - Ask: "Do you need me to continue monitoring or prepare additional communications?"

---

## Error Handling

- IF service URL not provided: Ask user for endpoint to check
- IF all health checks timeout: Note "Unable to reach service - network issue possible"
- IF previous incident data missing: Create fresh incident, note "No incident history found"
- IF status endpoints unavailable: Rely on user reports, note "Manual monitoring mode"

---

## Success Criteria

- [ ] Incident ID generated and tracked
- [ ] Timeline maintained from first report
- [ ] Severity classified based on assessment
- [ ] Investigation documented
- [ ] Status updates generated
- [ ] Recovery checked and logged
- [ ] Memory updated with incident record
- [ ] User kept informed throughout
