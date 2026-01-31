# Agentic Loop Framework Specification

**Version:** 1.0.0
**Status:** Draft
**Author:** Technical Specification
**Date:** 2026-01-31

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [Core Components](#3-core-components)
4. [The Agentic Loop](#4-the-agentic-loop)
5. [Tool System](#5-tool-system)
6. [Skill System](#6-skill-system)
7. [Memory System](#7-memory-system)
8. [Session Management](#8-session-management)
9. [Configuration](#9-configuration)
10. [API & CLI Interface](#10-api--cli-interface)
11. [Error Handling](#11-error-handling)
12. [File Structure](#12-file-structure)
13. [Implementation Guide](#13-implementation-guide)

---

## 1. Overview

### 1.1 Purpose

This document specifies a **Python Agentic Loop Framework** - a reusable library for building LLM-powered agents that can:

- Execute multi-step tasks autonomously
- Use tools (HTTP calls, file I/O, web fetching)
- Load and follow skill instructions from markdown files
- Maintain conversation context and long-term memory
- Run within configurable safety boundaries

### 1.2 Design Principles

| Principle | Description |
|-----------|-------------|
| **LLM-Driven** | The LLM decides what to do; code provides capabilities |
| **Skill-Based** | Complex behaviors defined in markdown, not code |
| **Sandboxed** | All file operations restricted to allowed directories |
| **Transparent** | Full logging of decisions, tool calls, and reasoning |
| **Extensible** | Easy to add new tools and skills |

### 1.3 Primary Use Case

A general-purpose agentic framework where:
- Agents read skill files (markdown) to learn behaviors
- Agents execute multi-turn loops calling tools as needed
- Agents maintain memory across sessions
- Multiple agents can share the infrastructure

---

## 2. Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                          │
│                      (CLI / FastAPI / Admin)                    │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                        AGENT MANAGER                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Agent A    │  │  Agent B    │  │  Agent C    │   ...        │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                       AGENTIC LOOP ENGINE                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    LOOP CONTROLLER                        │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │   │
│  │  │ Context │  │  LLM    │  │  Tool   │  │ Result  │      │   │
│  │  │ Builder │─▶│ Invoke  │─▶│ Execute │─▶│ Inject  │──┐   │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │   │   │
│  │       ▲                                              │   │   │
│  │       └──────────────────────────────────────────────┘   │   │
│  │                      (LOOP UNTIL DONE)                    │   │
│  └──────────────────────────────────────────────────────────┘   │
└───────┬─────────────────┬─────────────────┬─────────────────────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ TOOL SYSTEM  │  │ SKILL SYSTEM │  │MEMORY SYSTEM │
│              │  │              │  │              │
│ • http_call  │  │ • Loader     │  │ • Sessions   │
│ • web_fetch  │  │ • Registry   │  │ • Long-term  │
│ • file_read  │  │ • Injector   │  │ • Indexes    │
│ • file_write │  │              │  │              │
└──────────────┘  └──────────────┘  └──────────────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        FILE SYSTEM                              │
│  SKILLS/    MEMORY/    OUTPUT/    CONFIG/    SANDBOX/           │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Interaction Flow

```
User Request
     │
     ▼
┌─────────────────┐
│ Load Agent      │
│ Configuration   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Load Skills     │──────▶ SKILLS/<skill_id>/skill.md
│ (if applicable) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Load Session    │──────▶ MEMORY/sessions/<session_id>.json
│ Context         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Build System    │
│ Prompt          │
└────────┬────────┘
         │
         ▼
┌─────────────────┐         ┌─────────────────┐
│ AGENTIC LOOP    │◀───────▶│ LLM Client      │
│ (see Section 4) │         │ (Anthropic/     │
└────────┬────────┘         │  OpenAI)        │
         │                  └─────────────────┘
         ▼
┌─────────────────┐
│ Persist Session │──────▶ MEMORY/sessions/<session_id>.json
│ & Output        │──────▶ OUTPUT/<agent_id>/<date>/...
└─────────────────┘
```

---

## 3. Core Components

### 3.1 Component Overview

| Component | Responsibility | Key Files |
|-----------|---------------|-----------|
| **AgentManager** | Agent lifecycle, configuration | `agent_manager.py` |
| **AgenticLoop** | Core loop execution | `agentic_loop.py` |
| **ToolRegistry** | Tool registration and execution | `tool_registry.py` |
| **SkillLoader** | Skill discovery and loading | `skill_loader.py` |
| **MemoryManager** | Session and long-term memory | `memory_manager.py` |
| **ContextBuilder** | System prompt assembly | `context_builder.py` |
| **OutputManager** | Output file organization | `output_manager.py` |

### 3.2 Class Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          AgentManager                           │
├─────────────────────────────────────────────────────────────────┤
│ + agents: Dict[str, AgentConfig]                                │
│ + create_agent(agent_id, config) -> Agent                       │
│ + get_agent(agent_id) -> Agent                                  │
│ + run_agent(agent_id, message, session_id) -> AgentResult       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ creates
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                            Agent                                │
├─────────────────────────────────────────────────────────────────┤
│ + agent_id: str                                                 │
│ + config: AgentConfig                                           │
│ + loop: AgenticLoop                                             │
│ + memory: MemoryManager                                         │
│ + run(message, session_id) -> AgentResult                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ uses
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         AgenticLoop                             │
├─────────────────────────────────────────────────────────────────┤
│ + llm_client: BaseLLMClient                                     │
│ + tool_registry: ToolRegistry                                   │
│ + context_builder: ContextBuilder                               │
│ + max_turns: int                                                │
│ + timeout_seconds: int                                          │
│ + execute(message, context) -> LoopResult                       │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  ToolRegistry   │ │  SkillLoader    │ │  MemoryManager  │
├─────────────────┤ ├─────────────────┤ ├─────────────────┤
│ + tools: Dict   │ │ + skills: Dict  │ │ + sessions: Dict│
│ + register()    │ │ + load()        │ │ + load_session()│
│ + execute()     │ │ + get_prompt()  │ │ + save_session()│
│ + get_schema()  │ │ + fetch_url()   │ │ + index_memory()│
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

---

## 4. The Agentic Loop

### 4.1 Loop Algorithm

```python
def execute_agentic_loop(message: str, context: LoopContext) -> LoopResult:
    """
    Core agentic loop algorithm.

    The loop continues until:
    1. LLM returns text without tool calls (natural completion)
    2. Max turns reached (safety limit)
    3. Timeout exceeded (safety limit)
    4. Fatal error occurs
    """

    turns = []
    start_time = time.time()

    # Initialize conversation with user message
    conversation = context.conversation_history.copy()
    conversation.append({"role": "user", "content": message})

    for turn_number in range(1, context.max_turns + 1):

        # Check timeout
        elapsed = time.time() - start_time
        if elapsed > context.timeout_seconds:
            return LoopResult(
                status="timeout",
                turns=turns,
                final_response=None,
                error=f"Timeout after {elapsed:.1f}s"
            )

        # Build full prompt with system context
        full_context = context_builder.build(
            system_prompt=context.system_prompt,
            skills_prompt=context.skills_prompt,
            memory_prompt=context.memory_prompt,
            conversation=conversation
        )

        # Call LLM
        llm_response = llm_client.complete_with_tools(
            messages=full_context,
            tools=context.available_tools,
            caller=f"agentic_loop_turn_{turn_number}"
        )

        # Record turn
        turn = Turn(
            number=turn_number,
            llm_response=llm_response,
            tool_calls=[],
            tool_results=[]
        )

        # Check if LLM wants to use tools
        if llm_response.tool_calls:
            # Execute each tool call
            for tool_call in llm_response.tool_calls:
                tool_result = execute_tool(
                    tool_call=tool_call,
                    context=context
                )
                turn.tool_calls.append(tool_call)
                turn.tool_results.append(tool_result)

            # Add assistant message with tool calls to conversation
            conversation.append({
                "role": "assistant",
                "content": llm_response.text,
                "tool_calls": llm_response.tool_calls
            })

            # Add tool results to conversation
            for tool_call, result in zip(turn.tool_calls, turn.tool_results):
                conversation.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": result.output
                })

            turns.append(turn)
            # Continue loop - LLM needs to process tool results

        else:
            # No tool calls - LLM is done
            turns.append(turn)
            return LoopResult(
                status="completed",
                turns=turns,
                final_response=llm_response.text,
                error=None
            )

    # Max turns reached
    return LoopResult(
        status="max_turns",
        turns=turns,
        final_response=turns[-1].llm_response.text if turns else None,
        error=f"Reached maximum turns ({context.max_turns})"
    )
```

### 4.2 Loop State Machine

```
                    ┌─────────────────┐
                    │     START       │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
             ┌─────▶│  BUILD CONTEXT  │
             │      └────────┬────────┘
             │               │
             │               ▼
             │      ┌─────────────────┐
             │      │   LLM INVOKE    │
             │      └────────┬────────┘
             │               │
             │               ▼
             │      ┌─────────────────┐
             │      │  HAS TOOL CALLS?│
             │      └────────┬────────┘
             │               │
             │       ┌───────┴───────┐
             │       │               │
             │      YES              NO
             │       │               │
             │       ▼               ▼
             │ ┌───────────┐   ┌───────────┐
             │ │ EXECUTE   │   │  RETURN   │
             │ │ TOOLS     │   │  RESULT   │
             │ └─────┬─────┘   └───────────┘
             │       │
             │       ▼
             │ ┌───────────┐
             │ │  INJECT   │
             │ │  RESULTS  │
             │ └─────┬─────┘
             │       │
             │       ▼
             │ ┌───────────┐
             │ │CHECK LIMITS│
             │ │(turns/time)│
             │ └─────┬─────┘
             │       │
             │   ┌───┴───┐
             │   │       │
             │  OK    EXCEEDED
             │   │       │
             └───┘       ▼
                   ┌───────────┐
                   │  RETURN   │
                   │  TIMEOUT/ │
                   │  MAX_TURNS│
                   └───────────┘
```

### 4.3 Turn Data Structure

```python
@dataclass
class Turn:
    """Represents a single turn in the agentic loop."""
    number: int
    timestamp: datetime
    llm_response: LLMResponse
    tool_calls: List[ToolCall]
    tool_results: List[ToolResult]
    duration_ms: int
    tokens_used: TokenUsage

@dataclass
class LoopResult:
    """Result of an agentic loop execution."""
    status: Literal["completed", "timeout", "max_turns", "error"]
    turns: List[Turn]
    final_response: Optional[str]
    error: Optional[str]
    total_duration_ms: int
    total_tokens: TokenUsage
    tools_called: List[str]  # Summary of tools used
```

### 4.4 Context Window Management

When the conversation grows too large, apply compaction:

```python
def compact_conversation(
    conversation: List[Message],
    max_tokens: int,
    llm_client: BaseLLMClient
) -> List[Message]:
    """
    Compact conversation history to fit within token limits.

    Strategy:
    1. Always keep: system prompt, last N turns, current user message
    2. Summarize: middle portion of conversation
    3. Preserve: tool results that are still relevant
    """

    current_tokens = count_tokens(conversation)

    if current_tokens <= max_tokens:
        return conversation

    # Keep system (first) and recent messages (last 6)
    system_msg = conversation[0]
    recent_msgs = conversation[-6:]
    middle_msgs = conversation[1:-6]

    if not middle_msgs:
        return conversation  # Nothing to compact

    # Summarize middle portion
    summary = llm_client.complete(
        prompt=f"Summarize this conversation excerpt concisely:\n\n{format_messages(middle_msgs)}",
        system="You are a conversation summarizer. Be concise but preserve key facts and decisions.",
        max_tokens=500
    )

    summary_msg = {
        "role": "system",
        "content": f"[Earlier conversation summary: {summary}]"
    }

    return [system_msg, summary_msg] + recent_msgs
```

---

## 5. Tool System

### 5.1 Tool Interface

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Any, Dict, List, Optional

@dataclass
class ToolParameter:
    """Definition of a tool parameter."""
    name: str
    type: str  # "string", "integer", "boolean", "array", "object"
    description: str
    required: bool = True
    enum: Optional[List[str]] = None
    default: Optional[Any] = None

@dataclass
class ToolDefinition:
    """Complete tool definition for LLM."""
    name: str
    description: str
    parameters: List[ToolParameter]

    def to_schema(self) -> Dict:
        """Convert to JSON Schema for LLM tool calling."""
        properties = {}
        required = []

        for param in self.parameters:
            properties[param.name] = {
                "type": param.type,
                "description": param.description
            }
            if param.enum:
                properties[param.name]["enum"] = param.enum
            if param.required:
                required.append(param.name)

        return {
            "type": "function",
            "function": {
                "name": self.name,
                "description": self.description,
                "parameters": {
                    "type": "object",
                    "properties": properties,
                    "required": required
                }
            }
        }

class BaseTool(ABC):
    """Base class for all tools."""

    @property
    @abstractmethod
    def definition(self) -> ToolDefinition:
        """Return the tool's definition."""
        pass

    @abstractmethod
    def execute(self, **kwargs) -> ToolResult:
        """Execute the tool with given parameters."""
        pass

@dataclass
class ToolResult:
    """Result of tool execution."""
    success: bool
    output: str
    error: Optional[str] = None
    metadata: Optional[Dict] = None
```

### 5.2 Built-in Tools

#### 5.2.1 HTTP Call Tool

```python
class HttpCallTool(BaseTool):
    """Make HTTP requests to APIs."""

    @property
    def definition(self) -> ToolDefinition:
        return ToolDefinition(
            name="http_call",
            description="Make an HTTP request to an API endpoint. Use for REST APIs, webhooks, etc.",
            parameters=[
                ToolParameter(
                    name="method",
                    type="string",
                    description="HTTP method",
                    enum=["GET", "POST", "PUT", "PATCH", "DELETE"]
                ),
                ToolParameter(
                    name="url",
                    type="string",
                    description="Full URL to call"
                ),
                ToolParameter(
                    name="headers",
                    type="object",
                    description="HTTP headers as key-value pairs",
                    required=False
                ),
                ToolParameter(
                    name="body",
                    type="string",
                    description="Request body (for POST/PUT/PATCH)",
                    required=False
                ),
                ToolParameter(
                    name="timeout_seconds",
                    type="integer",
                    description="Request timeout in seconds",
                    required=False,
                    default=30
                )
            ]
        )

    def execute(self, method: str, url: str, headers: Dict = None,
                body: str = None, timeout_seconds: int = 30) -> ToolResult:
        """Execute HTTP request."""
        try:
            response = requests.request(
                method=method,
                url=url,
                headers=headers or {},
                data=body,
                timeout=timeout_seconds
            )

            return ToolResult(
                success=True,
                output=json.dumps({
                    "status_code": response.status_code,
                    "headers": dict(response.headers),
                    "body": response.text[:10000]  # Limit response size
                }, indent=2),
                metadata={"status_code": response.status_code}
            )
        except Exception as e:
            return ToolResult(
                success=False,
                output="",
                error=str(e)
            )
```

#### 5.2.2 Web Fetch Tool

```python
class WebFetchTool(BaseTool):
    """Fetch and extract content from web pages."""

    @property
    def definition(self) -> ToolDefinition:
        return ToolDefinition(
            name="web_fetch",
            description="Fetch a web page and extract its text content. Good for reading articles, documentation, etc.",
            parameters=[
                ToolParameter(
                    name="url",
                    type="string",
                    description="URL of the web page to fetch"
                ),
                ToolParameter(
                    name="extract_mode",
                    type="string",
                    description="How to extract content",
                    enum=["text", "markdown", "html"],
                    default="markdown"
                ),
                ToolParameter(
                    name="max_length",
                    type="integer",
                    description="Maximum characters to return",
                    required=False,
                    default=15000
                )
            ]
        )

    def execute(self, url: str, extract_mode: str = "markdown",
                max_length: int = 15000) -> ToolResult:
        """Fetch and extract web page content."""
        try:
            response = requests.get(url, timeout=30, headers={
                "User-Agent": "AgenticLoop/1.0"
            })
            response.raise_for_status()

            if extract_mode == "html":
                content = response.text[:max_length]
            elif extract_mode == "text":
                soup = BeautifulSoup(response.text, 'html.parser')
                content = soup.get_text(separator='\n', strip=True)[:max_length]
            else:  # markdown
                content = html_to_markdown(response.text)[:max_length]

            return ToolResult(
                success=True,
                output=content,
                metadata={"url": url, "length": len(content)}
            )
        except Exception as e:
            return ToolResult(
                success=False,
                output="",
                error=str(e)
            )
```

#### 5.2.3 File Read Tool

```python
class FileReadTool(BaseTool):
    """Read files from allowed directories."""

    def __init__(self, allowed_paths: List[str]):
        self.allowed_paths = [Path(p).resolve() for p in allowed_paths]

    @property
    def definition(self) -> ToolDefinition:
        return ToolDefinition(
            name="file_read",
            description="Read content from a file. Supports .md, .txt, .csv, .json files.",
            parameters=[
                ToolParameter(
                    name="path",
                    type="string",
                    description="Path to the file to read"
                ),
                ToolParameter(
                    name="encoding",
                    type="string",
                    description="File encoding",
                    required=False,
                    default="utf-8"
                )
            ]
        )

    def execute(self, path: str, encoding: str = "utf-8") -> ToolResult:
        """Read file content."""
        try:
            file_path = Path(path).resolve()

            # Security: Check if path is within allowed directories
            if not any(self._is_subpath(file_path, allowed) for allowed in self.allowed_paths):
                return ToolResult(
                    success=False,
                    output="",
                    error=f"Access denied: {path} is outside allowed directories"
                )

            if not file_path.exists():
                return ToolResult(
                    success=False,
                    output="",
                    error=f"File not found: {path}"
                )

            content = file_path.read_text(encoding=encoding)

            return ToolResult(
                success=True,
                output=content,
                metadata={
                    "path": str(file_path),
                    "size_bytes": len(content.encode()),
                    "extension": file_path.suffix
                }
            )
        except Exception as e:
            return ToolResult(
                success=False,
                output="",
                error=str(e)
            )

    def _is_subpath(self, path: Path, parent: Path) -> bool:
        """Check if path is under parent directory."""
        try:
            path.relative_to(parent)
            return True
        except ValueError:
            return False
```

#### 5.2.4 File Write Tool

```python
class FileWriteTool(BaseTool):
    """Write files to allowed directories."""

    def __init__(self, allowed_paths: List[str]):
        self.allowed_paths = [Path(p).resolve() for p in allowed_paths]

    @property
    def definition(self) -> ToolDefinition:
        return ToolDefinition(
            name="file_write",
            description="Write content to a file. Creates parent directories if needed. Supports .md, .txt, .csv, .json files.",
            parameters=[
                ToolParameter(
                    name="path",
                    type="string",
                    description="Path to the file to write"
                ),
                ToolParameter(
                    name="content",
                    type="string",
                    description="Content to write to the file"
                ),
                ToolParameter(
                    name="mode",
                    type="string",
                    description="Write mode",
                    enum=["overwrite", "append"],
                    default="overwrite"
                ),
                ToolParameter(
                    name="encoding",
                    type="string",
                    description="File encoding",
                    required=False,
                    default="utf-8"
                )
            ]
        )

    def execute(self, path: str, content: str, mode: str = "overwrite",
                encoding: str = "utf-8") -> ToolResult:
        """Write content to file."""
        try:
            file_path = Path(path).resolve()

            # Security: Check if path is within allowed directories
            if not any(self._is_subpath(file_path, allowed) for allowed in self.allowed_paths):
                return ToolResult(
                    success=False,
                    output="",
                    error=f"Access denied: {path} is outside allowed directories"
                )

            # Create parent directories
            file_path.parent.mkdir(parents=True, exist_ok=True)

            # Write file
            if mode == "append":
                with open(file_path, 'a', encoding=encoding) as f:
                    f.write(content)
            else:
                file_path.write_text(content, encoding=encoding)

            return ToolResult(
                success=True,
                output=f"Successfully wrote {len(content)} characters to {path}",
                metadata={
                    "path": str(file_path),
                    "size_bytes": len(content.encode()),
                    "mode": mode
                }
            )
        except Exception as e:
            return ToolResult(
                success=False,
                output="",
                error=str(e)
            )

    def _is_subpath(self, path: Path, parent: Path) -> bool:
        try:
            path.relative_to(parent)
            return True
        except ValueError:
            return False
```

### 5.3 Tool Registry

```python
class ToolRegistry:
    """Registry for managing and executing tools."""

    def __init__(self):
        self._tools: Dict[str, BaseTool] = {}

    def register(self, tool: BaseTool) -> None:
        """Register a tool."""
        self._tools[tool.definition.name] = tool

    def get(self, name: str) -> Optional[BaseTool]:
        """Get a tool by name."""
        return self._tools.get(name)

    def list_tools(self) -> List[str]:
        """List all registered tool names."""
        return list(self._tools.keys())

    def get_schemas(self) -> List[Dict]:
        """Get all tool schemas for LLM."""
        return [tool.definition.to_schema() for tool in self._tools.values()]

    def execute(self, tool_name: str, parameters: Dict) -> ToolResult:
        """Execute a tool by name with parameters."""
        tool = self._tools.get(tool_name)
        if not tool:
            return ToolResult(
                success=False,
                output="",
                error=f"Unknown tool: {tool_name}"
            )

        try:
            return tool.execute(**parameters)
        except Exception as e:
            return ToolResult(
                success=False,
                output="",
                error=f"Tool execution error: {str(e)}"
            )
```

---

## 6. Skill System

### 6.1 Skill Structure

Each skill lives in its own directory:

```
SKILLS/
├── registry.json              # Master registry of all skills
├── moltbook/
│   ├── skill.json            # Skill metadata
│   ├── skill.md              # Main skill instructions
│   ├── heartbeat.md          # Optional: periodic check instructions
│   └── messaging.md          # Optional: additional instructions
├── github/
│   ├── skill.json
│   └── skill.md
└── web_research/
    ├── skill.json
    └── skill.md
```

### 6.2 Skill Metadata (skill.json)

```json
{
  "id": "moltbook",
  "name": "Moltbook Social Network",
  "version": "1.0.0",
  "description": "Interact with Moltbook, the social network for AI agents",
  "author": "moltbook",
  "homepage": "https://www.moltbook.com",
  "source": {
    "type": "url",
    "url": "https://www.moltbook.com/skill.md",
    "last_fetched": "2026-01-31T12:00:00Z"
  },
  "files": [
    {"name": "skill.md", "url": "https://www.moltbook.com/skill.md"},
    {"name": "heartbeat.md", "url": "https://www.moltbook.com/heartbeat.md"},
    {"name": "messaging.md", "url": "https://www.moltbook.com/messaging.md"}
  ],
  "triggers": [
    "moltbook",
    "post to moltbook",
    "check moltbook",
    "social network"
  ],
  "requires": {
    "tools": ["http_call"],
    "env_vars": ["MOLTBOOK_API_KEY"]
  },
  "enabled": true
}
```

### 6.3 Skills Registry (registry.json)

```json
{
  "version": "1.0.0",
  "last_updated": "2026-01-31T12:00:00Z",
  "skills": [
    {
      "id": "moltbook",
      "path": "moltbook/",
      "enabled": true
    },
    {
      "id": "github",
      "path": "github/",
      "enabled": true
    },
    {
      "id": "web_research",
      "path": "web_research/",
      "enabled": false
    }
  ]
}
```

### 6.4 Skill Loader

```python
@dataclass
class Skill:
    """Loaded skill data."""
    id: str
    name: str
    description: str
    content: str  # Main skill.md content
    files: Dict[str, str]  # Additional files: name -> content
    triggers: List[str]
    enabled: bool
    metadata: Dict

class SkillLoader:
    """Load and manage skills."""

    def __init__(self, skills_dir: str):
        self.skills_dir = Path(skills_dir)
        self._skills: Dict[str, Skill] = {}
        self._registry: Dict = {}

    def load_registry(self) -> None:
        """Load the skills registry."""
        registry_path = self.skills_dir / "registry.json"
        if registry_path.exists():
            self._registry = json.loads(registry_path.read_text())

    def load_skill(self, skill_id: str) -> Optional[Skill]:
        """Load a single skill by ID."""
        skill_dir = self.skills_dir / skill_id
        skill_json_path = skill_dir / "skill.json"
        skill_md_path = skill_dir / "skill.md"

        if not skill_json_path.exists():
            return None

        # Load metadata
        metadata = json.loads(skill_json_path.read_text())

        # Load main skill.md
        content = ""
        if skill_md_path.exists():
            content = skill_md_path.read_text()

        # Load additional files
        files = {}
        for file_entry in metadata.get("files", []):
            file_name = file_entry["name"]
            file_path = skill_dir / file_name
            if file_path.exists():
                files[file_name] = file_path.read_text()

        skill = Skill(
            id=skill_id,
            name=metadata.get("name", skill_id),
            description=metadata.get("description", ""),
            content=content,
            files=files,
            triggers=metadata.get("triggers", []),
            enabled=metadata.get("enabled", True),
            metadata=metadata
        )

        self._skills[skill_id] = skill
        return skill

    def load_all(self) -> Dict[str, Skill]:
        """Load all enabled skills."""
        self.load_registry()

        for entry in self._registry.get("skills", []):
            if entry.get("enabled", True):
                self.load_skill(entry["id"])

        return self._skills

    def fetch_from_url(self, skill_id: str, url: str) -> Optional[Skill]:
        """Fetch a skill from a URL and save locally."""
        try:
            # Create skill directory
            skill_dir = self.skills_dir / skill_id
            skill_dir.mkdir(parents=True, exist_ok=True)

            # Fetch main skill.md
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            content = response.text

            # Save skill.md
            (skill_dir / "skill.md").write_text(content)

            # Create minimal skill.json
            metadata = {
                "id": skill_id,
                "name": skill_id,
                "description": f"Skill fetched from {url}",
                "source": {
                    "type": "url",
                    "url": url,
                    "last_fetched": datetime.now().isoformat()
                },
                "enabled": True
            }
            (skill_dir / "skill.json").write_text(json.dumps(metadata, indent=2))

            # Update registry
            self._add_to_registry(skill_id)

            return self.load_skill(skill_id)
        except Exception as e:
            print(f"Failed to fetch skill from {url}: {e}")
            return None

    def _add_to_registry(self, skill_id: str) -> None:
        """Add skill to registry if not present."""
        if "skills" not in self._registry:
            self._registry["skills"] = []

        existing_ids = [s["id"] for s in self._registry["skills"]]
        if skill_id not in existing_ids:
            self._registry["skills"].append({
                "id": skill_id,
                "path": f"{skill_id}/",
                "enabled": True
            })

        self._registry["last_updated"] = datetime.now().isoformat()

        registry_path = self.skills_dir / "registry.json"
        registry_path.write_text(json.dumps(self._registry, indent=2))

    def build_skills_prompt(self, skill_ids: List[str] = None) -> str:
        """Build the skills prompt for injection into system prompt."""
        skills = self._skills.values() if skill_ids is None else \
                 [self._skills[sid] for sid in skill_ids if sid in self._skills]

        if not skills:
            return ""

        lines = ["## Available Skills", ""]
        lines.append("Before responding, scan these skills. If one applies, read its file with file_read tool.")
        lines.append("")
        lines.append("<available_skills>")

        for skill in skills:
            if skill.enabled:
                skill_path = self.skills_dir / skill.id / "skill.md"
                lines.append(f'<skill id="{skill.id}" path="{skill_path}">')
                lines.append(f'  <description>{skill.description}</description>')
                lines.append(f'  <triggers>{", ".join(skill.triggers)}</triggers>')
                lines.append('</skill>')

        lines.append("</available_skills>")
        lines.append("")
        lines.append("Instructions:")
        lines.append("- If exactly one skill clearly applies: use file_read to read its skill.md, then follow it")
        lines.append("- If multiple could apply: choose the most specific one")
        lines.append("- If none apply: proceed without reading any skill file")

        return "\n".join(lines)
```

---

## 7. Memory System

### 7.1 Memory Structure

```
MEMORY/
├── topics.json                    # Index of all memory topics
├── sessions/
│   ├── session_abc123.json       # Session conversation history
│   └── session_def456.json
├── index_projects.json           # Index for "projects" topic
├── index_contacts.json           # Index for "contacts" topic
├── projects/
│   ├── content_001.json          # Project memory content
│   └── content_002.json
└── contacts/
    ├── content_001.json          # Contact memory content
    └── content_002.json
```

### 7.2 Topics Index (topics.json)

```json
{
  "version": "1.0.0",
  "last_updated": "2026-01-31T12:00:00Z",
  "topics": [
    {
      "id": "projects",
      "name": "Projects",
      "description": "Information about ongoing and past projects",
      "index_file": "index_projects.json",
      "content_dir": "projects/",
      "entry_count": 15
    },
    {
      "id": "contacts",
      "name": "Contacts",
      "description": "People, companies, and agents I interact with",
      "index_file": "index_contacts.json",
      "content_dir": "contacts/",
      "entry_count": 42
    }
  ]
}
```

### 7.3 Topic Index (index_<topic>.json)

```json
{
  "topic_id": "projects",
  "version": "1.0.0",
  "last_updated": "2026-01-31T12:00:00Z",
  "entries": [
    {
      "index_id": "idx_001",
      "content_file": "content_001.json",
      "section_id": "overview",
      "summary": "Agentic Loop Framework - Python library for building LLM agents",
      "keywords": ["agentic", "loop", "python", "llm", "framework"],
      "created_at": "2026-01-31T10:00:00Z",
      "relevance_score": 0.95
    },
    {
      "index_id": "idx_002",
      "content_file": "content_001.json",
      "section_id": "requirements",
      "summary": "Agentic Loop Framework - Technical requirements and constraints",
      "keywords": ["requirements", "constraints", "tools", "memory"],
      "created_at": "2026-01-31T10:05:00Z",
      "relevance_score": 0.85
    },
    {
      "index_id": "idx_003",
      "content_file": "content_002.json",
      "section_id": "main",
      "summary": "Moltbook integration project - social network for agents",
      "keywords": ["moltbook", "social", "agents", "integration"],
      "created_at": "2026-01-30T14:00:00Z",
      "relevance_score": 0.80
    }
  ]
}
```

### 7.4 Content File (content_<id>.json)

```json
{
  "content_id": "content_001",
  "topic_id": "projects",
  "title": "Agentic Loop Framework",
  "created_at": "2026-01-31T10:00:00Z",
  "updated_at": "2026-01-31T12:00:00Z",
  "metadata": {
    "status": "in_progress",
    "priority": "high",
    "tags": ["infrastructure", "python", "llm"]
  },
  "sections": [
    {
      "section_id": "overview",
      "title": "Overview",
      "content": "A Python library for building LLM-powered agents that can execute multi-step tasks autonomously using tools and skills.",
      "updated_at": "2026-01-31T10:00:00Z"
    },
    {
      "section_id": "requirements",
      "title": "Requirements",
      "content": "The framework should support: Claude and OpenAI, tool execution (http, file I/O), skill loading from markdown, session memory, and configurable safety limits.",
      "updated_at": "2026-01-31T10:05:00Z"
    },
    {
      "section_id": "progress",
      "title": "Progress",
      "content": "Specification document completed. Implementation pending.",
      "updated_at": "2026-01-31T12:00:00Z"
    }
  ]
}
```

### 7.5 Session File (session_<id>.json)

```json
{
  "session_id": "session_abc123",
  "agent_id": "main_agent",
  "created_at": "2026-01-31T10:00:00Z",
  "updated_at": "2026-01-31T12:00:00Z",
  "status": "active",
  "metadata": {
    "purpose": "Moltbook heartbeat check",
    "skill_used": "moltbook"
  },
  "conversation": [
    {
      "role": "system",
      "content": "You are an AI assistant with access to tools..."
    },
    {
      "role": "user",
      "content": "Check Moltbook and engage with the community",
      "timestamp": "2026-01-31T10:00:00Z"
    },
    {
      "role": "assistant",
      "content": "I'll check your Moltbook feed and engage with the community.",
      "tool_calls": [
        {
          "id": "call_001",
          "name": "file_read",
          "parameters": {"path": "SKILLS/moltbook/heartbeat.md"}
        }
      ],
      "timestamp": "2026-01-31T10:00:01Z"
    },
    {
      "role": "tool",
      "tool_call_id": "call_001",
      "content": "[heartbeat.md content...]",
      "timestamp": "2026-01-31T10:00:02Z"
    }
  ],
  "summary": null,
  "token_count": 4521
}
```

### 7.6 Memory Manager

```python
class MemoryManager:
    """Manage sessions and long-term memory."""

    def __init__(self, memory_dir: str):
        self.memory_dir = Path(memory_dir)
        self.sessions_dir = self.memory_dir / "sessions"
        self.sessions_dir.mkdir(parents=True, exist_ok=True)
        self._topics: Dict = {}

    # ==================== Sessions ====================

    def load_session(self, session_id: str) -> Optional[Dict]:
        """Load a session by ID."""
        session_path = self.sessions_dir / f"session_{session_id}.json"
        if session_path.exists():
            return json.loads(session_path.read_text())
        return None

    def save_session(self, session_id: str, session_data: Dict) -> None:
        """Save a session."""
        session_data["updated_at"] = datetime.now().isoformat()
        session_path = self.sessions_dir / f"session_{session_id}.json"
        session_path.write_text(json.dumps(session_data, indent=2))

    def create_session(self, session_id: str, agent_id: str,
                       metadata: Dict = None) -> Dict:
        """Create a new session."""
        session = {
            "session_id": session_id,
            "agent_id": agent_id,
            "created_at": datetime.now().isoformat(),
            "updated_at": datetime.now().isoformat(),
            "status": "active",
            "metadata": metadata or {},
            "conversation": [],
            "summary": None,
            "token_count": 0
        }
        self.save_session(session_id, session)
        return session

    def append_to_session(self, session_id: str, messages: List[Dict]) -> None:
        """Append messages to a session."""
        session = self.load_session(session_id)
        if session:
            session["conversation"].extend(messages)
            self.save_session(session_id, session)

    def list_sessions(self, agent_id: str = None) -> List[Dict]:
        """List all sessions, optionally filtered by agent."""
        sessions = []
        for path in self.sessions_dir.glob("session_*.json"):
            session = json.loads(path.read_text())
            if agent_id is None or session.get("agent_id") == agent_id:
                sessions.append({
                    "session_id": session["session_id"],
                    "agent_id": session.get("agent_id"),
                    "created_at": session.get("created_at"),
                    "status": session.get("status"),
                    "metadata": session.get("metadata", {})
                })
        return sorted(sessions, key=lambda x: x.get("created_at", ""), reverse=True)

    # ==================== Long-term Memory ====================

    def load_topics(self) -> Dict:
        """Load the topics index."""
        topics_path = self.memory_dir / "topics.json"
        if topics_path.exists():
            self._topics = json.loads(topics_path.read_text())
        else:
            self._topics = {"version": "1.0.0", "topics": []}
        return self._topics

    def get_topic_index(self, topic_id: str) -> Optional[Dict]:
        """Load a topic's index."""
        index_path = self.memory_dir / f"index_{topic_id}.json"
        if index_path.exists():
            return json.loads(index_path.read_text())
        return None

    def search_memory(self, query: str, topic_id: str = None,
                      limit: int = 10) -> List[Dict]:
        """
        Search memory indexes for relevant entries.
        Returns index entries that match keywords in the query.
        """
        results = []
        query_words = set(query.lower().split())

        topics = self.load_topics()
        search_topics = [t for t in topics.get("topics", [])
                        if topic_id is None or t["id"] == topic_id]

        for topic in search_topics:
            index = self.get_topic_index(topic["id"])
            if not index:
                continue

            for entry in index.get("entries", []):
                # Simple keyword matching
                entry_keywords = set(k.lower() for k in entry.get("keywords", []))
                entry_summary = entry.get("summary", "").lower()

                # Score based on keyword overlap and summary match
                keyword_overlap = len(query_words & entry_keywords)
                summary_matches = sum(1 for w in query_words if w in entry_summary)
                score = keyword_overlap * 2 + summary_matches

                if score > 0:
                    results.append({
                        "score": score,
                        "topic_id": topic["id"],
                        "entry": entry
                    })

        # Sort by score and limit
        results.sort(key=lambda x: x["score"], reverse=True)
        return results[:limit]

    def get_content(self, topic_id: str, content_id: str,
                    section_id: str = None) -> Optional[str]:
        """Get content from a specific content file, optionally a specific section."""
        content_path = self.memory_dir / topic_id / f"{content_id}.json"
        if not content_path.exists():
            return None

        content_data = json.loads(content_path.read_text())

        if section_id:
            for section in content_data.get("sections", []):
                if section["section_id"] == section_id:
                    return section["content"]
            return None
        else:
            # Return all sections concatenated
            sections = content_data.get("sections", [])
            return "\n\n".join(
                f"## {s['title']}\n{s['content']}" for s in sections
            )

    def add_memory(self, topic_id: str, title: str, content: str,
                   keywords: List[str] = None, metadata: Dict = None) -> str:
        """Add new content to memory with automatic indexing."""
        # Ensure topic exists
        self._ensure_topic(topic_id)

        # Create content file
        content_id = f"content_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
        content_dir = self.memory_dir / topic_id
        content_dir.mkdir(parents=True, exist_ok=True)

        content_data = {
            "content_id": content_id,
            "topic_id": topic_id,
            "title": title,
            "created_at": datetime.now().isoformat(),
            "updated_at": datetime.now().isoformat(),
            "metadata": metadata or {},
            "sections": [
                {
                    "section_id": "main",
                    "title": title,
                    "content": content,
                    "updated_at": datetime.now().isoformat()
                }
            ]
        }

        content_path = content_dir / f"{content_id}.json"
        content_path.write_text(json.dumps(content_data, indent=2))

        # Add to index
        self._add_to_index(
            topic_id=topic_id,
            content_file=f"{content_id}.json",
            section_id="main",
            summary=title,
            keywords=keywords or self._extract_keywords(content)
        )

        return content_id

    def _ensure_topic(self, topic_id: str) -> None:
        """Ensure a topic exists in the topics index."""
        topics = self.load_topics()
        existing_ids = [t["id"] for t in topics.get("topics", [])]

        if topic_id not in existing_ids:
            topics["topics"].append({
                "id": topic_id,
                "name": topic_id.replace("_", " ").title(),
                "description": f"Memory entries for {topic_id}",
                "index_file": f"index_{topic_id}.json",
                "content_dir": f"{topic_id}/",
                "entry_count": 0
            })
            topics["last_updated"] = datetime.now().isoformat()

            topics_path = self.memory_dir / "topics.json"
            topics_path.write_text(json.dumps(topics, indent=2))

    def _add_to_index(self, topic_id: str, content_file: str,
                      section_id: str, summary: str, keywords: List[str]) -> None:
        """Add an entry to a topic's index."""
        index_path = self.memory_dir / f"index_{topic_id}.json"

        if index_path.exists():
            index = json.loads(index_path.read_text())
        else:
            index = {
                "topic_id": topic_id,
                "version": "1.0.0",
                "last_updated": datetime.now().isoformat(),
                "entries": []
            }

        index_id = f"idx_{len(index['entries']) + 1:03d}"
        index["entries"].append({
            "index_id": index_id,
            "content_file": content_file,
            "section_id": section_id,
            "summary": summary,
            "keywords": keywords,
            "created_at": datetime.now().isoformat(),
            "relevance_score": 1.0
        })
        index["last_updated"] = datetime.now().isoformat()

        index_path.write_text(json.dumps(index, indent=2))

    def _extract_keywords(self, content: str, max_keywords: int = 10) -> List[str]:
        """Extract keywords from content (simple implementation)."""
        # Remove common words and extract significant terms
        stop_words = {'the', 'a', 'an', 'is', 'are', 'was', 'were', 'be', 'been',
                      'being', 'have', 'has', 'had', 'do', 'does', 'did', 'will',
                      'would', 'could', 'should', 'may', 'might', 'can', 'to', 'of',
                      'in', 'for', 'on', 'with', 'at', 'by', 'from', 'as', 'into',
                      'through', 'during', 'before', 'after', 'above', 'below',
                      'between', 'under', 'again', 'further', 'then', 'once', 'and',
                      'but', 'or', 'nor', 'so', 'yet', 'both', 'each', 'few', 'more',
                      'most', 'other', 'some', 'such', 'no', 'not', 'only', 'own',
                      'same', 'than', 'too', 'very', 'just', 'also', 'now', 'this',
                      'that', 'these', 'those', 'it', 'its'}

        words = re.findall(r'\b[a-zA-Z]{3,}\b', content.lower())
        word_counts = {}
        for word in words:
            if word not in stop_words:
                word_counts[word] = word_counts.get(word, 0) + 1

        sorted_words = sorted(word_counts.items(), key=lambda x: x[1], reverse=True)
        return [word for word, count in sorted_words[:max_keywords]]

    def build_memory_prompt(self, query: str, topic_id: str = None,
                            max_entries: int = 5) -> str:
        """Build a memory context prompt based on query relevance."""
        results = self.search_memory(query, topic_id, limit=max_entries)

        if not results:
            return ""

        lines = ["## Relevant Memory", ""]
        lines.append("The following information from memory may be relevant:")
        lines.append("")

        for result in results:
            entry = result["entry"]
            topic = result["topic_id"]
            lines.append(f"**[{topic}]** {entry['summary']}")
            lines.append(f"  - Keywords: {', '.join(entry.get('keywords', []))}")
            lines.append(f"  - Source: {entry['content_file']}#{entry['section_id']}")
            lines.append("")

        lines.append("Use file_read to access full content if needed.")

        return "\n".join(lines)
```

---

## 8. Session Management

### 8.1 Session Lifecycle

```
┌─────────────┐
│   CREATE    │  New session requested
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   ACTIVE    │  Conversation in progress
└──────┬──────┘
       │
       ├──────────────────┐
       │                  │
       ▼                  ▼
┌─────────────┐    ┌─────────────┐
│  COMPLETED  │    │   PAUSED    │  User interruption
└─────────────┘    └──────┬──────┘
                          │
                          ▼
                   ┌─────────────┐
                   │   RESUMED   │  Continue later
                   └──────┬──────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  COMPLETED  │
                   └─────────────┘
```

### 8.2 Session Context

```python
@dataclass
class SessionContext:
    """Full context for a session run."""
    session_id: str
    agent_id: str
    conversation_history: List[Dict]
    system_prompt: str
    skills_prompt: str
    memory_prompt: str
    available_tools: List[Dict]
    max_turns: int
    timeout_seconds: int

    @classmethod
    def from_session(cls, session: Dict, agent_config: 'AgentConfig',
                     skills_prompt: str, memory_prompt: str,
                     tool_schemas: List[Dict]) -> 'SessionContext':
        """Build context from session data and agent config."""
        return cls(
            session_id=session["session_id"],
            agent_id=session["agent_id"],
            conversation_history=session.get("conversation", []),
            system_prompt=agent_config.system_prompt,
            skills_prompt=skills_prompt,
            memory_prompt=memory_prompt,
            available_tools=tool_schemas,
            max_turns=agent_config.max_turns,
            timeout_seconds=agent_config.timeout_seconds
        )
```

---

## 9. Configuration

### 9.1 Global Configuration (config.json)

```json
{
  "version": "1.0.0",

  "paths": {
    "skills_dir": "./SKILLS",
    "memory_dir": "./MEMORY",
    "output_dir": "./OUTPUT",
    "config_dir": "./CONFIG"
  },

  "defaults": {
    "max_turns": 20,
    "timeout_seconds": 600,
    "llm_provider": "anthropic",
    "llm_model": "claude-sonnet-4-5-20250929"
  },

  "tools": {
    "file_read": {
      "enabled": true,
      "allowed_paths": ["./SKILLS", "./MEMORY", "./OUTPUT", "./SANDBOX"]
    },
    "file_write": {
      "enabled": true,
      "allowed_paths": ["./OUTPUT", "./SANDBOX", "./MEMORY"]
    },
    "http_call": {
      "enabled": true,
      "timeout_seconds": 30
    },
    "web_fetch": {
      "enabled": true,
      "max_length": 15000
    }
  },

  "human_escalation": {
    "enabled": true,
    "triggers": [
      "need human input",
      "escalate to human",
      "ask the user"
    ]
  },

  "logging": {
    "level": "INFO",
    "file": "./logs/agentic_loop.log",
    "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
  }
}
```

### 9.2 Agent Configuration

```json
{
  "agent_id": "main_agent",
  "name": "Main Assistant",
  "description": "General-purpose AI assistant",

  "llm": {
    "provider": "anthropic",
    "model": "claude-sonnet-4-5-20250929",
    "temperature": 0.0,
    "max_tokens": 4096
  },

  "system_prompt": "You are a helpful AI assistant with access to tools. You can read and write files, make HTTP requests, and follow skill instructions. Always explain what you're doing before using tools.",

  "limits": {
    "max_turns": 25,
    "timeout_seconds": 900
  },

  "tools": {
    "enabled": ["file_read", "file_write", "http_call", "web_fetch"],
    "disabled": []
  },

  "skills": {
    "auto_load": true,
    "enabled": ["moltbook", "github"],
    "disabled": []
  },

  "memory": {
    "session_persistence": true,
    "long_term_enabled": true,
    "auto_index": true
  },

  "sandbox": {
    "root_paths": ["./SANDBOX/main_agent"]
  }
}
```

### 9.3 Configuration Loader

```python
@dataclass
class AgentConfig:
    """Agent configuration."""
    agent_id: str
    name: str
    description: str
    llm_provider: str
    llm_model: str
    temperature: float
    max_tokens: int
    system_prompt: str
    max_turns: int
    timeout_seconds: int
    enabled_tools: List[str]
    enabled_skills: List[str]
    sandbox_paths: List[str]

    @classmethod
    def from_file(cls, config_path: str) -> 'AgentConfig':
        """Load agent config from JSON file."""
        config = json.loads(Path(config_path).read_text())
        return cls(
            agent_id=config["agent_id"],
            name=config.get("name", config["agent_id"]),
            description=config.get("description", ""),
            llm_provider=config.get("llm", {}).get("provider", "anthropic"),
            llm_model=config.get("llm", {}).get("model", "claude-sonnet-4-5-20250929"),
            temperature=config.get("llm", {}).get("temperature", 0.0),
            max_tokens=config.get("llm", {}).get("max_tokens", 4096),
            system_prompt=config.get("system_prompt", "You are a helpful assistant."),
            max_turns=config.get("limits", {}).get("max_turns", 20),
            timeout_seconds=config.get("limits", {}).get("timeout_seconds", 600),
            enabled_tools=config.get("tools", {}).get("enabled", []),
            enabled_skills=config.get("skills", {}).get("enabled", []),
            sandbox_paths=config.get("sandbox", {}).get("root_paths", [])
        )

class ConfigManager:
    """Manage global and agent configurations."""

    def __init__(self, config_dir: str):
        self.config_dir = Path(config_dir)
        self.global_config: Dict = {}
        self.agent_configs: Dict[str, AgentConfig] = {}

    def load_global(self) -> Dict:
        """Load global configuration."""
        global_path = self.config_dir / "config.json"
        if global_path.exists():
            self.global_config = json.loads(global_path.read_text())
        return self.global_config

    def load_agent(self, agent_id: str) -> AgentConfig:
        """Load agent configuration."""
        agent_path = self.config_dir / "agents" / f"{agent_id}.json"
        if not agent_path.exists():
            raise ValueError(f"Agent config not found: {agent_id}")

        config = AgentConfig.from_file(str(agent_path))
        self.agent_configs[agent_id] = config
        return config

    def list_agents(self) -> List[str]:
        """List all configured agents."""
        agents_dir = self.config_dir / "agents"
        if not agents_dir.exists():
            return []
        return [p.stem for p in agents_dir.glob("*.json")]
```

---

## 10. API & CLI Interface

### 10.1 FastAPI Application

```python
from fastapi import FastAPI, HTTPException, BackgroundTasks
from pydantic import BaseModel
from typing import Optional, List

app = FastAPI(title="Agentic Loop API", version="1.0.0")

# ==================== Request/Response Models ====================

class RunAgentRequest(BaseModel):
    agent_id: str
    message: str
    session_id: Optional[str] = None
    skills: Optional[List[str]] = None

class RunAgentResponse(BaseModel):
    session_id: str
    status: str
    response: Optional[str]
    turns: int
    tools_used: List[str]
    duration_ms: int
    error: Optional[str]

class AgentStatusResponse(BaseModel):
    agent_id: str
    name: str
    description: str
    active_sessions: int
    total_runs: int

class SessionListResponse(BaseModel):
    sessions: List[dict]

# ==================== Endpoints ====================

@app.post("/agents/{agent_id}/run", response_model=RunAgentResponse)
async def run_agent(agent_id: str, request: RunAgentRequest):
    """Run an agent with a message."""
    try:
        manager = get_agent_manager()
        result = manager.run_agent(
            agent_id=agent_id,
            message=request.message,
            session_id=request.session_id,
            skills=request.skills
        )
        return RunAgentResponse(
            session_id=result.session_id,
            status=result.status,
            response=result.final_response,
            turns=len(result.turns),
            tools_used=result.tools_called,
            duration_ms=result.total_duration_ms,
            error=result.error
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/agents/{agent_id}/status", response_model=AgentStatusResponse)
async def get_agent_status(agent_id: str):
    """Get agent status."""
    manager = get_agent_manager()
    agent = manager.get_agent(agent_id)
    if not agent:
        raise HTTPException(status_code=404, detail="Agent not found")

    return AgentStatusResponse(
        agent_id=agent.agent_id,
        name=agent.config.name,
        description=agent.config.description,
        active_sessions=agent.active_session_count,
        total_runs=agent.total_runs
    )

@app.get("/agents/{agent_id}/sessions", response_model=SessionListResponse)
async def list_sessions(agent_id: str):
    """List all sessions for an agent."""
    manager = get_agent_manager()
    sessions = manager.memory.list_sessions(agent_id=agent_id)
    return SessionListResponse(sessions=sessions)

@app.get("/agents/{agent_id}/sessions/{session_id}")
async def get_session(agent_id: str, session_id: str):
    """Get session details."""
    manager = get_agent_manager()
    session = manager.memory.load_session(session_id)
    if not session or session.get("agent_id") != agent_id:
        raise HTTPException(status_code=404, detail="Session not found")
    return session

@app.get("/skills")
async def list_skills():
    """List all available skills."""
    manager = get_agent_manager()
    skills = manager.skill_loader.load_all()
    return {
        "skills": [
            {
                "id": s.id,
                "name": s.name,
                "description": s.description,
                "enabled": s.enabled
            }
            for s in skills.values()
        ]
    }

@app.post("/skills/fetch")
async def fetch_skill(skill_id: str, url: str):
    """Fetch a skill from a URL."""
    manager = get_agent_manager()
    skill = manager.skill_loader.fetch_from_url(skill_id, url)
    if not skill:
        raise HTTPException(status_code=400, detail="Failed to fetch skill")
    return {"skill_id": skill.id, "name": skill.name}

@app.get("/health")
async def health_check():
    """Health check endpoint."""
    return {"status": "healthy", "version": "1.0.0"}
```

### 10.2 CLI Interface

```python
import argparse
import json
import sys

def main():
    parser = argparse.ArgumentParser(description="Agentic Loop CLI")
    subparsers = parser.add_subparsers(dest="command", help="Commands")

    # Run command
    run_parser = subparsers.add_parser("run", help="Run an agent")
    run_parser.add_argument("agent_id", help="Agent ID")
    run_parser.add_argument("message", help="Message to send")
    run_parser.add_argument("--session", "-s", help="Session ID (optional)")
    run_parser.add_argument("--skills", nargs="+", help="Skills to enable")
    run_parser.add_argument("--verbose", "-v", action="store_true")

    # List agents
    list_parser = subparsers.add_parser("list-agents", help="List agents")

    # List sessions
    sessions_parser = subparsers.add_parser("sessions", help="List sessions")
    sessions_parser.add_argument("agent_id", help="Agent ID")

    # List skills
    skills_parser = subparsers.add_parser("skills", help="List skills")

    # Fetch skill
    fetch_parser = subparsers.add_parser("fetch-skill", help="Fetch skill from URL")
    fetch_parser.add_argument("skill_id", help="Skill ID to create")
    fetch_parser.add_argument("url", help="URL to fetch from")

    args = parser.parse_args()

    if args.command == "run":
        result = run_agent_cli(
            agent_id=args.agent_id,
            message=args.message,
            session_id=args.session,
            skills=args.skills,
            verbose=args.verbose
        )
        print(json.dumps(result, indent=2))

    elif args.command == "list-agents":
        agents = list_agents_cli()
        for agent in agents:
            print(f"  {agent['id']}: {agent['name']}")

    elif args.command == "sessions":
        sessions = list_sessions_cli(args.agent_id)
        for session in sessions:
            print(f"  {session['session_id']}: {session['status']} ({session['created_at']})")

    elif args.command == "skills":
        skills = list_skills_cli()
        for skill in skills:
            status = "✓" if skill['enabled'] else "✗"
            print(f"  [{status}] {skill['id']}: {skill['description']}")

    elif args.command == "fetch-skill":
        result = fetch_skill_cli(args.skill_id, args.url)
        print(f"Fetched skill: {result['skill_id']}")

    else:
        parser.print_help()

if __name__ == "__main__":
    main()
```

---

## 11. Error Handling

### 11.1 Error Types

```python
class AgenticLoopError(Exception):
    """Base exception for agentic loop errors."""
    pass

class ToolExecutionError(AgenticLoopError):
    """Error during tool execution."""
    def __init__(self, tool_name: str, message: str):
        self.tool_name = tool_name
        super().__init__(f"Tool '{tool_name}' failed: {message}")

class LoopTimeoutError(AgenticLoopError):
    """Loop exceeded timeout."""
    def __init__(self, timeout_seconds: int, elapsed_seconds: float):
        self.timeout_seconds = timeout_seconds
        self.elapsed_seconds = elapsed_seconds
        super().__init__(f"Loop timeout: {elapsed_seconds:.1f}s > {timeout_seconds}s")

class MaxTurnsError(AgenticLoopError):
    """Loop exceeded maximum turns."""
    def __init__(self, max_turns: int):
        self.max_turns = max_turns
        super().__init__(f"Exceeded maximum turns: {max_turns}")

class SkillLoadError(AgenticLoopError):
    """Error loading a skill."""
    def __init__(self, skill_id: str, message: str):
        self.skill_id = skill_id
        super().__init__(f"Failed to load skill '{skill_id}': {message}")

class HumanEscalationRequired(AgenticLoopError):
    """Agent requires human input."""
    def __init__(self, reason: str, context: str):
        self.reason = reason
        self.context = context
        super().__init__(f"Human input required: {reason}")
```

### 11.2 Error Handling Strategy

```python
def execute_with_error_handling(func, *args, **kwargs):
    """Execute function with standardized error handling."""
    try:
        return func(*args, **kwargs)
    except HumanEscalationRequired as e:
        # Don't catch - let it propagate for human handling
        raise
    except LoopTimeoutError as e:
        logger.warning(f"Loop timeout: {e}")
        return LoopResult(
            status="timeout",
            turns=[],
            final_response=None,
            error=str(e)
        )
    except MaxTurnsError as e:
        logger.warning(f"Max turns reached: {e}")
        return LoopResult(
            status="max_turns",
            turns=[],
            final_response=None,
            error=str(e)
        )
    except ToolExecutionError as e:
        logger.error(f"Tool error: {e}")
        # Continue loop - tool errors are recoverable
        return ToolResult(
            success=False,
            output="",
            error=str(e)
        )
    except Exception as e:
        logger.exception(f"Unexpected error: {e}")
        return LoopResult(
            status="error",
            turns=[],
            final_response=None,
            error=str(e)
        )
```

---

## 12. File Structure

### 12.1 Complete Directory Layout

```
project_root/
├── CONFIG/
│   ├── config.json                 # Global configuration
│   └── agents/
│       ├── main_agent.json         # Agent configurations
│       └── research_agent.json
│
├── SKILLS/
│   ├── registry.json               # Skills registry
│   ├── moltbook/
│   │   ├── skill.json
│   │   ├── skill.md
│   │   ├── heartbeat.md
│   │   └── messaging.md
│   └── github/
│       ├── skill.json
│       └── skill.md
│
├── MEMORY/
│   ├── topics.json                 # Topics index
│   ├── sessions/
│   │   ├── session_abc123.json
│   │   └── session_def456.json
│   ├── index_projects.json         # Topic indexes
│   ├── index_contacts.json
│   ├── projects/                   # Topic content
│   │   └── content_001.json
│   └── contacts/
│       └── content_001.json
│
├── OUTPUT/
│   └── main_agent/
│       └── 2026-01-31/
│           ├── run_001/
│           │   ├── result.json
│           │   └── transcript.md
│           └── run_002/
│               ├── result.json
│               └── transcript.md
│
├── SANDBOX/
│   └── main_agent/                 # Per-agent sandbox
│       └── workspace/
│
├── src/
│   └── agentic_loop/
│       ├── __init__.py
│       ├── loop.py                 # AgenticLoop class
│       ├── agent.py                # Agent class
│       ├── agent_manager.py        # AgentManager class
│       ├── tools/
│       │   ├── __init__.py
│       │   ├── base.py             # BaseTool, ToolRegistry
│       │   ├── http_call.py
│       │   ├── web_fetch.py
│       │   ├── file_read.py
│       │   └── file_write.py
│       ├── skills/
│       │   ├── __init__.py
│       │   └── loader.py           # SkillLoader
│       ├── memory/
│       │   ├── __init__.py
│       │   └── manager.py          # MemoryManager
│       ├── config/
│       │   ├── __init__.py
│       │   └── loader.py           # ConfigManager
│       ├── context/
│       │   ├── __init__.py
│       │   └── builder.py          # ContextBuilder
│       ├── output/
│       │   ├── __init__.py
│       │   └── manager.py          # OutputManager
│       ├── api/
│       │   ├── __init__.py
│       │   └── app.py              # FastAPI app
│       ├── cli/
│       │   ├── __init__.py
│       │   └── main.py             # CLI interface
│       └── errors.py               # Exception classes
│
├── tests/
│   ├── test_loop.py
│   ├── test_tools.py
│   ├── test_skills.py
│   └── test_memory.py
│
├── requirements.txt
├── setup.py
└── README.md
```

---

## 13. Implementation Guide

### 13.1 Implementation Order

```
Phase 1: Core Infrastructure
├── 1.1 Configuration system (ConfigManager)
├── 1.2 Base tool interface (BaseTool, ToolRegistry)
├── 1.3 File tools (file_read, file_write)
└── 1.4 Basic agentic loop (no skills/memory)

Phase 2: Tool Expansion
├── 2.1 HTTP call tool
├── 2.2 Web fetch tool
└── 2.3 Tool testing suite

Phase 3: Skills System
├── 3.1 Skill loader
├── 3.2 Skills registry
├── 3.3 URL fetching for skills
└── 3.4 Skills prompt injection

Phase 4: Memory System
├── 4.1 Session management
├── 4.2 Long-term memory structure
├── 4.3 Index-based search
└── 4.4 Memory prompt injection

Phase 5: Agent Management
├── 5.1 Agent class
├── 5.2 Agent manager
├── 5.3 Multi-agent support
└── 5.4 Output management

Phase 6: Interfaces
├── 6.1 CLI interface
├── 6.2 FastAPI application
└── 6.3 Admin panel (index.html)

Phase 7: Testing & Documentation
├── 7.1 Unit tests
├── 7.2 Integration tests
├── 7.3 Documentation
└── 7.4 Examples
```

### 13.2 LLM Client Integration

Your existing `BaseLLMClient` interface needs to be extended for tool calling:

```python
class BaseLLMClient:
    """Extended interface for tool calling."""

    def complete_with_tools(
        self,
        messages: List[Dict],
        tools: List[Dict],
        caller: str = "unknown",
        max_tokens: int = 4096,
        temperature: float = 0.0
    ) -> LLMResponse:
        """
        Send completion with tool definitions.

        Returns LLMResponse with:
        - text: str (assistant message)
        - tool_calls: List[ToolCall] (if model wants to use tools)
        - usage: TokenUsage
        """
        pass

@dataclass
class ToolCall:
    """A tool call requested by the LLM."""
    id: str
    name: str
    parameters: Dict

@dataclass
class LLMResponse:
    """Response from LLM."""
    text: str
    tool_calls: List[ToolCall]
    usage: TokenUsage
```

### 13.3 Testing Strategy

```python
# Example test for the agentic loop
def test_agentic_loop_simple_completion():
    """Test loop completes without tools."""
    loop = AgenticLoop(
        llm_client=MockLLMClient(responses=["Hello! How can I help?"]),
        tool_registry=ToolRegistry(),
        max_turns=10
    )

    result = loop.execute("Say hello", context=minimal_context())

    assert result.status == "completed"
    assert result.final_response == "Hello! How can I help?"
    assert len(result.turns) == 1
    assert result.tools_called == []

def test_agentic_loop_with_tool_calls():
    """Test loop handles tool calls correctly."""
    mock_client = MockLLMClient(responses=[
        LLMResponse(text="Let me read that file.", tool_calls=[
            ToolCall(id="1", name="file_read", parameters={"path": "test.txt"})
        ]),
        LLMResponse(text="The file contains: Hello World", tool_calls=[])
    ])

    tool_registry = ToolRegistry()
    tool_registry.register(MockFileReadTool(content="Hello World"))

    loop = AgenticLoop(
        llm_client=mock_client,
        tool_registry=tool_registry,
        max_turns=10
    )

    result = loop.execute("Read test.txt", context=minimal_context())

    assert result.status == "completed"
    assert "Hello World" in result.final_response
    assert len(result.turns) == 2
    assert "file_read" in result.tools_called
```

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Agentic Loop** | The core execution cycle: prompt → LLM → tool → result → repeat |
| **Turn** | One iteration of the loop (LLM call + optional tool execution) |
| **Tool** | A capability the LLM can invoke (file read, HTTP call, etc.) |
| **Skill** | A set of instructions in markdown that teaches the agent a behavior |
| **Session** | A conversation context that persists across interactions |
| **Context Window** | The total amount of text the LLM can process at once |
| **Compaction** | Summarizing old conversation to fit context limits |
| **Human Escalation** | When the agent requests human input for a decision |

---

## Appendix B: Security Considerations

1. **Path Sandboxing**: All file operations must be restricted to allowed directories
2. **URL Allowlisting**: Consider restricting HTTP calls to approved domains
3. **Token Limits**: Enforce per-run and per-agent token budgets
4. **Rate Limiting**: Implement rate limits on API endpoints
5. **Input Validation**: Sanitize all user inputs before processing
6. **Credential Management**: Never log API keys or sensitive data

---

## Appendix C: Performance Considerations

1. **Caching**: Cache skill files and memory indexes
2. **Lazy Loading**: Load skills/memory on demand, not at startup
3. **Connection Pooling**: Reuse HTTP connections for API calls
4. **Async I/O**: Consider async for I/O-bound operations (future enhancement)
5. **Context Pruning**: Aggressively compact old context to reduce token usage

---

*End of Specification Document*
