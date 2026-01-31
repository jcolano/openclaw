# Ultimate Test 08: Customer Onboarding Workflow Skill

**Trigger:** "onboard customer [NAME]" or "new customer setup [COMPANY]"

**Expected Duration:** 5-10 minutes for full onboarding workflow

---

## Skill Instructions

When user asks to onboard a new customer, follow these steps exactly:

### Phase 1: Information Gathering

1. **Acknowledge and start**:
   - Confirm customer name/company
   - Generate customer ID: `CUST-[YYYYMMDD]-[sequence]`
   - Create `OUTPUT/onboarding/[CUSTOMER_ID]/`

2. **Create customer record**: Write to `customer_record.json`:
   ```json
   {
     "customer_id": "[ID]",
     "company_name": "[COMPANY]",
     "contact_name": "[NAME if provided]",
     "onboarding_started": "[timestamp]",
     "status": "in_progress",
     "checklist": {
       "info_collected": false,
       "account_created": false,
       "welcome_sent": false,
       "training_scheduled": false,
       "documents_shared": false,
       "integration_setup": false,
       "success_call_scheduled": false
     }
   }
   ```

3. **Check for existing customer**:
   - Search memory for [COMPANY] or [NAME]
   - IF existing relationship found:
     a. Note previous interactions
     b. Flag as "returning customer" or "expansion"
     c. Adjust onboarding for context

4. **Gather required information**:
   - Check if user provided: contact email, company size, use case
   - FOR EACH missing field:
     a. Note as "needed from user"
     b. Continue with available info

5. **Research customer** (if company name provided):
   - Fetch company website via `web_fetch`
   - Extract: industry, size estimate, products/services
   - Note potential use cases based on business type
   - Write to `OUTPUT/onboarding/[CUSTOMER_ID]/customer_research.md`

### Phase 2: Account Setup

6. **Prepare account creation**:
   - Generate temporary password (note: for demo purposes only)
   - Determine account tier based on company size:
     - 1-10 employees: Starter
     - 11-100 employees: Professional
     - 100+ employees: Enterprise

7. **Create account** (simulated via API):
   ```
   POST /api/accounts
   {
     "customer_id": "[ID]",
     "company": "[COMPANY]",
     "tier": "[TIER]",
     "admin_email": "[EMAIL if known]"
   }
   ```
   - IF API call succeeds: Note account ID
   - IF API call fails: Log error, mark for manual creation

8. **Configure account settings**:
   - Set default preferences based on tier
   - Enable appropriate features
   - Set usage limits
   - Log all configurations to `OUTPUT/onboarding/[CUSTOMER_ID]/account_config.md`

9. **Update checklist**: Mark "account_created" as true

### Phase 3: Welcome Communication

10. **Generate welcome email**: Write to `OUTPUT/onboarding/[CUSTOMER_ID]/welcome_email.md`:
    ```
    Subject: Welcome to [Product Name], [CONTACT_NAME]!

    Hi [CONTACT_NAME],

    Welcome to [Product Name]! We're excited to have [COMPANY] on board.

    Here's what you need to get started:

    **Your Account Details**
    - Account ID: [ID]
    - Admin Portal: [URL]
    - Temporary Password: [TEMP_PASS] (please change on first login)

    **Quick Start**
    1. Log in to your admin portal
    2. Complete your profile
    3. Invite your team members
    4. Start your first project

    **Resources**
    - Getting Started Guide: [URL]
    - Video Tutorials: [URL]
    - Support: [EMAIL]

    **Your Onboarding Specialist**
    I'm [AGENT_NAME], your dedicated onboarding specialist. I'm here to help you succeed.

    Next Steps:
    - [ ] Log in and explore
    - [ ] Schedule your kickoff call: [CALENDAR_LINK]

    Questions? Just reply to this email!

    Best,
    [AGENT_NAME]
    ```

11. **Generate personalized elements**:
    - IF specific use case known: Add relevant tips
    - IF enterprise tier: Add VIP contact information
    - IF returning customer: Reference previous relationship

12. **Send welcome email** (simulated):
    ```
    POST /api/email/send
    {
      "to": "[EMAIL]",
      "template": "welcome",
      "customer_id": "[ID]"
    }
    ```

13. **Update checklist**: Mark "welcome_sent" as true

### Phase 4: Resource Preparation

14. **Prepare onboarding documents**:
    a. Getting started guide (copy from template)
    b. Best practices for [INDUSTRY]
    c. FAQ document
    d. Training schedule template
    - Write document list to `OUTPUT/onboarding/[CUSTOMER_ID]/documents.md`

15. **Customize documents**:
    - Replace placeholders with customer info
    - Add industry-specific examples
    - Remove irrelevant sections based on tier

16. **Share documents** (simulated):
    - Create shared folder
    - Set permissions
    - Log share links

17. **Update checklist**: Mark "documents_shared" as true

### Phase 5: Training Setup

18. **Create training plan**: Write to `OUTPUT/onboarding/[CUSTOMER_ID]/training_plan.md`:
    ```
    # Training Plan for [COMPANY]

    ## Recommended Training Path

    ### Week 1: Fundamentals
    - [ ] Platform Overview (30 min)
    - [ ] Admin Setup (45 min)
    - [ ] User Management (30 min)

    ### Week 2: Core Features
    - [ ] [Feature 1] Deep Dive (1 hr)
    - [ ] [Feature 2] Workshop (1 hr)

    ### Week 3: Advanced
    - [ ] Integrations (45 min)
    - [ ] Advanced Workflows (1 hr)
    - [ ] Best Practices Review (30 min)

    ## Training Resources
    - Self-paced videos: [LINK]
    - Live webinars: [SCHEDULE]
    - Documentation: [LINK]

    ## Milestones
    - Day 7: Admin setup complete
    - Day 14: First workflow created
    - Day 30: Full team onboarded
    ```

19. **Schedule kickoff call**:
    - Propose 3 available time slots
    - Create calendar invite draft
    - Write to `OUTPUT/onboarding/[CUSTOMER_ID]/kickoff_invite.md`

20. **Update checklist**: Mark "training_scheduled" as true (pending confirmation)

### Phase 6: Integration & Success Planning

21. **Assess integration needs**:
    - Based on industry, suggest common integrations
    - Check if customer mentioned specific tools
    - Create integration recommendation document

22. **Prepare integration guide**: Write to `OUTPUT/onboarding/[CUSTOMER_ID]/integrations.md`:
    ```
    # Integration Guide for [COMPANY]

    ## Recommended Integrations

    ### High Priority
    1. [Integration 1] - [Reason]
       - Setup time: ~30 min
       - Documentation: [LINK]

    ### Nice to Have
    1. [Integration 2] - [Reason]

    ## Integration Support
    - Self-service guides: [LINK]
    - Integration support: [EMAIL]
    ```

23. **Create success metrics**: Write to `OUTPUT/onboarding/[CUSTOMER_ID]/success_plan.md`:
    ```
    # Success Plan for [COMPANY]

    ## Goals (30/60/90 Days)

    ### 30 Days
    - [ ] All admins logged in
    - [ ] Core workflow created
    - [ ] [Metric]: [Target]

    ### 60 Days
    - [ ] Full team active
    - [ ] 3+ workflows running
    - [ ] [Metric]: [Target]

    ### 90 Days
    - [ ] Fully operational
    - [ ] Integrations complete
    - [ ] Ready for renewal discussion

    ## Check-in Schedule
    - Day 7: Quick check-in
    - Day 14: Progress review
    - Day 30: Success review
    - Day 60: Expansion discussion
    - Day 90: Renewal preparation
    ```

### Phase 7: Completion

24. **Update customer record**:
    - Mark all completed steps
    - Set status to "active" or "pending_confirmation"
    - Record completion timestamp

25. **Remember customer**: Store in memory:
    - "Onboarded [COMPANY] on [DATE]"
    - "Customer ID: [ID]"
    - "Tier: [TIER]"
    - "Key contact: [NAME]"

26. **Generate onboarding summary**: Write to `OUTPUT/onboarding/[CUSTOMER_ID]/summary.md`:
    ```
    # Onboarding Summary: [COMPANY]

    ## Customer Details
    - Customer ID: [ID]
    - Company: [COMPANY]
    - Contact: [NAME]
    - Tier: [TIER]

    ## Onboarding Status
    | Step | Status | Date |
    |------|--------|------|
    | Account Created | ✓ | [DATE] |
    | Welcome Sent | ✓ | [DATE] |
    | Documents Shared | ✓ | [DATE] |
    | Training Scheduled | Pending | - |
    | Kickoff Call | Pending | - |

    ## Documents Created
    - welcome_email.md
    - training_plan.md
    - integrations.md
    - success_plan.md

    ## Next Steps
    1. [Next action]
    2. [Next action]

    ## Notes
    [Any special considerations]
    ```

27. **Report to user**:
    - "Customer [COMPANY] onboarding initiated"
    - "Customer ID: [ID]"
    - "Status: [X/Y steps complete]"
    - "Pending: [list pending items]"
    - "Full details in OUTPUT/onboarding/[CUSTOMER_ID]/"

---

## Error Handling

- IF email not provided: Continue without sending, note "Welcome email pending - need email address"
- IF account creation fails: Log error, create manual task
- IF company research fails: Continue with provided information only
- IF this is a duplicate customer: Alert user, offer to update existing record

---

## Success Criteria

- [ ] Customer ID generated and tracked
- [ ] Customer record created with all available info
- [ ] Account created (or manual task logged)
- [ ] Welcome email drafted
- [ ] Training plan created
- [ ] Success metrics defined
- [ ] All documents in customer folder
- [ ] Memory updated with customer record
- [ ] User received comprehensive summary
