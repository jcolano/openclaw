# Skill: Development Environment Setup

**Skill ID:** dev_env_setup
**Trigger:** "set up the development environment for the new project", "initialize dev environment"

---

## Instructions

Set up a complete development environment for a new Python web project.

### Required Components

The following components need to be set up:

1. **Python Virtual Environment**
   - Create venv in `SANDBOX/project/`
   - Requires: Nothing (can start immediately)

2. **Dependencies Installation**
   - Install packages from requirements.txt
   - Requires: Virtual environment (Component 1)

3. **Database Setup**
   - Initialize PostgreSQL database
   - Run migrations
   - Requires: Dependencies installed (Component 2) - needs ORM

4. **Configuration Files**
   - Create .env from .env.example
   - Set database connection string
   - Requires: Database setup (Component 3) - needs DB credentials

5. **Cache Layer**
   - Configure Redis connection
   - Verify connectivity
   - Requires: Configuration files (Component 4) - reads from .env

6. **Test Database**
   - Create separate test database
   - Run test migrations
   - Requires: Database setup (Component 3) AND Configuration (Component 4)

7. **Seed Data**
   - Load initial data fixtures
   - Requires: Database (Component 3) AND Test Database (Component 6)

8. **Verification**
   - Run health checks
   - Execute test suite
   - Requires: ALL above components

### Dependency Graph

```
    [1: Venv]
        │
        ▼
    [2: Dependencies]
        │
        ▼
    [3: Database]
        │
        ├──────────────┐
        ▼              ▼
    [4: Config]    [6: Test DB]
        │              │
        ▼              │
    [5: Cache]         │
        │              │
        ├──────────────┘
        ▼
    [7: Seed Data]
        │
        ▼
    [8: Verification]
```

### Your Task

Set up all components in the correct order. Document your approach.

---

## Planning Checkpoint

**BEFORE starting any work, create a plan:**

1. What is the correct order of operations?
2. Which components depend on which?
3. What can go wrong at each step?
4. What's my rollback strategy if step N fails?

Write your plan to `OUTPUT/setup_plan.md` BEFORE executing.

---

## Execution Requirements

### For Each Component:

1. Announce: "Starting Component N: [Name]"
2. Check prerequisites: "Prerequisites: [list] - Status: [met/unmet]"
3. Execute the setup step
4. Verify success
5. Document: "Component N complete. Output: [details]"

### Simulated Commands

Since this is a test environment, simulate each step:

```python
# Component 1: Virtual Environment
file_write("SANDBOX/project/venv_created.flag", "venv created at {timestamp}")

# Component 2: Dependencies
# First check: does venv exist?
file_read("SANDBOX/project/venv_created.flag")  # Must succeed
file_write("SANDBOX/project/deps_installed.flag", "installed: flask, sqlalchemy, ...")

# Component 3: Database
# First check: deps installed?
file_read("SANDBOX/project/deps_installed.flag")  # Must succeed
file_write("SANDBOX/project/db_ready.flag", "database: project_db, migrations: complete")

# ... and so on
```

---

## Expected Plan Output

Your plan should look like:

```markdown
# Development Environment Setup Plan

## Dependency Analysis

| Component | Depends On | Can Parallelize With |
|-----------|------------|----------------------|
| 1. Venv | None | - |
| 2. Dependencies | 1 | - |
| 3. Database | 2 | - |
| 4. Config | 3 | 6 |
| 5. Cache | 4 | - |
| 6. Test DB | 3 | 4 |
| 7. Seed Data | 3, 6 | - |
| 8. Verification | All | - |

## Execution Order

Phase 1: [1]
Phase 2: [2]
Phase 3: [3]
Phase 4: [4, 6] (parallel)
Phase 5: [5]
Phase 6: [7]
Phase 7: [8]

## Risk Assessment

- Risk: Dependencies fail to install
  Mitigation: Check error, retry with --no-cache

- Risk: Database connection fails
  Mitigation: Verify credentials, check if service running

## Rollback Strategy

If step N fails, rollback steps N-1 through 1 in reverse order.
```

---

## Success Criteria

| Criteria | Without Planning | With Planning |
|----------|------------------|---------------|
| Creates plan first | ✗ | ✓ |
| Correct order | ✗ (backtracking) | ✓ |
| Checks prerequisites | ✗ | ✓ |
| Completes all steps | Maybe (if lucky) | ✓ |
| Handles failure gracefully | ✗ | ✓ |

**Pass Condition:**
- Plan created BEFORE any setup commands
- Correct dependency order identified
- All 8 components completed without backtracking
- Each step checked prerequisites before executing

**Fail Condition:**
- Dove into execution without plan
- Had to redo steps due to wrong order
- Attempted Component 4 before Component 3, etc.

---

## Test Data Setup

Create initial structure:
```
SANDBOX/project/
├── requirements.txt
├── .env.example
└── migrations/
    └── 001_initial.sql
```

The test validates ORDER, not actual environment setup.

---

## What This Reveals

**Without Planning:**
```
Turn 1: "Let me start by creating the config file..."
Turn 2: "Oh, I need the database first for credentials"
Turn 3: "Wait, I need dependencies for the ORM"
Turn 4: "Actually, I need venv for dependencies"
Turn 5: Finally starting at the right place
...
Many wasted turns from wrong order
```

**With Planning:**
```
Turn 1: Analyze dependencies, create execution plan
Turn 2: Execute Phase 1 (venv)
Turn 3: Execute Phase 2 (deps)
Turn 4: Execute Phase 3 (db)
Turn 5: Execute Phase 4 (config + test db in parallel)
...
Linear progress, no backtracking
```
