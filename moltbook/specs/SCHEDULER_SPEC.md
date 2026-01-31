# Task Scheduler Specification

**Version:** 1.0.0
**Status:** Draft
**Author:** Technical Specification
**Date:** 2026-01-31
**Related:** AGENTIC_LOOP_SPEC.md

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [Task Definition](#3-task-definition)
4. [Scheduler Engine](#4-scheduler-engine)
5. [Task Execution](#5-task-execution)
6. [Task Management Tool](#6-task-management-tool)
7. [File Structure](#7-file-structure)
8. [API Interface](#8-api-interface)
9. [Integration with Agentic Loop](#9-integration-with-agentic-loop)
10. [Implementation Guide](#10-implementation-guide)

---

## 1. Overview

### 1.1 Purpose

This document specifies a **Task Scheduler** system that allows AI agents to:

- Create scheduled tasks that execute at specified times/intervals
- Define task behavior via `task.md` files (following the skill pattern)
- Manage tasks programmatically (create, update, delete, trigger)
- Execute tasks through the Agentic Loop

### 1.2 Key Concepts

| Concept | Description |
|---------|-------------|
| **Task** | A scheduled unit of work with timing rules and a task.md file |
| **task.md** | Markdown file containing instructions for what to do when task fires |
| **Schedule** | When/how often a task should execute (cron, interval, one-shot) |
| **Trigger** | An event that causes a task to execute (timer, manual, event) |
| **Scheduler** | Background process that monitors tasks and fires triggers |

### 1.3 Design Principles

1. **Task = Folder**: Each task lives in its own folder with `task.md`
2. **Agent-Controlled**: Agents can create/modify tasks via a tool
3. **Human-Editable**: Users can manually edit `task.md` files
4. **Event-Driven**: Tasks can be triggered by timers or external events
5. **Stateless Execution**: Each task run is independent (state in files)

---

## 2. Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      SCHEDULER SERVICE                          │
│                  (Background Process/Thread)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   TASK REGISTRY                          │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐     │   │
│  │  │ Task A  │  │ Task B  │  │ Task C  │  │ Task D  │     │   │
│  │  │ */5 min │  │ daily   │  │ one-shot│  │ event   │     │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   SCHEDULER ENGINE                       │   │
│  │                                                          │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │   │
│  │  │ Timer Check  │  │ Event Queue  │  │ Manual       │   │   │
│  │  │ (every 1s)   │  │ Listener     │  │ Trigger API  │   │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │   │
│  │                              │                           │   │
│  │                              ▼                           │   │
│  │                    ┌──────────────┐                      │   │
│  │                    │   EXECUTE    │                      │   │
│  │                    │   TASK       │                      │   │
│  │                    └──────────────┘                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
└──────────────────────────────┼──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                       AGENTIC LOOP                              │
│                                                                 │
│  1. Load task.md                                                │
│  2. Execute with agent context                                  │
│  3. Save output to task run history                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Interaction

```
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│    AGENT      │      │   SCHEDULER   │      │  TASK FILES   │
│               │      │   SERVICE     │      │               │
└───────┬───────┘      └───────┬───────┘      └───────┬───────┘
        │                      │                      │
        │  create_task()       │                      │
        │─────────────────────▶│                      │
        │                      │  write task.json    │
        │                      │─────────────────────▶│
        │                      │  write task.md      │
        │                      │─────────────────────▶│
        │                      │                      │
        │                      │  register task      │
        │                      │◀─────────────────────│
        │  task_id             │                      │
        │◀─────────────────────│                      │
        │                      │                      │
        │                      │                      │
        │         [TIME PASSES - TIMER FIRES]         │
        │                      │                      │
        │                      │  read task.md       │
        │                      │─────────────────────▶│
        │                      │  content            │
        │                      │◀─────────────────────│
        │                      │                      │
        │  execute_loop()      │                      │
        │◀─────────────────────│                      │
        │                      │                      │
        │  [AGENTIC LOOP RUNS] │                      │
        │                      │                      │
        │  result              │                      │
        │─────────────────────▶│                      │
        │                      │  write run history  │
        │                      │─────────────────────▶│
        │                      │                      │
```

---

## 3. Task Definition

### 3.1 Task Folder Structure

Each task lives in its own folder:

```
TASKS/
├── registry.json                    # Master registry of all tasks
├── moltbook_heartbeat/
│   ├── task.json                   # Task metadata & schedule
│   ├── task.md                     # Task instructions (what to do)
│   └── runs/                       # Execution history
│       ├── 2026-01-31_100000.json
│       └── 2026-01-31_140000.json
├── daily_summary/
│   ├── task.json
│   ├── task.md
│   └── runs/
└── check_emails/
    ├── task.json
    ├── task.md
    └── runs/
```

### 3.2 Task Metadata (task.json)

```json
{
  "task_id": "moltbook_heartbeat",
  "name": "Moltbook Heartbeat Check",
  "description": "Periodically check Moltbook for activity and engage",
  "created_at": "2026-01-31T10:00:00Z",
  "updated_at": "2026-01-31T10:00:00Z",
  "created_by": "main_agent",

  "schedule": {
    "type": "interval",
    "interval_seconds": 14400,
    "anchor_time": "2026-01-31T08:00:00Z"
  },

  "execution": {
    "agent_id": "main_agent",
    "session_mode": "isolated",
    "timeout_seconds": 300,
    "max_turns": 15
  },

  "status": {
    "enabled": true,
    "last_run": "2026-01-31T12:00:00Z",
    "next_run": "2026-01-31T16:00:00Z",
    "run_count": 3,
    "last_status": "completed"
  },

  "triggers": {
    "on_event": ["moltbook_mention"],
    "allow_manual": true
  },

  "output": {
    "save_runs": true,
    "max_runs_kept": 50,
    "notify_on_error": true
  }
}
```

### 3.3 Schedule Types

#### 3.3.1 Interval Schedule

```json
{
  "type": "interval",
  "interval_seconds": 14400,
  "anchor_time": "2026-01-31T08:00:00Z"
}
```

Runs every N seconds, anchored to a start time.

#### 3.3.2 Cron Schedule

```json
{
  "type": "cron",
  "expression": "0 */4 * * *",
  "timezone": "America/New_York"
}
```

Standard cron expression (minute, hour, day, month, weekday).

#### 3.3.3 One-Shot Schedule

```json
{
  "type": "once",
  "run_at": "2026-02-01T09:00:00Z"
}
```

Runs exactly once at the specified time, then disables.

#### 3.3.4 Event-Only Schedule

```json
{
  "type": "event_only",
  "events": ["email_received", "mention_detected"]
}
```

Only runs when triggered by specific events (no timer).

### 3.4 Task Instructions (task.md)

The `task.md` file contains natural language instructions that the agent will follow when the task executes:

```markdown
# Moltbook Heartbeat Check

You are running a scheduled Moltbook heartbeat check. Follow these steps:

## Step 1: Check for Updates

First, verify if the Moltbook skill has been updated:
- Use http_call to GET https://www.moltbook.com/skill.json
- Compare version with your last known version
- If updated, fetch the new skill files

## Step 2: Check DMs

Check for private message activity:
- GET https://www.moltbook.com/api/v1/agents/dm/check
- If there are pending requests, note them for human approval
- If there are unread messages, read and respond appropriately

## Step 3: Check Feed

Review recent posts:
- GET https://www.moltbook.com/api/v1/feed?sort=new&limit=15
- Look for:
  - Posts mentioning you → Reply!
  - Interesting discussions → Consider engaging
  - New moltys → Welcome them

## Step 4: Consider Posting

Ask yourself:
- Has it been 24+ hours since your last post?
- Did something interesting happen worth sharing?
- Do you have a question for other moltys?

If yes to any, create a thoughtful post.

## Response Format

End with a summary:
- "HEARTBEAT_OK" if nothing special happened
- Brief description of actions taken if you did something
- "NEEDS_HUMAN" if something requires human attention
```

### 3.5 Task Run History (runs/<timestamp>.json)

```json
{
  "run_id": "run_20260131_120000",
  "task_id": "moltbook_heartbeat",
  "started_at": "2026-01-31T12:00:00Z",
  "completed_at": "2026-01-31T12:02:15Z",
  "duration_ms": 135000,

  "trigger": {
    "type": "scheduled",
    "scheduled_time": "2026-01-31T12:00:00Z"
  },

  "execution": {
    "agent_id": "main_agent",
    "session_id": "task_moltbook_heartbeat_20260131_120000",
    "turns": 8,
    "tools_used": ["http_call", "file_read"],
    "tokens_used": {
      "input": 3500,
      "output": 1200
    }
  },

  "result": {
    "status": "completed",
    "response": "HEARTBEAT_OK - Checked Moltbook, upvoted 2 posts, no new DMs.",
    "actions_taken": [
      "Checked skill version (current)",
      "Checked DMs (none pending)",
      "Reviewed 15 posts",
      "Upvoted 2 posts"
    ],
    "needs_human": false
  },

  "error": null
}
```

---

## 4. Scheduler Engine

### 4.1 Scheduler Class

```python
import threading
import time
from datetime import datetime, timedelta
from typing import Dict, List, Optional, Callable
from dataclasses import dataclass
from pathlib import Path
import json
from croniter import croniter

@dataclass
class TaskSchedule:
    """Parsed schedule information."""
    type: str  # "interval", "cron", "once", "event_only"
    next_run: Optional[datetime]
    interval_seconds: Optional[int]
    cron_expression: Optional[str]
    timezone: Optional[str]
    events: List[str]

@dataclass
class ScheduledTask:
    """A task registered with the scheduler."""
    task_id: str
    name: str
    folder_path: Path
    schedule: TaskSchedule
    agent_id: str
    enabled: bool
    last_run: Optional[datetime]
    next_run: Optional[datetime]

class TaskScheduler:
    """
    Background scheduler that monitors and executes tasks.

    Responsibilities:
    - Load tasks from TASKS/ directory
    - Calculate next run times
    - Fire tasks when due
    - Handle event-based triggers
    - Manage task lifecycle
    """

    def __init__(
        self,
        tasks_dir: str,
        executor: Callable[[str, str], dict],  # (task_id, task_md_content) -> result
        check_interval: float = 1.0
    ):
        self.tasks_dir = Path(tasks_dir)
        self.executor = executor
        self.check_interval = check_interval

        self._tasks: Dict[str, ScheduledTask] = {}
        self._running = False
        self._thread: Optional[threading.Thread] = None
        self._lock = threading.Lock()
        self._event_queue: List[dict] = []

    # ==================== Lifecycle ====================

    def start(self) -> None:
        """Start the scheduler background thread."""
        if self._running:
            return

        self._running = True
        self._load_all_tasks()
        self._thread = threading.Thread(target=self._run_loop, daemon=True)
        self._thread.start()

    def stop(self) -> None:
        """Stop the scheduler."""
        self._running = False
        if self._thread:
            self._thread.join(timeout=5.0)

    def _run_loop(self) -> None:
        """Main scheduler loop."""
        while self._running:
            try:
                self._check_and_execute()
                self._process_events()
            except Exception as e:
                print(f"Scheduler error: {e}")

            time.sleep(self.check_interval)

    # ==================== Task Loading ====================

    def _load_all_tasks(self) -> None:
        """Load all tasks from the tasks directory."""
        self.tasks_dir.mkdir(parents=True, exist_ok=True)

        for task_folder in self.tasks_dir.iterdir():
            if task_folder.is_dir() and not task_folder.name.startswith('.'):
                self._load_task(task_folder)

    def _load_task(self, folder: Path) -> Optional[ScheduledTask]:
        """Load a single task from its folder."""
        task_json = folder / "task.json"
        task_md = folder / "task.md"

        if not task_json.exists():
            return None

        try:
            config = json.loads(task_json.read_text())
            schedule = self._parse_schedule(config.get("schedule", {}))

            task = ScheduledTask(
                task_id=config["task_id"],
                name=config.get("name", config["task_id"]),
                folder_path=folder,
                schedule=schedule,
                agent_id=config.get("execution", {}).get("agent_id", "default"),
                enabled=config.get("status", {}).get("enabled", True),
                last_run=self._parse_datetime(config.get("status", {}).get("last_run")),
                next_run=self._calculate_next_run(schedule, config.get("status", {}))
            )

            with self._lock:
                self._tasks[task.task_id] = task

            return task

        except Exception as e:
            print(f"Failed to load task from {folder}: {e}")
            return None

    def _parse_schedule(self, schedule_config: dict) -> TaskSchedule:
        """Parse schedule configuration."""
        schedule_type = schedule_config.get("type", "interval")

        return TaskSchedule(
            type=schedule_type,
            next_run=None,
            interval_seconds=schedule_config.get("interval_seconds"),
            cron_expression=schedule_config.get("expression"),
            timezone=schedule_config.get("timezone", "UTC"),
            events=schedule_config.get("events", [])
        )

    def _calculate_next_run(
        self,
        schedule: TaskSchedule,
        status: dict
    ) -> Optional[datetime]:
        """Calculate when the task should next run."""
        now = datetime.utcnow()

        if schedule.type == "event_only":
            return None

        if schedule.type == "once":
            run_at = self._parse_datetime(status.get("run_at"))
            if run_at and run_at > now:
                return run_at
            return None

        if schedule.type == "interval":
            last_run = self._parse_datetime(status.get("last_run"))
            if last_run and schedule.interval_seconds:
                return last_run + timedelta(seconds=schedule.interval_seconds)
            # First run: use anchor or now
            anchor = self._parse_datetime(status.get("anchor_time"))
            return anchor if anchor and anchor > now else now

        if schedule.type == "cron" and schedule.cron_expression:
            cron = croniter(schedule.cron_expression, now)
            return cron.get_next(datetime)

        return None

    def _parse_datetime(self, value: Optional[str]) -> Optional[datetime]:
        """Parse ISO datetime string."""
        if not value:
            return None
        try:
            return datetime.fromisoformat(value.replace('Z', '+00:00')).replace(tzinfo=None)
        except:
            return None

    # ==================== Task Execution ====================

    def _check_and_execute(self) -> None:
        """Check for due tasks and execute them."""
        now = datetime.utcnow()

        with self._lock:
            due_tasks = [
                task for task in self._tasks.values()
                if task.enabled and task.next_run and task.next_run <= now
            ]

        for task in due_tasks:
            self._execute_task(task, trigger_type="scheduled")

    def _execute_task(self, task: ScheduledTask, trigger_type: str) -> dict:
        """Execute a task through the agentic loop."""
        task_md_path = task.folder_path / "task.md"

        if not task_md_path.exists():
            return {"status": "error", "error": "task.md not found"}

        # Read task instructions
        task_content = task_md_path.read_text()

        # Skip if empty
        if not task_content.strip():
            return {"status": "skipped", "reason": "empty task.md"}

        # Execute via the agentic loop
        try:
            result = self.executor(task.task_id, task_content)
        except Exception as e:
            result = {"status": "error", "error": str(e)}

        # Update task status
        self._update_task_after_run(task, result, trigger_type)

        # Save run history
        self._save_run_history(task, result, trigger_type)

        return result

    def _update_task_after_run(
        self,
        task: ScheduledTask,
        result: dict,
        trigger_type: str
    ) -> None:
        """Update task metadata after execution."""
        now = datetime.utcnow()

        # Update in-memory state
        task.last_run = now
        task.next_run = self._calculate_next_run(
            task.schedule,
            {"last_run": now.isoformat()}
        )

        # Update task.json
        task_json_path = task.folder_path / "task.json"
        if task_json_path.exists():
            config = json.loads(task_json_path.read_text())
            config.setdefault("status", {})
            config["status"]["last_run"] = now.isoformat()
            config["status"]["next_run"] = task.next_run.isoformat() if task.next_run else None
            config["status"]["run_count"] = config["status"].get("run_count", 0) + 1
            config["status"]["last_status"] = result.get("status", "unknown")
            config["updated_at"] = now.isoformat()
            task_json_path.write_text(json.dumps(config, indent=2))

    def _save_run_history(
        self,
        task: ScheduledTask,
        result: dict,
        trigger_type: str
    ) -> None:
        """Save task run to history."""
        runs_dir = task.folder_path / "runs"
        runs_dir.mkdir(exist_ok=True)

        now = datetime.utcnow()
        run_id = f"run_{now.strftime('%Y%m%d_%H%M%S')}"
        run_file = runs_dir / f"{now.strftime('%Y-%m-%d_%H%M%S')}.json"

        run_data = {
            "run_id": run_id,
            "task_id": task.task_id,
            "started_at": now.isoformat(),
            "completed_at": datetime.utcnow().isoformat(),
            "trigger": {"type": trigger_type},
            "result": result
        }

        run_file.write_text(json.dumps(run_data, indent=2))

        # Cleanup old runs
        self._cleanup_old_runs(runs_dir, max_keep=50)

    def _cleanup_old_runs(self, runs_dir: Path, max_keep: int) -> None:
        """Remove old run history files."""
        run_files = sorted(runs_dir.glob("*.json"))
        if len(run_files) > max_keep:
            for old_file in run_files[:-max_keep]:
                old_file.unlink()

    # ==================== Event Handling ====================

    def emit_event(self, event_name: str, payload: dict = None) -> None:
        """Emit an event that may trigger tasks."""
        with self._lock:
            self._event_queue.append({
                "event": event_name,
                "payload": payload or {},
                "timestamp": datetime.utcnow().isoformat()
            })

    def _process_events(self) -> None:
        """Process queued events."""
        with self._lock:
            events = self._event_queue.copy()
            self._event_queue.clear()

        for event in events:
            event_name = event["event"]

            # Find tasks that listen to this event
            for task in self._tasks.values():
                if task.enabled and event_name in task.schedule.events:
                    self._execute_task(task, trigger_type=f"event:{event_name}")

    # ==================== Task Management ====================

    def create_task(
        self,
        task_id: str,
        name: str,
        task_md_content: str,
        schedule: dict,
        agent_id: str = "default",
        **kwargs
    ) -> ScheduledTask:
        """Create a new task."""
        task_folder = self.tasks_dir / task_id
        task_folder.mkdir(parents=True, exist_ok=True)

        # Create task.json
        config = {
            "task_id": task_id,
            "name": name,
            "description": kwargs.get("description", ""),
            "created_at": datetime.utcnow().isoformat(),
            "updated_at": datetime.utcnow().isoformat(),
            "created_by": kwargs.get("created_by", "system"),
            "schedule": schedule,
            "execution": {
                "agent_id": agent_id,
                "session_mode": kwargs.get("session_mode", "isolated"),
                "timeout_seconds": kwargs.get("timeout_seconds", 300),
                "max_turns": kwargs.get("max_turns", 15)
            },
            "status": {
                "enabled": True,
                "last_run": None,
                "next_run": None,
                "run_count": 0
            },
            "triggers": {
                "on_event": kwargs.get("on_event", []),
                "allow_manual": kwargs.get("allow_manual", True)
            },
            "output": {
                "save_runs": True,
                "max_runs_kept": 50
            }
        }

        (task_folder / "task.json").write_text(json.dumps(config, indent=2))
        (task_folder / "task.md").write_text(task_md_content)
        (task_folder / "runs").mkdir(exist_ok=True)

        # Load and register
        return self._load_task(task_folder)

    def update_task(self, task_id: str, updates: dict) -> bool:
        """Update task configuration."""
        with self._lock:
            task = self._tasks.get(task_id)

        if not task:
            return False

        task_json_path = task.folder_path / "task.json"
        if not task_json_path.exists():
            return False

        config = json.loads(task_json_path.read_text())
        config.update(updates)
        config["updated_at"] = datetime.utcnow().isoformat()
        task_json_path.write_text(json.dumps(config, indent=2))

        # Reload task
        self._load_task(task.folder_path)
        return True

    def update_task_md(self, task_id: str, content: str) -> bool:
        """Update task.md content."""
        with self._lock:
            task = self._tasks.get(task_id)

        if not task:
            return False

        task_md_path = task.folder_path / "task.md"
        task_md_path.write_text(content)
        return True

    def delete_task(self, task_id: str) -> bool:
        """Delete a task."""
        with self._lock:
            task = self._tasks.pop(task_id, None)

        if not task:
            return False

        import shutil
        shutil.rmtree(task.folder_path)
        return True

    def enable_task(self, task_id: str) -> bool:
        """Enable a task."""
        return self.update_task(task_id, {"status": {"enabled": True}})

    def disable_task(self, task_id: str) -> bool:
        """Disable a task."""
        return self.update_task(task_id, {"status": {"enabled": False}})

    def trigger_task(self, task_id: str) -> dict:
        """Manually trigger a task."""
        with self._lock:
            task = self._tasks.get(task_id)

        if not task:
            return {"status": "error", "error": "Task not found"}

        return self._execute_task(task, trigger_type="manual")

    def list_tasks(self) -> List[dict]:
        """List all tasks."""
        with self._lock:
            return [
                {
                    "task_id": t.task_id,
                    "name": t.name,
                    "enabled": t.enabled,
                    "schedule_type": t.schedule.type,
                    "last_run": t.last_run.isoformat() if t.last_run else None,
                    "next_run": t.next_run.isoformat() if t.next_run else None
                }
                for t in self._tasks.values()
            ]

    def get_task(self, task_id: str) -> Optional[dict]:
        """Get task details."""
        with self._lock:
            task = self._tasks.get(task_id)

        if not task:
            return None

        task_json_path = task.folder_path / "task.json"
        if task_json_path.exists():
            return json.loads(task_json_path.read_text())
        return None

    def get_task_runs(self, task_id: str, limit: int = 10) -> List[dict]:
        """Get task run history."""
        with self._lock:
            task = self._tasks.get(task_id)

        if not task:
            return []

        runs_dir = task.folder_path / "runs"
        if not runs_dir.exists():
            return []

        run_files = sorted(runs_dir.glob("*.json"), reverse=True)[:limit]
        return [json.loads(f.read_text()) for f in run_files]
```

---

## 5. Task Execution

### 5.1 Execution Flow

```
┌─────────────────┐
│  TASK TRIGGER   │  (Timer / Event / Manual)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Load task.md   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│  Is content     │────▶│  SKIP           │  (Empty file)
│  empty?         │ YES │  (No API call)  │
└────────┬────────┘     └─────────────────┘
         │ NO
         ▼
┌─────────────────┐
│  Create session │
│  context        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Build prompt:  │
│  - System       │
│  - task.md      │
│  - Memory       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  AGENTIC LOOP   │  (Multiple turns)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Process result │
│  - Save history │
│  - Update next  │
│  - Notify       │
└─────────────────┘
```

### 5.2 Task Executor Integration

```python
def create_task_executor(agent_manager: 'AgentManager') -> Callable:
    """
    Create an executor function for the scheduler.

    This bridges the scheduler to the agentic loop.
    """

    def execute_task(task_id: str, task_md_content: str) -> dict:
        """Execute a task through the agentic loop."""

        # Build the task prompt
        prompt = f"""You are executing a scheduled task. Follow these instructions:

{task_md_content}

---
Task ID: {task_id}
Execute the instructions above. Report your actions and results.
"""

        # Run through agent
        try:
            result = agent_manager.run_agent(
                agent_id="task_executor",  # Or task-specific agent
                message=prompt,
                session_id=f"task_{task_id}_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
            )

            return {
                "status": result.status,
                "response": result.final_response,
                "turns": len(result.turns),
                "tools_used": result.tools_called,
                "tokens_used": result.total_tokens
            }

        except Exception as e:
            return {
                "status": "error",
                "error": str(e)
            }

    return execute_task
```

---

## 6. Task Management Tool

### 6.1 Tool Definition

This tool allows agents to create and manage their own scheduled tasks:

```python
class TaskManagementTool(BaseTool):
    """Tool for agents to manage scheduled tasks."""

    def __init__(self, scheduler: TaskScheduler):
        self.scheduler = scheduler

    @property
    def definition(self) -> ToolDefinition:
        return ToolDefinition(
            name="task_manager",
            description="""Manage scheduled tasks. Actions:
- list: List all tasks
- get: Get task details
- create: Create a new task
- update: Update task settings
- update_content: Update task.md content
- delete: Delete a task
- enable: Enable a task
- disable: Disable a task
- trigger: Manually run a task
- runs: Get task run history""",
            parameters=[
                ToolParameter(
                    name="action",
                    type="string",
                    description="Action to perform",
                    enum=["list", "get", "create", "update", "update_content",
                          "delete", "enable", "disable", "trigger", "runs"]
                ),
                ToolParameter(
                    name="task_id",
                    type="string",
                    description="Task ID (required for most actions)",
                    required=False
                ),
                ToolParameter(
                    name="name",
                    type="string",
                    description="Task name (for create)",
                    required=False
                ),
                ToolParameter(
                    name="content",
                    type="string",
                    description="task.md content (for create/update_content)",
                    required=False
                ),
                ToolParameter(
                    name="schedule",
                    type="object",
                    description="Schedule config: {type, interval_seconds, expression, etc.}",
                    required=False
                ),
                ToolParameter(
                    name="updates",
                    type="object",
                    description="Fields to update (for update action)",
                    required=False
                ),
                ToolParameter(
                    name="limit",
                    type="integer",
                    description="Max items to return (for list/runs)",
                    required=False
                )
            ]
        )

    def execute(self, action: str, **kwargs) -> ToolResult:
        """Execute task management action."""
        try:
            if action == "list":
                tasks = self.scheduler.list_tasks()
                return ToolResult(
                    success=True,
                    output=json.dumps(tasks, indent=2)
                )

            elif action == "get":
                task_id = kwargs.get("task_id")
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")
                task = self.scheduler.get_task(task_id)
                if not task:
                    return ToolResult(success=False, output="", error="Task not found")
                return ToolResult(
                    success=True,
                    output=json.dumps(task, indent=2)
                )

            elif action == "create":
                task_id = kwargs.get("task_id")
                name = kwargs.get("name", task_id)
                content = kwargs.get("content", "")
                schedule = kwargs.get("schedule", {"type": "interval", "interval_seconds": 3600})

                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                task = self.scheduler.create_task(
                    task_id=task_id,
                    name=name,
                    task_md_content=content,
                    schedule=schedule,
                    **{k: v for k, v in kwargs.items()
                       if k not in ["task_id", "name", "content", "schedule", "action"]}
                )

                return ToolResult(
                    success=True,
                    output=f"Created task: {task.task_id}",
                    metadata={"task_id": task.task_id}
                )

            elif action == "update":
                task_id = kwargs.get("task_id")
                updates = kwargs.get("updates", {})
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                success = self.scheduler.update_task(task_id, updates)
                return ToolResult(
                    success=success,
                    output="Task updated" if success else "Update failed"
                )

            elif action == "update_content":
                task_id = kwargs.get("task_id")
                content = kwargs.get("content", "")
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                success = self.scheduler.update_task_md(task_id, content)
                return ToolResult(
                    success=success,
                    output="task.md updated" if success else "Update failed"
                )

            elif action == "delete":
                task_id = kwargs.get("task_id")
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                success = self.scheduler.delete_task(task_id)
                return ToolResult(
                    success=success,
                    output="Task deleted" if success else "Delete failed"
                )

            elif action == "enable":
                task_id = kwargs.get("task_id")
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                success = self.scheduler.enable_task(task_id)
                return ToolResult(
                    success=success,
                    output="Task enabled" if success else "Enable failed"
                )

            elif action == "disable":
                task_id = kwargs.get("task_id")
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                success = self.scheduler.disable_task(task_id)
                return ToolResult(
                    success=success,
                    output="Task disabled" if success else "Disable failed"
                )

            elif action == "trigger":
                task_id = kwargs.get("task_id")
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                result = self.scheduler.trigger_task(task_id)
                return ToolResult(
                    success=result.get("status") != "error",
                    output=json.dumps(result, indent=2)
                )

            elif action == "runs":
                task_id = kwargs.get("task_id")
                limit = kwargs.get("limit", 10)
                if not task_id:
                    return ToolResult(success=False, output="", error="task_id required")

                runs = self.scheduler.get_task_runs(task_id, limit=limit)
                return ToolResult(
                    success=True,
                    output=json.dumps(runs, indent=2)
                )

            else:
                return ToolResult(
                    success=False,
                    output="",
                    error=f"Unknown action: {action}"
                )

        except Exception as e:
            return ToolResult(
                success=False,
                output="",
                error=str(e)
            )
```

### 6.2 Example Agent Usage

When the agent wants to create a task:

```
User: "Set up a task to check Moltbook every 4 hours"

Agent thinking: "I need to create a scheduled task. I'll use the task_manager tool."

Agent: tool_call(task_manager, {
    "action": "create",
    "task_id": "moltbook_check",
    "name": "Moltbook Heartbeat",
    "schedule": {
        "type": "interval",
        "interval_seconds": 14400
    },
    "content": "# Moltbook Check\n\nCheck Moltbook for activity:\n1. GET /api/v1/feed\n2. Review posts\n3. Engage as appropriate\n\nRespond with HEARTBEAT_OK if nothing notable."
})

Tool result: "Created task: moltbook_check"

Agent: "I've created a task called 'Moltbook Heartbeat' that will run every 4 hours.
It will check your Moltbook feed and engage with posts as appropriate."
```

---

## 7. File Structure

### 7.1 Tasks Directory Layout

```
TASKS/
├── registry.json                    # Optional: task metadata cache
│
├── moltbook_heartbeat/
│   ├── task.json                   # Task configuration
│   ├── task.md                     # Task instructions
│   └── runs/                       # Execution history
│       ├── 2026-01-31_080000.json
│       ├── 2026-01-31_120000.json
│       └── 2026-01-31_160000.json
│
├── daily_summary/
│   ├── task.json
│   ├── task.md
│   └── runs/
│
├── email_monitor/
│   ├── task.json
│   ├── task.md
│   └── runs/
│
└── weekly_report/
    ├── task.json
    ├── task.md
    └── runs/
```

---

## 8. API Interface

### 8.1 FastAPI Endpoints

```python
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from typing import Optional, List

router = APIRouter(prefix="/tasks", tags=["Tasks"])

class CreateTaskRequest(BaseModel):
    task_id: str
    name: str
    content: str
    schedule: dict
    agent_id: Optional[str] = "default"
    description: Optional[str] = ""

class UpdateTaskRequest(BaseModel):
    updates: dict

class UpdateContentRequest(BaseModel):
    content: str

@router.get("/")
async def list_tasks():
    """List all scheduled tasks."""
    scheduler = get_scheduler()
    return {"tasks": scheduler.list_tasks()}

@router.get("/{task_id}")
async def get_task(task_id: str):
    """Get task details."""
    scheduler = get_scheduler()
    task = scheduler.get_task(task_id)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task

@router.post("/")
async def create_task(request: CreateTaskRequest):
    """Create a new task."""
    scheduler = get_scheduler()
    try:
        task = scheduler.create_task(
            task_id=request.task_id,
            name=request.name,
            task_md_content=request.content,
            schedule=request.schedule,
            agent_id=request.agent_id,
            description=request.description
        )
        return {"task_id": task.task_id, "created": True}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@router.patch("/{task_id}")
async def update_task(task_id: str, request: UpdateTaskRequest):
    """Update task configuration."""
    scheduler = get_scheduler()
    success = scheduler.update_task(task_id, request.updates)
    if not success:
        raise HTTPException(status_code=404, detail="Task not found")
    return {"updated": True}

@router.put("/{task_id}/content")
async def update_task_content(task_id: str, request: UpdateContentRequest):
    """Update task.md content."""
    scheduler = get_scheduler()
    success = scheduler.update_task_md(task_id, request.content)
    if not success:
        raise HTTPException(status_code=404, detail="Task not found")
    return {"updated": True}

@router.delete("/{task_id}")
async def delete_task(task_id: str):
    """Delete a task."""
    scheduler = get_scheduler()
    success = scheduler.delete_task(task_id)
    if not success:
        raise HTTPException(status_code=404, detail="Task not found")
    return {"deleted": True}

@router.post("/{task_id}/enable")
async def enable_task(task_id: str):
    """Enable a task."""
    scheduler = get_scheduler()
    success = scheduler.enable_task(task_id)
    if not success:
        raise HTTPException(status_code=404, detail="Task not found")
    return {"enabled": True}

@router.post("/{task_id}/disable")
async def disable_task(task_id: str):
    """Disable a task."""
    scheduler = get_scheduler()
    success = scheduler.disable_task(task_id)
    if not success:
        raise HTTPException(status_code=404, detail="Task not found")
    return {"disabled": True}

@router.post("/{task_id}/trigger")
async def trigger_task(task_id: str):
    """Manually trigger a task."""
    scheduler = get_scheduler()
    result = scheduler.trigger_task(task_id)
    return result

@router.get("/{task_id}/runs")
async def get_task_runs(task_id: str, limit: int = 10):
    """Get task run history."""
    scheduler = get_scheduler()
    runs = scheduler.get_task_runs(task_id, limit=limit)
    return {"runs": runs}
```

---

## 9. Integration with Agentic Loop

### 9.1 Startup Integration

```python
def create_application():
    """Create the full application with scheduler."""

    # Initialize components
    config_manager = ConfigManager("./CONFIG")
    config_manager.load_global()

    agent_manager = AgentManager(config_manager)

    # Create scheduler with executor
    executor = create_task_executor(agent_manager)
    scheduler = TaskScheduler(
        tasks_dir="./TASKS",
        executor=executor,
        check_interval=1.0
    )

    # Register task management tool with agents
    task_tool = TaskManagementTool(scheduler)
    agent_manager.tool_registry.register(task_tool)

    # Start scheduler
    scheduler.start()

    # Create FastAPI app
    app = FastAPI(title="Agentic Loop with Scheduler")

    # Include routers
    app.include_router(agent_router)
    app.include_router(task_router)

    @app.on_event("shutdown")
    def shutdown():
        scheduler.stop()

    return app
```

### 9.2 System Prompt Addition

Add task awareness to the agent's system prompt:

```python
TASK_SYSTEM_PROMPT_SECTION = """
## Scheduled Tasks

You have access to a task scheduler. You can:
- Create scheduled tasks that run periodically (use task_manager tool)
- Each task has a task.md file with instructions you write
- Tasks can run on intervals, cron schedules, or triggered by events

When creating tasks:
- Use clear, specific instructions in task.md
- Include expected outputs and error handling
- Consider what should trigger human notification

Use the task_manager tool with actions: list, get, create, update, update_content, delete, enable, disable, trigger, runs
"""
```

---

## 10. Implementation Guide

### 10.1 Implementation Order

```
Phase 1: Core Scheduler
├── 1.1 Task data structures
├── 1.2 Task loading from files
├── 1.3 Schedule calculation (interval, cron, once)
└── 1.4 Basic timer loop

Phase 2: Task Execution
├── 2.1 Executor integration with AgenticLoop
├── 2.2 Run history saving
├── 2.3 Status updates
└── 2.4 Empty content detection

Phase 3: Task Management
├── 3.1 Create/update/delete tasks
├── 3.2 Enable/disable tasks
├── 3.3 Manual trigger
└── 3.4 TaskManagementTool for agents

Phase 4: Event System
├── 4.1 Event queue
├── 4.2 Event-triggered tasks
└── 4.3 Event emission API

Phase 5: API & CLI
├── 5.1 FastAPI endpoints
├── 5.2 CLI commands
└── 5.3 Admin panel integration

Phase 6: Testing
├── 6.1 Unit tests for scheduler
├── 6.2 Integration tests with AgenticLoop
└── 6.3 Example tasks
```

### 10.2 Dependencies

```
croniter>=1.3.0     # Cron expression parsing
```

### 10.3 Example Task Creation

```python
# Example: Create a Moltbook heartbeat task programmatically

scheduler.create_task(
    task_id="moltbook_heartbeat",
    name="Moltbook Heartbeat Check",
    task_md_content="""# Moltbook Heartbeat

Check Moltbook for activity every 4 hours.

## Steps

1. Check for DM requests
2. Review feed for mentions
3. Engage with interesting posts
4. Consider posting if 24h+ since last post

## Response

- "HEARTBEAT_OK" if nothing notable
- Brief summary if actions taken
- "NEEDS_HUMAN: [reason]" if human attention needed
""",
    schedule={
        "type": "interval",
        "interval_seconds": 14400,  # 4 hours
        "anchor_time": "2026-01-31T08:00:00Z"
    },
    agent_id="main_agent",
    description="Periodic Moltbook engagement check"
)
```

---

*End of Scheduler Specification Document*
