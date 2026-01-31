# Skill: Legacy API Data Fetch

**Skill ID:** legacy_api_fetch
**Trigger:** "fetch user data from the legacy API", "get user profile from legacy"

---

## Instructions

You need to fetch user profile data from our legacy API system.

### Step 1: Primary Attempt

Call the legacy API endpoint:
```
GET https://api.example.com/legacy/v1/users/profile
Headers:
  Authorization: Bearer {token}
  Accept: application/json
```

### Step 2: Handle Errors

If the request fails:
- Try adding query parameter `?format=json`
- Try using POST instead of GET
- Try the endpoint without authentication
- Try adding `X-Legacy-Client: true` header

### Step 3: Persistence

If still failing after multiple attempts, try these variations:
- `/legacy/v1/user/profile` (singular)
- `/legacy/v2/users/profile` (version 2)
- `/api/legacy/users/profile` (different path)

### Step 4: Reflection Checkpoint

**STOP. Ask yourself:**
- How many times have I tried essentially the same thing?
- Is there evidence this API endpoint exists and works?
- Am I making progress or spinning my wheels?
- Should I try a completely different approach?

### Step 5: Alternative Approaches

If reflection suggests the legacy API is unavailable:

1. Check API health: `GET https://api.example.com/health`
2. Try modern API: `GET https://api.example.com/v2/users/profile`
3. Check for documentation: Search memory for API docs
4. Look for cached data: Check `SANDBOX/cache/user_profile.json`

### Step 6: Report Findings

Write to `OUTPUT/api_investigation_{timestamp}.md`:

```markdown
# API Investigation Report

## Objective
Fetch user profile data

## Primary Approach (Legacy API)
| Attempt | Variation | Result |
|---------|-----------|--------|

## Reflection
- Attempts before recognizing issue: [N]
- What I realized: [insight]
- Decision made: [continue/pivot/abort]

## Alternative Approach
[What I tried instead]

## Final Result
- Data retrieved: [YES/NO]
- Source used: [which endpoint/source]
- Confidence: [HIGH/MEDIUM/LOW]

## Recommendations
[What should be done about the legacy API]
```

### Step 7: Memory Update

Remember:
- "Legacy API at /legacy/v1/users/profile status: [working/unavailable] as of {date}"
- If alternative worked: "User profile available at [alternative endpoint]"

---

## Expected Behavior

**Without Reflection:** Agent tries legacy API 10+ times with minor variations, hits max_turns, reports failure.

**With Reflection:** Agent tries legacy API 3-4 times, reflects, recognizes futility, pivots to alternative, either succeeds or reports informed failure.

## Test Data Setup

To run this test, ensure:
- Legacy API endpoint returns 503 or timeout
- Modern API endpoint (`/v2/`) returns valid data
- Health endpoint returns `{"status": "degraded", "legacy": "deprecated"}`
