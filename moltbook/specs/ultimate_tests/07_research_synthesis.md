# Ultimate Test 07: Research Synthesis Skill

**Trigger:** "research [TOPIC] from multiple sources" or "synthesize research on [TOPIC]"

**Expected Duration:** 8-15 minutes for comprehensive synthesis

---

## Skill Instructions

When user asks for research synthesis, follow these steps exactly:

### Phase 1: Research Planning

1. **Understand the request**:
   - Extract main topic
   - Identify subtopics or specific questions
   - Determine scope: broad overview vs. deep dive
   - Note any constraints: time period, geography, perspective

2. **Create research workspace**:
   - Create `OUTPUT/research_[TOPIC]_[timestamp]/`
   - Write research plan to `plan.md`:
     ```
     # Research Plan: [TOPIC]

     ## Research Question
     [Primary question]

     ## Sub-questions
     1. [Sub-question 1]
     2. [Sub-question 2]
     3. [Sub-question 3]

     ## Source Strategy
     - Academic: [approach]
     - Industry: [approach]
     - News: [approach]

     ## Scope Boundaries
     - Include: [what's in scope]
     - Exclude: [what's out of scope]
     ```

3. **Define source categories**:
   - Academic/Research papers
   - Industry reports
   - News articles
   - Expert opinions/blogs
   - Official documentation
   - Statistics/data sources

### Phase 2: Source Collection

4. **Academic sources**:
   - Search via `web_fetch` on: `https://scholar.google.com/scholar?q=[TOPIC]`
   - Extract: titles, authors, publication years, abstracts
   - Prioritize: recent papers, high citation counts
   - IF no results: Note "Limited academic coverage"
   - Save to `OUTPUT/research_[TOPIC]_[timestamp]/sources_academic.md`

5. **Industry sources**:
   - Search for "[TOPIC] industry report" or "[TOPIC] market analysis"
   - Fetch from: consulting firms, industry associations
   - Extract key findings and statistics
   - Save to `OUTPUT/research_[TOPIC]_[timestamp]/sources_industry.md`

6. **News sources**:
   - Search recent news (last 12 months)
   - Identify: major events, trends, controversies
   - Note publication dates for timeliness
   - Save to `OUTPUT/research_[TOPIC]_[timestamp]/sources_news.md`

7. **Expert sources**:
   - Search for expert blogs, thought leadership
   - Look for conference talks, interviews
   - Note credentials and potential biases
   - Save to `OUTPUT/research_[TOPIC]_[timestamp]/sources_experts.md`

8. **Compile source list**: Write to `sources_master.md`:
   ```
   # Source Master List

   ## Sources Collected
   | # | Type | Title | Author/Org | Date | URL | Reliability |
   |---|------|-------|------------|------|-----|-------------|
   | 1 | Academic | ... | ... | ... | ... | High/Med/Low |

   ## Source Quality Assessment
   - Total sources: [N]
   - Academic: [A]
   - Industry: [B]
   - News: [C]
   - Expert: [D]

   ## Coverage Gaps
   [Areas with insufficient sources]
   ```

### Phase 3: Source Analysis

9. **FOR EACH major source** (up to 10):
   a. Read full content via `web_fetch` or `file_read`
   b. Extract key claims with supporting evidence
   c. Note methodology (if applicable)
   d. Identify biases or limitations
   e. Create source note in `OUTPUT/research_[TOPIC]_[timestamp]/notes/`

10. **Identify key claims**: For each significant claim:
    - Statement of the claim
    - Supporting evidence
    - Source(s) making this claim
    - Confidence level: Strong/Moderate/Weak

11. **Cross-reference claims**:
    - IF multiple sources agree: Mark as "consensus"
    - IF sources contradict: Note as "disputed"
    - IF single source: Mark as "unverified"
    - Write to `OUTPUT/research_[TOPIC]_[timestamp]/claims_analysis.md`

12. **Identify patterns**:
    - Common themes across sources
    - Evolution of thinking over time
    - Geographic or sector differences
    - Gaps in current knowledge

### Phase 4: Synthesis

13. **Create synthesis framework**:
    - Organize findings by theme or sub-question
    - Identify relationships between findings
    - Note contradictions and their possible explanations

14. **Write theme summaries**: FOR EACH major theme:
    ```
    ## Theme: [Theme Name]

    ### Key Finding
    [Main insight in 1-2 sentences]

    ### Supporting Evidence
    - [Evidence 1] (Source: [X])
    - [Evidence 2] (Source: [Y])

    ### Dissenting Views
    [If any]

    ### Confidence Level
    [High/Medium/Low] - [Reason]
    ```

15. **Build narrative**: Connect themes into coherent story:
    - Introduction: Why this matters
    - Current state: What we know
    - Debates: What's contested
    - Trends: Where things are heading
    - Gaps: What we still don't know

16. **Create visual summary**: Write to `OUTPUT/research_[TOPIC]_[timestamp]/visual_summary.md`:
    ```
    # Visual Summary: [TOPIC]

    ## Concept Map
    [Main Topic]
    ├── [Theme 1]
    │   ├── [Finding 1.1]
    │   └── [Finding 1.2]
    ├── [Theme 2]
    │   ├── [Finding 2.1]
    │   └── [Finding 2.2]
    └── [Theme 3]

    ## Timeline (if temporal)
    [YEAR] - [Event/Development]
    [YEAR] - [Event/Development]

    ## Key Statistics
    - [Stat 1]
    - [Stat 2]
    ```

### Phase 5: Quality Assurance

17. **Verify key facts**:
    - Cross-check important statistics
    - Verify dates and attributions
    - Flag any unverifiable claims

18. **Assess synthesis quality**:
    - Coverage: Did we address all sub-questions?
    - Balance: Are multiple perspectives represented?
    - Depth: Is evidence sufficient for conclusions?
    - Recency: Are sources up to date?

19. **Write limitations section**:
    ```
    ## Research Limitations

    ### Source Limitations
    - [Limitation 1]
    - [Limitation 2]

    ### Methodology Limitations
    - [Limitation]

    ### Recommendations for Further Research
    - [Area needing more investigation]
    ```

### Phase 6: Deliverables

20. **Generate executive summary**: Write to `OUTPUT/research_[TOPIC]_[timestamp]/executive_summary.md`:
    ```
    # Executive Summary: [TOPIC]

    ## In Brief (30 seconds)
    [2-3 sentences capturing the essence]

    ## Key Findings
    1. [Finding 1]
    2. [Finding 2]
    3. [Finding 3]
    4. [Finding 4]
    5. [Finding 5]

    ## Implications
    [What this means for the reader]

    ## Confidence Level
    [Overall assessment of research reliability]
    ```

21. **Generate full report**: Write to `OUTPUT/research_[TOPIC]_[timestamp]/full_report.md`:
    - Table of contents
    - Executive summary
    - Methodology
    - Findings by theme
    - Synthesis and discussion
    - Limitations
    - Appendix: Source list

22. **Remember research**: Store in memory:
    - "Research synthesis on [TOPIC] completed [DATE]"
    - "Key finding: [most important insight]"
    - "Sources: [N] sources from [X] categories"

23. **Present to user**:
    - Provide executive summary (5 bullet points max)
    - Note confidence level
    - Offer: "Full report available. Want deep dive on any theme?"

---

## Error Handling

- IF source unavailable: Note and continue with other sources
- IF paywall encountered: Extract visible content, note "Full content behind paywall"
- IF contradictory sources: Present both views, don't resolve artificially
- IF insufficient sources (<5): Warn user about limited data, suggest scope adjustment
- IF topic too broad: Suggest narrowing, proceed with disclaimer

---

## Success Criteria

- [ ] Research plan documented
- [ ] At least 5 sources from 2+ categories collected
- [ ] Source quality assessed
- [ ] Key claims identified and cross-referenced
- [ ] Themes synthesized with evidence
- [ ] Contradictions and limitations acknowledged
- [ ] Executive summary captures essence
- [ ] Full report is comprehensive and cited
- [ ] Memory updated with research record
