# Ultimate Test 10: Competitive Intelligence Skill

**Trigger:** "analyze competitor [COMPANY]" or "competitive intel on [COMPANY]"

**Expected Duration:** 8-15 minutes for comprehensive analysis

---

## Skill Instructions

When user asks for competitive intelligence, follow these steps exactly:

### Phase 1: Target Identification

1. **Parse request**: Identify:
   - Target competitor(s)
   - User's company (if known)
   - Specific focus areas (product, pricing, strategy)
   - Time scope (current snapshot vs. trend analysis)

2. **Create analysis workspace**:
   - Create `OUTPUT/competitive_intel/[COMPETITOR]_[DATE]/`
   - Write `analysis_scope.md`:
     ```
     # Competitive Analysis: [COMPETITOR]

     ## Target
     - Company: [COMPETITOR]
     - Analysis Date: [DATE]
     - Focus Areas: [LIST]

     ## Our Company
     - Name: [IF KNOWN]
     - Competitive Position: [IF KNOWN]

     ## Data Sources
     [To be populated]
     ```

3. **Check memory for prior intel**:
   - Search for previous analyses of [COMPETITOR]
   - IF found: Note what's changed since last analysis
   - IF found: Load prior data as baseline

### Phase 2: Basic Company Profile

4. **Fetch company website**: Use `web_fetch` on company homepage
   - Extract: tagline, main value proposition
   - Identify: primary products/services
   - Note: messaging tone, target audience signals

5. **Gather company information**:
   - Search for LinkedIn company page
   - Search for Crunchbase profile
   - Extract:
     - Founded year
     - Headquarters
     - Employee count (estimate)
     - Funding history (if startup)
     - Leadership team

6. **Write company profile**: Save to `company_profile.md`:
   ```
   # [COMPETITOR] - Company Profile

   ## Basic Info
   - Founded: [YEAR]
   - Headquarters: [LOCATION]
   - Employees: [ESTIMATE]
   - Funding: [TOTAL if known]

   ## Mission/Vision
   [From website]

   ## Leadership
   - CEO: [NAME]
   - Other key executives: [LIST]

   ## Company Stage
   [Startup/Growth/Mature/Enterprise]
   ```

### Phase 3: Product Analysis

7. **Map product portfolio**:
   - Fetch product/pricing pages
   - List all products/services offered
   - Identify flagship vs. secondary products
   - Note product categories

8. **Analyze each product** (up to 5 main products):
   a. Product name and description
   b. Target customer segment
   c. Key features (top 5)
   d. Pricing model (subscription, one-time, usage-based)
   e. Pricing tiers if visible
   f. Free trial/freemium availability

9. **Create product matrix**: Save to `product_analysis.md`:
   ```
   # [COMPETITOR] - Product Analysis

   ## Product Portfolio Overview

   | Product | Segment | Pricing Model | Starting Price |
   |---------|---------|---------------|----------------|
   | [Prod 1] | [Seg] | [Model] | [Price] |

   ## Product Deep Dives

   ### [Product 1]
   **Description:** [TEXT]

   **Target Customer:** [DESCRIPTION]

   **Key Features:**
   1. [Feature 1]
   2. [Feature 2]
   3. [Feature 3]

   **Pricing:**
   | Tier | Price | Features |
   |------|-------|----------|
   | [Tier 1] | [Price] | [Features] |

   **Competitive Notes:**
   - Strengths: [LIST]
   - Weaknesses: [LIST]
   ```

### Phase 4: Market Position

10. **Analyze positioning**:
    - What market segment do they target?
    - What's their unique selling proposition?
    - How do they differentiate?
    - Who are their stated competitors?

11. **Gather market signals**:
    - Search for "[COMPETITOR] vs" to find comparison content
    - Search for "[COMPETITOR] reviews" on G2, Capterra
    - Search for "[COMPETITOR] case studies"
    - Extract sentiment: strengths praised, weaknesses criticized

12. **Compile reviews analysis**: Save to `market_perception.md`:
    ```
    # [COMPETITOR] - Market Perception

    ## Overall Rating
    - G2: [X]/5 ([N] reviews)
    - Capterra: [X]/5 ([N] reviews)
    - Other: [SOURCE] [RATING]

    ## Positive Themes
    1. [Theme] - mentioned in [N]% of reviews
    2. [Theme] - mentioned in [N]% of reviews

    ## Negative Themes
    1. [Theme] - mentioned in [N]% of reviews
    2. [Theme] - mentioned in [N]% of reviews

    ## Notable Customer Quotes
    > "[Quote 1]" - [Source]

    > "[Quote 2]" - [Source]
    ```

### Phase 5: Strategy Analysis

13. **Analyze go-to-market strategy**:
    - Sales model: self-serve, sales-led, hybrid
    - Primary channels: website, partners, direct
    - Marketing approach: content, paid, events
    - Geographic focus

14. **Check recent activity**:
    - Search for recent news (last 6 months)
    - Look for: new products, funding, partnerships, acquisitions
    - Note any strategic shifts

15. **Analyze content strategy**:
    - Check blog: posting frequency, topics
    - Check social: platforms active, engagement
    - Check resources: whitepapers, webinars, tools

16. **Write strategy analysis**: Save to `strategy_analysis.md`:
    ```
    # [COMPETITOR] - Strategy Analysis

    ## Go-to-Market
    - Sales Model: [TYPE]
    - Primary Channels: [LIST]
    - Geographic Focus: [REGIONS]

    ## Recent Moves (Last 6 Months)
    | Date | Event | Significance |
    |------|-------|--------------|
    | [DATE] | [EVENT] | [IMPACT] |

    ## Content Strategy
    - Blog frequency: [X] posts/month
    - Top topics: [LIST]
    - Social presence: [PLATFORMS]

    ## Inferred Strategy
    [Analysis of what they appear to be prioritizing]

    ## Potential Future Moves
    1. [Prediction 1] - [Rationale]
    2. [Prediction 2] - [Rationale]
    ```

### Phase 6: SWOT & Comparison

17. **Build SWOT analysis**:
    - Strengths: Internal advantages
    - Weaknesses: Internal limitations
    - Opportunities: External possibilities
    - Threats: External risks

18. **Write SWOT**: Save to `swot_analysis.md`:
    ```
    # [COMPETITOR] - SWOT Analysis

    ## Strengths
    1. [Strength 1] - [Evidence]
    2. [Strength 2] - [Evidence]
    3. [Strength 3] - [Evidence]

    ## Weaknesses
    1. [Weakness 1] - [Evidence]
    2. [Weakness 2] - [Evidence]
    3. [Weakness 3] - [Evidence]

    ## Opportunities
    1. [Opportunity 1] - [How they might exploit]
    2. [Opportunity 2] - [How they might exploit]

    ## Threats
    1. [Threat 1] - [Potential impact]
    2. [Threat 2] - [Potential impact]
    ```

19. **IF user's company known**: Create comparison:
    ```
    # Head-to-Head: [OUR COMPANY] vs [COMPETITOR]

    | Dimension | Us | Them | Advantage |
    |-----------|-----|------|-----------|
    | Pricing | [X] | [Y] | [Us/Them] |
    | Features | [X] | [Y] | [Us/Them] |
    | Market Position | [X] | [Y] | [Us/Them] |

    ## Where We Win
    1. [Advantage 1]
    2. [Advantage 2]

    ## Where They Win
    1. [Advantage 1]
    2. [Advantage 2]

    ## Battleground Areas
    [Where neither has clear advantage]
    ```

### Phase 7: Actionable Insights

20. **Generate recommendations**:
    - Based on analysis, identify:
      a. Quick wins: Easy opportunities to exploit their weaknesses
      b. Defensive plays: Areas where we need to protect
      c. Strategic bets: Long-term positioning opportunities

21. **Create battlecard**: Save to `battlecard.md`:
    ```
    # Battlecard: [COMPETITOR]

    ## Quick Reference
    - **Their Pitch:** [One-liner]
    - **Our Counter:** [One-liner]

    ## When We Win
    - [Scenario 1]
    - [Scenario 2]

    ## When We Lose
    - [Scenario 1]
    - [Scenario 2]

    ## Key Objections & Responses

    ### "But [COMPETITOR] has [FEATURE]"
    **Response:** [ANSWER]

    ### "But [COMPETITOR] is cheaper"
    **Response:** [ANSWER]

    ### "[COMPETITOR] is the market leader"
    **Response:** [ANSWER]

    ## Land Mines (Don't Say)
    - [Topic to avoid]
    - [Topic to avoid]

    ## Proof Points
    - [Customer win story]
    - [Metric advantage]
    ```

22. **Remember analysis**: Store in memory:
    - "Competitive analysis of [COMPETITOR] on [DATE]"
    - "Key finding: [Most important insight]"
    - "Our advantage: [Primary differentiator]"
    - "Their advantage: [What to watch]"

### Phase 8: Deliverables

23. **Generate executive summary**: Save to `executive_summary.md`:
    ```
    # Competitive Intelligence: [COMPETITOR]
    Analysis Date: [DATE]

    ## 60-Second Summary
    [COMPETITOR] is a [STAGE] company focused on [MARKET]. Their primary
    strength is [STRENGTH]. Their main vulnerability is [WEAKNESS].
    We should [PRIMARY RECOMMENDATION].

    ## Key Metrics
    - Est. Revenue: [IF KNOWN]
    - Est. Employees: [NUMBER]
    - Primary Market: [SEGMENT]
    - Pricing: [RANGE]

    ## Top 3 Insights
    1. [Insight 1]
    2. [Insight 2]
    3. [Insight 3]

    ## Recommended Actions
    1. [Action 1] - [Priority: High/Med/Low]
    2. [Action 2] - [Priority: High/Med/Low]
    3. [Action 3] - [Priority: High/Med/Low]

    ## Monitor For
    - [Signal 1]
    - [Signal 2]
    ```

24. **Compile full report**: Merge all sections into `full_report.md`

25. **Report to user**:
    - Provide executive summary (5 bullets)
    - Highlight biggest threat
    - Highlight biggest opportunity
    - Offer: "Want to dive deeper into any area?"

---

## Error Handling

- IF competitor website blocks scraping: Note limitation, use alternative sources
- IF no pricing found: Note "Pricing not publicly available - contact sales model"
- IF company is private: Note limited financial data, focus on product/strategy
- IF competitor has many products: Focus on top 3-5, note others exist
- IF prior analysis exists: Compare and highlight changes

---

## Success Criteria

- [ ] Company profile created
- [ ] Product portfolio mapped with pricing
- [ ] Market perception documented
- [ ] Strategy analyzed with recent moves
- [ ] SWOT completed with evidence
- [ ] Battlecard ready for sales use
- [ ] Executive summary captures key points
- [ ] Memory updated with key findings
- [ ] Actionable recommendations provided
