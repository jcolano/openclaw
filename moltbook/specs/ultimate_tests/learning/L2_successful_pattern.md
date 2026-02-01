# Skill: API Integration

**Skill ID:** api_integration
**Trigger:** "integrate with the payments API", "connect to external service"

---

## Instructions

Integrate with external APIs. This test validates whether the agent remembers and reuses successful approaches.

### Scenario Setup

You've previously integrated with the Stripe API successfully. Now you need to integrate with similar payment APIs.

---

### Task 1: Stripe Integration (Reference Experience)

*[This represents a past successful experience - either from memory or earlier in session]*

The agent previously solved Stripe integration with this pattern:

```python
# The Successful Pattern (stored in memory or discovered)
class StripeClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.stripe.com/v1"
        self.session = self._create_session()

    def _create_session(self):
        session = requests.Session()
        session.auth = (self.api_key, '')  # Stripe uses basic auth with key as username
        session.headers.update({
            'Content-Type': 'application/x-www-form-urlencoded',  # Stripe uses form encoding
            'Stripe-Version': '2023-10-16'  # Pin API version
        })
        # Retry logic for transient failures
        retries = Retry(total=3, backoff_factor=0.5, status_forcelist=[500, 502, 503])
        session.mount('https://', HTTPAdapter(max_retries=retries))
        return session

    def _handle_response(self, response):
        if response.status_code == 429:
            # Rate limited - exponential backoff
            retry_after = int(response.headers.get('Retry-After', 1))
            time.sleep(retry_after)
            raise RateLimitError("Rate limited, retry after backoff")
        elif response.status_code >= 400:
            error = response.json().get('error', {})
            raise PaymentError(error.get('message'), error.get('code'))
        return response.json()

    def create_payment(self, amount, currency, customer_id):
        # Idempotency key prevents duplicate charges
        idempotency_key = f"payment_{customer_id}_{amount}_{int(time.time())}"
        response = self.session.post(
            f"{self.base_url}/payment_intents",
            data={'amount': amount, 'currency': currency, 'customer': customer_id},
            headers={'Idempotency-Key': idempotency_key}
        )
        return self._handle_response(response)
```

**Key patterns that made this successful:**
1. Session with retry logic
2. API version pinning
3. Rate limit handling with backoff
4. Idempotency keys for payments
5. Structured error handling

Store this pattern:
```
memory_store({
  type: "successful_pattern",
  category: "api_integration",
  context: "payment_api",
  pattern: {
    name: "robust_payment_client",
    elements: [
      "session_with_retries",
      "api_version_pinning",
      "rate_limit_handling",
      "idempotency_keys",
      "structured_errors"
    ]
  },
  code_reference: "stripe_client.py",
  outcome: "success - zero duplicate payments, graceful degradation"
})
```

---

### Task 2: PayPal Integration

User request: "Now integrate with PayPal's payment API"

Agent should:
1. Recognize this is similar to Stripe (payment API)
2. Search memory: `memory_search("payment api integration pattern")`
3. Retrieve the successful Stripe pattern
4. Adapt for PayPal's specifics:

```python
class PayPalClient:
    def __init__(self, client_id, client_secret):
        self.client_id = client_id
        self.client_secret = client_secret
        self.base_url = "https://api.paypal.com/v2"  # PayPal uses v2
        self.session = self._create_session()
        self._authenticate()

    def _create_session(self):
        session = requests.Session()
        # ADAPTED: PayPal uses OAuth, not basic auth
        session.headers.update({
            'Content-Type': 'application/json',  # ADAPTED: PayPal uses JSON, not form
        })
        # REUSED: Retry logic (same pattern)
        retries = Retry(total=3, backoff_factor=0.5, status_forcelist=[500, 502, 503])
        session.mount('https://', HTTPAdapter(max_retries=retries))
        return session

    def _authenticate(self):
        # ADAPTED: PayPal OAuth flow
        auth_response = requests.post(
            f"{self.base_url.replace('/v2', '/v1')}/oauth2/token",
            auth=(self.client_id, self.client_secret),
            data={'grant_type': 'client_credentials'}
        )
        token = auth_response.json()['access_token']
        self.session.headers['Authorization'] = f'Bearer {token}'

    def _handle_response(self, response):
        # REUSED: Rate limit handling (same pattern)
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 1))
            time.sleep(retry_after)
            raise RateLimitError("Rate limited")
        # ADAPTED: PayPal error structure differs
        elif response.status_code >= 400:
            error = response.json()
            raise PaymentError(
                error.get('message', error.get('details', [{}])[0].get('description')),
                error.get('name')
            )
        return response.json()

    def create_payment(self, amount, currency, description):
        # REUSED: Idempotency concept (PayPal calls it PayPal-Request-Id)
        request_id = f"payment_{int(time.time())}_{uuid.uuid4().hex[:8]}"
        response = self.session.post(
            f"{self.base_url}/checkout/orders",
            json={  # ADAPTED: PayPal order structure
                'intent': 'CAPTURE',
                'purchase_units': [{
                    'amount': {'currency_code': currency, 'value': str(amount/100)}
                }]
            },
            headers={'PayPal-Request-Id': request_id}
        )
        return self._handle_response(response)
```

**Test Point:** Agent should explicitly note what was reused vs adapted:

```markdown
## PayPal Integration - Pattern Reuse Log

### Retrieved Pattern: robust_payment_client (from Stripe)

### Reused Elements
| Element | Stripe | PayPal | Status |
|---------|--------|--------|--------|
| Session with retries | ✓ | ✓ | REUSED directly |
| Rate limit handling | Retry-After header | Same | REUSED directly |
| Idempotency | Idempotency-Key | PayPal-Request-Id | REUSED (adapted header) |
| Structured errors | ✓ | ✓ | REUSED (adapted parsing) |

### Adapted Elements
| Element | Stripe | PayPal | Change |
|---------|--------|--------|--------|
| Auth method | Basic auth | OAuth2 | NEW: OAuth flow added |
| Content type | form-urlencoded | JSON | CHANGED |
| API structure | /payment_intents | /checkout/orders | CHANGED |
| Version pinning | Stripe-Version header | URL path /v2 | ADAPTED |

### New Elements (PayPal-specific)
- OAuth token refresh logic
- PayPal order lifecycle (create → capture)
```

---

### Task 3: Square Integration

User request: "Also add Square payments integration"

Agent should now have TWO reference patterns (Stripe + PayPal).

```markdown
## Square Integration - Pattern Reuse Log

### Retrieved Patterns
1. robust_payment_client (Stripe) - original
2. paypal_oauth_client (PayPal) - OAuth variant

### Pattern Selection
- Square uses OAuth like PayPal → start from PayPal pattern
- Square uses JSON like PayPal → confirmed
- Square has idempotency like both → reuse

### Implementation Notes
[Details of what was reused and adapted]
```

**Test Point:** Agent should recognize it now has multiple reference patterns and choose appropriately.

---

### Task 4: Store Generalized Pattern

After three integrations, agent should abstract a generalized pattern:

```
memory_store({
  type: "generalized_pattern",
  category: "api_integration",
  context: "payment_apis",
  pattern: {
    name: "payment_api_integration_template",
    required_elements: [
      "session_management_with_retries",
      "rate_limit_handling",
      "idempotency_mechanism",
      "structured_error_handling"
    ],
    variable_elements: [
      {name: "auth_method", variants: ["basic_auth", "oauth2", "api_key_header"]},
      {name: "content_type", variants: ["form", "json"]},
      {name: "idempotency_header", variants: ["Idempotency-Key", "PayPal-Request-Id", "X-Idempotency-Key"]}
    ],
    checklist: [
      "Identify auth method from docs",
      "Check content type requirements",
      "Find idempotency header name",
      "Locate rate limit response format",
      "Map error response structure"
    ]
  },
  derived_from: ["stripe_integration", "paypal_integration", "square_integration"],
  confidence: "high"
})
```

---

### Task 5: New Payment API (Test Generalization)

User request: "We need to add Adyen payments"

Agent should:
1. Retrieve generalized pattern
2. Apply checklist to Adyen docs
3. Fill in variable elements
4. Implement with confidence

```markdown
## Adyen Integration - Using Generalized Pattern

### Pattern Applied: payment_api_integration_template

### Checklist Completion
- [x] Auth method: API key in header (X-API-Key)
- [x] Content type: JSON
- [x] Idempotency: X-Idempotency-Key header
- [x] Rate limits: 429 with Retry-After
- [x] Error structure: {status, errorCode, message}

### Implementation
[Code applying the pattern with Adyen-specific values]

### Confidence: HIGH
Using proven pattern from 3 previous successful integrations.
```

---

## Expected Behavior Log

Write to `OUTPUT/pattern_learning_log.md`:

```markdown
# Pattern Learning Log: Payment API Integrations

## Phase 1: Initial Pattern (Stripe)
- Created successful pattern: [YES/NO]
- Stored in memory: [YES/NO]
- Key elements identified: [list]

## Phase 2: First Reuse (PayPal)
- Searched for relevant patterns: [YES/NO]
- Retrieved Stripe pattern: [YES/NO]
- Explicitly noted reused elements: [YES/NO]
- Explicitly noted adaptations: [YES/NO]
- Stored PayPal variant: [YES/NO]

## Phase 3: Multiple References (Square)
- Retrieved multiple patterns: [YES/NO]
- Selected most appropriate base: [YES/NO]
- Justified selection: [YES/NO]

## Phase 4: Generalization
- Abstracted common pattern: [YES/NO]
- Identified required vs variable elements: [YES/NO]
- Created reusable checklist: [YES/NO]
- Stored generalized pattern: [YES/NO]

## Phase 5: Application (Adyen)
- Retrieved generalized pattern: [YES/NO]
- Applied checklist: [YES/NO]
- Filled variable elements: [YES/NO]
- Expressed confidence: [YES/NO]

## Learning Progression
| Integration | Approach | Efficiency |
|-------------|----------|------------|
| Stripe | From scratch | Baseline |
| PayPal | Pattern + adaptation | Faster |
| Square | Best pattern selection | Faster |
| Adyen | Generalized template | Fastest |
```

---

## Success Criteria

| Criteria | Non-Learning Agent | Learning Agent |
|----------|-------------------|----------------|
| Stripe: Creates pattern | Yes | Yes + stores |
| PayPal: Starts fresh | Yes (from scratch) | No (retrieves pattern) |
| PayPal: Notes reuse | No | Yes explicitly |
| Square: Multiple patterns | N/A | Selects best fit |
| Generalizes | No | Yes (abstracts template) |
| Adyen: Uses template | From scratch again | Applies with confidence |

**Pass Condition:**
- Pattern retrieved for PayPal (not starting fresh)
- Explicit documentation of reused vs adapted elements
- Generalized pattern stored after 3 integrations
- Adyen implemented using template with high confidence

**Fail Condition:**
- Each integration treated as completely new
- No memory storage of patterns
- No generalization after multiple similar tasks
- Same "discovery" process repeated each time
