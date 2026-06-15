# Chapter 99 — Scripting for Real Workflows

Scripts are small programs written to get work done.

That sounds modest.

But scripts are everywhere in professional software work.

They rename files.

They clean data.

They generate reports.

They run checks.

They migrate records.

They create releases.

They call APIs.

They prepare datasets.

They automate developer workflows.

They glue together tools that were not originally designed to work together.

Python is excellent for scripting because it lets you start small and grow carefully.

A useful script can begin as:

```python
print("hello")
```

Then become:

* a command-line tool
* a scheduled job
* a team utility
* a migration script
* a data preparation tool
* a maintenance command
* a deployment helper
* an internal automation product

This chapter is about that growth.

Not every script needs to become a framework.

But every script that affects real work should become safe enough for its responsibility.

The central lesson is:

```text
a script becomes professional when another person can run it safely
```

---

# Why Scripting Matters

Scripting matters because many tasks are too small for a full application but too important to do manually forever.

Examples:

* convert a folder of images
* validate a CSV export
* generate Markdown files
* download invoices
* update issue labels
* check broken links
* rename thousands of files
* archive old logs
* create database seed data
* run a release checklist
* compare two JSON files
* prepare a training dataset
* call an API for every row in a spreadsheet

Manual work has problems:

* it is slow
* it is inconsistent
* it is hard to audit
* it is hard to repeat
* it is easy to forget a step
* it does not scale

A script captures the process.

Once captured, the process can be reviewed, tested, improved, scheduled, shared, and reused.

This is why scripting is not junior work.

Good scripting is operational leverage.

---

# Scripts Are Interfaces

A script is an interface between a human and a workflow.

That means design matters.

A script should answer:

* What does it do?
* What inputs does it need?
* What will it change?
* How do I run it?
* How do I preview it?
* What happens if it fails?
* What output should I expect?
* Is it safe to run twice?
* Where are logs?

If only the author can run the script safely, the script is not a team tool yet.

Good scripts communicate.

They have help text.

They validate inputs.

They print useful progress.

They exit with meaningful status codes.

They avoid surprising destructive behavior.

They are boring to operate.

Boring is the goal.

---

# Start with a Clear Job

Before writing code, describe the job.

Weak description:

```text
clean files
```

Better description:

```text
read all CSV files from an input directory, validate required columns, remove blank rows, write cleaned files to an output directory, and produce a summary report
```

The clearer description identifies:

* input
* validation
* transformation
* output
* reporting

This makes implementation easier.

It also makes testing easier.

If the script's purpose cannot be described clearly, the code will drift.

Many bad scripts are bad because the goal was never made explicit.

Write the job in a comment, README, or help description before the script grows.

---

# The Main Function Pattern

A professional script should usually have a `main` function.

Example:

```python
def main() -> int:
    print("running workflow")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

This pattern has several benefits.

It makes the entry point clear.

It lets the script return an exit code.

It avoids running work when the module is imported.

It makes functions easier to test.

`raise SystemExit(main())` tells Python to exit with the return code from `main`.

By convention:

```text
0 means success
nonzero means failure
```

Small details like this matter when scripts run in CI, cron, or automation platforms.

---

# Command-Line Arguments

Hard-coded paths and settings make scripts brittle.

Better scripts accept arguments.

Python's standard library provides `argparse`.

Example:

```python
import argparse
from pathlib import Path


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Clean CSV files and write normalized outputs.",
    )
    parser.add_argument("input_dir", type=Path)
    parser.add_argument("output_dir", type=Path)
    parser.add_argument("--dry-run", action="store_true")
    return parser.parse_args()
```

Now the script can be run as:

```bash
python clean_csvs.py ./incoming ./cleaned --dry-run
```

Arguments make scripts reusable.

They also make behavior visible.

Avoid scripts that require editing source code just to change an input path.

That is an invitation to mistakes.

---

# Help Text

Good command-line tools explain themselves.

With `argparse`, help comes naturally:

```bash
python clean_csvs.py --help
```

The user should see:

* what the script does
* required arguments
* optional flags
* default values where useful
* examples if the script is complex

Help text is not decoration.

It is part of the interface.

If a teammate has to open the source code to learn basic usage, the script is under-designed.

For internal tools, good help text saves repeated explanations.

It also reduces dangerous misuse.

---

# Paths with pathlib

Use `pathlib` for filesystem paths.

It gives an object-oriented path API.

Example:

```python
from pathlib import Path

input_dir = Path("incoming")

for path in input_dir.glob("*.csv"):
    print(path.name)
```

Common operations:

```python
path.exists()
path.is_file()
path.is_dir()
path.mkdir(parents=True, exist_ok=True)
path.read_text(encoding="utf-8")
path.write_text("content", encoding="utf-8")
```

`pathlib` makes path code clearer than manual string concatenation.

Avoid:

```python
filename = directory + "/" + name
```

Use:

```python
filename = directory / name
```

Path handling seems simple until scripts run on different machines.

Use the standard tool designed for it.

---

# Input Validation

Scripts should validate inputs before doing work.

Example:

```python
def validate_input_dir(path: Path) -> None:
    if not path.exists():
        raise ValueError(f"input directory does not exist: {path}")

    if not path.is_dir():
        raise ValueError(f"input path is not a directory: {path}")
```

Validate:

* paths exist
* paths have the expected type
* files have expected extensions
* required columns exist
* configuration values are valid
* credentials are present
* output directories are writable
* destructive actions are explicitly confirmed

Fail early.

Fail clearly.

A script that fails after modifying half the data is much harder to recover from than a script that refuses to start.

---

# Dry Runs

Dry runs are essential for scripts that modify things.

A dry run shows what would happen without doing it.

Example:

```python
def move_file(source: Path, destination: Path, dry_run: bool) -> None:
    if dry_run:
        print(f"would move {source} -> {destination}")
        return

    source.rename(destination)
```

Dry runs are useful for:

* file renaming
* data migrations
* API updates
* deletion scripts
* bulk edits
* report generation
* deployment steps

The dry run should be honest.

It should exercise the same discovery and validation logic as the real run.

Only the final mutation should be skipped.

If dry run and real run use different paths through the code, dry run can become misleading.

---

# Confirmations

Some scripts should ask for explicit confirmation.

Example:

```python
def confirm(message: str) -> bool:
    answer = input(f"{message} [y/N] ")
    return answer.lower() == "y"
```

Use confirmation for:

* deletion
* overwriting
* production changes
* irreversible API calls
* large migrations
* expensive operations

But confirmation is not always appropriate.

Scheduled jobs and CI scripts cannot wait for interactive input.

For those, use explicit flags:

```bash
python delete_old_exports.py --confirm-delete
```

Make dangerous actions intentional.

Accidental execution should not be enough.

---

# Logging

Use logging for scripts that matter.

Example:

```python
import logging

logger = logging.getLogger(__name__)


def main() -> int:
    logging.basicConfig(level=logging.INFO)
    logger.info("starting workflow")
    return 0
```

Log:

* start
* inputs
* counts
* skipped items
* warnings
* errors
* summary

Do not log:

* passwords
* API keys
* private tokens
* unnecessary personal data
* huge payloads

Use levels:

* `DEBUG` for detailed development information
* `INFO` for normal progress
* `WARNING` for recoverable concerns
* `ERROR` for failures
* `EXCEPTION` when logging an exception traceback

Good logs make scripts operable.

---

# Exit Codes

Exit codes let other systems know whether a script succeeded.

Convention:

```text
0 means success
1 or other nonzero value means failure
```

Example:

```python
def main() -> int:
    try:
        run_workflow()
    except Exception:
        logger.exception("workflow failed")
        return 1

    return 0
```

CI systems, cron jobs, shell scripts, and automation platforms use exit codes.

If your script prints an error but exits successfully, automation may think everything is fine.

That is dangerous.

Make failure visible through the process exit code.

---

# Standard Input and Output

Some scripts work well as command-line filters.

They read from standard input and write to standard output.

Example:

```bash
cat users.json | python extract_emails.py > emails.txt
```

This style fits Unix pipelines.

It is useful for simple transformations.

But not every script should be a filter.

If the workflow needs multiple files, logs, dry runs, or complex validation, explicit arguments may be clearer.

Use standard input and output when it makes composition easier.

Use explicit files when clarity and safety matter more.

---

# Working with CSV

CSV is common in real workflows.

Python's standard library includes `csv`.

Example:

```python
import csv
from pathlib import Path


def read_customers(path: Path) -> list[dict[str, str]]:
    with path.open(newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)
        return list(reader)
```

`DictReader` maps column names to values.

This is often better than relying on column positions.

Validate required columns:

```python
required = {"email", "name", "status"}
missing = required - set(reader.fieldnames or [])

if missing:
    raise ValueError(f"missing columns: {sorted(missing)}")
```

CSV looks simple.

Real CSV files can include commas, quotes, encodings, blank lines, and inconsistent columns.

Use the `csv` module rather than hand-splitting lines.

---

# Working with JSON

JSON is common for configuration, API data, and structured output.

```python
import json
from pathlib import Path


def load_config(path: Path) -> dict:
    return json.loads(path.read_text(encoding="utf-8"))
```

Pretty output:

```python
path.write_text(
    json.dumps(data, indent=2, sort_keys=True),
    encoding="utf-8",
)
```

Be careful when reading untrusted or very large JSON.

Validate structure after parsing.

Do not assume:

```python
data["customers"][0]["email"]
```

exists just because it existed in one sample file.

Scripts should fail with clear messages when input structure is wrong.

---

# Temporary Files

Temporary files are useful when intermediate data is needed.

Python provides `tempfile`.

Example:

```python
import tempfile
from pathlib import Path

with tempfile.TemporaryDirectory() as directory:
    work_dir = Path(directory)
    temp_file = work_dir / "output.txt"
    temp_file.write_text("hello", encoding="utf-8")
```

Temporary directories are cleaned up automatically when the context exits.

Use temporary files for:

* downloads
* extracted archives
* intermediate reports
* test data
* atomic writes

Avoid writing random temporary files into the project directory.

Temporary data should have a lifecycle.

---

# Atomic Writes

If a script writes an output file, consider atomic writes.

Problem:

```text
script starts writing report.csv
script crashes halfway
downstream system reads partial report.csv
```

Safer pattern:

```text
write report.csv.tmp
finish and flush
rename report.csv.tmp to report.csv
```

On the same filesystem, rename is usually atomic.

Example:

```python
temp_path = output_path.with_suffix(output_path.suffix + ".tmp")
temp_path.write_text(content, encoding="utf-8")
temp_path.replace(output_path)
```

This helps prevent consumers from seeing partial files.

Atomic output is especially important in scheduled jobs and file-based integrations.

---

# Running External Commands

Python can run external commands with `subprocess`.

Example:

```python
import subprocess

result = subprocess.run(
    ["git", "status", "--short"],
    check=True,
    text=True,
    capture_output=True,
)

print(result.stdout)
```

Important details:

* pass arguments as a list
* use `check=True` to raise on nonzero exit
* use `text=True` for strings instead of bytes
* use `capture_output=True` when you need output

Avoid `shell=True` with untrusted input.

Bad:

```python
subprocess.run(f"rm {filename}", shell=True)
```

This can become command injection.

Prefer:

```python
subprocess.run(["rm", filename], check=True)
```

Even then, be careful with destructive commands.

---

# Environment Variables

Scripts often read configuration from environment variables.

Example:

```python
import os

api_key = os.environ["API_KEY"]
```

Use `os.environ["NAME"]` when the variable is required.

It fails clearly if missing.

Use `os.environ.get("NAME", default)` when a default is reasonable.

Environment variables are useful for:

* secrets
* environment names
* API URLs
* feature flags
* timeouts
* log levels

Do not print all environment variables in logs.

They may contain secrets.

Do not rely on environment variables so heavily that script behavior becomes invisible.

Document required variables.

---

# Configuration Files

When configuration becomes complex, use a config file.

Examples:

* JSON
* TOML
* YAML with a third-party library
* INI

Python has standard support for TOML reading through `tomllib`.

Example:

```python
import tomllib
from pathlib import Path

config = tomllib.loads(Path("config.toml").read_text(encoding="utf-8"))
```

Config files are useful when:

* there are many settings
* the settings should be versioned
* different environments need different values
* non-programmers need to review settings

Secrets usually do not belong in committed config files.

Separate secret values from ordinary configuration.

---

# Designing Output

A script's output should match its audience.

For humans:

```text
Processed 420 files.
Cleaned: 417
Skipped: 2
Failed: 1
Report written to cleaned/summary.json
```

For machines:

```json
{
  "processed": 420,
  "cleaned": 417,
  "skipped": 2,
  "failed": 1
}
```

Sometimes support both:

```bash
python clean_csvs.py incoming cleaned
python clean_csvs.py incoming cleaned --json
```

Human-readable output is good for manual use.

Machine-readable output is good for automation.

Do not mix logs and machine-readable output on the same stream unless you know what you are doing.

Many tools write logs to stderr and data to stdout.

That allows composition.

---

# Progress Reporting

Long-running scripts should show progress.

Useful progress:

* number of files found
* current file
* records processed
* batches completed
* estimated remaining work
* final summary

But progress should not overwhelm logs.

Bad:

```text
printed line for every one of 10 million records
```

Better:

```text
log every 10,000 records
```

Progress reporting helps humans trust that work is happening.

It also helps diagnose where a script got stuck.

---

# Error Handling

Scripts need clear error handling.

Some errors should stop the script.

Some errors should skip one record and continue.

Some errors should be retried.

Some errors should create a quarantine file.

Decide based on risk.

Example:

```python
for record in records:
    try:
        process_record(record)
    except ValueError:
        logger.warning("invalid record skipped", extra={"record_id": record.get("id")})
        skipped += 1
```

If one bad record should not stop a batch, handle it locally.

If configuration is invalid, stop immediately.

If an output write fails, stop.

Error handling is policy.

The code should reflect the workflow's risk.

---

# Testing Scripts

Scripts are easier to test when logic is in functions.

Bad:

```python
args = parse_args()
data = read_file(args.input)
cleaned = clean(data)
write_file(args.output, cleaned)
```

at top level.

Better:

```python
def clean_rows(rows):
    ...


def run(input_path: Path, output_path: Path) -> Summary:
    ...


def main() -> int:
    args = parse_args()
    run(args.input, args.output)
    return 0
```

Now `clean_rows` and `run` can be tested.

Test:

* valid input
* invalid input
* missing files
* empty files
* duplicate records
* dry run
* output formatting
* error paths

If a script matters, test its important behavior.

---

# Packaging Scripts

At first, running a script directly is fine:

```bash
python scripts/clean_csvs.py incoming cleaned
```

As scripts become team tools, packaging may help.

Options:

* keep scripts in a `scripts/` directory
* expose console entry points
* create a package command
* use `python -m package.command`
* containerize the script
* run through CI workflows

The goal is repeatability.

If every teammate runs the script differently, behavior becomes inconsistent.

Document the expected invocation.

Pin dependencies when needed.

Make the tool easy to run correctly.

---

# Documentation

A useful script should have minimal documentation.

Include:

* purpose
* example command
* required inputs
* outputs
* dry-run behavior
* required environment variables
* failure behavior
* owner or team

This can live in:

* script help text
* README
* module docstring
* internal runbook

Documentation does not need to be long.

It needs to answer the questions a runner will have at the moment of use.

The most important example is often:

```bash
python script.py --dry-run ...
```

Show the safe first command.

---

# A Practical Script Skeleton

Here is a useful skeleton:

```python
import argparse
import logging
from dataclasses import dataclass
from pathlib import Path

logger = logging.getLogger(__name__)


@dataclass
class Summary:
    processed: int = 0
    skipped: int = 0
    failed: int = 0


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Process input files and write normalized outputs.",
    )
    parser.add_argument("input_dir", type=Path)
    parser.add_argument("output_dir", type=Path)
    parser.add_argument("--dry-run", action="store_true")
    parser.add_argument("--verbose", action="store_true")
    return parser.parse_args()


def validate_args(args: argparse.Namespace) -> None:
    if not args.input_dir.is_dir():
        raise ValueError(f"input directory does not exist: {args.input_dir}")

    if args.input_dir == args.output_dir:
        raise ValueError("input and output directories must be different")


def run(input_dir: Path, output_dir: Path, dry_run: bool) -> Summary:
    summary = Summary()

    if not dry_run:
        output_dir.mkdir(parents=True, exist_ok=True)

    for path in sorted(input_dir.glob("*.txt")):
        summary.processed += 1
        destination = output_dir / path.name

        if dry_run:
            logger.info("would process %s -> %s", path, destination)
            continue

        text = path.read_text(encoding="utf-8")
        destination.write_text(text.strip() + "\n", encoding="utf-8")

    return summary


def main() -> int:
    args = parse_args()
    logging.basicConfig(
        level=logging.DEBUG if args.verbose else logging.INFO,
        format="%(levelname)s %(message)s",
    )

    try:
        validate_args(args)
        summary = run(args.input_dir, args.output_dir, args.dry_run)
    except Exception:
        logger.exception("script failed")
        return 1

    logger.info(
        "complete: processed=%s skipped=%s failed=%s",
        summary.processed,
        summary.skipped,
        summary.failed,
    )
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

This skeleton includes:

* argument parsing
* path handling
* dry run
* logging
* validation
* summary object
* testable `run` function
* exit code

You can adapt this shape for many real workflows.

---

# Common Mistakes

The first common mistake is writing everything at top level.

Use functions and a `main` entry point.

The second common mistake is hard-coding paths.

Use command-line arguments or configuration.

The third common mistake is doing destructive work without dry run or confirmation.

Make dangerous actions intentional.

The fourth common mistake is using string concatenation for paths.

Use `pathlib`.

The fifth common mistake is hand-parsing CSV.

Use the `csv` module or a suitable data library.

The sixth common mistake is using `shell=True` with untrusted input.

Prefer argument lists for `subprocess`.

The seventh common mistake is printing errors but exiting with success.

Use meaningful exit codes.

The eighth common mistake is having no logs or summary.

Users need to know what happened.

The ninth common mistake is making scripts impossible to test.

Keep logic in functions.

The tenth common mistake is letting a one-off script quietly become critical infrastructure without improving it.

When importance grows, engineering standards must grow too.

---

# Professional Scripting Checklist

Before sharing a script with a team, check:

* Is the purpose clear?
* Does it have a `main` function?
* Does it use `argparse` or a clear interface?
* Does `--help` explain usage?
* Are paths handled with `pathlib`?
* Are inputs validated before mutation?
* Is there a dry-run mode for risky changes?
* Are destructive actions confirmed or gated?
* Are logs useful?
* Are secrets avoided in logs?
* Are exit codes meaningful?
* Are outputs designed for humans, machines, or both?
* Are partial writes avoided?
* Are external commands run safely?
* Is configuration documented?
* Are important functions testable?
* Are unhappy paths tested?
* Is the script safe to run twice?
* Is ownership clear?
* Is there a short example command?

This checklist is how a script becomes a dependable internal tool.

---

# Summary

Scripts turn repeatable work into executable tools.

Python is excellent for scripting because it combines readability, standard library strength, ecosystem breadth, and easy command-line use.

Professional scripts are not merely code that works once.

They have clear goals, command-line arguments, help text, input validation, dry runs, confirmations, logging, exit codes, safe path handling, careful file writes, safe subprocess usage, configuration, progress reporting, error handling, tests, and documentation.

The `main` function pattern keeps scripts importable and testable.

`argparse` makes command-line interfaces user-friendly.

`pathlib` makes filesystem work clearer.

`subprocess` can run external commands, but must be used carefully.

The central lesson is:

```text
scripts are small products for workflows
```

Treat important scripts with product-level care.

That is how teams turn repeated effort into reliable leverage.

---

# Exercises

1. Write a script with a `main` function that returns an exit code.

2. Add `argparse` arguments for input and output paths.

3. Add `--dry-run` and `--verbose` flags.

4. Use `pathlib` to find all `.txt` files in a directory.

5. Validate that the input directory exists before doing work.

6. Create the output directory only when not in dry-run mode.

7. Add logging for start, progress, and summary.

8. Write a CSV reader using `csv.DictReader`.

9. Validate required CSV columns.

10. Write JSON output with sorted keys and indentation.

11. Write output through a temporary file and then replace the final path.

12. Run an external command with `subprocess.run` and `check=True`.

13. Explain why `shell=True` can be dangerous.

14. Add environment-variable configuration for a script setting.

15. Add a TOML config file for multiple settings.

16. Add confirmation before deleting files.

17. Write tests for a pure transformation function.

18. Write a test for invalid input handling.

19. Design machine-readable JSON summary output.

20. Write a short README section showing how to run the script safely.

---

# Part IV Closing

Part IV of Volume IV covered Automation and Integration.

Chapter 98 showed how Python connects systems through APIs, HTTP clients, JSON, authentication, retries, rate limits, pagination, webhooks, schedules, files, queues, validation, logging, observability, and secure automation practices.

Chapter 99 showed how Python scripts turn everyday workflows into repeatable tools through command-line interfaces, path handling, dry runs, confirmations, safe subprocess calls, logging, exit codes, configuration, testing, and documentation.

Together, these chapters show why Python is so valuable beyond application development and data science.

Python is a language for connecting work.

It can connect services.

It can connect files.

It can connect teams.

It can connect one-off tasks into repeatable systems.

The professional lesson is:

```text
small automation deserves real engineering when real work depends on it
```

---

# Preview of Chapter 100

Chapter 99 studied Scripting for Real Workflows.

We learned how scripts become safe, usable tools through clear entry points, command-line arguments, help text, validation, dry runs, confirmations, logging, exit codes, path handling, file safety, subprocess safety, configuration, testing, packaging, and documentation.

Next we begin Part V of Volume IV: Career Paths.

The first chapter in that part is Backend Engineering.

The transition is:

```text
automation connects tasks and systems
career paths show how Python skills become professional roles
```

Chapter 100 will show how Python fits into backend engineering through APIs, services, databases, queues, authentication, observability, deployment, and production ownership.
