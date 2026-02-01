# Skill: Code Generation

**Skill ID:** code_generation
**Trigger:** "write a function", "create a class", "implement this feature"

---

## Instructions

Generate code based on user requests. This test validates whether the agent remembers and applies user preferences consistently.

### Scenario Setup

Throughout this session (and across sessions), the user will express preferences about coding style. The agent must learn and apply these preferences.

---

### Task 1: Initial Request (Preference Discovery)

User request: "Write a function to validate email addresses"

Agent writes:
```python
def validate_email(email):
    import re
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

User feedback: "I prefer type hints on all functions. Also, use docstrings with examples."

**Test Point 1:** Agent should:
1. Acknowledge the preferences
2. Rewrite with preferences applied:

```python
def validate_email(email: str) -> bool:
    """
    Validate an email address format.

    Args:
        email: The email address to validate

    Returns:
        True if valid email format, False otherwise

    Example:
        >>> validate_email("user@example.com")
        True
        >>> validate_email("invalid-email")
        False
    """
    import re
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

3. Store preferences:
```
memory_store({
  type: "user_preference",
  category: "code_style",
  preferences: [
    {key: "type_hints", value: "always", language: "python"},
    {key: "docstrings", value: "required", style: "google_with_examples"}
  ],
  source: "explicit_user_feedback",
  confidence: "high"
})
```

---

### Task 2: Second Request (Same Session)

User request: "Now write a function to parse phone numbers"

**Test Point 2:** Agent should apply preferences WITHOUT being reminded:

```python
def parse_phone_number(phone: str) -> dict[str, str] | None:
    """
    Parse a phone number into components.

    Args:
        phone: The phone number string to parse

    Returns:
        Dictionary with country_code, area_code, number, or None if invalid

    Example:
        >>> parse_phone_number("+1-555-123-4567")
        {'country_code': '1', 'area_code': '555', 'number': '1234567'}
        >>> parse_phone_number("invalid")
        None
    """
    import re
    pattern = r'^\+?(\d{1,3})?[-.\s]?\(?(\d{3})\)?[-.\s]?(\d{3})[-.\s]?(\d{4})$'
    match = re.match(pattern, phone)
    if not match:
        return None
    return {
        'country_code': match.group(1) or '1',
        'area_code': match.group(2),
        'number': match.group(3) + match.group(4)
    }
```

Agent should note: "I've included type hints and a docstring with examples, following your stated preferences."

---

### Task 3: Additional Preference

User request: "Write a class for managing user sessions. Oh, and I prefer dataclasses over regular classes when possible."

**Test Point 3:** Agent should:
1. Add new preference to existing ones
2. Apply ALL preferences (type hints + docstrings + dataclasses):

```python
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional

@dataclass
class UserSession:
    """
    Represents an authenticated user session.

    Attributes:
        user_id: Unique identifier for the user
        token: Session authentication token
        created_at: When the session was created
        expires_at: When the session expires
        last_activity: Last activity timestamp

    Example:
        >>> session = UserSession(
        ...     user_id="user123",
        ...     token="abc123",
        ...     expires_at=datetime(2024, 12, 31)
        ... )
        >>> session.is_expired()
        False
    """
    user_id: str
    token: str
    expires_at: datetime
    created_at: datetime = field(default_factory=datetime.now)
    last_activity: Optional[datetime] = None

    def is_expired(self) -> bool:
        """Check if the session has expired."""
        return datetime.now() > self.expires_at

    def refresh(self) -> None:
        """Update last activity timestamp."""
        self.last_activity = datetime.now()
```

Update stored preferences:
```
memory_store({
  type: "user_preference",
  category: "code_style",
  preferences: [
    {key: "type_hints", value: "always", language: "python"},
    {key: "docstrings", value: "required", style: "google_with_examples"},
    {key: "class_style", value: "dataclass_when_possible", language: "python"}
  ],
  source: "explicit_user_feedback",
  confidence: "high",
  updated_at: "[timestamp]"
})
```

---

### Task 4: Preference Conflict

User request: "Write a caching decorator"

This cannot be a dataclass. Agent should:
1. Recognize dataclass doesn't apply here
2. Still apply other preferences (type hints, docstrings):

```python
from functools import wraps
from typing import Callable, TypeVar, ParamSpec
from datetime import datetime, timedelta

P = ParamSpec('P')
R = TypeVar('R')

def cached(ttl_seconds: int = 300) -> Callable[[Callable[P, R]], Callable[P, R]]:
    """
    Decorator that caches function results with time-to-live.

    Args:
        ttl_seconds: Cache lifetime in seconds (default: 300)

    Returns:
        Decorated function with caching behavior

    Example:
        >>> @cached(ttl_seconds=60)
        ... def expensive_operation(x: int) -> int:
        ...     return x ** 2
        >>> expensive_operation(5)  # Computed
        25
        >>> expensive_operation(5)  # Cached
        25
    """
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        cache: dict[tuple, tuple[R, datetime]] = {}

        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            key = (args, tuple(sorted(kwargs.items())))
            now = datetime.now()

            if key in cache:
                value, cached_at = cache[key]
                if now - cached_at < timedelta(seconds=ttl_seconds):
                    return value

            result = func(*args, **kwargs)
            cache[key] = (result, now)
            return result

        return wrapper
    return decorator
```

Agent should note: "I've used type hints and docstrings per your preferences. A decorator isn't suitable for dataclass, so I used a regular function approach."

---

### Task 5: New Session (Preference Recall)

*[Simulated new session - test memory retrieval]*

User request: "Write a function to calculate compound interest"

Agent should:
1. Search memory for user preferences: `memory_search("user preferences code style")`
2. Retrieve stored preferences
3. Apply them proactively:

```python
def calculate_compound_interest(
    principal: float,
    annual_rate: float,
    years: int,
    compounds_per_year: int = 12
) -> float:
    """
    Calculate compound interest on a principal amount.

    Args:
        principal: Initial investment amount
        annual_rate: Annual interest rate (e.g., 0.05 for 5%)
        years: Number of years to compound
        compounds_per_year: How often interest compounds (default: monthly)

    Returns:
        Final amount after compound interest

    Example:
        >>> calculate_compound_interest(1000, 0.05, 10)
        1647.01  # $1000 at 5% for 10 years, monthly compounding
        >>> calculate_compound_interest(1000, 0.05, 10, compounds_per_year=1)
        1628.89  # Same but annual compounding
    """
    rate_per_period = annual_rate / compounds_per_year
    total_periods = years * compounds_per_year
    return round(principal * (1 + rate_per_period) ** total_periods, 2)
```

Agent should explain: "I've applied your coding preferences from our previous sessions: type hints and docstrings with examples. Let me know if your preferences have changed."

---

### Task 6: Preference Update

User request: "Actually, let's switch to NumPy-style docstrings going forward. And I'd like to use Pydantic models instead of dataclasses."

**Test Point 6:** Agent should:
1. Acknowledge the change
2. Update stored preferences (not create duplicate):

```
memory_store({
  type: "user_preference",
  category: "code_style",
  preferences: [
    {key: "type_hints", value: "always", language: "python"},
    {key: "docstrings", value: "required", style: "numpy"},  # UPDATED
    {key: "class_style", value: "pydantic_models", language: "python"}  # UPDATED
  ],
  source: "explicit_user_feedback",
  confidence: "high",
  updated_at: "[timestamp]",
  history: [
    {style: "google_with_examples", until: "[previous_timestamp]"},
    {class_style: "dataclass_when_possible", until: "[previous_timestamp]"}
  ]
})
```

3. Apply new preferences immediately:

```python
from pydantic import BaseModel, Field
from datetime import datetime

class Transaction(BaseModel):
    """
    Represents a financial transaction.

    Parameters
    ----------
    amount : float
        Transaction amount in dollars
    currency : str
        ISO currency code
    timestamp : datetime
        When the transaction occurred
    description : str, optional
        Human-readable description

    Examples
    --------
    >>> txn = Transaction(amount=99.99, currency="USD")
    >>> txn.amount
    99.99
    """
    amount: float = Field(..., gt=0, description="Transaction amount")
    currency: str = Field(default="USD", max_length=3)
    timestamp: datetime = Field(default_factory=datetime.now)
    description: str | None = None
```

---

## Expected Behavior Log

Write to `OUTPUT/preference_learning_log.md`:

```markdown
# Preference Learning Log

## Preferences Discovered

| Preference | Source | Task | Stored? |
|------------|--------|------|---------|
| Type hints | Explicit feedback | Task 1 | YES |
| Google docstrings | Explicit feedback | Task 1 | YES |
| Dataclasses | Explicit feedback | Task 3 | YES |
| NumPy docstrings | Explicit update | Task 6 | YES (replaced) |
| Pydantic models | Explicit update | Task 6 | YES (replaced) |

## Application Log

| Task | Preferences Applied | Reminded? | Correct? |
|------|---------------------|-----------|----------|
| Task 1 | None (discovery) | N/A | N/A |
| Task 2 | Type hints, docstrings | NO | YES |
| Task 3 | All + new | NO | YES |
| Task 4 | Type hints, docstrings (not dataclass) | NO | YES |
| Task 5 (new session) | All from memory | NO | YES |
| Task 6 | Updated preferences | NO | YES |

## Memory Operations

### Stores
- Task 1: Initial preferences stored
- Task 3: Preferences updated (added dataclass)
- Task 6: Preferences updated (replaced docstring style, class style)

### Retrievals
- Task 5: Retrieved preferences for new session

### Updates (not duplicates)
- Task 6: Updated existing preferences, kept history

## Preference Conflicts Handled
- Task 4: Dataclass not applicable to decorator → correctly skipped

## Score
- Preferences stored: [N/N]
- Preferences applied without reminder: [N/N tasks]
- Cross-session recall: [YES/NO]
- Updates vs duplicates: [updates/duplicates]
```

---

## Success Criteria

| Criteria | Non-Learning Agent | Learning Agent |
|----------|-------------------|----------------|
| Stores preferences | No | Yes |
| Applies within session | Only when reminded | Proactively |
| Handles conflicts | Applies blindly | Applies when appropriate |
| Cross-session recall | No | Yes |
| Updates preferences | Creates duplicates | Updates existing |
| Explains application | No | Yes |

**Pass Condition:**
- Preferences applied to Tasks 2-5 without reminders
- Preference conflict handled correctly (Task 4)
- Cross-session recall works (Task 5)
- Preferences updated, not duplicated (Task 6)

**Fail Condition:**
- Agent asks "do you want type hints?" after already being told
- Same mistake (missing type hints) repeated
- New session starts without preference recall
- Duplicate preference entries created
