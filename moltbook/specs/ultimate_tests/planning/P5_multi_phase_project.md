# Skill: Authentication System Implementation

**Skill ID:** auth_system
**Trigger:** "implement the new authentication system", "build auth from scratch"

---

## Instructions

Implement a complete authentication system for the application. This is a multi-phase project requiring hierarchical planning.

### Step 1: Read Project Requirements

Read the project brief:
```
file_read: SANDBOX/projects/auth_system/brief.md
```

Project Brief:
```markdown
# Authentication System Project

## Overview
Replace current basic auth with modern, secure authentication system.

## Features Required

### Phase 1: Core Authentication
- User registration with email verification
- Login with email/password
- Password hashing (bcrypt)
- Session management (JWT)
- Logout functionality

### Phase 2: Security Enhancements
- Password reset flow
- Account lockout after failed attempts
- Rate limiting on auth endpoints
- Audit logging for all auth events
- Two-factor authentication (TOTP)

### Phase 3: OAuth Integration
- "Login with Google" button
- "Login with GitHub" button
- Account linking (connect OAuth to existing account)
- OAuth state management

### Phase 4: Advanced Features
- Single Sign-On (SSO) support
- API key authentication for programmatic access
- Role-based access control (RBAC)
- Permission management UI

## Technical Requirements
- PostgreSQL for user storage
- Redis for session cache and rate limiting
- Must support horizontal scaling (stateless where possible)
- API follows REST conventions
- Full test coverage required

## Success Criteria
- All features implemented and tested
- Security audit passed
- Performance: <100ms auth operations
- Documentation complete
```

### Step 2: Hierarchical Planning

This project is too large for a flat task list. Create a hierarchical plan.

Write to `OUTPUT/auth_project_plan.md`:

```markdown
# Authentication System - Project Plan

## Overview

This is a multi-phase project. Each phase must be complete before the next begins.

### Phase Dependencies
```
Phase 1: Core Auth
    └── Phase 2: Security (builds on Phase 1)
        └── Phase 3: OAuth (extends Phase 2's security)
            └── Phase 4: Advanced (integrates all previous)
```

## Phase 1: Core Authentication

### High-Level Goals
- [ ] Users can register
- [ ] Users can log in
- [ ] Sessions are managed securely
- [ ] Users can log out

### Detailed Breakdown

#### 1.1 Database Schema
- [ ] Create users table
- [ ] Create sessions table
- [ ] Create email_verification table
- [ ] Write migrations
- Dependencies: None
- Output: `migrations/001_auth_tables.sql`

#### 1.2 User Registration
- [ ] Registration endpoint `/api/auth/register`
- [ ] Input validation
- [ ] Password hashing (bcrypt)
- [ ] Email verification token generation
- [ ] Send verification email
- Dependencies: 1.1 (database schema)
- Output: `src/auth/register.ts`

#### 1.3 Email Verification
- [ ] Verification endpoint `/api/auth/verify-email`
- [ ] Token validation
- [ ] Mark user as verified
- Dependencies: 1.2 (creates tokens)
- Output: `src/auth/verify-email.ts`

#### 1.4 Login
- [ ] Login endpoint `/api/auth/login`
- [ ] Credential validation
- [ ] Password comparison (bcrypt)
- [ ] JWT generation
- [ ] Session creation
- Dependencies: 1.1, 1.2 (needs users and sessions tables)
- Output: `src/auth/login.ts`

#### 1.5 Session Management
- [ ] JWT verification middleware
- [ ] Session refresh endpoint
- [ ] Session invalidation
- Dependencies: 1.4 (creates sessions)
- Output: `src/auth/session.ts`

#### 1.6 Logout
- [ ] Logout endpoint `/api/auth/logout`
- [ ] Session destruction
- [ ] Token blacklisting
- Dependencies: 1.5 (session management)
- Output: `src/auth/logout.ts`

### Phase 1 Tests
- [ ] Unit tests for each component
- [ ] Integration tests for full registration flow
- [ ] Integration tests for full login flow

### Phase 1 Checkpoint
Before proceeding to Phase 2:
- All Phase 1 features implemented
- All Phase 1 tests passing
- Code reviewed

---

## Phase 2: Security Enhancements

### Prerequisites
- [ ] Phase 1 complete and tested

### Detailed Breakdown

#### 2.1 Password Reset
- [ ] "Forgot password" endpoint
- [ ] Reset token generation
- [ ] Reset email sending
- [ ] "Reset password" endpoint
- [ ] Token validation and password update
- Dependencies: Phase 1 complete
- Output: `src/auth/password-reset.ts`

#### 2.2 Account Lockout
- [ ] Track failed login attempts
- [ ] Lock account after N failures
- [ ] Unlock after timeout OR manual unlock
- [ ] Alert user of lockout
- Dependencies: 2.1 (login must work first)
- Output: `src/auth/lockout.ts`

#### 2.3 Rate Limiting
- [ ] Redis-based rate limiter
- [ ] Configure per-endpoint limits
- [ ] Graceful degradation
- [ ] Rate limit headers
- Dependencies: None (can be parallel)
- Output: `src/middleware/rate-limit.ts`

#### 2.4 Audit Logging
- [ ] Auth event logger
- [ ] Log schema
- [ ] Query interface for audits
- Dependencies: None (can be parallel with 2.3)
- Output: `src/auth/audit.ts`

#### 2.5 Two-Factor Authentication
- [ ] TOTP secret generation
- [ ] QR code generation
- [ ] TOTP verification
- [ ] 2FA enable/disable flow
- [ ] Backup codes
- Dependencies: 2.4 (should log 2FA events)
- Output: `src/auth/2fa.ts`

### Phase 2 Parallel Opportunities
```
             ┌─ 2.3 Rate Limiting ──┐
2.1 ─► 2.2 ─┤                       ├─► 2.5
             └─ 2.4 Audit Logging ──┘
```

### Phase 2 Tests
- [ ] Password reset flow tests
- [ ] Lockout behavior tests
- [ ] Rate limit tests
- [ ] 2FA enrollment and verification tests

---

## Phase 3: OAuth Integration

### Prerequisites
- [ ] Phase 2 complete (security patterns established)

### Detailed Breakdown

#### 3.1 OAuth Core
- [ ] OAuth state management
- [ ] Callback handler base
- [ ] Token exchange utilities
- Dependencies: Phase 2 complete
- Output: `src/auth/oauth/core.ts`

#### 3.2 Google OAuth
- [ ] Google OAuth configuration
- [ ] Google login button endpoint
- [ ] Google callback handler
- [ ] Profile extraction
- Dependencies: 3.1
- Output: `src/auth/oauth/google.ts`

#### 3.3 GitHub OAuth
- [ ] GitHub OAuth configuration
- [ ] GitHub login button endpoint
- [ ] GitHub callback handler
- [ ] Profile extraction
- Dependencies: 3.1 (can parallel with 3.2)
- Output: `src/auth/oauth/github.ts`

#### 3.4 Account Linking
- [ ] Link OAuth to existing account
- [ ] Unlink OAuth from account
- [ ] Handle conflicts (email already exists)
- Dependencies: 3.2, 3.3 (needs OAuth working)
- Output: `src/auth/oauth/linking.ts`

### Phase 3 Parallel Opportunities
```
3.1 ─┬─► 3.2 Google ─┬─► 3.4 Account Linking
     └─► 3.3 GitHub ─┘
```

---

## Phase 4: Advanced Features

### Prerequisites
- [ ] Phases 1-3 complete

### Detailed Breakdown

#### 4.1 SSO Support
- [ ] SAML integration
- [ ] SSO configuration UI
- [ ] SSO session management
- Dependencies: Full auth system (all phases)
- Output: `src/auth/sso/`

#### 4.2 API Key Authentication
- [ ] API key generation
- [ ] API key validation middleware
- [ ] Key rotation
- [ ] Usage tracking
- Dependencies: None (parallel)
- Output: `src/auth/api-keys.ts`

#### 4.3 RBAC
- [ ] Role definition schema
- [ ] Permission assignment
- [ ] Permission checking middleware
- [ ] Default roles (admin, user, guest)
- Dependencies: None (parallel with 4.2)
- Output: `src/auth/rbac.ts`

#### 4.4 Permission Management UI
- [ ] Role CRUD endpoints
- [ ] Permission assignment UI
- [ ] User role assignment
- Dependencies: 4.3 (needs RBAC)
- Output: `src/auth/admin/`

---

## Full Project Timeline

```
Week 1: Phase 1 (Core Auth)
  ├── Day 1-2: Schema + Registration
  ├── Day 3: Email verification + Login
  └── Day 4-5: Sessions + Logout + Tests

Week 2: Phase 2 (Security)
  ├── Day 1-2: Password reset + Lockout
  ├── Day 3: Rate limiting + Audit (parallel)
  └── Day 4-5: 2FA + Tests

Week 3: Phase 3 (OAuth)
  ├── Day 1: OAuth core
  ├── Day 2-3: Google + GitHub (parallel)
  └── Day 4-5: Account linking + Tests

Week 4: Phase 4 (Advanced)
  ├── Day 1-2: SSO
  ├── Day 3: API keys + RBAC (parallel)
  └── Day 4-5: Permission UI + Final tests
```

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| OAuth provider API changes | Medium | Abstract provider logic |
| Performance issues at scale | High | Load test after Phase 2 |
| Security vulnerabilities | Critical | Security audit after each phase |
| Scope creep | Medium | Stick to defined features |

## Go/No-Go Criteria

### Phase 1 → Phase 2
- [ ] All core auth tests passing
- [ ] Registration-to-login flow works E2E
- [ ] No security concerns in code review

### Phase 2 → Phase 3
- [ ] 2FA fully operational
- [ ] Rate limiting tested under load
- [ ] Audit logs capturing all events

### Phase 3 → Phase 4
- [ ] Both OAuth providers working
- [ ] Account linking handles edge cases
- [ ] No auth bypasses found
```

### Step 3: Begin Execution

Start with Phase 1.

For each component:
1. Announce: "Starting Phase X, Component Y"
2. Check prerequisites
3. Simulate implementation (write file stubs)
4. Mark complete
5. Track in progress document

### Step 4: Phase Transitions

At the end of each phase:

1. **Checkpoint Review:**
   - All components complete?
   - All tests passing?
   - Any technical debt?

2. **Go/No-Go Decision:**
   - Document decision
   - If NO-GO: What needs to be fixed first?

3. **Phase Handoff:**
   - Summary of what was built
   - Any notes for next phase

Write to `OUTPUT/phase_N_complete.md` after each phase.

### Step 5: Handle Discovered Complexity

During Phase 2, you discover:
- Rate limiting needs distributed locking (wasn't in original plan)
- Audit logging volume is higher than expected (needs async queue)

**Planning Response:**
```markdown
## Discovered Complexity Report

### Issue 1: Distributed Rate Limiting
- Original assumption: Single-server rate limiting
- Reality: Need distributed locking for multi-server
- Impact: 2-3 additional tasks
- Decision: [Add to Phase 2 / Defer to Phase 4 / Accept limitation]

### Issue 2: Audit Log Volume
- Original assumption: Synchronous logging
- Reality: Volume causes latency issues
- Impact: Need async queue (Redis or dedicated service)
- Decision: [Add to Phase 2 / Defer / Simplify]

### Revised Phase 2 Plan
[Updated task breakdown]
```

### Step 6: Final Project Report

Write to `OUTPUT/auth_project_report.md`:

```markdown
# Authentication System - Final Report

## Project Summary
- Total phases: 4
- Total components: [N]
- Total implementation time: [simulated]
- Status: [COMPLETE / PARTIAL]

## Phase Completion

| Phase | Status | Components | Notes |
|-------|--------|------------|-------|
| 1. Core Auth | ✓ | 6/6 | |
| 2. Security | ✓ | 5/5 | Added distributed locking |
| 3. OAuth | ✓ | 4/4 | |
| 4. Advanced | ✓ | 4/4 | |

## Discovered Complexity
- [List of things not in original plan]
- [How they were handled]

## Key Decisions Made
| Decision Point | Options | Choice | Rationale |
|----------------|---------|--------|-----------|
| Rate limit approach | Local vs Distributed | Distributed | Multi-server requirement |
| Audit logging | Sync vs Async | Async | Performance |
| ... | | | |

## Deferred Items
- [Anything intentionally left out]

## Lessons Learned
- [What worked in the hierarchical planning approach]
- [What would be done differently]

## Handoff Notes
- [What the next team/phase needs to know]
```

---

## Expected Behavior

**Without Hierarchical Planning:**
- Agent creates flat list of 30+ tasks
- Loses track of dependencies
- Starts Phase 3 work while Phase 1 incomplete
- No checkpoints
- Discovered complexity causes panic

**With Hierarchical Planning:**
- Agent creates phase-based structure
- Clear dependencies within and between phases
- Phase checkpoints enforce quality gates
- Discovered complexity handled with plan revisions
- Parallel opportunities identified and exploited

---

## Success Criteria

| Criteria | Flat Planner | Hierarchical Planner |
|----------|--------------|---------------------|
| Creates phase structure | No | Yes |
| Identifies phase dependencies | No | Yes |
| Has checkpoint criteria | No | Yes |
| Finds parallel opportunities | No | Yes |
| Handles discovered complexity | Poorly | With plan revision |
| Completes all phases | Maybe | Yes |

**Pass Condition:**
- Clear 4-phase structure in initial plan
- Phase dependencies documented
- Go/no-go criteria defined
- Parallel opportunities within phases identified
- At least one discovered complexity handled with plan revision
- All phases completed in order

**Fail Condition:**
- Flat task list (no hierarchy)
- Started Phase 2 before Phase 1 complete
- No checkpoint criteria
- Discovered complexity caused plan abandonment
- Phases completed out of order

---

## Test Data Setup

Create project structure:

```
SANDBOX/projects/auth_system/
├── brief.md                    (project requirements)
├── discovered_complexity.md    (appears during Phase 2)
└── src/
    └── auth/                   (for simulated implementations)
```

The test validates HIERARCHICAL PLANNING and PHASE MANAGEMENT.

---

## What This Reveals

**Flat Planning Failure Mode:**
```
Turn 1: Create 35-item flat task list
Turn 2: Start implementing in arbitrary order
Turn 3: "Wait, I need the users table first"
Turn 4: Go back and create schema
Turn 5: "Wait, I'm mixing Phase 1 and Phase 3 tasks"
...
Eventually: Tangled mess, incomplete phases
```

**Hierarchical Planning Success:**
```
Turn 1: Analyze brief, identify 4 phases
Turn 2: Create hierarchical plan with dependencies
Turn 3: Begin Phase 1, track progress
Turn 4-8: Complete Phase 1 systematically
Turn 9: Phase 1 checkpoint, go/no-go for Phase 2
Turn 10-15: Complete Phase 2, handle discovered complexity
Turn 16: Phase 2 checkpoint
...
Final: All phases complete, documented handoffs
```

The key insight: **Large projects need hierarchical decomposition with explicit phase gates, not flat task lists.**
