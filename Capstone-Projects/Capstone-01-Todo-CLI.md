# Capstone 01 - Todo CLI

## Project Brief

In this project, you will build a command-line todo manager that stores tasks in a local JSON file and lets users add, list, complete, reopen, edit, delete, inspect, and clear tasks from the terminal.

The project is intentionally simple on the surface, but it is designed to teach professional habits: clean project structure, explicit data modeling, safe file persistence, command-line interface design, validation, application-specific errors, testable service logic, and packaging as an installable command.

By the end, the reader should have a working `todo` command and a clear understanding of how a small local Python program becomes real software.

---

The first capstone project is a command-line todo application.

That may sound small.

It is not small if we build it professionally.

A todo CLI touches many ideas from the book:

* command-line interfaces
* functions
* modules
* lists
* dictionaries
* dataclasses
* files
* JSON
* paths
* errors
* validation
* testing
* packaging
* user experience
* safe writes
* incremental design

The point is not to build a toy.

The point is to build a small real program with the habits of a professional engineer.

A beginner version of a todo app stores a list in memory and prints it.

A professional learning version asks:

```text
How are tasks represented?
Where is data stored?
How are IDs assigned?
How are invalid commands handled?
How do we avoid corrupting the data file?
How do we test the program?
How do we keep the CLI pleasant?
How do we structure the code so it can grow?
```

This project is deliberately local and simple.

There is no database.

There is no web server.

There is no authentication.

There is no deployment.

That is good.

The first capstone should let the reader focus on program shape, data modeling, persistence, tests, and user interaction without the noise of a networked system.

---

# What You Will Build

You will build a command-line program named `todo`.

It will let a user:

```text
add a task
list tasks
mark a task as done
reopen a task
delete a task
clear completed tasks
show task details
edit task text
filter tasks by status
store tasks in a JSON file
```

The final command-line experience should feel like this:

```bash
todo add "Read Chapter 76"
todo add "Build the first capstone"
todo list
todo done 2
todo list --all
todo edit 1 "Review packaging chapter"
todo delete 2
```

The exact output can vary, but it should be readable.

For example:

```text
ID  Status  Task
1   open    Review packaging chapter
2   done    Build the first capstone
```

The program should store data between runs.

If you add a task, close the terminal, and run `todo list` later, the task should still be there.

That means the application needs persistence.

For this capstone, persistence will be a JSON file.

---

# Why This Project Comes First

The Todo CLI is a perfect first capstone because it is small enough to finish and rich enough to matter.

It forces a reader to connect earlier ideas:

```text
strings -> task descriptions
booleans -> done status
lists -> collections of tasks
dictionaries -> JSON-like records
dataclasses -> structured domain objects
files -> persistence
pathlib -> safe file paths
argparse -> command-line interface
exceptions -> invalid operations
tests -> confidence
packaging -> installable command
```

The project also teaches an important professional lesson:

```text
small programs still deserve design
```

Many bugs in real systems are not caused by advanced algorithms.

They are caused by weak boundaries:

* unclear data models
* unsafe file writes
* missing validation
* hidden global state
* commands that do too much
* tests that depend on real user files
* confusing output

This capstone gives the reader a place to practice those basics.

---

# Requirements

Before writing code, define the requirements.

Requirements keep the project from becoming vague.

The Todo CLI must support these commands:

```text
todo add TEXT
todo list
todo list --all
todo list --done
todo list --open
todo done ID
todo reopen ID
todo edit ID TEXT
todo delete ID
todo show ID
todo clear-done
```

It must persist tasks in a JSON file.

It must assign stable integer IDs.

It must not reuse IDs after deletion.

It must handle missing files by starting with an empty task list.

It must fail clearly when the user requests an unknown task ID.

It must write data safely.

It must be testable without touching the user's real todo file.

It must have a clean module structure.

It should be installable as a command-line tool.

---

# Non-Requirements

Knowing what not to build is part of design.

This first capstone will not include:

* recurring tasks
* due dates
* priorities
* tags
* sync
* database storage
* terminal colors
* interactive prompts
* curses interface
* web API
* multi-user support

These are useful features.

They are not needed yet.

The capstone should teach the foundation.

Extra features can be added after the core is solid.

---

# Project Structure

A clean project structure might look like this:

```text
todo-cli/
    pyproject.toml
    README.md
    src/
        todo_cli/
            __init__.py
            __main__.py
            cli.py
            model.py
            store.py
            service.py
            errors.py
    tests/
        test_model.py
        test_store.py
        test_service.py
        test_cli.py
```

Each file has a job.

`model.py` defines the task object.

`store.py` reads and writes JSON.

`service.py` implements task operations.

`cli.py` parses commands and prints output.

`errors.py` defines application-specific exceptions.

`__main__.py` lets the package run with:

```bash
python -m todo_cli
```

The structure separates concerns:

```text
CLI layer      -> talks to the user
service layer  -> changes tasks
store layer    -> reads and writes files
model layer    -> defines data shape
```

This separation is the first architectural lesson.

Do not mix everything into one file unless the program is truly tiny.

This program is small, but it is a capstone.

We want it to teach growth.

---

# Data Model

The central object is a task.

A task needs:

* an ID
* text
* done status

It may also include timestamps:

* created time
* updated time
* completed time

For the first version, use:

```python
from dataclasses import dataclass
from datetime import datetime


@dataclass
class Task:
    id: int
    text: str
    done: bool = False
    created_at: str = ""
    updated_at: str = ""
    completed_at: str | None = None
```

This uses strings for timestamps because JSON stores strings naturally.

Later, a richer version could parse them into `datetime` objects.

But the first version should avoid unnecessary complexity.

The data model should be easy to serialize.

---

# Task IDs

Task IDs should be stable.

If task 3 exists today, it should still be task 3 tomorrow.

If task 3 is deleted, a new task should not automatically reuse ID 3.

Why?

Because reused IDs can confuse users.

Imagine this sequence:

```text
todo add "Pay electricity bill"      -> id 3
todo delete 3
todo add "Buy notebook"              -> id 3
```

If a user remembers task 3 as the electricity bill, reuse can cause mistakes.

A simple solution is to compute the next ID as:

```text
max(existing IDs) + 1
```

This means deleted IDs are not reused unless all higher IDs disappear.

That is good enough for this project.

For a more explicit design, store a separate `next_id` value in the JSON file.

That design looks like:

```json
{
  "next_id": 4,
  "tasks": [
    {
      "id": 1,
      "text": "Read Chapter 76",
      "done": false,
      "created_at": "2026-06-15T10:30:00Z",
      "updated_at": "2026-06-15T10:30:00Z",
      "completed_at": null
    }
  ]
}
```

Using `next_id` makes ID assignment explicit.

This capstone should use the explicit design because it teaches state clearly.

---

# Storage Format

The JSON storage file should contain two things:

* the next ID
* the task list

Example:

```json
{
  "next_id": 3,
  "tasks": [
    {
      "id": 1,
      "text": "Read Chapter 76",
      "done": false,
      "created_at": "2026-06-15T10:30:00Z",
      "updated_at": "2026-06-15T10:30:00Z",
      "completed_at": null
    },
    {
      "id": 2,
      "text": "Build Todo CLI",
      "done": true,
      "created_at": "2026-06-15T10:40:00Z",
      "updated_at": "2026-06-15T11:00:00Z",
      "completed_at": "2026-06-15T11:00:00Z"
    }
  ]
}
```

The storage format is part of the application's contract.

Once users have data, changing the format becomes a migration problem.

This is why even a simple JSON file deserves thought.

---

# Choosing the Data File Path

Where should the todo file live?

Hard-coding a path like this is poor design:

```python
DATA_FILE = "/Users/narendra/todos.json"
```

That only works for one user on one machine.

A better default is:

```text
~/.todo-cli/tasks.json
```

Using `pathlib`, this becomes:

```python
from pathlib import Path


DEFAULT_DATA_PATH = Path.home() / ".todo-cli" / "tasks.json"
```

But tests should not write to the real home directory.

So the storage path must be injectable.

For example:

```bash
todo --data-file /tmp/tasks.json add "Test task"
```

The CLI can have a global option:

```text
--data-file PATH
```

This is useful for:

* tests
* demos
* backups
* multiple task files
* debugging

This is a small design choice with large benefits.

---

# Safe Writes

Writing JSON directly to the final file can corrupt data if the program crashes mid-write.

Dangerous pattern:

```python
path.write_text(json.dumps(data), encoding="utf-8")
```

If the process stops halfway, the file may contain partial JSON.

A safer pattern is:

```text
write to temporary file
flush data
replace final file atomically
```

In Python:

```python
import json
import os
from pathlib import Path


def write_json_atomic(path: Path, data: dict) -> None:
    path.parent.mkdir(parents=True, exist_ok=True)
    tmp_path = path.with_suffix(path.suffix + ".tmp")
    tmp_path.write_text(
        json.dumps(data, indent=2, sort_keys=True),
        encoding="utf-8",
    )
    os.replace(tmp_path, path)
```

`os.replace()` replaces the destination if it exists.

On the same filesystem, this is atomic on common operating systems.

This means readers should see either the old file or the new file, not a half-written file.

This is a professional habit.

Small local programs can still protect user data.

---

# Application Errors

Do not raise generic errors everywhere.

Define application-specific exceptions:

```python
class TodoError(Exception):
    pass


class TaskNotFound(TodoError):
    pass


class InvalidTaskText(TodoError):
    pass
```

Why?

Because the CLI can catch `TodoError` and show a clean user-facing message:

```python
try:
    run_command(args)
except TodoError as error:
    print(f"error: {error}", file=sys.stderr)
    return 1
```

Unexpected errors should still fail loudly during development.

Expected application errors should be controlled.

This is the difference between:

```text
Traceback...
TaskNotFound: no task with id 99
```

and:

```text
error: no task with id 99
```

Users deserve clear errors.

Developers deserve useful exceptions.

Both can coexist.

---

# Service Layer

The service layer contains operations.

It should not parse command-line arguments.

It should not print tables.

It should not know about terminal formatting.

It should operate on application state.

Example operations:

```python
def add_task(state: TodoState, text: str) -> Task:
    ...


def list_tasks(state: TodoState, status: str = "open") -> list[Task]:
    ...


def mark_done(state: TodoState, task_id: int) -> Task:
    ...


def reopen_task(state: TodoState, task_id: int) -> Task:
    ...


def delete_task(state: TodoState, task_id: int) -> Task:
    ...
```

This makes testing easier.

You can test task behavior without running a subprocess or parsing CLI output.

The CLI becomes a thin adapter.

That is a major design lesson:

```text
keep business logic out of the interface layer
```

---

# State Object

It is useful to represent the whole file as an object.

For example:

```python
from dataclasses import dataclass, field


@dataclass
class TodoState:
    next_id: int = 1
    tasks: list[Task] = field(default_factory=list)
```

This is better than passing around separate variables:

```python
next_id, tasks
```

The state object represents the whole persistent state.

It can be loaded from JSON and written back to JSON.

It can also be used directly in tests.

---

# Serialization

The model needs conversion functions.

For example:

```python
def task_to_dict(task: Task) -> dict:
    return {
        "id": task.id,
        "text": task.text,
        "done": task.done,
        "created_at": task.created_at,
        "updated_at": task.updated_at,
        "completed_at": task.completed_at,
    }
```

And:

```python
def task_from_dict(data: dict) -> Task:
    return Task(
        id=int(data["id"]),
        text=str(data["text"]),
        done=bool(data.get("done", False)),
        created_at=str(data.get("created_at", "")),
        updated_at=str(data.get("updated_at", "")),
        completed_at=data.get("completed_at"),
    )
```

This is a deliberate boundary.

Inside the program, use `Task`.

At the file boundary, use dictionaries and JSON.

Do not let file dictionaries leak everywhere.

---

# Validation

Task text should be validated.

Invalid examples:

```text
""
"   "
```

The user should not be able to create empty tasks.

Validation function:

```python
def normalize_task_text(text: str) -> str:
    normalized = text.strip()
    if not normalized:
        raise InvalidTaskText("task text cannot be empty")
    return normalized
```

This function should be used by both `add` and `edit`.

Validation should live close to the domain logic, not only in the CLI.

Why?

Because someone may call the service layer directly in tests or future code.

The domain rule is:

```text
a task cannot have empty text
```

That rule belongs in the application logic.

---

# Time Handling

Timestamps are useful, but time can make tests difficult.

If `add_task()` calls `datetime.now()` directly, tests become less predictable.

One simple approach is to allow a clock function:

```python
from collections.abc import Callable


Clock = Callable[[], str]


def utc_now() -> str:
    return datetime.now(timezone.utc).isoformat(timespec="seconds")
```

Then:

```python
def add_task(state: TodoState, text: str, clock: Clock = utc_now) -> Task:
    now = clock()
    ...
```

Tests can pass:

```python
def fake_clock() -> str:
    return "2026-06-15T10:00:00+00:00"
```

This is dependency injection in a small form.

It avoids global time dependency.

---

# CLI Design

Use `argparse`.

The CLI should have subcommands:

```text
add
list
done
reopen
edit
delete
show
clear-done
```

The parser shape:

```python
import argparse


def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(prog="todo")
    parser.add_argument(
        "--data-file",
        help="Path to the todo JSON file",
    )

    subparsers = parser.add_subparsers(dest="command", required=True)

    add_parser = subparsers.add_parser("add")
    add_parser.add_argument("text")

    list_parser = subparsers.add_parser("list")
    status = list_parser.add_mutually_exclusive_group()
    status.add_argument("--all", action="store_true")
    status.add_argument("--done", action="store_true")
    status.add_argument("--open", action="store_true")

    done_parser = subparsers.add_parser("done")
    done_parser.add_argument("id", type=int)

    return parser
```

This is only the beginning.

Each subcommand should map clearly to one service operation.

The CLI should not be clever.

It should be predictable.

---

# Output Design

Output matters.

The program should not print Python dictionaries.

Weak output:

```text
[{'id': 1, 'text': 'Read', 'done': False}]
```

Better output:

```text
ID  Status  Task
1   open    Read
```

For `show`, include more detail:

```text
ID:           1
Status:       open
Task:         Read Chapter 76
Created:      2026-06-15T10:30:00+00:00
Updated:      2026-06-15T10:30:00+00:00
Completed:    -
```

Human-readable output is part of the product.

Even command-line tools have user experience.

---

# Exit Codes

The CLI should use exit codes.

Common convention:

```text
0 -> success
1 -> application error
2 -> command-line usage error
```

`argparse` usually handles usage errors and exits with code 2.

Your application can return 1 for known todo errors.

For example:

```bash
todo done 999
```

Output:

```text
error: no task with id 999
```

Exit code:

```text
1
```

This matters for scripting.

Command-line tools are often used by other programs.

Exit codes are part of the interface.

---

# Testing Strategy

Test the project in layers.

Model tests:

```text
Can a Task be converted to a dictionary?
Can a dictionary be converted back to a Task?
Are missing optional fields handled?
```

Store tests:

```text
Does missing file load as empty state?
Does save create parent directory?
Does saved state load back correctly?
Does invalid JSON produce a useful error?
```

Service tests:

```text
Can a task be added?
Does next_id increment?
Can a task be marked done?
Can a done task be reopened?
Can a task be edited?
Can a task be deleted?
Does unknown ID raise TaskNotFound?
Does empty text raise InvalidTaskText?
```

CLI tests:

```text
Does add command print a success message?
Does list command show expected tasks?
Does --data-file isolate test data?
Does invalid ID produce a clean error?
```

The first three groups should be pure and fast.

CLI tests can be fewer because they are more integration-like.

---

# Testing With Temporary Directories

Tests should not use the real todo file.

Use temporary directories.

Example with `pytest`:

```python
def test_save_and_load_round_trip(tmp_path):
    path = tmp_path / "tasks.json"
    state = TodoState(next_id=2, tasks=[Task(id=1, text="Read")])

    save_state(path, state)

    loaded = load_state(path)

    assert loaded == state
```

`tmp_path` gives each test its own temporary directory.

This prevents tests from interfering with real data or each other.

Testing should be safe by default.

---

# CLI Integration Testing

One way to test the CLI is to call the main function directly.

For example:

```python
def test_add_command(tmp_path, capsys):
    data_file = tmp_path / "tasks.json"

    exit_code = main([
        "--data-file",
        str(data_file),
        "add",
        "Read",
    ])

    captured = capsys.readouterr()

    assert exit_code == 0
    assert "added task 1" in captured.out
```

This is better than spawning a subprocess for every test.

It is faster and easier to inspect.

Later, one or two subprocess tests can verify the installed command works.

---

# Packaging

The project should be installable.

`pyproject.toml` should define:

```toml
[project]
name = "todo-cli"
version = "0.1.0"
description = "A learning-focused command-line todo application"
requires-python = ">=3.12"
dependencies = []

[project.scripts]
todo = "todo_cli.cli:main"
```

This creates a command named `todo`.

After installation:

```bash
todo add "Read Chapter 76"
```

The packaging chapter taught why this matters.

The capstone makes it real.

---

# Development Workflow

A practical workflow:

```bash
uv init todo-cli
cd todo-cli
mkdir -p src/todo_cli tests
uv add --dev pytest
uv run pytest
```

Or with standard tools:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install pytest
python -m pytest
```

The tool is less important than the workflow:

```text
write small behavior
test it
run the CLI
refactor
repeat
```

Do not write the entire project before running anything.

Build in milestones.

---

# Milestone 1 - Model and Serialization

Build:

* `Task`
* `TodoState`
* `task_to_dict()`
* `task_from_dict()`
* `state_to_dict()`
* `state_from_dict()`

Tests:

* task round trip
* state round trip
* missing optional task fields

Completion criteria:

```text
You can convert application objects to JSON-compatible dictionaries and back.
```

This milestone teaches data boundaries.

---

# Milestone 2 - Storage

Build:

* `load_state(path)`
* `save_state(path, state)`
* atomic JSON writing
* missing file behavior
* invalid JSON error

Tests:

* missing file returns empty state
* save and load round trip
* parent directory is created
* invalid JSON raises a useful application error

Completion criteria:

```text
The program can persist and recover state safely.
```

This milestone teaches files and reliability.

---

# Milestone 3 - Service Operations

Build:

* `add_task`
* `list_tasks`
* `get_task`
* `mark_done`
* `reopen_task`
* `edit_task`
* `delete_task`
* `clear_done`

Tests:

* add increments ID
* list filters correctly
* done changes status and completed timestamp
* reopen clears completed timestamp
* edit changes text and updated timestamp
* delete removes task
* clear removes only completed tasks
* unknown ID raises error

Completion criteria:

```text
All todo behavior works without the CLI.
```

This milestone teaches business logic.

---

# Milestone 4 - CLI

Build:

* parser
* subcommands
* output formatting
* error handling
* exit codes
* data file option

Tests:

* `add`
* `list`
* `done`
* invalid ID
* `--data-file` isolation

Completion criteria:

```text
The user can manage tasks from the terminal.
```

This milestone teaches interfaces.

---

# Milestone 5 - Packaging and Polish

Build:

* `pyproject.toml`
* console script entry point
* README usage examples
* release checklist

Tests:

* install in a clean environment
* run `todo --help`
* run a basic command

Completion criteria:

```text
The project can be installed and used as a real command-line tool.
```

This milestone teaches distribution.

---

# Suggested Implementation Order

Use this order:

1. Create project structure.
2. Define errors.
3. Define model dataclasses.
4. Write serialization functions.
5. Write model tests.
6. Write storage load/save.
7. Write storage tests.
8. Write service operations.
9. Write service tests.
10. Write CLI parser.
11. Connect CLI to service and store.
12. Write CLI tests.
13. Add packaging metadata.
14. Install and run the command.
15. Polish README.

This order prevents a common mistake:

```text
building the interface before the behavior is solid
```

Interfaces are easier when the underlying logic is already reliable.

---

# Common Mistakes

The first common mistake is storing tasks only in memory.

That teaches command parsing but not persistence.

The second common mistake is letting the CLI do everything.

If command parsing, task logic, JSON writing, and printing all live in one function, the program becomes hard to test.

The third common mistake is writing to the real user file during tests.

Tests must use temporary paths.

The fourth common mistake is reusing deleted IDs.

Stable IDs make the tool easier to reason about.

The fifth common mistake is ignoring invalid input.

Empty tasks, unknown IDs, corrupted JSON, and missing files should be handled intentionally.

The sixth common mistake is printing tracebacks for expected user errors.

Expected errors should be clean.

Unexpected errors can still show tracebacks during development.

The seventh common mistake is adding too many features too early.

Finish the core first.

Then extend.

---

# Extension Ideas

After the core project works, add features one at a time.

Possible extensions:

* priorities
* due dates
* tags
* search
* sorting
* archive command
* export to Markdown
* import from CSV
* colored output
* shell completion
* configuration file
* recurring tasks
* SQLite storage
* sync with a REST API

Each extension should come with tests.

Do not turn extensions into chaos.

The project should remain understandable.

---

# What This Capstone Teaches

This capstone teaches that even simple software has layers.

The Todo CLI is not only about tasks.

It is about:

```text
data model
serialization
persistence
safe writes
validation
service logic
command-line interface
testing
packaging
user experience
```

It also teaches restraint.

A good first version is not the version with every feature.

It is the version with a clear model, working behavior, tests, and room to grow.

---

# Completion Checklist

The capstone is complete when:

```text
The project has a src layout.
The project has a pyproject.toml.
The todo command can be installed.
The add command works.
The list command works.
The done command works.
The reopen command works.
The edit command works.
The delete command works.
The show command works.
The clear-done command works.
Tasks persist between runs.
The data file path can be overridden.
The program does not write test data to the real home directory.
The JSON file is written safely.
Expected errors are clean.
Tests cover model, store, service, and CLI layers.
The README explains usage.
```

If all of those are true, the reader has built a real command-line tool.

Small, yes.

Real, also yes.

---

# Exercises

1. Implement the project structure exactly as described.

2. Add `Task` and `TodoState` dataclasses.

3. Write serialization functions for tasks and state.

4. Write tests for serialization round trips.

5. Implement `load_state()` for missing and existing files.

6. Implement `save_state()` using an atomic replace.

7. Write tests that use `tmp_path`.

8. Implement `add_task()`.

9. Implement `mark_done()` and `reopen_task()`.

10. Implement `edit_task()` with empty text validation.

11. Implement `delete_task()`.

12. Implement `clear_done()`.

13. Build an `argparse` CLI with subcommands.

14. Add `--data-file`.

15. Format `todo list` as a readable table.

16. Format `todo show` as a detail view.

17. Catch application errors and return exit code 1.

18. Add a console script entry point.

19. Install the project in a clean environment and run the command.

20. Add one extension feature only after the core checklist passes.

---

# Preview of Capstone 02

Capstone 01 built a Todo CLI.

It connected functions, data structures, dataclasses, files, JSON, paths, validation, errors, command-line interfaces, tests, packaging, and safe local persistence.

Capstone 02 will build a File Organizer.

The File Organizer will scan a directory, classify files, plan moves, support dry runs, avoid destructive behavior, handle name conflicts, log actions, and produce a summary.

The transition is:

```text
Todo CLI manages structured user data
File Organizer safely automates filesystem work
```

The next project will make the reader practice paths, file operations, dry-run design, safety checks, logging, and real-world automation discipline.
