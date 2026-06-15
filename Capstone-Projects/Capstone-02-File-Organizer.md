# Capstone 02 - File Organizer

## Project Brief

In this project, you will build a safe command-line file organizer that scans a directory, classifies files by rules, plans where each file should move, shows a dry-run preview, handles filename conflicts, performs the moves only after explicit execution, logs what happened, and produces a final summary.

The project teaches real-world automation discipline. It combines `pathlib`, file metadata, dictionaries, functions, dataclasses, JSON or TOML configuration, logging, dry-run design, validation, error handling, tests with temporary directories, and careful thinking about destructive filesystem operations.

By the end, the reader should have a working `organize` command and a stronger instinct for building automation that is useful without being reckless.

---

# Why This Project Matters

Many people write their first automation script as a quick file-moving program.

They have a messy downloads folder.

They want images in one folder.

They want PDFs in another.

They want archives somewhere else.

The first version often looks like this:

```python
import os
import shutil

for name in os.listdir("Downloads"):
    if name.endswith(".pdf"):
        shutil.move("Downloads/" + name, "Documents/" + name)
```

That script may work once.

It is also dangerous.

It has hidden assumptions:

* the source directory exists
* the destination directory exists
* filenames do not conflict
* file extensions are lowercase
* every `.pdf` should be moved
* no file is currently being downloaded
* the user intended real changes
* errors can be ignored
* paths can be joined with strings

Professional automation does not rely on luck.

It plans.

It previews.

It validates.

It logs.

It handles conflicts.

It gives the user control.

This capstone turns a common script idea into a reliable command-line tool.

---

# What You Will Build

You will build a command-line program named `organize`.

It will scan a source directory and organize files into destination folders based on file extension.

The command-line experience should feel like this:

```bash
organize ~/Downloads --dry-run
organize ~/Downloads --execute
organize ~/Downloads --execute --config organizer.toml
organize ~/Downloads --dry-run --summary-json
```

The program should support:

```text
dry-run mode
execute mode
extension-based categories
configurable rules
safe path handling
destination directory creation
filename conflict handling
skipping directories
logging
summary output
tests with temporary directories
```

The tool should never move files unless the user explicitly asks it to execute.

The default should be safe:

```text
dry-run first
execute only when requested
```

That rule is the soul of the project.

---

# Example Behavior

Suppose the source directory contains:

```text
Downloads/
    invoice.pdf
    photo.JPG
    archive.zip
    notes.txt
    app.dmg
    unfinished.crdownload
```

A dry run might print:

```text
Plan:
  invoice.pdf          -> Documents/invoice.pdf
  photo.JPG            -> Images/photo.JPG
  archive.zip          -> Archives/archive.zip
  notes.txt            -> Text/notes.txt
  app.dmg              -> Applications/app.dmg
  unfinished.crdownload -> skipped temporary download

Summary:
  planned: 5
  skipped: 1
  conflicts: 0
```

Nothing moves in dry-run mode.

In execute mode, the program performs the planned moves:

```bash
organize ~/Downloads --execute
```

Output:

```text
Moved:
  invoice.pdf -> Documents/invoice.pdf
  photo.JPG -> Images/photo.JPG
  archive.zip -> Archives/archive.zip
  notes.txt -> Text/notes.txt
  app.dmg -> Applications/app.dmg

Skipped:
  unfinished.crdownload: temporary download

Summary:
  moved: 5
  skipped: 1
  conflicts: 0
```

The output should make the operation understandable.

Automation should not feel mysterious.

---

# Requirements

The File Organizer must:

* accept a source directory
* default to dry-run mode
* require `--execute` for real moves
* skip directories by default
* classify files by extension
* support case-insensitive extensions
* create destination directories when executing
* handle filename conflicts safely
* skip temporary download files
* log actions
* return useful exit codes
* support tests without touching real user files

It should support these commands:

```bash
organize PATH
organize PATH --dry-run
organize PATH --execute
organize PATH --config organizer.toml
organize PATH --summary-json
organize PATH --verbose
```

The command should fail clearly if the source path does not exist.

It should fail clearly if the source path is not a directory.

It should not silently overwrite files.

It should not follow surprising behavior just because `shutil.move()` allows it.

---

# Non-Requirements

This capstone will not include:

* recursive directory organization
* duplicate file hashing
* cloud sync
* GUI interface
* file content inspection
* machine learning classification
* undo history
* background daemon mode
* cross-device performance tuning

These are interesting.

They are later extensions.

The core project should focus on safe local automation.

---

# Project Structure

A clean structure:

```text
file-organizer/
    pyproject.toml
    README.md
    src/
        file_organizer/
            __init__.py
            __main__.py
            cli.py
            config.py
            model.py
            planner.py
            executor.py
            reporting.py
            errors.py
    tests/
        test_config.py
        test_planner.py
        test_executor.py
        test_cli.py
```

Each module has a clear responsibility.

`model.py` defines the plan objects.

`config.py` loads organization rules.

`planner.py` scans files and creates a move plan.

`executor.py` performs moves.

`reporting.py` formats summaries.

`cli.py` parses command-line arguments and connects the pieces.

`errors.py` defines application-specific exceptions.

The separation matters because planning and execution are different.

Planning asks:

```text
what would happen?
```

Execution asks:

```text
perform the approved plan
```

Never blur those two ideas in a file-moving tool.

---

# The Core Design

The File Organizer should have two phases:

```text
scan and plan
execute plan
```

The planner should not move files.

It should only produce data describing intended actions.

For example:

```python
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class MoveAction:
    source: Path
    destination: Path
    category: str
```

Skipped files can also be represented:

```python
@dataclass(frozen=True)
class SkipAction:
    source: Path
    reason: str
```

A plan can contain both:

```python
@dataclass(frozen=True)
class OrganizePlan:
    moves: list[MoveAction]
    skips: list[SkipAction]
    conflicts: list[SkipAction]
```

This is more structured than printing as you scan.

Structured plans are testable.

They are inspectable.

They are safer.

---

# Rule Model

Rules map file extensions to categories.

Example:

```python
DEFAULT_RULES = {
    ".jpg": "Images",
    ".jpeg": "Images",
    ".png": "Images",
    ".gif": "Images",
    ".pdf": "Documents",
    ".txt": "Text",
    ".md": "Text",
    ".zip": "Archives",
    ".tar": "Archives",
    ".gz": "Archives",
    ".dmg": "Applications",
}
```

Extensions should be normalized:

```python
extension = path.suffix.lower()
```

This means:

```text
photo.JPG -> .jpg
photo.jpg -> .jpg
photo.JpG -> .jpg
```

Case-insensitive extension matching avoids a common real-world annoyance.

---

# Unknown Files

What should happen to unknown extensions?

There are several options:

* skip unknown files
* move them into `Other`
* fail
* ask interactively

For this project, choose:

```text
move unknown files into Other
```

Why?

Because a file organizer should organize.

But the behavior should be visible in the plan:

```text
we did not recognize this extension, so it goes to Other
```

The category can be configurable later.

---

# Temporary Files

Some files should be skipped.

Examples:

```text
.crdownload
.part
.tmp
.download
```

These may represent incomplete downloads or temporary work files.

Moving them can confuse other applications.

Define skip suffixes:

```python
TEMP_SUFFIXES = {
    ".crdownload",
    ".part",
    ".tmp",
    ".download",
}
```

If a file has one of these suffixes, the planner should create a `SkipAction`.

Do not silently ignore it.

Report it.

The user should know why it did not move.

---

# Destination Layout

The simplest destination layout is inside the source directory:

```text
Downloads/
    Documents/
    Images/
    Archives/
    Text/
    Applications/
    Other/
```

This keeps everything local.

It avoids moving files across the user's home directory without clear consent.

The user can later configure a different destination root.

For the first version:

```text
destination root = source directory
```

So:

```text
~/Downloads/invoice.pdf
```

becomes:

```text
~/Downloads/Documents/invoice.pdf
```

This is safer than sending files to unrelated global folders.

---

# Conflict Handling

The program must not overwrite files.

Suppose this exists:

```text
Downloads/
    invoice.pdf
    Documents/
        invoice.pdf
```

Moving `invoice.pdf` into `Documents/invoice.pdf` would conflict.

There are several strategies:

* skip the file
* rename automatically
* fail the whole operation
* ask the user

For this capstone, choose automatic rename with a suffix:

```text
invoice.pdf
invoice-1.pdf
invoice-2.pdf
```

The planner can find the first available path.

Example:

```python
def unique_destination(path: Path) -> Path:
    if not path.exists():
        return path

    stem = path.stem
    suffix = path.suffix
    parent = path.parent

    counter = 1
    while True:
        candidate = parent / f"{stem}-{counter}{suffix}"
        if not candidate.exists():
            return candidate
        counter += 1
```

This function must be tested.

Conflict handling is where file automation often becomes dangerous.

---

# Dry Run

Dry run is the default mode.

Dry run means:

```text
show what would happen, but do not change the filesystem
```

This allows the user to inspect the plan before executing.

In dry run:

* do not create destination directories
* do not move files
* do not rename files
* do not delete files

Only inspect and report.

Dry run should still detect likely conflicts based on existing paths.

It should show the final planned destination.

This is what makes the command trustworthy.

---

# Execute Mode

Execute mode performs the plan.

It should require:

```bash
--execute
```

The program may also accept `--dry-run`, but dry-run should be default.

Avoid this design:

```text
organize ~/Downloads
```

and immediately moving files.

That is too risky.

Better:

```text
organize ~/Downloads
```

shows the plan.

Then:

```text
organize ~/Downloads --execute
```

moves files.

The user must cross a clear boundary.

---

# Safe Path Handling

Use `pathlib`.

Avoid string path concatenation.

Weak:

```python
destination = str(source_dir) + "/" + category + "/" + filename
```

Better:

```python
destination = source_dir / category / filename
```

`Path` objects make path operations clearer.

They also help with tests because temporary directories are represented as paths naturally.

The code should validate:

```text
source path exists
source path is a directory
source path can be read
```

Do not scan paths that are invalid.

Fail early.

---

# Configuration

The first version can use default rules.

The next version should support a TOML config file.

Example:

```toml
[categories]
Images = [".jpg", ".jpeg", ".png", ".gif"]
Documents = [".pdf", ".docx"]
Text = [".txt", ".md"]
Archives = [".zip", ".tar", ".gz"]
Applications = [".dmg", ".pkg"]

[settings]
unknown_category = "Other"
```

The config loader should convert this into an extension-to-category mapping.

For example:

```python
{
    ".jpg": "Images",
    ".jpeg": "Images",
    ".png": "Images",
}
```

Validation matters.

The config should reject:

* empty category names
* extensions without leading dots
* duplicate extensions in different categories
* non-list category values

Bad config should fail before files move.

---

# Logging

Use `logging`.

Do not rely only on `print()`.

The CLI can print user-facing output.

The logger can record operational details.

Examples:

```text
scanning source directory
planned move
skipped temporary file
created destination directory
moved file
failed move
```

With `--verbose`, the program can set log level to `DEBUG`.

Without it, use `WARNING` or keep logs quiet.

Logging should help debug problems without overwhelming normal users.

---

# Summary Output

At the end, print a summary:

```text
Summary:
  planned: 12
  moved: 0
  skipped: 2
  conflicts resolved: 1
  mode: dry-run
```

In execute mode:

```text
Summary:
  planned: 12
  moved: 12
  skipped: 2
  conflicts resolved: 1
  mode: execute
```

The summary gives the user confidence.

It also makes the command easier to use in scripts.

---

# JSON Summary

Support:

```bash
organize ~/Downloads --summary-json
```

This prints machine-readable output:

```json
{
  "mode": "dry-run",
  "planned": 5,
  "moved": 0,
  "skipped": 1,
  "conflicts_resolved": 1
}
```

Why include this?

Because professional command-line tools often need both human-readable and machine-readable modes.

Human output is for people.

JSON output is for scripts.

---

# Error Handling

Define application errors:

```python
class OrganizerError(Exception):
    pass


class InvalidSourceDirectory(OrganizerError):
    pass


class InvalidConfig(OrganizerError):
    pass


class ExecutionError(OrganizerError):
    pass
```

The CLI should catch `OrganizerError` and print a clean message.

Unexpected exceptions should still be visible during development.

Expected failures should not produce scary tracebacks.

Example:

```text
error: source path does not exist: /missing/path
```

This is much better than a raw stack trace.

---

# Testing Strategy

Test the planner heavily.

The planner is the heart of the project.

Planner tests:

* PDF goes to Documents
* JPG goes to Images
* uppercase extension is normalized
* unknown extension goes to Other
* temporary download is skipped
* directories are skipped
* conflict produces unique destination

Executor tests:

* execute creates destination directory
* execute moves file
* execute does not overwrite existing file
* dry run does not move file

Config tests:

* valid config loads
* duplicate extension fails
* extension without dot fails

CLI tests:

* default mode is dry-run
* `--execute` moves files
* invalid source returns exit code 1
* `--summary-json` prints valid JSON

Use temporary directories for every filesystem test.

Never test against real Downloads.

---

# Milestone 1 - Models

Build:

* `MoveAction`
* `SkipAction`
* `OrganizePlan`
* summary helpers

Completion criteria:

```text
The program can represent planned moves and skipped files without touching the filesystem.
```

This milestone teaches modeling operations before doing them.

---

# Milestone 2 - Default Rules

Build:

* default extension mapping
* extension normalization
* unknown category handling
* temporary suffix detection

Completion criteria:

```text
The program can classify a file path into a category or skip reason.
```

This milestone teaches rule design.

---

# Milestone 3 - Planner

Build:

* source directory validation
* file scanning
* directory skipping
* destination path creation
* conflict-safe destination selection
* dry-run plan generation

Completion criteria:

```text
The program can produce a complete plan without moving anything.
```

This milestone teaches separation of planning from execution.

---

# Milestone 4 - Executor

Build:

* destination directory creation
* file moves
* execution logging
* execution result summary
* failure handling

Completion criteria:

```text
The program can execute a previously created plan safely.
```

This milestone teaches controlled side effects.

---

# Milestone 5 - CLI

Build:

* `argparse` interface
* default dry-run behavior
* `--execute`
* `--config`
* `--summary-json`
* `--verbose`
* clean error handling

Completion criteria:

```text
The user can run the organizer from the terminal and understand what happened.
```

This milestone teaches user-facing automation.

---

# Milestone 6 - Configuration

Build:

* TOML config loading
* config validation
* category-to-extension conversion
* duplicate detection

Completion criteria:

```text
The user can customize organization rules safely.
```

This milestone teaches configuration as part of software design.

---

# Packaging

The project should expose a console command:

```toml
[project.scripts]
organize = "file_organizer.cli:main"
```

After installation:

```bash
organize ~/Downloads
```

should show a dry-run plan.

And:

```bash
organize ~/Downloads --execute
```

should perform the plan.

This project turns automation into a reusable tool.

---

# Common Mistakes

The first common mistake is moving files immediately by default.

That is unsafe.

The second common mistake is overwriting files during conflicts.

Never overwrite user files silently.

The third common mistake is mixing planning and moving in one loop.

That makes dry-run behavior harder to trust.

The fourth common mistake is testing with real directories.

Use temporary directories.

The fifth common mistake is ignoring directories.

If the source directory already contains category folders, scanning them incorrectly can create strange nested moves.

The sixth common mistake is assuming extensions are lowercase.

Real files are messy.

The seventh common mistake is making configuration too flexible too early.

Start with extension rules.

Then extend.

---

# Extension Ideas

After the core works, add:

* recursive organization
* ignore patterns
* undo log
* duplicate detection by hash
* date-based folders
* MIME type detection
* interactive confirmation
* config initialization command
* preview as Markdown
* move history
* restore command

Each extension must preserve the safety model:

```text
plan before action
dry run before execute
never overwrite silently
```

---

# What This Capstone Teaches

This capstone teaches safe automation.

It uses:

```text
paths
file operations
dataclasses
configuration
planning
execution
logging
JSON output
tests
packaging
```

The central lesson is:

```text
automation should make work safer, not merely faster
```

A file organizer that moves files quickly but dangerously is not a good tool.

A file organizer that previews, validates, handles conflicts, and logs clearly is a professional tool.

---

# Completion Checklist

The capstone is complete when:

```text
The project has a src layout.
The organize command is installable.
The default command performs a dry run.
The --execute option performs real moves.
Source paths are validated.
Directories are skipped.
Temporary download files are skipped.
Extensions are matched case-insensitively.
Unknown extensions go to Other.
Destination folders are created only during execution.
Conflicts do not overwrite files.
The plan is represented as structured data.
Human-readable summary output exists.
JSON summary output exists.
Logging exists.
Tests use temporary directories.
Planner, executor, config, and CLI tests exist.
```

If all of these are true, the reader has built a safe filesystem automation tool.

---

# Exercises

1. Create the project structure.

2. Define `MoveAction`, `SkipAction`, and `OrganizePlan`.

3. Build default extension rules.

4. Write a function that normalizes file extensions.

5. Write a function that detects temporary download files.

6. Write a function that maps a file to a category.

7. Write tests for uppercase and unknown extensions.

8. Implement source directory validation.

9. Implement the planner without moving files.

10. Write tests for the planner using `tmp_path`.

11. Implement conflict-safe destination naming.

12. Write tests for filename conflicts.

13. Implement the executor.

14. Ensure dry run does not create directories or move files.

15. Add logging.

16. Add `argparse` CLI options.

17. Add `--summary-json`.

18. Add TOML configuration.

19. Add config validation tests.

20. Package the project with a console script.

---

# Preview of Capstone 03

Capstone 02 built a File Organizer.

It connected paths, file operations, dry-run design, planning, execution, safe conflict handling, configuration, logging, tests, and packaging.

Capstone 03 will build a REST API.

The REST API will introduce HTTP routes, request parsing, response models, validation, persistence, error handling, tests, and API design.

The transition is:

```text
File Organizer safely automates local filesystem work
REST API exposes application behavior over HTTP
```

The next project moves from local command-line tools to networked software.
