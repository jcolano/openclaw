# Ultimate Test 01: Market Research Skill

**Trigger:** "research [TOPIC] market"

**Expected Duration:** 5-10 minutes autonomous execution

---

## Skill Instructions

When user asks to "research [TOPIC] market", follow these steps exactly:

### Phase 1: Initial Research

1. **Acknowledge and plan**: Tell user you're starting market research on [TOPIC]. Write your research plan to `OUTPUT/research_[TOPIC]/plan.md` including today's date and the 5 research areas you'll cover.

2. **Search for news**: Use `http_call` to fetch recent news from a news API:
   ```
   GET https://newsapi.org/v2/everything?q=[TOPIC]&sortBy=publishedAt
   ```
   - IF response contains articles, extract top 5 headlines with sources
   - IF no articles found, note "No recent news coverage found"
   - IF API fails, try alternative: `web_fetch` on `https://news.google.com/search?q=[TOPIC]`

3. **Save news findings**: Write news summary to `OUTPUT/research_[TOPIC]/news.md`

### Phase 2: Competitor Analysis

4. **Identify competitors**: Use `web_fetch` to search for "[TOPIC] companies" or "[TOPIC] competitors"
   - Extract company names mentioned
   - IF more than 10 found, keep only top 10 by mention frequency

5. **FOR EACH competitor** (up to 5):
   a. Attempt to fetch their homepage using `web_fetch`
   b. Extract: company description, key products, pricing if visible
   c. IF fetch fails, note "Could not access [competitor] website"
   d. Write findings to `OUTPUT/research_[TOPIC]/competitor_[name].md`

6. **Create competitor matrix**: Read all competitor files, create comparison table in `OUTPUT/research_[TOPIC]/competitor_matrix.md` with columns: Name, Products, Pricing, Strengths, Weaknesses

### Phase 3: Market Data

7. **Search for market size**: Use `web_fetch` on market research sites:
   - Try: `https://www.statista.com/search/?q=[TOPIC]+market+size`
   - IF blocked or no data, try: `https://www.grandviewresearch.com/industry-analysis/[TOPIC]-market`

8. **Extract market figures**:
   - IF market size found, extract: current size, growth rate (CAGR), forecast
   - IF figures in non-USD currency, note the currency (don't convert)
   - IF no figures found, note "Market size data not publicly available"

9. **Search for trends**: Fetch "[TOPIC] industry trends 2026" via web search
   - Extract top 5 trends mentioned across sources
   - Note which trends appear in multiple sources (higher confidence)

### Phase 4: Synthesis

10. **Read all generated files**: Use `file_read` on each file in `OUTPUT/research_[TOPIC]/`

11. **Identify patterns**:
    - IF 3+ competitors have similar pricing, note "Price convergence around $X"
    - IF news sentiment is mostly negative, flag as "Market concern: negative press"
    - IF growth rate > 15%, flag as "High growth market"

12. **Remember key facts**: Store in memory:
    - "Researched [TOPIC] market on [DATE]"
    - "Key competitors: [list top 3]"
    - "Market size: [figure if found]"

### Phase 5: Report Generation

13. **Generate executive summary**: Write `OUTPUT/research_[TOPIC]/executive_summary.md`:
    ```
    # [TOPIC] Market Research - Executive Summary
    Date: [today]

    ## Key Findings
    [3-5 bullet points]

    ## Market Overview
    [Size, growth, key players]

    ## Competitive Landscape
    [Summary of competitor matrix]

    ## Trends & Outlook
    [Top trends identified]

    ## Recommendations
    [2-3 actionable recommendations]
    ```

14. **Generate full report**: Compile all sections into `OUTPUT/research_[TOPIC]/full_report.md`
    - Include all data from individual files
    - Add table of contents
    - Add source citations

15. **Summarize for user**: Provide 5 bullet points of key findings. Ask if they want deeper analysis on any section.

---

## Error Handling

- IF any API returns 429 (rate limit): Wait and note "Rate limited, partial data collected"
- IF any API returns 401/403: Skip that source, note "Source unavailable: [source]"
- IF file write fails: Report error, continue with remaining steps
- IF less than 3 competitors found: Note "Limited competitive data - niche market?"
- IF ALL external fetches fail: Use file_read to check for cached previous research

---

## Success Criteria

- [ ] Created `OUTPUT/research_[TOPIC]/` directory structure
- [ ] Generated at least 5 files: plan.md, news.md, competitor_matrix.md, executive_summary.md, full_report.md
- [ ] Memory contains research record
- [ ] User received final summary
- [ ] Errors were handled gracefully, not fatal
