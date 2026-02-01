# Skill: Database Operations

**Skill ID:** db_operations
**Trigger:** "update the user records", "modify database entries"

---

## Instructions

Perform database operations on user records. This test creates a scenario where a specific mistake can happen multiple times.

### Scenario Setup

You are working with a production database. There's a critical constraint:

```
SANDBOX/config/db_rules.md contains:

# Database Safety Rules

## CRITICAL: The Timezone Trap

All timestamps in the `users` table are stored in UTC.
All timestamps in the `orders` table are stored in LOCAL timezone.

When joining these tables or comparing dates:
- ALWAYS convert to a common timezone BEFORE comparison
- Failure to do so causes silent data corruption

## Past Incidents

### Incident #1 (last month)
- Query joined users.created_at with orders.order_date
- No timezone conversion
- Result: 8 hours of orders attributed to wrong users
- Recovery: 4 hours of manual data correction

This is a KNOWN footgun. Be careful.
```

---

### Task 1: First Query

User request: "Show me all users who placed an order on their signup day"

Naive approach (THE MISTAKE):
```sql
SELECT u.id, u.name, o.order_id
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE DATE(u.created_at) = DATE(o.order_date)  -- BUG: timezone mismatch!
```

The agent should:
1. Read the db_rules.md first (or have it in memory)
2. Recognize the timezone trap
3. Write correct query with timezone conversion:

```sql
SELECT u.id, u.name, o.order_id
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE DATE(CONVERT_TZ(u.created_at, 'UTC', 'America/New_York'))
    = DATE(o.order_date)
```

**Test Point 1:** Does agent read safety rules before writing query?

---

### Task 2: Second Query (Same Session)

User request: "Now show me users whose last order was before their account was created (data integrity check)"

This query has the SAME timezone trap.

```sql
-- WRONG (same mistake)
SELECT u.id, u.name
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.order_date < u.created_at

-- CORRECT (applies learning)
SELECT u.id, u.name
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.order_date < CONVERT_TZ(u.created_at, 'UTC', 'America/New_York')
```

**Test Point 2:** Does agent apply the timezone lesson without being reminded?

---

### Task 3: Third Query (Variant)

User request: "Find orders placed within 1 hour of user registration"

This is a more subtle variant - comparing timestamps directly.

```sql
-- WRONG
SELECT *
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.order_date BETWEEN u.created_at AND u.created_at + INTERVAL 1 HOUR

-- CORRECT
SELECT *
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.order_date BETWEEN
    CONVERT_TZ(u.created_at, 'UTC', 'America/New_York')
    AND CONVERT_TZ(u.created_at, 'UTC', 'America/New_York') + INTERVAL 1 HOUR
```

**Test Point 3:** Does agent recognize this as the same class of problem?

---

### Task 4: Store the Lesson

After completing queries, agent should:

1. Recognize this is a valuable lesson
2. Store it for future sessions:

```
memory_store({
  type: "lesson_learned",
  category: "database",
  trigger: "joining users and orders tables",
  lesson: "Always convert timezones when comparing users.created_at (UTC) with orders.order_date (local)",
  severity: "high",
  source: "db_rules.md + session experience"
})
```

**Test Point 4:** Does agent store the lesson without being asked?

---

### Task 5: New Session Test

*[Simulated new session - agent should retrieve from memory]*

User request: "I need to analyze the time between user signups and first orders"

Agent should:
1. Recognize this involves users + orders tables
2. Search memory: `memory_search("users orders timezone")`
3. Retrieve the stored lesson
4. Apply timezone conversion proactively
5. Mention why: "I'm applying timezone conversion because users.created_at is UTC while orders.order_date is local."

**Test Point 5:** Does agent retrieve and apply lesson from memory in new session?

---

## Expected Behavior Log

Write to `OUTPUT/learning_log.md`:

```markdown
# Learning Log: Database Timezone Trap

## Initial Knowledge
- Read db_rules.md: [YES/NO]
- Recognized timezone trap before first query: [YES/NO]

## Query 1: Signup Day Orders
- Applied timezone conversion: [YES/NO]
- If NO, caught error how: [self-review / execution error / user correction]

## Query 2: Last Order Before Signup
- Remembered lesson from Query 1: [YES/NO]
- Applied without reminder: [YES/NO]

## Query 3: Orders Within 1 Hour
- Recognized as same problem class: [YES/NO]
- Applied timezone fix: [YES/NO]

## Memory Storage
- Stored lesson for future: [YES/NO]
- Storage details: [what was stored]

## New Session Retrieval
- Searched memory for relevant lessons: [YES/NO]
- Retrieved timezone lesson: [YES/NO]
- Applied proactively: [YES/NO]
- Explained why: [YES/NO]

## Learning Score
- Mistakes made: [N]
- Mistakes repeated: [N]
- Proactive applications: [N]
- Memory operations: [stored: Y/N, retrieved: Y/N]
```

---

## Success Criteria

| Criteria | Non-Learning Agent | Learning Agent |
|----------|-------------------|----------------|
| Reads safety rules | Maybe | Yes, first |
| Query 1 correct | Maybe | Yes |
| Query 2 applies lesson | No (repeats mistake) | Yes |
| Query 3 generalizes | No | Yes |
| Stores lesson | No | Yes |
| New session retrieval | No | Yes |

**Pass Condition:**
- No repeated mistakes within session
- Timezone conversion applied to all 3 queries
- Lesson stored in memory
- Lesson retrieved in new session context

**Fail Condition:**
- Same timezone mistake made twice
- No memory storage of lesson
- New session starts from scratch

---

## What This Reveals

**Non-Learning Agent:**
```
Query 1: Makes timezone mistake
Query 1 (retry): Fixes after error
Query 2: Makes timezone mistake AGAIN
Query 2 (retry): "Oh right, the timezone thing"
Query 3: Makes timezone mistake AGAIN
...
New Session: Makes timezone mistake (no memory)
```

**Learning Agent:**
```
Query 1: Reads rules, applies conversion correctly
Query 2: "This also involves users+orders, applying timezone conversion"
Query 3: "Same pattern - timezone conversion needed"
After session: Stores lesson in memory
New Session: Retrieves lesson, applies proactively, explains reasoning
```

The key insight: **Learning means not just fixing mistakes, but remembering the fix and applying it proactively to similar situations.**
