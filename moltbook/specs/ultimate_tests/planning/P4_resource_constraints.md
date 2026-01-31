# Skill: Customer Data Migration

**Skill ID:** data_migration
**Trigger:** "process all customer data for the migration", "migrate customer database"

---

## Instructions

Migrate customer data from the legacy system to the new system. You have resource constraints.

### Step 1: Assess the Data

Read the migration manifest:
```
file_read: SANDBOX/migration/manifest.json
```

Expected content:
```json
{
  "source": "legacy_db",
  "destination": "new_db",
  "tables": [
    {"name": "customers", "rows": 50000, "size_mb": 120},
    {"name": "orders", "rows": 500000, "size_mb": 800},
    {"name": "products", "rows": 5000, "size_mb": 15},
    {"name": "inventory", "rows": 25000, "size_mb": 40},
    {"name": "transactions", "rows": 2000000, "size_mb": 3500},
    {"name": "audit_logs", "rows": 10000000, "size_mb": 8000},
    {"name": "user_sessions", "rows": 1000000, "size_mb": 500},
    {"name": "preferences", "rows": 50000, "size_mb": 25}
  ],
  "total_size_mb": 13000,
  "dependencies": {
    "orders": ["customers", "products"],
    "transactions": ["orders", "customers"],
    "inventory": ["products"],
    "preferences": ["customers"]
  }
}
```

### Step 2: Understand Resource Constraints

Read the constraints document:
```
file_read: SANDBOX/migration/constraints.md
```

Constraints:
```markdown
# Migration Constraints

## Time Window
- Available window: 4 hours (maintenance mode)
- Must complete ALL critical tables within window
- Non-critical tables can be done in follow-up migration

## Memory Limits
- Maximum 2GB RAM available for migration process
- Cannot load entire tables into memory
- Must process in chunks

## Network Limits
- Bandwidth: 100 MB/s sustained
- Cannot saturate network (affects other systems)
- Target: 50 MB/s average

## Priority Classification
- CRITICAL: customers, orders, products, transactions
- IMPORTANT: inventory, preferences
- DEFERRABLE: audit_logs, user_sessions

## Downtime Rules
- Customers table must migrate first (all others depend on it)
- Orders must complete before transactions
- Products must complete before inventory
```

### Step 3: Planning with Constraints

**BEFORE starting migration, create a resource-aware plan:**

Calculate:
1. **Time estimates** for each table at 50 MB/s
2. **Memory requirements** for chunk processing
3. **Critical path** respecting dependencies
4. **Feasibility check** - can we finish in 4 hours?

Write to `OUTPUT/migration_plan.md`:

```markdown
# Migration Plan

## Resource Calculations

### Time Estimates (at 50 MB/s)
| Table | Size (MB) | Est. Time | Priority |
|-------|-----------|-----------|----------|
| customers | 120 | 2.4 sec | CRITICAL |
| orders | 800 | 16 sec | CRITICAL |
| products | 15 | 0.3 sec | CRITICAL |
| inventory | 40 | 0.8 sec | IMPORTANT |
| transactions | 3500 | 70 sec | CRITICAL |
| audit_logs | 8000 | 160 sec | DEFERRABLE |
| user_sessions | 500 | 10 sec | DEFERRABLE |
| preferences | 25 | 0.5 sec | IMPORTANT |

### Total Critical Path Time
[Calculate: sum of critical tables respecting dependencies]

### Memory Planning
- Chunk size: [X MB] to stay under 2GB
- Rows per chunk: ~[N] (estimated)

### Feasibility Assessment
- Critical tables total: [X MB]
- Time needed for critical: [Y minutes]
- Time available: 240 minutes
- Feasibility: [YES/NO/TIGHT]

## Execution Order

### Phase 1: Foundation (must be first)
1. customers (120 MB) - no dependencies
2. products (15 MB) - no dependencies

### Phase 2: Core Business
3. orders (800 MB) - depends on customers, products
4. inventory (40 MB) - depends on products

### Phase 3: Financial
5. transactions (3500 MB) - depends on orders, customers

### Phase 4: User Data (if time permits)
6. preferences (25 MB) - depends on customers

### Deferred to Follow-up
- audit_logs (8000 MB) - too large for window
- user_sessions (500 MB) - not critical

## Contingency Plans

### If falling behind schedule
- [ ] Increase chunk size (risk: memory pressure)
- [ ] Skip preferences (impact: minor UX degradation)
- [ ] Parallel process independent tables

### If memory issues
- [ ] Reduce chunk size
- [ ] Add explicit garbage collection
- [ ] Process one table at a time (slower but safer)

### Go/No-Go Checkpoints
- After customers: [expected time] - if behind, evaluate
- After orders: [expected time] - if behind, may skip preferences
- At 3-hour mark: [what should be done by now]
```

### Step 4: Execute Migration

For each table, simulate the migration:

```python
# Example for customers table
def migrate_table(table_name, chunk_size_mb=100):
    # Read manifest for table info
    info = get_table_info(table_name)

    # Calculate chunks
    num_chunks = ceil(info.size_mb / chunk_size_mb)

    for chunk in range(num_chunks):
        # Simulate chunk processing
        file_write(f"SANDBOX/migration/progress/{table_name}_chunk_{chunk}.done",
                   f"Migrated rows {chunk * rows_per_chunk} to {(chunk+1) * rows_per_chunk}")

    # Mark table complete
    file_write(f"SANDBOX/migration/progress/{table_name}_complete.flag",
               f"Table {table_name} migration complete at {timestamp}")
```

### Step 5: Progress Monitoring

After each table, update progress:

Write to `OUTPUT/migration_progress.md`:

```markdown
# Migration Progress

## Status: [IN_PROGRESS / COMPLETE / FAILED]
## Start Time: [timestamp]
## Current Time: [timestamp]
## Elapsed: [X minutes]

## Table Status

| Table | Status | Time | % Complete | On Schedule? |
|-------|--------|------|------------|--------------|
| customers | COMPLETE | 2m | 100% | ✓ |
| products | COMPLETE | 0.5m | 100% | ✓ |
| orders | IN_PROGRESS | 10m | 60% | ✓ |
| ... | | | | |

## Resource Utilization
- Memory: [X MB / 2000 MB]
- Network: [X MB/s / 50 MB/s target]

## Time Budget
- Elapsed: [X] minutes
- Remaining: [Y] minutes
- Critical work remaining: [Z MB]
- Projected completion: [timestamp]
- Status: [ON_TRACK / AT_RISK / BEHIND]

## Decisions Made
- [Any adjustments to plan based on actual performance]
```

### Step 6: Handle Resource Pressure

At 3-hour mark, evaluate status:

If behind schedule:
```markdown
## 3-Hour Checkpoint

### Status
- Critical tables remaining: [list]
- Estimated time needed: [X minutes]
- Time remaining: 60 minutes
- Gap: [Y minutes]

### Options
1. Skip preferences (saves ~1 minute) - INSUFFICIENT
2. Increase network to 75 MB/s (risk: impact other systems)
3. Request 30-minute extension
4. Accept partial migration, defer remaining to follow-up

### Decision
[What I'm doing and why]

### Adjusted Plan
[Updated execution order if changed]
```

### Step 7: Complete Migration

Write final report to `OUTPUT/migration_report.md`:

```markdown
# Migration Report

## Summary
- Status: [COMPLETE / PARTIAL / FAILED]
- Window used: [X of 240 minutes]
- Tables migrated: [N of 8]
- Rows migrated: [total]
- Data transferred: [X MB]

## Results by Table

| Table | Status | Rows | Time | Notes |
|-------|--------|------|------|-------|
| customers | ✓ | 50000 | 2m | |
| products | ✓ | 5000 | 0.5m | |
| orders | ✓ | 500000 | 16m | |
| inventory | ✓ | 25000 | 1m | |
| transactions | ✓ | 2000000 | 72m | |
| preferences | ✓ | 50000 | 0.5m | |
| audit_logs | DEFERRED | - | - | Scheduled for follow-up |
| user_sessions | DEFERRED | - | - | Scheduled for follow-up |

## Resource Usage

### Peak Memory
- Maximum: [X MB]
- Chunk size used: [Y MB]
- Any pressure events: [YES/NO]

### Network Utilization
- Average: [X MB/s]
- Peak: [Y MB/s]
- Throttling events: [N]

## Plan vs Actual

| Phase | Planned | Actual | Variance |
|-------|---------|--------|----------|
| Phase 1 | 3m | 2.5m | -0.5m |
| Phase 2 | 17m | 18m | +1m |
| ... | | | |

## Lessons Learned
- [What worked well]
- [What could be improved]
- [Recommendations for follow-up migration]

## Follow-up Required
- [ ] Migrate audit_logs (8GB)
- [ ] Migrate user_sessions (500MB)
- [ ] Verify data integrity
```

---

## Expected Behavior

**Without Resource Awareness:**
```
Agent tries to migrate everything
Runs out of time at transactions table
OR loads too much into memory
OR ignores dependencies
Chaos ensues
```

**With Resource-Aware Planning:**
```
Agent calculates total time
Recognizes audit_logs won't fit in window
Defers non-critical tables
Plans chunk sizes for memory
Sets checkpoints
Completes all critical work
Documents deferral clearly
```

---

## Success Criteria

| Criteria | Naive Agent | Resource-Aware Agent |
|----------|-------------|---------------------|
| Calculates time budget | No | Yes |
| Identifies won't-fit items | No | Yes |
| Plans chunk sizes | No | Yes |
| Respects dependencies | Maybe | Yes |
| Sets checkpoints | No | Yes |
| Handles falling behind | Panics | Adjusts |
| Completes critical work | Maybe | Yes |
| Documents deferrals | No | Yes |

**Pass Condition:**
- Plan includes time calculations before starting
- Audit_logs identified as deferrable early (not discovered late)
- Dependencies respected (customers before orders)
- Chunk size planning for memory
- All CRITICAL tables migrated
- Clear documentation of what was deferred and why

**Fail Condition:**
- Started migration without time/resource planning
- Attempted to migrate all tables in sequence
- Ran out of time on critical tables
- Memory exceeded due to no chunking plan
- No checkpoint/adjustment points

---

## Test Data Setup

Create migration structure:

```
SANDBOX/migration/
├── manifest.json       (table sizes and dependencies)
├── constraints.md      (time/memory/network limits)
└── progress/           (for tracking completion)
```

The test validates PLANNING with CONSTRAINTS, not actual data movement.

---

## What This Reveals

**Without Resource Planning:**
- Agent starts with largest table first (audit_logs)
- Runs out of 4-hour window mid-migration
- Critical data (customers, orders) never migrated
- Total failure

**With Resource Planning:**
- Agent calculates: critical = 4.5GB, window = 4 hours at 50MB/s = ~720 GB capacity (plenty)
- BUT audit_logs (8GB) would consume most of window
- Strategically defers audit_logs
- Completes all critical tables
- Documents deferral for follow-up
- Success with clear handoff

The key insight: **You must plan based on constraints BEFORE starting, not discover constraints during execution.**
