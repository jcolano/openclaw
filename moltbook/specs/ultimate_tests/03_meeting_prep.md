# Ultimate Test 03: Meeting Preparation Skill

**Trigger:** "prepare for meeting with [PERSON/COMPANY]" or "meeting prep [NAME]"

**Expected Duration:** 5-8 minutes autonomous execution

---

## Skill Instructions

When user asks to prepare for a meeting, follow these steps exactly:

### Phase 1: Context Gathering

1. **Acknowledge and create workspace**:
   - Confirm meeting preparation request for [NAME]
   - Create `OUTPUT/meeting_prep_[NAME]_[DATE]/`
   - Write initial notes file with meeting target and date

2. **Check memory for history**:
   - Search memory for previous interactions with [NAME]
   - Search memory for [COMPANY] if company name mentioned
   - IF history found, summarize in `OUTPUT/meeting_prep_[NAME]_[DATE]/history.md`
   - IF no history, note "First known interaction with [NAME]"

3. **Ask user for context** (if needed):
   - IF meeting purpose unclear, formulate a question but continue with research
   - Note: "Proceeding with general research. Purpose can be refined later."

### Phase 2: Person Research

4. **Search for person information**:
   - Use `web_fetch` on LinkedIn search: `https://www.linkedin.com/search/results/people/?keywords=[NAME]`
   - IF blocked, try: `web_fetch` on `https://www.google.com/search?q=[NAME]+[COMPANY]`

5. **Extract person details**:
   - Current role and company
   - Previous roles (career trajectory)
   - Education background
   - Shared connections or interests (if visible)
   - Recent posts or articles (if available)

6. **Write person profile**: Save to `OUTPUT/meeting_prep_[NAME]_[DATE]/person_profile.md`:
   ```
   # [NAME] - Profile

   ## Current Role
   [Title] at [Company]

   ## Background
   [Career summary]

   ## Potential Talking Points
   - [Based on their interests/posts]
   - [Shared connections]
   - [Recent company news]
   ```

### Phase 3: Company Research

7. **IF company identified**:
   a. Fetch company website homepage
   b. Look for: About page, News/Press page, Leadership page
   c. Extract: company mission, recent news, key products/services

8. **Search for recent company news**:
   - Use `web_fetch` on news search for "[COMPANY] news"
   - Extract headlines from last 30 days
   - Note any significant events: funding, acquisitions, layoffs, product launches

9. **Check company financials** (if public):
   - IF public company, search for recent earnings
   - Note stock performance trend (up/down/stable)
   - IF private, note "Private company - financial data limited"

10. **Write company brief**: Save to `OUTPUT/meeting_prep_[NAME]_[DATE]/company_brief.md`

### Phase 4: Prepare Materials

11. **Generate talking points**: Based on research, create:
    - 3 rapport-building topics (personal connection points)
    - 3 business discussion topics (relevant to likely meeting purpose)
    - 3 questions to ask them (shows preparation)

12. **Identify potential concerns**:
    - IF negative news found, prepare response strategy
    - IF competitor relationship found, note sensitivity
    - IF recent changes at company, note potential instability

13. **Create agenda template**: Write to `OUTPUT/meeting_prep_[NAME]_[DATE]/agenda_template.md`:
    ```
    # Meeting with [NAME]
    Date: [DATE]
    Duration: [TBD]

    ## Objectives
    1. [Primary goal]
    2. [Secondary goal]
    3. [Relationship building]

    ## Agenda Items
    1. Introduction / Rapport (5 min)
       - Topics: [from talking points]
    2. Main Discussion (20 min)
       - [Topic 1]
       - [Topic 2]
    3. Next Steps (5 min)
       - [Action items]

    ## Questions to Ask
    - [Question 1]
    - [Question 2]
    - [Question 3]

    ## Notes Space
    [Leave blank for meeting notes]
    ```

### Phase 5: Risk Assessment

14. **Analyze meeting risks**:
    - IF person is senior executive: Note "Expect limited time, be concise"
    - IF company in financial trouble: Note "May have budget constraints"
    - IF first meeting: Note "Focus on relationship building over sales"
    - IF competitor mentioned: Note "Avoid discussing [competitor] negatively"

15. **Create risk matrix**: Write to `OUTPUT/meeting_prep_[NAME]_[DATE]/risks.md`:
    ```
    # Meeting Risks & Mitigations

    | Risk | Likelihood | Impact | Mitigation |
    |------|------------|--------|------------|
    | [Risk 1] | High/Med/Low | High/Med/Low | [Strategy] |
    ```

### Phase 6: Synthesis

16. **Remember key facts**: Store in memory:
    - "[NAME] works at [COMPANY] as [ROLE]"
    - "Meeting prepared on [DATE]"
    - "Key interests: [list 2-3]"

17. **Generate executive brief**: Write to `OUTPUT/meeting_prep_[NAME]_[DATE]/executive_brief.md`:
    ```
    # Meeting Prep: [NAME] @ [COMPANY]
    Prepared: [DATE]

    ## 30-Second Summary
    [2-3 sentences: who they are, why meeting matters, key approach]

    ## Must-Know Facts
    1. [Critical fact 1]
    2. [Critical fact 2]
    3. [Critical fact 3]

    ## Icebreakers
    - [Topic 1]
    - [Topic 2]

    ## Key Questions
    - [Question 1]
    - [Question 2]

    ## Watch Out For
    - [Risk/sensitivity 1]
    - [Risk/sensitivity 2]
    ```

18. **Present to user**: Provide the 30-second summary and ask:
    - "Want me to elaborate on any section?"
    - "Is there a specific angle for this meeting I should focus on?"

---

## Error Handling

- IF person not found online: Note "Limited public information available" and focus on company research
- IF company not identified: Ask user to clarify, proceed with person-only research
- IF all web fetches fail: Create template documents with placeholder sections marked "[RESEARCH NEEDED]"
- IF person has common name: Note ambiguity, ask user to confirm if multiple profiles found

---

## Success Criteria

- [ ] Created meeting prep directory with at least 5 files
- [ ] Person profile includes current role and background
- [ ] Talking points generated
- [ ] Agenda template created
- [ ] Memory updated with key facts
- [ ] Executive brief ready for quick review
