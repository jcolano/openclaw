# Skill: Stock Price Alert

**Skill ID:** stock_price_alert
**Trigger:** "get the current price of [STOCK] and alert if above [THRESHOLD]", "monitor [STOCK] price"

---

## Instructions

Monitor stock price and send alert if threshold is exceeded.

### Step 1: Parse Request

Extract from user request:
- Stock symbol: [STOCK]
- Alert threshold: [THRESHOLD]
- Default threshold if not specified: $50

### Step 2: Fetch Current Price

Call the stock API:
```
GET https://api.example.com/stocks/{SYMBOL}/price
```

Expected response:
```json
{
  "symbol": "ACME",
  "price": 52.34,
  "timestamp": "2026-01-15T10:30:00Z"
}
```

**Note:** This API has caching issues - it may return different prices on consecutive calls.

### Step 3: Verify Price

Fetch the price again to verify:
```
GET https://api.example.com/stocks/{SYMBOL}/price
```

Compare the two readings:
- If within 1%: Price is stable, proceed
- If different by >1%: Note the discrepancy

### Step 4: Handle Discrepancy

If prices differ significantly (>1%), **reflect:**

- Is this normal market volatility?
- Is the API caching inconsistently?
- Can I trust any single reading?
- What's my confidence in the "true" price?

### Step 5: Reliability Strategies

If data seems unreliable, choose an approach:

**Strategy A: Multi-Sample Average**
```
1. Fetch price 5 times (with small delays)
2. Calculate: mean, std_dev, min, max
3. If std_dev < 2% of mean: Use mean
4. If std_dev > 2%: Flag as unreliable
```

**Strategy B: Multi-Source Verification**
```
1. Try alternative endpoint: /stocks/{SYMBOL}/quote
2. Try backup API: https://backup-api.example.com/price/{SYMBOL}
3. Compare sources
4. Use most consistent source
```

**Strategy C: Confidence-Weighted Decision**
```
1. Calculate confidence based on variance
2. If confidence > 80%: Make alert decision
3. If confidence 50-80%: Alert with caveat
4. If confidence < 50%: Report uncertainty, don't alert
```

### Step 6: Alert Decision Logic

```
IF confidence >= HIGH:
    IF price > threshold:
        ALERT("Price ${price} exceeds ${threshold}")
    ELSE:
        NO_ALERT("Price ${price} below ${threshold}")

ELSE IF confidence >= MEDIUM:
    IF price > threshold:
        ALERT_WITH_CAVEAT("Price appears to be ${price}, above ${threshold}, but data quality is uncertain")
    ELSE:
        NO_ALERT_WITH_NOTE("Price appears below threshold but verify manually")

ELSE (LOW confidence):
    REPORT_UNCERTAINTY("Cannot determine reliable price. Readings ranged from ${min} to ${max}")
```

### Step 7: Document Decision

Write to `OUTPUT/alerts/price_check_{SYMBOL}_{timestamp}.md`:

```markdown
# Price Alert Check: {SYMBOL}

## Request
- Symbol: {SYMBOL}
- Threshold: ${THRESHOLD}
- Check time: {timestamp}

## Data Collection

### Price Readings
| # | Time | Price | Source |
|---|------|-------|--------|
| 1 | | | |
| 2 | | | |
...

### Data Quality Analysis
- Reading count: [N]
- Mean: $[X]
- Std Dev: $[Y] ([Z]%)
- Range: $[min] - $[max]

### Reliability Assessment
- Variance: [LOW/MEDIUM/HIGH]
- Consistency: [STABLE/UNSTABLE]
- Confidence level: [X]%

## Reflection Notes
- Issue detected: [what inconsistency was noticed]
- Strategy applied: [how I handled it]
- Reasoning: [why this was appropriate]

## Decision

### Alert Status: [TRIGGERED / NOT_TRIGGERED / UNCERTAIN]

### Rationale
[Why this decision was made]

### Caveats
[Any uncertainty in this decision]

### Recommended Action
[What user should do with this information]
```

### Step 8: Memory Update

Remember:
- "Stock API for {SYMBOL} reliability: [HIGH/MEDIUM/LOW] as of {date}"
- "Price check strategy [X] worked for handling inconsistent readings"
- Alert decision for audit trail

---

## Expected Behavior

**Without Reflection:** Agent takes first reading, makes alert decision, ignores inconsistency when re-checking, or flip-flops on decision with each reading.

**With Reflection:** Agent notices price inconsistency, reflects on data reliability, chooses appropriate strategy, makes decision with appropriate confidence level and caveats.

## Test Data Setup

Configure the mock API to return varying prices:
```
Call 1: $52.34
Call 2: $47.89
Call 3: $51.23
Call 4: $53.01
Call 5: $48.56
```

This ~10% variance should trigger reflection about data reliability.

Alternative reliable source should return consistent: $50.25

## Edge Cases to Test

1. **Threshold boundary:** True price is exactly at threshold - how does uncertainty affect decision?
2. **High volatility:** 20%+ variance - should refuse to make decision?
3. **One outlier:** 4 consistent readings + 1 wild outlier - how handled?
4. **All sources disagree:** No reliable data available - graceful reporting?
