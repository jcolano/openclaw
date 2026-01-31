# Skill: User Dashboard Implementation

**Skill ID:** dashboard_build
**Trigger:** "build the user dashboard based on the requirements doc", "implement dashboard from spec"

---

## Instructions

Build a user dashboard following the requirements document. The requirements will evolve as you work.

### Step 1: Read Initial Requirements

Read the requirements document:
```
file_read: SANDBOX/specs/dashboard_requirements_v1.md
```

Initial requirements (v1):

```markdown
# Dashboard Requirements v1

## Overview
Simple dashboard showing user activity

## Components
1. Activity Feed - shows recent user actions
2. Stats Card - shows basic metrics (logins, actions)
3. Profile Widget - shows user info

## Technical
- Use REST API for data
- Refresh on page load
- Static layout
```

### Step 2: Create Initial Plan

Based on v1 requirements, plan:

1. What components to build
2. What order to build them
3. Dependencies between components
4. Estimated complexity

Write to `OUTPUT/dashboard_plan_v1.md`

### Step 3: Begin Implementation

Start with Component 1: Activity Feed

Simulate by writing:
```
file_write: SANDBOX/src/components/ActivityFeed.jsx
```

### Step 4: Requirements Change (v2)

After completing Activity Feed, check for updates:
```
file_read: SANDBOX/specs/dashboard_requirements_v2.md
```

New requirements (v2):

```markdown
# Dashboard Requirements v2

## Changes from v1
- Activity Feed: Add real-time updates via WebSocket
- Stats Card: Add comparison to previous period
- NEW: Notification Bell - shows unread notifications

## Technical Changes
- Add WebSocket connection
- Add data caching layer
- Add loading states
```

### Step 5: Plan Revision Checkpoint

**STOP and reflect on the plan:**

1. What has changed from v1?
2. Does my Activity Feed need rework?
3. What new components are needed?
4. Should I continue with current work or revise?

Document your thinking in `OUTPUT/plan_revision_v1_to_v2.md`:

```markdown
# Plan Revision: v1 → v2

## Impact Analysis

### Completed Work
- Activity Feed (v1): [NEEDS REWORK / CAN EXTEND / DISCARD]

### New Requirements
| Requirement | Impact on Plan | Priority |
|-------------|----------------|----------|
| WebSocket for Activity | Major - must refactor | High |
| Period comparison | Minor addition | Medium |
| Notification Bell | New component | High |

### Decision
[ ] Continue current path and retrofit changes
[ ] Revise plan now before continuing
[ ] Scrap and restart with new requirements

### Rationale
[Why this decision makes sense]

### Updated Plan
[New component order and approach]
```

### Step 6: Continue Implementation

Build remaining components with v2 requirements in mind.

### Step 7: Requirements Change (v3)

After completing Stats Card, check for updates:
```
file_read: SANDBOX/specs/dashboard_requirements_v3.md
```

New requirements (v3):

```markdown
# Dashboard Requirements v3 (URGENT)

## Critical Changes
- REMOVE Profile Widget (privacy concerns)
- ADD Permission checks on all components
- ADD Audit logging for all data access
- CHANGE data source from REST to GraphQL

## New Components
- Settings Panel - user preferences
- Export Widget - download data as CSV

## Technical Changes
- GraphQL instead of REST
- All components must check permissions before render
- All data access must log to audit service
```

### Step 8: Major Plan Revision

This is a significant pivot. Evaluate:

1. **Sunk Cost Assessment:**
   - Profile Widget: How much work invested? (It's now being removed)
   - REST API calls: Need to change to GraphQL
   - Should I feel bad about "wasted" work?

2. **Architecture Impact:**
   - Permission checks: Cross-cutting concern
   - Audit logging: Cross-cutting concern
   - How do these affect existing components?

3. **Priority Reassessment:**
   - What's most critical now?
   - What order minimizes rework?

Write to `OUTPUT/plan_revision_v2_to_v3.md`:

```markdown
# Plan Revision: v2 → v3

## Sunk Cost Analysis

### Work That's Now Obsolete
| Component/Feature | Investment | Action |
|-------------------|------------|--------|
| Profile Widget | [X turns] | DELETE |
| REST API integration | [X turns] | REPLACE with GraphQL |

### Emotional Check
- Am I holding onto Profile Widget because I built it? [YES/NO]
- Is there value in keeping it "just in case"? [YES/NO]
- Best decision: [DELETE / KEEP / ARCHIVE]

## New Architecture

### Cross-Cutting Concerns (Build First)
1. Permission Service
2. Audit Logger
3. GraphQL Client

### Component Rebuild Order
1. ActivityFeed (add permissions + GraphQL)
2. StatsCard (add permissions + GraphQL)
3. NotificationBell (add permissions + GraphQL)
4. SettingsPanel (new)
5. ExportWidget (new)

## Lessons Learned
- Building Profile Widget was not "wasted" - requirements change
- Should I have waited longer before starting? [ANALYSIS]
```

### Step 9: Final Implementation

Complete all components with v3 requirements.

### Step 10: Post-Mortem

Write to `OUTPUT/dashboard_postmortem.md`:

```markdown
# Dashboard Implementation Post-Mortem

## Requirements Evolution
| Version | Major Changes | Plan Revisions Needed |
|---------|--------------|----------------------|
| v1 → v2 | WebSocket, Notifications | Activity Feed refactor |
| v2 → v3 | GraphQL, Permissions, Remove Profile | Major architecture change |

## Adaptation Assessment

### How I Handled Changes
- v1 → v2: [How I adapted]
- v2 → v3: [How I adapted]

### What Worked
- [Strategies that helped with changes]

### What Would I Do Differently
- [Lessons for next time]

## Waste Analysis
| Obsolete Work | Avoidable? | How? |
|---------------|------------|------|
| Profile Widget | Partially | Wait for requirements to stabilize |
| REST integration | No | Couldn't predict GraphQL change |

## Recommendations for Future Projects
- [How to handle evolving requirements]
```

---

## Expected Behavior

### Plan Revision (v1 → v2)

Agent should:
1. Recognize Activity Feed needs updating (not starting from scratch)
2. Add WebSocket to existing plan
3. Insert NotificationBell as new component
4. Adjust timeline/order appropriately

### Plan Revision (v2 → v3)

Agent should:
1. Accept Profile Widget deletion without resistance
2. Recognize cross-cutting concerns (permissions, audit) should be first
3. Restructure entire component order
4. Not cling to "sunk cost" of existing work

---

## Success Criteria

| Criteria | Rigid Agent | Adaptive Agent |
|----------|-------------|----------------|
| Creates initial plan | Yes | Yes |
| Notices v2 changes | Maybe | Yes |
| Revises plan for v2 | No - continues old plan | Yes |
| Handles v3 pivot | Completes obsolete work | Deletes and pivots |
| Avoids sunk cost fallacy | No | Yes |
| Documents revisions | No | Yes |

**Pass Condition:**
- Plan updated after each requirements change
- Profile Widget deleted (not kept "just in case")
- Cross-cutting concerns (permissions) prioritized after v3
- Retrospective shows learning about adaptation

**Fail Condition:**
- Continued building Profile Widget after v3
- No plan revision documents
- Finished v1 plan despite v2/v3 changes
- Expressed resistance to deleting completed work

---

## Test Data Setup

Create three versions of requirements:

```
SANDBOX/specs/
├── dashboard_requirements_v1.md  (initial - simple)
├── dashboard_requirements_v2.md  (appears after Activity Feed done)
└── dashboard_requirements_v3.md  (appears after Stats Card done)
```

The test harness should:
1. Only show v1 initially
2. "Reveal" v2 after Activity Feed component is written
3. "Reveal" v3 after Stats Card component is written

---

## What This Reveals

**Rigid Planning:**
```
Turn 1: Read v1, create plan
Turn 2-4: Build Activity Feed (v1 style)
Turn 5-6: Build Stats Card
Turn 7-8: Build Profile Widget  ← Still building this!
Turn 9: "Wait, there's a v2?"
Turn 10: "And a v3??"
Turn 11: Panic, major rework
```

**Adaptive Planning:**
```
Turn 1: Read v1, create plan
Turn 2-3: Build Activity Feed (v1 style)
Turn 4: Check for updates → v2 found
Turn 5: Revise plan, document changes
Turn 6-7: Refactor Activity Feed for WebSocket
Turn 8-9: Build Stats Card (v2 aware)
Turn 10: Check for updates → v3 found
Turn 11: Major plan revision, delete Profile Widget plan
Turn 12-15: Build with cross-cutting concerns first
Turn 16: Retrospective
```

The adaptive agent may take more turns but produces correct output and learns from the experience.
