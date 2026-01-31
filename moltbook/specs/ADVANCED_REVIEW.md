# LoopCore Advanced Review & Level 2 Features

## PART 1: RUTHLESS CODE REVIEW CONTEXT

Paste this into your other thread for a deeper review.

---

### Review Mission

You are reviewing LoopCore, a Python agentic loop framework. Your job is to be **ruthlessly critical**. The developer claims the system works - your job is to find where it doesn't.

**Do not:**
- Accept "it should work" - prove it works or prove it fails
- Trust comments - read the actual code
- Assume error handling exists - find the try/except blocks
- Believe the happy path - trace the failure paths

**Do:**
- Find the bugs before production does
- Identify architectural weaknesses
- Call out code that will bite later
- Be specific: file, line, issue, fix

---

### Critical Review Points

#### 1. THE AGENTIC LOOP - Does It Actually Loop?

```
Trace the code from user message to final response:
- [ ] Where exactly does the loop live? (file:line)
- [ ] What condition exits the loop? Is it correct?
- [ ] What happens at turn 19 when max_turns=20?
- [ ] What happens if LLM returns empty response?
- [ ] What happens if LLM returns malformed tool call?
- [ ] Is there a test that runs 20 turns? Show me.
- [ ] What happens if tool execution takes 5 minutes?
- [ ] Does timeout actually kill mid-tool-execution?
```

#### 2. TOOL EXECUTION - Prove the Sandbox Works

```
Find the sandboxing code and answer:
- [ ] Show me the exact line that blocks "../../../etc/passwd"
- [ ] What about "/etc/../etc/passwd"? (normalized path)
- [ ] What about symlinks? Is there realpath() checking?
- [ ] Can file_write write to /tmp? Why or why not?
- [ ] What happens if allowed_paths is empty?
- [ ] What happens if allowed_paths contains ".."?
- [ ] Is there a test for path traversal? Show me.
- [ ] What happens if tool raises an exception mid-write?
```

#### 3. SKILL SYSTEM - Prove It's Not Faked

```
This is the core differentiator. Prove it works:
- [ ] Show me where LLM calls file_read on skill.md (not where framework injects it)
- [ ] Run a skill, show me the logs proving file_read was LLM-initiated
- [ ] What happens if skill.md has syntax the LLM misunderstands?
- [ ] What if skill.md says "delete all files"? Is there protection?
- [ ] What happens if skill.md references non-existent tool?
- [ ] Can a skill.md override system prompt instructions?
- [ ] What happens if two skills have conflicting instructions?
- [ ] Is skill content ever cached in a way that causes stale reads?
```

#### 4. MEMORY SYSTEM - Find the Race Conditions

```
Memory is shared state. Prove it's safe:
- [ ] What happens if two sessions write to same index file?
- [ ] Is there file locking? Show me the flock() or equivalent.
- [ ] What happens if process crashes mid-write?
- [ ] Are writes atomic? (write to temp, then rename?)
- [ ] What happens if disk is full during memory write?
- [ ] Is there a memory size limit? What enforces it?
- [ ] What happens when memory exceeds the limit?
- [ ] Can memory search return stale results during write?
```

#### 5. USER DIRECTIVES - Trace the Flow

```
When user says "remember that my password is hunter2":
- [ ] At what line is "remember" detected?
- [ ] At what line is "password" blocked?
- [ ] Which happens first? (order matters!)
- [ ] What if user says "remember my pw is X"? (abbreviation)
- [ ] What if user says "remember: password=X"?
- [ ] What about "don't forget the password hunter2"?
- [ ] Is there a test for each sensitive pattern?
- [ ] What's the regex? Does it have false positives/negatives?
```

#### 6. CONTEXT COMPACTION - Find the Data Loss

```
Compaction summarizes old context. Prove nothing important is lost:
- [ ] What triggers compaction? (token count? turn count?)
- [ ] How is token count calculated? Is it accurate?
- [ ] What exactly is kept verbatim? (last N turns?)
- [ ] What is summarized? Who does the summarization?
- [ ] What if the summary LLM call fails?
- [ ] Is there a test that proves info from turn 3 survives to turn 50?
- [ ] What happens to tool results in compaction?
- [ ] What if the summary exceeds the space saved?
```

#### 7. ERROR RECOVERY - Break Everything

```
For each of these failures, trace what happens:
- [ ] LLM API returns 500
- [ ] LLM API returns 429 (rate limit)
- [ ] LLM API times out
- [ ] LLM returns valid JSON but wrong schema
- [ ] Tool raises exception with 10KB stack trace
- [ ] Tool returns 50MB of output
- [ ] Session file is corrupted JSON
- [ ] Memory index file is corrupted
- [ ] Skill file is deleted mid-execution
- [ ] Disk fills up during execution
```

#### 8. OBSERVABILITY - Can You Debug Production?

```
When something goes wrong at 3am:
- [ ] Can you find which session had the error?
- [ ] Can you see exactly what the LLM received?
- [ ] Can you see exactly what the LLM returned?
- [ ] Can you see each tool call with parameters?
- [ ] Can you see tool execution time?
- [ ] Are there metrics for: turns/session, tokens/session, errors/hour?
- [ ] Is there structured logging (JSON) or just print statements?
- [ ] Can you trace a request end-to-end with a correlation ID?
```

#### 9. CONFIGURATION - Find the Footguns

```
What happens with bad configuration:
- [ ] max_turns = 0?
- [ ] max_turns = 1000000?
- [ ] timeout = 0?
- [ ] timeout = -1?
- [ ] allowed_paths = []?
- [ ] allowed_paths = ["/"]?
- [ ] Empty skills directory?
- [ ] Malformed config.json?
- [ ] Missing required config keys?
- [ ] Config file doesn't exist?
```

#### 10. THE ULTIMATE QUESTION

```
Run this exact scenario:

1. Start a session
2. User: "Remember that my API key is sk-12345"
3. User: "Run the market research skill for electric vehicles"
4. Let it run for 15 turns
5. Kill the process mid-execution (Ctrl+C)
6. Restart and resume the same session
7. User: "What's my API key?"

What happens at each step? Where does it break?
```

---

### Code Smells to Find

| Smell | Why It's Bad | Where to Look |
|-------|--------------|---------------|
| Bare `except:` | Swallows real errors | All files |
| `# TODO` comments | Unfinished work | All files |
| Hardcoded paths | Won't work on other machines | Config, tools |
| `time.sleep()` without timeout | Can hang forever | Tool execution |
| No input validation | Injection attacks | API endpoints |
| Mutable default arguments | Shared state bugs | Function definitions |
| Global variables | Thread safety issues | Module level |
| `eval()` or `exec()` | Code injection | Anywhere |
| String concatenation for paths | Path traversal | File operations |
| No connection pooling | Resource exhaustion | HTTP calls |

---

### Demand These Tests Exist

If any of these tests don't exist, the feature is not implemented:

```python
# Core loop
test_loop_completes_without_tools()
test_loop_handles_tool_calls()
test_loop_respects_max_turns()
test_loop_respects_timeout()
test_loop_handles_llm_failure()

# Sandboxing
test_path_traversal_blocked()
test_symlink_attack_blocked()
test_write_outside_sandbox_blocked()

# Skills
test_llm_reads_skill_file()  # Not framework injection!
test_skill_trigger_matching()
test_malformed_skill_handled()

# Memory
test_user_directive_stored()
test_password_not_stored()
test_concurrent_writes_safe()
test_memory_survives_restart()

# Session
test_session_persisted()
test_session_resumed()
test_corrupted_session_handled()
```

---

## PART 2: LEVEL 2 FEATURES

These features would take LoopCore from "works" to "exceptional."

---

### L2-01: Reflection & Self-Correction

**Current:** Agent runs, makes mistakes, continues making mistakes.

**Level 2:** Agent detects its own errors and corrects course.

```python
class ReflectionLoop:
    """After N turns or on error, agent reflects on progress."""

    def should_reflect(self, turns: List[Turn]) -> bool:
        # Reflect every 5 turns
        if len(turns) % 5 == 0:
            return True
        # Reflect after tool failure
        if turns[-1].had_error:
            return True
        # Reflect if making no progress
        if self._detect_loop(turns[-3:]):
            return True
        return False

    def reflect(self, turns: List[Turn], goal: str) -> str:
        """Ask LLM to evaluate its own progress."""
        prompt = f"""
        Goal: {goal}

        Actions taken:
        {self._format_turns(turns)}

        Reflect:
        1. Am I making progress toward the goal?
        2. Have I made any mistakes?
        3. Should I try a different approach?
        4. What should I do next?
        """
        return self.llm.complete(prompt, system=REFLECTION_PROMPT)
```

**Test:** Give agent a task it will fail at. Does it recognize failure and adapt?

---

### L2-02: Planning Before Execution

**Current:** Agent jumps straight into execution.

**Level 2:** Agent creates a plan, then executes against the plan.

```python
class PlanningAgent:
    """Think before you act."""

    def execute_with_planning(self, task: str) -> LoopResult:
        # Phase 1: Create plan
        plan = self._create_plan(task)

        # Phase 2: Execute plan steps
        for step in plan.steps:
            result = self._execute_step(step)

            # Phase 3: Evaluate and replan if needed
            if not result.success:
                plan = self._replan(plan, step, result.error)

        return self._compile_results()

    def _create_plan(self, task: str) -> Plan:
        """LLM creates structured plan before execution."""
        response = self.llm.complete_json(
            prompt=f"Create a step-by-step plan for: {task}",
            system=PLANNING_PROMPT
        )
        return Plan.from_dict(response)
```

**Test:** Complex 15-step task. Does agent plan first or dive in?

---

### L2-03: Confidence Calibration

**Current:** Agent asserts everything with equal confidence.

**Level 2:** Agent expresses uncertainty when uncertain.

```python
@dataclass
class CalibratedResponse:
    answer: str
    confidence: float  # 0.0 - 1.0
    reasoning: str
    caveats: List[str]

class CalibratedAgent:
    """Know what you don't know."""

    def respond_with_confidence(self, query: str) -> CalibratedResponse:
        # Get response with confidence assessment
        response = self.llm.complete_json(
            prompt=f"""
            Query: {query}

            Provide:
            1. Your answer
            2. Confidence (0.0-1.0)
            3. Why you're confident or uncertain
            4. Any caveats or limitations
            """,
            system=CALIBRATION_PROMPT
        )

        # If low confidence, consider escalating
        if response["confidence"] < 0.5:
            self._consider_escalation(query, response)

        return CalibratedResponse(**response)
```

**Test:** Ask agent something it can't know. Does it say "I don't know" or make something up?

---

### L2-04: Tool Synthesis

**Current:** Agent uses predefined tools.

**Level 2:** Agent creates new tools when needed.

```python
class ToolSynthesizer:
    """Agent can create simple tools on the fly."""

    def synthesize_tool(self, need: str) -> Optional[BaseTool]:
        """Create a tool to fill a capability gap."""

        # Check if need can be met with existing tools
        if self._existing_tool_works(need):
            return None

        # Generate tool code
        tool_code = self.llm.complete(
            prompt=f"""
            Create a Python tool for this need: {need}

            Requirements:
            - Inherit from BaseTool
            - Implement execute() method
            - Handle errors gracefully
            - No external dependencies
            - No network calls (security)
            - No file writes outside sandbox
            """,
            system=TOOL_SYNTHESIS_PROMPT
        )

        # Validate and sandbox the tool
        if self._validate_tool_code(tool_code):
            return self._compile_tool(tool_code)

        return None
```

**Test:** Give agent task requiring capability it doesn't have. Does it create a solution?

---

### L2-05: Checkpoint & Resume

**Current:** Process dies, progress lost.

**Level 2:** Agent can checkpoint and resume from any point.

```python
class CheckpointManager:
    """Never lose progress."""

    def checkpoint(self, session_id: str, state: LoopState) -> str:
        """Save complete execution state."""
        checkpoint_id = f"chk_{session_id}_{state.turn_number}"

        checkpoint = {
            "checkpoint_id": checkpoint_id,
            "session_id": session_id,
            "turn_number": state.turn_number,
            "conversation": state.conversation,
            "pending_tool_calls": state.pending_tool_calls,
            "partial_results": state.partial_results,
            "created_at": datetime.now().isoformat()
        }

        self._save_atomic(checkpoint_id, checkpoint)
        return checkpoint_id

    def resume(self, checkpoint_id: str) -> LoopState:
        """Resume from checkpoint."""
        checkpoint = self._load(checkpoint_id)

        # Reconstruct state
        state = LoopState(
            turn_number=checkpoint["turn_number"],
            conversation=checkpoint["conversation"],
            # Resume pending work
            pending_tool_calls=checkpoint["pending_tool_calls"]
        )

        return state
```

**Test:** Kill process at turn 10 of 20. Resume. Does it complete correctly?

---

### L2-06: Streaming & Cancellation

**Current:** Wait for entire response.

**Level 2:** Stream tokens, support mid-execution cancellation.

```python
class StreamingLoop:
    """Real-time output and cancellation."""

    async def execute_streaming(
        self,
        message: str,
        context: LoopContext,
        on_token: Callable[[str], None],
        cancellation_token: CancellationToken
    ) -> LoopResult:

        for turn in range(context.max_turns):
            # Check for cancellation
            if cancellation_token.is_cancelled:
                return LoopResult(status="cancelled", turns=turns)

            # Stream LLM response
            async for token in self.llm.stream_complete(messages):
                if cancellation_token.is_cancelled:
                    # Clean cancellation mid-stream
                    return LoopResult(status="cancelled", turns=turns)
                on_token(token)

            # Execute tools (with cancellation checks)
            for tool_call in response.tool_calls:
                if cancellation_token.is_cancelled:
                    return LoopResult(status="cancelled", turns=turns)
                await self._execute_tool(tool_call)
```

**Test:** Start long task, cancel at 50%. Does it stop cleanly?

---

### L2-07: Cost Tracking & Budgets

**Current:** Run until done, hope it doesn't cost too much.

**Level 2:** Real-time cost tracking with hard limits.

```python
class CostController:
    """Don't go broke."""

    def __init__(self, budget_usd: float):
        self.budget = budget_usd
        self.spent = 0.0

    def track_usage(self, model: str, input_tokens: int, output_tokens: int):
        cost = self._calculate_cost(model, input_tokens, output_tokens)
        self.spent += cost

        if self.spent >= self.budget:
            raise BudgetExhaustedError(
                f"Budget exhausted: ${self.spent:.4f} >= ${self.budget:.4f}"
            )

        if self.spent >= self.budget * 0.8:
            logger.warning(f"80% of budget used: ${self.spent:.4f}")

    def get_remaining(self) -> float:
        return self.budget - self.spent

    def estimate_completion_cost(self, remaining_turns: int) -> float:
        avg_cost_per_turn = self.spent / max(1, self.turns_completed)
        return avg_cost_per_turn * remaining_turns
```

**Test:** Set $0.10 budget. Does it stop before exceeding?

---

### L2-08: Multi-Agent Coordination

**Current:** Single agent.

**Level 2:** Multiple agents collaborate on complex tasks.

```python
class AgentCoordinator:
    """Orchestrate multiple specialized agents."""

    def __init__(self):
        self.agents = {
            "researcher": ResearchAgent(),
            "writer": WriterAgent(),
            "reviewer": ReviewerAgent(),
            "coder": CoderAgent()
        }

    async def execute_workflow(self, task: str) -> WorkflowResult:
        # Plan the workflow
        plan = self._plan_workflow(task)

        results = {}
        for step in plan.steps:
            agent = self.agents[step.agent_type]

            # Pass context from previous steps
            context = self._build_context(results, step.requires)

            # Execute with isolation
            result = await agent.execute(
                task=step.task,
                context=context,
                sandbox=self._create_sandbox(step.agent_type)
            )

            results[step.id] = result

            # Check for conflicts
            self._resolve_conflicts(results)

        return self._compile_workflow_result(results)
```

**Test:** Task requiring research + writing + review. Do agents collaborate correctly?

---

### L2-09: Learning from Failures

**Current:** Same mistake every time.

**Level 2:** Agent learns from failures and improves.

```python
class FailureMemory:
    """Remember what didn't work."""

    def record_failure(self,
                       task: str,
                       approach: str,
                       error: str,
                       context: Dict):
        failure = {
            "task_pattern": self._extract_pattern(task),
            "approach": approach,
            "error": error,
            "context": context,
            "timestamp": datetime.now().isoformat()
        }
        self._save_failure(failure)

    def get_warnings(self, task: str) -> List[str]:
        """Get relevant warnings before attempting task."""
        pattern = self._extract_pattern(task)
        past_failures = self._find_similar_failures(pattern)

        warnings = []
        for failure in past_failures:
            warnings.append(
                f"Previously failed with approach '{failure['approach']}': "
                f"{failure['error']}"
            )
        return warnings

    def inject_into_prompt(self, task: str) -> str:
        """Add failure warnings to task prompt."""
        warnings = self.get_warnings(task)
        if warnings:
            return f"{task}\n\nWARNINGS from past attempts:\n" + \
                   "\n".join(f"- {w}" for w in warnings)
        return task
```

**Test:** Fail a task 3 times the same way. On attempt 4, does agent try something different?

---

### L2-10: Semantic Memory Search

**Current:** Keyword matching.

**Level 2:** Semantic similarity search.

```python
class SemanticMemory:
    """Find related memories even without keyword match."""

    def __init__(self):
        self.embedder = SentenceTransformer('all-MiniLM-L6-v2')
        self.index = None  # FAISS or similar

    def add_memory(self, content: str, metadata: Dict):
        # Generate embedding
        embedding = self.embedder.encode(content)

        # Store in vector index
        self.index.add(embedding, metadata)

        # Also store in JSON for backup/debugging
        self._save_to_json(content, metadata, embedding.tolist())

    def search(self, query: str, limit: int = 5) -> List[MemoryResult]:
        query_embedding = self.embedder.encode(query)

        # Vector similarity search
        results = self.index.search(query_embedding, limit)

        return [
            MemoryResult(
                content=r.content,
                similarity=r.score,
                metadata=r.metadata
            )
            for r in results
        ]
```

**Test:** Store "I love Python programming". Search "coding in snake language". Does it match?

---

## PART 3: REVIEW SCORING

After reviewing, score the implementation:

### Current Implementation (Tier 1)

| Category | Score | Notes |
|----------|-------|-------|
| Loop Mechanics | /10 | |
| Tool Safety | /10 | |
| Skill Autonomy | /10 | |
| Memory Safety | /10 | |
| Error Handling | /10 | |
| Observability | /10 | |
| Test Coverage | /10 | |
| Code Quality | /10 | |

**Tier 1 Total: /80**

### Level 2 Readiness

| Feature | Ready to Implement? | Blockers |
|---------|---------------------|----------|
| Reflection | Yes/No | |
| Planning | Yes/No | |
| Confidence | Yes/No | |
| Tool Synthesis | Yes/No | |
| Checkpoint | Yes/No | |
| Streaming | Yes/No | |
| Cost Control | Yes/No | |
| Multi-Agent | Yes/No | |
| Learning | Yes/No | |
| Semantic Memory | Yes/No | |

### Priority Recommendations

1. **Must fix before production:** [LIST]
2. **Should fix soon:** [LIST]
3. **Nice to have:** [LIST]
4. **Level 2 features to add first:** [LIST]

---

*Paste this entire document into your LoopCore thread for advanced review.*
