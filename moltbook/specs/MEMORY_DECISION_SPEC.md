# Memory Decision Logic Specification

**Version:** 1.0.0
**Status:** Draft
**Date:** 2026-01-31
**Parent Document:** AGENTIC_LOOP_SPEC.md (Section 7: Memory System)

---

## Overview

This addendum specifies **when and how** the agent decides what information to store in long-term memory. The parent specification (AGENTIC_LOOP_SPEC.md Section 7) defines the storage mechanisms; this document defines the decision logic.

### Design Goals

| Goal | Description |
|------|-------------|
| **User-Directed** | User can explicitly tell agent what to remember |
| **Selective** | Automatic storage only for meaningful information |
| **Automatic** | Agent also decides autonomously without requiring approval |
| **Transparent** | User can see what was stored (but doesn't approve each item) |
| **Safe** | Never store sensitive data (passwords, API keys) |
| **Evolving** | Old memories fade; related memories consolidate |

### Two Primary Triggers

1. **User Directive** (highest priority): User explicitly says "remember this", "note that", "don't forget", etc.
2. **Hybrid Automatic**: Background scanning + session-end LLM review for implicit facts

---

## 1. Memory Decision Architecture

### 1.1 Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    AGENTIC LOOP (per turn)                      │
│                                                                 │
│  User Message → LLM → Tool Calls → Results → Response           │
│                           │                                     │
│                           ▼                                     │
│              ┌─────────────────────────┐                        │
│              │  BACKGROUND SCANNER     │  (runs after each turn)│
│              │  - Extract candidates   │                        │
│              │  - Score importance     │                        │
│              │  - Queue for review     │                        │
│              └─────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SESSION END REVIEW                           │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │ Scan Session │───▶│ LLM Extract  │───▶│ Store/Notify │      │
│  │   History    │    │  Key Facts   │    │    User      │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  PERIODIC MAINTENANCE                           │
│  (runs on schedule - e.g., daily)                               │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │ Consolidate  │    │    Apply     │    │   Cleanup    │      │
│  │   Related    │    │    Decay     │    │   Orphans    │      │
│  │   Memories   │    │   Scores     │    │              │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Four Memory Decision Points

| Decision Point | When | Priority | What Happens |
|----------------|------|----------|--------------|
| **User Directive** | Immediately on request | **Highest** | User explicitly says "remember X" |
| **Background Scan** | After each turn | Medium | Quick extraction of obvious facts |
| **Session End Review** | When session completes | Medium | LLM-driven deep extraction |
| **Periodic Maintenance** | Scheduled (daily) | Low | Consolidation, decay, cleanup |

---

## 2. User Directives (Highest Priority)

### 2.1 Purpose

When the user explicitly tells the agent to remember something, that takes highest priority and is stored immediately with maximum importance.

### 2.2 Trigger Patterns

```python
USER_DIRECTIVE_PATTERNS = [
    # Explicit remember requests
    r"remember (?:that |this[:\s])?(.+)",
    r"don'?t forget (?:that )?(.+)",
    r"note (?:that |this[:\s])?(.+)",
    r"keep in mind (?:that )?(.+)",
    r"make a note[:\s]?(.+)",
    r"store (?:this|that)[:\s]?(.+)",
    r"save (?:this|that) (?:for later|to memory)[:\s]?(.+)",

    # Implicit strong signals
    r"(?:my|i) (?:always|never) (.+)",
    r"(?:it'?s|that'?s) important (?:that|to) (.+)",
]
```

### 2.3 Implementation

```python
class UserDirectiveHandler:
    """Handles explicit user memory requests."""

    def __init__(self, memory_manager: MemoryManager):
        self.memory_manager = memory_manager

    def check_for_directive(self, user_message: str) -> Optional[DirectiveResult]:
        """
        Check if user message contains a memory directive.

        Called BEFORE the agentic loop processes the message.
        If directive found, store immediately and acknowledge.
        """
        message_lower = user_message.lower().strip()

        for pattern in USER_DIRECTIVE_PATTERNS:
            match = re.search(pattern, message_lower, re.IGNORECASE)
            if match:
                content = match.group(1).strip()

                # Filter sensitive data even for explicit requests
                is_sensitive, reason = contains_sensitive_data(content)
                if is_sensitive:
                    return DirectiveResult(
                        found=True,
                        stored=False,
                        message=f"I can't store that because it appears to contain {reason}."
                    )

                # Determine topic from content
                topic = self._infer_topic(content)

                # Store with highest importance
                content_id = self.memory_manager.add_memory(
                    topic_id=topic,
                    title=self._generate_title(content),
                    content=content,
                    keywords=self._extract_keywords(content),
                    metadata={
                        "source": "user_directive",
                        "importance": 1.0,  # Maximum importance
                        "user_requested": True,
                        "original_message": user_message[:200],
                        "stored_at": datetime.now().isoformat()
                    }
                )

                return DirectiveResult(
                    found=True,
                    stored=True,
                    content_id=content_id,
                    message=f"Got it, I'll remember that."
                )

        return DirectiveResult(found=False, stored=False, message=None)

    def _infer_topic(self, content: str) -> str:
        """Infer appropriate topic from content."""
        content_lower = content.lower()

        topic_keywords = {
            "preferences": ["prefer", "like", "favorite", "always", "never", "hate", "love"],
            "user_info": ["name is", "work at", "live in", "age is", "birthday"],
            "contacts": ["email", "phone", "contact", "address", "colleague", "friend"],
            "projects": ["project", "working on", "building", "developing", "task"],
            "decisions": ["decided", "decision", "agreed", "commitment", "plan to"],
        }

        for topic, keywords in topic_keywords.items():
            if any(kw in content_lower for kw in keywords):
                return topic

        return "general"

    def _generate_title(self, content: str) -> str:
        """Generate a brief title for the memory."""
        # Take first 50 chars or first sentence
        if len(content) <= 50:
            return content

        # Try to find first sentence
        sentence_end = content.find('.')
        if 0 < sentence_end <= 60:
            return content[:sentence_end]

        return content[:47] + "..."

    def _extract_keywords(self, content: str) -> List[str]:
        """Extract keywords from content."""
        # Reuse the keyword extraction from MemoryManager
        stop_words = {'the', 'a', 'an', 'is', 'are', 'was', 'were', 'be', 'been',
                      'have', 'has', 'had', 'do', 'does', 'did', 'will', 'would',
                      'could', 'should', 'may', 'might', 'can', 'to', 'of', 'in',
                      'for', 'on', 'with', 'at', 'by', 'from', 'that', 'this',
                      'i', 'my', 'me', 'we', 'our', 'you', 'your', 'it', 'its'}

        words = re.findall(r'\b[a-zA-Z]{3,}\b', content.lower())
        word_counts = {}
        for word in words:
            if word not in stop_words:
                word_counts[word] = word_counts.get(word, 0) + 1

        sorted_words = sorted(word_counts.items(), key=lambda x: x[1], reverse=True)
        return [word for word, count in sorted_words[:8]]


@dataclass
class DirectiveResult:
    """Result of checking for user directive."""
    found: bool
    stored: bool
    content_id: Optional[str] = None
    message: Optional[str] = None
```

### 2.4 Integration Point

```python
class AgenticLoop:
    """Loop with user directive handling."""

    def __init__(self, ...):
        self.directive_handler = UserDirectiveHandler(self.memory_manager)
        # ... rest of init ...

    def execute(self, message: str, context: LoopContext) -> LoopResult:
        """Execute with directive check first."""

        # FIRST: Check for explicit user memory directive
        directive_result = self.directive_handler.check_for_directive(message)

        if directive_result.found and directive_result.stored:
            # Acknowledge the directive in the response
            context.memory_acknowledgment = directive_result.message

        # Continue with normal loop execution
        # ... rest of execute method ...
```

### 2.5 Examples

| User Says | Agent Action |
|-----------|--------------|
| "Remember that I prefer dark mode" | Store immediately with topic="preferences", importance=1.0 |
| "Don't forget my meeting with John is at 3pm" | Store immediately with topic="general", importance=1.0 |
| "Note: the API endpoint is /v2/users" | Store immediately with topic="projects", importance=1.0 |
| "Remember my password is abc123" | REJECT - sensitive data detected |
| "Keep in mind I have a peanut allergy" | Store immediately (health info explicitly requested), importance=1.0 |

---

## 3. Background Scanner (Automatic)

### 2.1 Purpose

Runs after each turn to catch obvious, high-value facts without waiting for session end.

### 2.2 Algorithm

```python
class BackgroundScanner:
    """Scans turns for memory-worthy content."""

    def __init__(self, memory_manager: MemoryManager):
        self.memory_manager = memory_manager
        self.pending_queue: List[MemoryCandidate] = []

    def scan_turn(self, turn: Turn, session_context: SessionContext) -> None:
        """
        Scan a completed turn for memory candidates.
        Called after each turn in the agentic loop.
        """
        candidates = []

        # 1. Scan user message for explicit statements
        user_content = self._get_user_content(turn)
        if user_content:
            candidates.extend(self._extract_explicit_facts(user_content))

        # 2. Scan tool results for significant outcomes
        for result in turn.tool_results:
            if result.success:
                candidates.extend(self._extract_from_tool_result(result))

        # 3. Score and filter candidates
        for candidate in candidates:
            score = self._calculate_importance(candidate, session_context)
            if score >= BACKGROUND_THRESHOLD:
                candidate.importance_score = score
                self.pending_queue.append(candidate)

    def _extract_explicit_facts(self, content: str) -> List[MemoryCandidate]:
        """
        Extract facts from explicit user statements.

        Patterns that indicate memory-worthy content:
        - "My name is..."
        - "I work at..."
        - "Remember that..."
        - "I prefer..."
        - "My email is..." (but NOT passwords/keys)
        """
        candidates = []

        # Pattern matching for explicit facts
        patterns = [
            (r"my name is (\w+)", "user_info", "name"),
            (r"i work (?:at|for) (.+?)(?:\.|$)", "user_info", "employer"),
            (r"remember (?:that )?(.+?)(?:\.|$)", "explicit_memory", "user_stated"),
            (r"i prefer (.+?)(?:\.|$)", "preferences", "stated_preference"),
            (r"my (?:email|e-mail) is (\S+@\S+)", "contact_info", "email"),
            (r"i live in (.+?)(?:\.|$)", "user_info", "location"),
            (r"my (?:phone|number) is ([\d\-\+\s]+)", "contact_info", "phone"),
        ]

        content_lower = content.lower()
        for pattern, topic, fact_type in patterns:
            matches = re.findall(pattern, content_lower, re.IGNORECASE)
            for match in matches:
                candidates.append(MemoryCandidate(
                    content=match.strip(),
                    topic=topic,
                    fact_type=fact_type,
                    source="user_explicit",
                    raw_text=content
                ))

        return candidates

    def _extract_from_tool_result(self, result: ToolResult) -> List[MemoryCandidate]:
        """Extract facts from tool execution results."""
        candidates = []

        # Skip failed tool calls
        if not result.success:
            return candidates

        # Extract from specific tool types
        metadata = result.metadata or {}
        tool_name = metadata.get("tool_name", "")

        if tool_name == "http_call":
            # API responses might contain structured data worth remembering
            # (e.g., user profile from an API)
            pass  # Implementation depends on API patterns

        return candidates

    def _calculate_importance(self, candidate: MemoryCandidate,
                             context: SessionContext) -> float:
        """
        Calculate importance score for a memory candidate.

        Scoring factors:
        - Explicit user request ("remember this") = +0.4
        - Personal information = +0.3
        - Repeated mention = +0.2
        - Relevant to current task = +0.1

        Returns: 0.0 to 1.0
        """
        score = 0.0

        # Explicit memory request
        if candidate.source == "user_explicit" and candidate.fact_type == "user_stated":
            score += 0.4

        # Personal information
        if candidate.topic in ["user_info", "contact_info", "preferences"]:
            score += 0.3

        # Check for repeated mentions in session
        mentions = self._count_mentions(candidate.content, context.conversation_history)
        if mentions >= 2:
            score += 0.2

        # Task relevance (simplified)
        if self._is_task_relevant(candidate, context):
            score += 0.1

        return min(score, 1.0)

    def _count_mentions(self, content: str, history: List[Dict]) -> int:
        """Count how many times content appears in conversation."""
        count = 0
        content_lower = content.lower()
        for msg in history:
            if content_lower in str(msg.get("content", "")).lower():
                count += 1
        return count

    def _is_task_relevant(self, candidate: MemoryCandidate,
                          context: SessionContext) -> bool:
        """Check if candidate relates to current task."""
        # Simple keyword overlap check
        task_words = set(context.conversation_history[-1].get("content", "").lower().split())
        candidate_words = set(candidate.content.lower().split())
        overlap = len(task_words & candidate_words)
        return overlap >= 2


@dataclass
class MemoryCandidate:
    """A candidate for long-term memory storage."""
    content: str
    topic: str
    fact_type: str
    source: str  # "user_explicit", "tool_result", "inferred"
    raw_text: str
    importance_score: float = 0.0
    timestamp: datetime = field(default_factory=datetime.now)


# Threshold for background storage (relatively high to avoid noise)
BACKGROUND_THRESHOLD = 0.5
```

---

## 4. Session End Review (Automatic)

### 3.1 Purpose

When a session completes, use the LLM to perform a deeper review and extract key facts that should be remembered.

### 3.2 Algorithm

```python
class SessionEndReviewer:
    """Reviews completed sessions for memory extraction."""

    def __init__(self, llm_client: BaseLLMClient, memory_manager: MemoryManager):
        self.llm_client = llm_client
        self.memory_manager = memory_manager

    def review_session(self, session: Dict) -> SessionReviewResult:
        """
        Review a completed session and extract memories.

        Called when:
        - Session status changes to "completed"
        - Session timeout/max_turns reached
        - Explicit session end request
        """
        conversation = session.get("conversation", [])

        if len(conversation) < 3:
            # Too short to extract meaningful memories
            return SessionReviewResult(memories_created=0, notification=None)

        # Build extraction prompt
        prompt = self._build_extraction_prompt(conversation)

        # Ask LLM to extract facts
        response = self.llm_client.complete_json(
            prompt=prompt,
            system=MEMORY_EXTRACTION_SYSTEM_PROMPT,
            caller="session_end_review",
            max_tokens=2000
        )

        if not response:
            return SessionReviewResult(memories_created=0, notification=None)

        # Process extracted facts
        facts = response.get("facts", [])
        memories_created = 0
        stored_items = []

        for fact in facts:
            # Filter sensitive data
            if self._is_sensitive(fact):
                continue

            # Filter health data unless explicitly requested
            if self._is_health_data(fact) and not fact.get("user_requested", False):
                continue

            # Store the memory
            content_id = self.memory_manager.add_memory(
                topic_id=fact.get("topic", "general"),
                title=fact.get("title", "Untitled"),
                content=fact.get("content"),
                keywords=fact.get("keywords", []),
                metadata={
                    "source": "session_review",
                    "session_id": session["session_id"],
                    "importance": fact.get("importance", 0.5),
                    "extracted_at": datetime.now().isoformat()
                }
            )

            memories_created += 1
            stored_items.append({
                "title": fact.get("title"),
                "topic": fact.get("topic")
            })

        # Build user notification
        notification = None
        if stored_items:
            notification = self._build_notification(stored_items)

        return SessionReviewResult(
            memories_created=memories_created,
            notification=notification,
            items=stored_items
        )

    def _build_extraction_prompt(self, conversation: List[Dict]) -> str:
        """Build prompt for LLM memory extraction."""
        # Format conversation for review
        formatted = []
        for msg in conversation:
            role = msg.get("role", "unknown")
            content = msg.get("content", "")
            if isinstance(content, list):
                content = " ".join(
                    block.get("text", "")
                    for block in content
                    if block.get("type") == "text"
                )
            formatted.append(f"{role.upper()}: {content[:500]}")

        conversation_text = "\n".join(formatted[-20:])  # Last 20 messages

        return f"""Review this conversation and extract facts worth remembering long-term.

CONVERSATION:
{conversation_text}

Extract facts that are:
- Personal information about the user (name, preferences, work, etc.)
- Decisions made during this conversation
- Important context for future conversations
- Explicit requests to remember something

DO NOT extract:
- Passwords, API keys, or security credentials
- Temporary or session-specific details
- Health/medical information (unless user explicitly asked to remember it)
- Mundane conversation details

Return JSON:
{{
  "facts": [
    {{
      "title": "Brief title",
      "content": "The fact to remember",
      "topic": "user_info|preferences|decisions|projects|contacts|general",
      "keywords": ["keyword1", "keyword2"],
      "importance": 0.0-1.0,
      "user_requested": true|false
    }}
  ]
}}"""

    def _is_sensitive(self, fact: Dict) -> bool:
        """Check if fact contains sensitive data that should never be stored."""
        content = fact.get("content", "").lower()
        title = fact.get("title", "").lower()

        sensitive_patterns = [
            r"password",
            r"api[_\s]?key",
            r"secret[_\s]?key",
            r"access[_\s]?token",
            r"private[_\s]?key",
            r"credentials",
            r"ssh[_\s]?key",
            r"bearer[_\s]?token",
            r"auth[_\s]?token",
        ]

        combined = f"{title} {content}"
        for pattern in sensitive_patterns:
            if re.search(pattern, combined, re.IGNORECASE):
                return True

        return False

    def _is_health_data(self, fact: Dict) -> bool:
        """Check if fact contains health/medical information."""
        content = fact.get("content", "").lower()
        topic = fact.get("topic", "").lower()

        health_indicators = [
            "diagnosis", "symptom", "medication", "prescription",
            "doctor", "hospital", "medical", "health condition",
            "treatment", "therapy", "disease", "illness"
        ]

        for indicator in health_indicators:
            if indicator in content or indicator in topic:
                return True

        return False

    def _build_notification(self, items: List[Dict]) -> str:
        """Build user notification about stored memories."""
        if not items:
            return None

        lines = ["I've noted the following for future reference:"]
        for item in items[:5]:  # Limit to 5 items in notification
            lines.append(f"  - {item['title']} ({item['topic']})")

        if len(items) > 5:
            lines.append(f"  - ...and {len(items) - 5} more items")

        return "\n".join(lines)


MEMORY_EXTRACTION_SYSTEM_PROMPT = """You are a memory extraction assistant. Your job is to identify facts from conversations that are worth remembering long-term.

Focus on:
- User identity and preferences
- Important decisions and agreements
- Project names and details
- Contact information (except passwords)
- Explicit "remember this" requests

Be selective. Only extract facts that would genuinely help in future conversations. Ignore temporary details, pleasantries, and session-specific context."""


@dataclass
class SessionReviewResult:
    """Result of session end review."""
    memories_created: int
    notification: Optional[str]
    items: List[Dict] = field(default_factory=list)
```

---

## 5. Memory Consolidation

### 4.1 Purpose

Periodically merge related memories to reduce redundancy and strengthen important information.

### 4.2 Algorithm

```python
class MemoryConsolidator:
    """Consolidates related memories."""

    def __init__(self, llm_client: BaseLLMClient, memory_manager: MemoryManager):
        self.llm_client = llm_client
        self.memory_manager = memory_manager

    def run_consolidation(self, topic_id: str = None) -> ConsolidationResult:
        """
        Consolidate memories within a topic.

        Process:
        1. Load all memories for topic
        2. Group by keyword similarity
        3. For groups with 3+ items, ask LLM to merge
        4. Replace old entries with consolidated version
        """
        topics = self.memory_manager.load_topics()
        target_topics = [t for t in topics.get("topics", [])
                        if topic_id is None or t["id"] == topic_id]

        total_merged = 0
        total_removed = 0

        for topic in target_topics:
            result = self._consolidate_topic(topic["id"])
            total_merged += result.get("merged", 0)
            total_removed += result.get("removed", 0)

        return ConsolidationResult(
            topics_processed=len(target_topics),
            memories_merged=total_merged,
            memories_removed=total_removed
        )

    def _consolidate_topic(self, topic_id: str) -> Dict:
        """Consolidate a single topic."""
        index = self.memory_manager.get_topic_index(topic_id)
        if not index:
            return {"merged": 0, "removed": 0}

        entries = index.get("entries", [])
        if len(entries) < 3:
            return {"merged": 0, "removed": 0}

        # Group by keyword similarity
        groups = self._group_by_similarity(entries)

        merged = 0
        removed = 0

        for group in groups:
            if len(group) >= 3:
                # Load full content for each entry
                contents = []
                for entry in group:
                    content = self.memory_manager.get_content(
                        topic_id,
                        entry["content_file"].replace(".json", ""),
                        entry["section_id"]
                    )
                    if content:
                        contents.append({
                            "entry": entry,
                            "content": content
                        })

                if len(contents) >= 3:
                    # Ask LLM to consolidate
                    consolidated = self._llm_consolidate(contents)

                    if consolidated:
                        # Add consolidated memory
                        self.memory_manager.add_memory(
                            topic_id=topic_id,
                            title=consolidated["title"],
                            content=consolidated["content"],
                            keywords=consolidated["keywords"],
                            metadata={
                                "consolidated_from": [c["entry"]["index_id"] for c in contents],
                                "consolidated_at": datetime.now().isoformat()
                            }
                        )
                        merged += 1

                        # Mark old entries for removal
                        for c in contents:
                            self._mark_for_removal(topic_id, c["entry"]["index_id"])
                            removed += 1

        return {"merged": merged, "removed": removed}

    def _group_by_similarity(self, entries: List[Dict]) -> List[List[Dict]]:
        """Group entries by keyword similarity."""
        groups = []
        used = set()

        for i, entry in enumerate(entries):
            if i in used:
                continue

            group = [entry]
            used.add(i)
            keywords_i = set(entry.get("keywords", []))

            for j, other in enumerate(entries):
                if j in used:
                    continue

                keywords_j = set(other.get("keywords", []))

                # Calculate Jaccard similarity
                if keywords_i and keywords_j:
                    intersection = len(keywords_i & keywords_j)
                    union = len(keywords_i | keywords_j)
                    similarity = intersection / union if union > 0 else 0

                    if similarity >= 0.5:  # 50% keyword overlap
                        group.append(other)
                        used.add(j)

            if len(group) >= 2:
                groups.append(group)

        return groups

    def _llm_consolidate(self, contents: List[Dict]) -> Optional[Dict]:
        """Use LLM to consolidate multiple memories into one."""
        memories_text = "\n\n".join(
            f"Memory {i+1}:\n{c['content']}"
            for i, c in enumerate(contents)
        )

        prompt = f"""Consolidate these related memories into a single, comprehensive memory:

{memories_text}

Combine the information, remove redundancy, and keep all important details.

Return JSON:
{{
  "title": "Consolidated title",
  "content": "Consolidated content with all key information",
  "keywords": ["keyword1", "keyword2", ...]
}}"""

        response = self.llm_client.complete_json(
            prompt=prompt,
            system="You consolidate related memories into concise, comprehensive summaries.",
            caller="memory_consolidation",
            max_tokens=1000
        )

        return response

    def _mark_for_removal(self, topic_id: str, index_id: str) -> None:
        """Mark an index entry for removal."""
        # Implementation: either delete immediately or mark with tombstone
        # For safety, we use soft delete (mark as consolidated)
        pass


@dataclass
class ConsolidationResult:
    """Result of consolidation run."""
    topics_processed: int
    memories_merged: int
    memories_removed: int
```

---

## 6. Memory Decay

### 5.1 Purpose

Old, unused memories should fade over time. This prevents memory bloat and keeps the system focused on relevant information.

### 5.2 Algorithm

```python
class MemoryDecay:
    """Applies decay to memory relevance scores."""

    # Decay configuration
    DECAY_RATE = 0.05           # 5% decay per period
    DECAY_PERIOD_DAYS = 7       # Apply decay weekly
    MIN_RELEVANCE = 0.1         # Below this, memory is archived
    BOOST_ON_ACCESS = 0.2       # Boost when memory is accessed

    def __init__(self, memory_manager: MemoryManager):
        self.memory_manager = memory_manager

    def apply_decay(self) -> DecayResult:
        """
        Apply decay to all memory relevance scores.

        Decay formula:
        new_score = old_score * (1 - DECAY_RATE)

        Memories below MIN_RELEVANCE are archived (not deleted).
        """
        topics = self.memory_manager.load_topics()

        decayed = 0
        archived = 0

        for topic in topics.get("topics", []):
            topic_id = topic["id"]
            index = self.memory_manager.get_topic_index(topic_id)

            if not index:
                continue

            entries = index.get("entries", [])
            updated_entries = []

            for entry in entries:
                old_score = entry.get("relevance_score", 1.0)
                last_accessed = entry.get("last_accessed")

                # Check if decay should be applied
                if self._should_decay(entry):
                    new_score = old_score * (1 - self.DECAY_RATE)
                    entry["relevance_score"] = new_score
                    entry["decayed_at"] = datetime.now().isoformat()
                    decayed += 1

                    if new_score < self.MIN_RELEVANCE:
                        entry["status"] = "archived"
                        archived += 1

                updated_entries.append(entry)

            # Save updated index
            index["entries"] = updated_entries
            index["last_updated"] = datetime.now().isoformat()
            self._save_index(topic_id, index)

        return DecayResult(
            memories_decayed=decayed,
            memories_archived=archived
        )

    def boost_on_access(self, topic_id: str, index_id: str) -> None:
        """
        Boost relevance score when a memory is accessed.

        Called when:
        - Memory appears in search results and is used
        - Memory is explicitly referenced in conversation
        """
        index = self.memory_manager.get_topic_index(topic_id)
        if not index:
            return

        for entry in index.get("entries", []):
            if entry.get("index_id") == index_id:
                old_score = entry.get("relevance_score", 0.5)
                new_score = min(1.0, old_score + self.BOOST_ON_ACCESS)
                entry["relevance_score"] = new_score
                entry["last_accessed"] = datetime.now().isoformat()
                entry["access_count"] = entry.get("access_count", 0) + 1
                break

        self._save_index(topic_id, index)

    def _should_decay(self, entry: Dict) -> bool:
        """Check if entry should have decay applied."""
        # Don't decay archived entries
        if entry.get("status") == "archived":
            return False

        # Check last decay time
        last_decay = entry.get("decayed_at")
        if last_decay:
            last_decay_dt = datetime.fromisoformat(last_decay)
            days_since = (datetime.now() - last_decay_dt).days
            return days_since >= self.DECAY_PERIOD_DAYS

        # Check creation time for new entries
        created = entry.get("created_at")
        if created:
            created_dt = datetime.fromisoformat(created)
            days_since = (datetime.now() - created_dt).days
            return days_since >= self.DECAY_PERIOD_DAYS

        return True

    def _save_index(self, topic_id: str, index: Dict) -> None:
        """Save updated index to disk."""
        index_path = self.memory_manager.memory_dir / f"index_{topic_id}.json"
        index_path.write_text(json.dumps(index, indent=2))


@dataclass
class DecayResult:
    """Result of decay application."""
    memories_decayed: int
    memories_archived: int
```

---

## 7. Sensitive Data Handling

### 6.1 Rules

| Data Type | Action | Rationale |
|-----------|--------|-----------|
| Passwords | **NEVER store** | Security risk |
| API keys | **NEVER store** | Security risk |
| SSH keys | **NEVER store** | Security risk |
| Access tokens | **NEVER store** | Security risk |
| Credit card numbers | **NEVER store** | Security/legal risk |
| SSN/ID numbers | **NEVER store** | Security/legal risk |
| Personal health info | **Only if user explicitly asks** | Privacy sensitive |
| Email addresses | Store | Useful for context |
| Phone numbers | Store | Useful for context |
| Names | Store | Useful for personalization |
| Preferences | Store | Useful for personalization |

### 6.2 Detection Patterns

```python
SENSITIVE_PATTERNS = {
    # Never store - security credentials
    "password": r"(?:password|passwd|pwd)\s*[:=]\s*\S+",
    "api_key": r"(?:api[_\s]?key|apikey)\s*[:=]\s*\S+",
    "secret_key": r"(?:secret[_\s]?key|secretkey)\s*[:=]\s*\S+",
    "access_token": r"(?:access[_\s]?token|bearer)\s*[:=]\s*\S+",
    "private_key": r"-----BEGIN (?:RSA |EC |DSA )?PRIVATE KEY-----",
    "ssh_key": r"ssh-(?:rsa|ed25519|dss)\s+\S+",

    # Never store - financial
    "credit_card": r"\b(?:\d{4}[- ]?){3}\d{4}\b",
    "ssn": r"\b\d{3}-\d{2}-\d{4}\b",

    # Conditional - health (only if user_requested=True)
    "health_condition": r"(?:diagnosed|diagnosis|symptom|medication|prescription)",
}

def contains_sensitive_data(text: str) -> Tuple[bool, str]:
    """
    Check if text contains sensitive data.

    Returns: (is_sensitive, reason)
    """
    text_lower = text.lower()

    for category, pattern in SENSITIVE_PATTERNS.items():
        if re.search(pattern, text_lower, re.IGNORECASE):
            return (True, category)

    return (False, None)
```

---

## 8. User Visibility

### 7.1 Notification Strategy

The agent notifies the user about stored memories without requiring approval.

```python
class MemoryNotifier:
    """Notifies user about memory operations."""

    def notify_session_end(self, result: SessionReviewResult) -> Optional[str]:
        """
        Generate notification for session-end memory storage.

        Only notify if memories were actually stored.
        """
        if result.memories_created == 0:
            return None

        return result.notification

    def notify_consolidation(self, result: ConsolidationResult) -> Optional[str]:
        """
        Generate notification for memory consolidation.

        Only notify if significant consolidation occurred.
        """
        if result.memories_merged == 0:
            return None

        return f"I've consolidated {result.memories_merged} related memories to keep things organized."

    def format_memory_list(self, topic_id: str = None, limit: int = 10) -> str:
        """
        Format a list of stored memories for user review.

        Can be called when user asks "what do you remember about me?"
        """
        # Implementation depends on MemoryManager
        pass
```

### 7.2 User Commands

Users can interact with memory through natural language:

| User Says | Agent Action |
|-----------|--------------|
| "What do you remember about me?" | List stored memories |
| "Forget that I work at Acme Corp" | Remove specific memory |
| "Remember that my favorite color is blue" | Explicit memory creation |
| "Show me my stored preferences" | List preferences topic |
| "Clear all my memories" | Archive all (with confirmation) |

---

## 9. Integration with Agentic Loop

### 8.1 Hook Points

```python
class AgenticLoop:
    """Extended with memory decision hooks."""

    def __init__(self, ...):
        # ... existing init ...
        self.background_scanner = BackgroundScanner(self.memory_manager)
        self.session_reviewer = SessionEndReviewer(self.llm_client, self.memory_manager)

    def execute(self, message: str, context: LoopContext) -> LoopResult:
        """Execute loop with memory hooks."""

        # ... existing loop logic ...

        for turn_number in range(1, context.max_turns + 1):
            # ... turn execution ...

            # HOOK: Background scan after each turn
            self.background_scanner.scan_turn(turn, context)

        # HOOK: Session end review
        if result.status in ["completed", "timeout", "max_turns"]:
            session_data = self.memory_manager.load_session(context.session_id)
            review_result = self.session_reviewer.review_session(session_data)

            # Append notification to final response if any
            if review_result.notification:
                result.memory_notification = review_result.notification

        return result
```

### 8.2 Scheduled Tasks

Add these to the scheduler (see SCHEDULER_SPEC.md):

```json
{
  "task_id": "memory_consolidation",
  "name": "Weekly Memory Consolidation",
  "schedule": {
    "type": "cron",
    "cron": "0 3 * * 0"
  },
  "payload": {
    "action": "consolidate_memories"
  }
}
```

```json
{
  "task_id": "memory_decay",
  "name": "Weekly Memory Decay",
  "schedule": {
    "type": "cron",
    "cron": "0 4 * * 0"
  },
  "payload": {
    "action": "apply_memory_decay"
  }
}
```

---

## 10. Configuration

### 9.1 Memory Decision Settings

Add to agent configuration:

```json
{
  "memory": {
    "session_persistence": true,
    "long_term_enabled": true,
    "auto_index": true,

    "decision": {
      "background_scan_enabled": true,
      "background_threshold": 0.5,
      "session_end_review_enabled": true,
      "consolidation_enabled": true,
      "consolidation_schedule": "0 3 * * 0",
      "decay_enabled": true,
      "decay_rate": 0.05,
      "decay_period_days": 7,
      "min_relevance_score": 0.1
    },

    "sensitive_data": {
      "never_store": ["password", "api_key", "credit_card", "ssn"],
      "conditional_store": {
        "health_info": "user_requested_only"
      }
    },

    "notifications": {
      "on_session_end_storage": true,
      "on_consolidation": true,
      "max_items_in_notification": 5
    }
  }
}
```

---

## 11. Summary

### 11.1 Decision Flow

```
User Input
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 1: USER DIRECTIVE CHECK (Highest Priority)        │
│                                                         │
│  "Remember that...", "Don't forget...", "Note:..."     │
│           │                                             │
│       FOUND?                                            │
│       │     │                                           │
│      YES    NO                                          │
│       │     │                                           │
│       ▼     └──────────────────────┐                    │
│  ┌─────────────────┐               │                    │
│  │ Store immediately│               │                    │
│  │ importance=1.0   │               │                    │
│  │ Acknowledge user │               │                    │
│  └─────────────────┘               │                    │
└────────────────────────────────────┼────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 2: BACKGROUND SCANNER (After Each Turn)           │
│                                                         │
│  - Extract explicit facts from user message             │
│  - Score importance (patterns, repetition, relevance)   │
│  - Store if score >= 0.5                                │
└────────────────────────────────────┬────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 3: SESSION END REVIEW (On Session Complete)       │
│                                                         │
│  - LLM deep extraction of key facts                     │
│  - Filter sensitive data (passwords, keys)              │
│  - Filter health data (unless user requested)           │
│  - Store memories                                       │
│  - Notify user what was stored                          │
└────────────────────────────────────┬────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 4: PERIODIC MAINTENANCE (Weekly/Scheduled)        │
│                                                         │
│  - Consolidate related memories (reduce redundancy)     │
│  - Apply decay to old/unused memories                   │
│  - Archive memories below threshold                     │
│  - Boost recently-accessed memories                     │
└─────────────────────────────────────────────────────────┘
```

### 11.2 Key Principles

1. **User directive first**: Explicit "remember this" requests have highest priority and are stored immediately
2. **Hybrid automatic**: Background scanning + session review catch implicit facts
3. **Transparent**: Agent tells user what was stored (no approval required)
4. **Selective**: High threshold prevents noise; only meaningful facts are stored
5. **Safe**: Passwords/credentials never stored; health info only on explicit request
6. **Evolving**: Memories consolidate and decay naturally over time
7. **User control**: User can ask what's remembered, request deletions, or add memories anytime

---

*End of Memory Decision Specification*
