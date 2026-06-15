# Chapter 42 — Virtual Environments

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a virtual environment is.
* Why Python projects need isolated environments.
* How virtual environments relate to installed packages.
* How installed packages relate to imports.
* Why different projects can require different package versions.
* How `python`, `pip`, and `site-packages` connect.
* What activation does and does not do.
* Why `python -m pip` is often safer than plain `pip`.
* How to check which Python interpreter is running.
* How virtual environments affect `sys.path`.
* Why virtual environments should not be committed to Git.
* Why virtual environments are disposable and recreatable.
* How virtual environments prepare us for packaging, dependency management, and professional tooling.

This chapter completes Part IX.

We started with modules.

Then packages.

Then the import system.

Then namespaces.

Now we step outside the program and look at the environment around it.

Namespaces answer:

```text
Where do names live inside a running Python program?
```

Virtual environments answer:

```text
Where do installed packages live for this project?
```

Both questions matter.

If modules and packages organize code, virtual environments organize dependencies.

---

# Concept Overview

A virtual environment is an isolated Python environment for a project.

It contains:

* A Python interpreter or links to one.
* A place for installed packages.
* Scripts installed by packages.
* Configuration describing the environment.

Typical project:

```text
my_project/
    .venv/
    main.py
    requirements.txt
```

The `.venv` directory is the virtual environment.

Your project code is outside it.

Installed third-party packages go inside it.

Conceptually:

```text
project code:
    main.py
    app/
    tests/

virtual environment:
    Python executable
    installed packages
    installed command-line scripts
```

The virtual environment creates a boundary around dependencies.

That boundary prevents one project's packages from interfering with another project's packages.

---

# Why Virtual Environments Exist

Imagine two projects.

Project A needs:

```text
requests version 2.x
```

Project B needs:

```text
requests version 3.x
```

If both projects install packages into the same global Python environment, they may conflict.

Installing one version can break the other project.

Virtual environments solve this by giving each project its own installed package space.

Example:

```text
project_a/
    .venv/
        requests 2.x

project_b/
    .venv/
        requests 3.x
```

Each project can use the version it needs.

This is dependency isolation.

Without isolation, Python development becomes fragile.

With isolation, projects become reproducible and easier to reason about.

---

# The Dependency Problem

Your code is not the only code your program uses.

Even a small program may depend on:

* Web libraries.
* Database drivers.
* Testing tools.
* Formatters.
* Linters.
* Type checkers.
* Data libraries.
* Frameworks.

Each dependency may depend on other packages.

Example:

```text
your app
    depends on fastapi
        depends on starlette
        depends on pydantic
```

Dependency graphs can become large.

Different projects can need different versions.

Your operating system may also use Python for its own tools.

Installing project packages globally can pollute or break unrelated code.

Virtual environments keep project dependencies local to the project.

They are one of the first habits that separate casual scripting from professional Python work.

---

# What `venv` Is

Python includes a standard module named `venv`.

It creates virtual environments.

Command:

```text
python -m venv .venv
```

This means:

```text
use the current Python interpreter
run the venv module
create a virtual environment in .venv
```

After creation, your project may look like:

```text
my_project/
    .venv/
    main.py
```

Inside `.venv`, Python creates platform-specific files and directories.

On macOS/Linux, you commonly see:

```text
.venv/
    bin/
    lib/
    pyvenv.cfg
```

On Windows, you commonly see:

```text
.venv/
    Scripts/
    Lib/
    pyvenv.cfg
```

You do not normally edit these files manually.

The environment is a tool-created directory.

---

# Why `.venv`

The directory name `.venv` is a convention.

The leading dot makes it hidden on many systems.

You may also see:

```text
venv/
env/
.env/
```

But `.venv` is common and clear.

Example:

```text
project/
    .venv/
    app/
    tests/
    README.md
```

The name says:

```text
this is the virtual environment for this project
```

It should not contain your source code.

It should not be the place where you write modules.

It should not be committed to Git.

Project code and project environment are separate.

That separation is important.

---

# Creating a Virtual Environment

From your project directory:

```text
python -m venv .venv
```

This creates the environment.

Then activate it.

On macOS/Linux with bash or zsh:

```text
source .venv/bin/activate
```

On Windows PowerShell:

```text
.venv\Scripts\Activate.ps1
```

On Windows cmd:

```text
.venv\Scripts\activate.bat
```

After activation, your shell prompt may show:

```text
(.venv)
```

That prompt is a convenience.

It reminds you that the environment is active.

The exact activation command depends on your shell and operating system.

---

# What Activation Does

Activation changes your shell environment.

Most importantly, it adjusts `PATH`.

`PATH` is the operating system search path for commands.

After activation, when you type:

```text
python
```

your shell finds the Python executable inside `.venv` first.

When you type:

```text
pip
```

your shell finds the `pip` associated with `.venv` first.

Conceptually:

```text
before activation:
    python -> system or other Python
    pip    -> system or other pip

after activation:
    python -> project/.venv/bin/python
    pip    -> project/.venv/bin/pip
```

Activation does not change your Python code.

It changes which Python-related commands your shell resolves first.

That is why activation affects installation and execution.

---

# Activation Is Convenient, Not Magic

You do not strictly need to activate a virtual environment to use it.

You can call its Python directly.

macOS/Linux:

```text
.venv/bin/python main.py
```

Windows:

```text
.venv\Scripts\python main.py
```

This runs the environment's Python without activation.

Activation is useful because it lets you type:

```text
python main.py
```

instead of the full path.

But the environment is really selected by the Python executable being run.

This distinction matters for debugging.

The active shell prompt may say one thing.

The running process may still be using another interpreter if your editor, test runner, or tool is configured differently.

Always verify when confused.

---

# Checking Which Python Is Running

Inside Python:

```python
import sys

print(sys.executable)
```

This prints the Python executable running the program.

In a virtual environment, it may look like:

```text
/path/to/project/.venv/bin/python
```

On Windows:

```text
C:\path\to\project\.venv\Scripts\python.exe
```

You can also check:

```python
import sys

print(sys.prefix)
print(sys.base_prefix)
```

When running inside a virtual environment:

```text
sys.prefix != sys.base_prefix
```

At a beginner level:

```text
sys.executable tells you which Python is running
sys.prefix helps identify the active environment
```

When imports or installs seem wrong, print these values.

They remove guesswork.

---

# `pip` and Python Must Match

`pip` installs packages into a Python environment.

Problem:

```text
pip install requests
python main.py
```

This only works correctly if `pip` and `python` belong to the same environment.

If they do not, you may install a package into one environment and run Python from another.

Then:

```python
import requests
```

fails.

Safer habit:

```text
python -m pip install requests
```

This means:

```text
use this exact Python interpreter
run its pip module
install into this interpreter's environment
```

Inside an activated virtual environment:

```text
python -m pip install requests
```

is usually the clearest command.

It ties installation to the Python you are using.

---

# Installed Packages and Imports

When you install a package:

```text
python -m pip install requests
```

the package is placed into the environment's package directory, commonly called `site-packages`.

Then Python can import it:

```python
import requests
```

Connection:

```text
pip install -> places package files in environment
import      -> loads package/module from environment
```

Installing is not importing.

Importing is not installing.

They are separate steps.

Install:

```text
make code available in the environment
```

Import:

```text
load code into the running Python process
```

This distinction prevents many beginner mistakes.

---

# `site-packages`

`site-packages` is the directory where third-party packages are commonly installed.

Inside a virtual environment, it lives inside the environment.

macOS/Linux example:

```text
.venv/lib/python3.14/site-packages/
```

Windows example:

```text
.venv/Lib/site-packages/
```

You usually do not edit files there manually.

Package installers manage it.

When Python starts, it configures import paths so this directory can be searched.

That is how installed packages become importable.

Conceptually:

```text
virtual environment
    contains site-packages
        contains installed packages
            become importable through sys.path
```

This connects virtual environments directly to the import system from Chapter 40.

---

# How Virtual Environments Affect `sys.path`

Virtual environments affect where Python searches for installed packages.

Inside a virtual environment:

```python
import sys

for entry in sys.path:
    print(entry)
```

You will usually see an entry pointing into the virtual environment's `site-packages`.

That entry is what lets Python import packages installed into the environment.

Example:

```text
/path/to/project/.venv/lib/python3.14/site-packages
```

This is the import connection:

```text
active Python environment
    determines sys.path entries
    determines which installed packages can be imported
```

If your program cannot import a package you installed, ask:

```text
did I install it into the same environment this Python is using?
```

That question solves many issues.

---

# Isolated Environments

By default, a virtual environment is isolated from packages installed in the base Python environment.

Meaning:

```text
packages installed globally are not automatically available inside the venv
```

This is good.

It means the project only sees packages installed for that project.

There is an option:

```text
--system-site-packages
```

which allows access to system site packages.

Example:

```text
python -m venv --system-site-packages .venv
```

Do not use this casually.

It weakens isolation.

For most projects, prefer the default:

```text
isolated virtual environment
```

Isolation is the point.

---

# Deactivating

If a virtual environment is activated, you can deactivate it:

```text
deactivate
```

This changes your shell back so `python` and `pip` resolve as they did before activation.

Deactivation does not delete the environment.

It only changes the current shell session.

The `.venv` directory remains.

You can activate it again later:

```text
source .venv/bin/activate
```

or run its Python directly:

```text
.venv/bin/python main.py
```

Activation state is shell state.

The environment directory is filesystem state.

They are related but different.

---

# Virtual Environments Are Disposable

A virtual environment should be easy to delete and recreate.

This is important.

The environment contains installed packages, generated scripts, and interpreter links.

It should not contain your project source code.

If the environment becomes broken, you should be able to recreate it:

```text
delete .venv
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

This is why dependency records matter.

The environment itself is disposable.

The instructions for recreating it are important.

Professional project rule:

```text
commit dependency descriptions
do not commit the virtual environment directory
```

---

# Do Not Commit `.venv`

Do not commit `.venv` to Git.

Reasons:

* It can be large.
* It contains generated files.
* It is specific to your machine.
* It may contain absolute paths.
* It can differ by operating system.
* It can be recreated.

Your `.gitignore` should include:

```text
.venv/
venv/
env/
```

Modern Python versions may create a `.gitignore` inside new virtual environments for Git convenience, but your project should still intentionally ignore environment directories.

Commit:

```text
source code
tests
README
dependency files
configuration
```

Do not commit:

```text
.venv/
__pycache__/
generated package installs
```

This keeps the repository clean and portable.

---

# Requirements Files

A common way to record dependencies is:

```text
requirements.txt
```

Example:

```text
requests==2.32.3
pytest==8.3.3
```

Install:

```text
python -m pip install -r requirements.txt
```

This installs listed packages into the active environment.

The file lets another developer recreate the environment.

Important distinction:

```text
.venv contains installed packages
requirements.txt describes packages to install
```

The exact dependency-management approach can vary.

Modern projects may use:

* `requirements.txt`
* `pyproject.toml`
* lock files
* tools like Poetry, Hatch, PDM, uv, or pip-tools

We will study professional packaging later.

For now, understand the basic relationship:

```text
environment is disposable
dependency description is source-controlled
```

---

# `pip freeze`

The command:

```text
python -m pip freeze
```

prints installed packages and versions in the current environment.

Example:

```text
requests==2.32.3
urllib3==2.2.3
certifi==2024.8.30
```

You may see packages you did not install directly.

Those can be transitive dependencies.

Example:

```text
requests depends on urllib3
```

`pip freeze` captures the environment state.

It can be useful.

But it may include more than your direct dependencies.

Professional dependency management distinguishes:

```text
direct dependencies
locked full environment
```

That distinction is later.

For now, `pip freeze` helps you see what is installed.

---

# Version Conflicts

Virtual environments solve many version conflicts by isolating projects.

Example:

Project A:

```text
framework==1.0
```

Project B:

```text
framework==2.0
```

Without virtual environments:

```text
one global environment
one installed framework version
conflict
```

With virtual environments:

```text
project_a/.venv -> framework 1.0
project_b/.venv -> framework 2.0
```

Both projects can work.

Virtual environments do not solve all dependency problems.

They do not automatically choose compatible versions.

They do provide separate spaces where compatible sets can exist.

That separation is essential.

---

# Python Version Matters

A virtual environment is created from a base Python.

Example:

```text
python3.12 -m venv .venv
```

creates an environment based on Python 3.12.

Another project might use:

```text
python3.14 -m venv .venv
```

The Python version matters because:

* Language features differ.
* Standard library behavior can differ.
* Package compatibility can differ.
* Compiled packages may depend on Python version.

Check:

```text
python --version
```

Inside Python:

```python
import sys

print(sys.version)
```

When documenting a project, specify the expected Python version.

Dependency isolation is not only about package versions.

It is also about Python version.

---

# Virtual Environment Structure

A virtual environment has internal structure.

Typical macOS/Linux:

```text
.venv/
    bin/
        python
        pip
        activate
    lib/
        python3.x/
            site-packages/
    pyvenv.cfg
```

Typical Windows:

```text
.venv/
    Scripts/
        python.exe
        pip.exe
        Activate.ps1
    Lib/
        site-packages/
    pyvenv.cfg
```

Important pieces:

```text
python executable -> runs code in this environment
pip executable    -> installs packages into this environment
site-packages     -> installed package location
pyvenv.cfg        -> environment configuration
activation script -> adjusts shell PATH
```

You do not need to memorize every file.

You need to understand the roles.

---

# `pyvenv.cfg`

The file:

```text
pyvenv.cfg
```

lives inside the virtual environment.

It records configuration about the environment.

It commonly includes information about the base Python used to create the environment.

Example idea:

```text
home = /path/to/base/python
include-system-site-packages = false
version = 3.14.x
```

You usually do not edit this file manually.

But it explains why environments are tied to their creation context.

The environment knows where it came from.

This is one reason virtual environments are not generally portable by moving or copying folders.

If you move a project, it is often better to recreate the environment.

---

# Environments Are Not Portable

Virtual environments often contain absolute paths.

Installed scripts may point directly to the environment's Python interpreter.

That means copying `.venv` to another location can break it.

Bad habit:

```text
copy .venv from one machine to another
```

Better:

```text
copy project code
create fresh .venv
install dependencies from dependency file
```

If a parent folder moves, recreate the environment if anything behaves strangely.

Model:

```text
source code is portable
dependency description is portable
virtual environment is recreatable
```

This is why professional projects invest in repeatable setup instructions.

---

# Editors and Virtual Environments

Your terminal can use one Python.

Your editor can use another.

Example:

In terminal:

```text
which python
/path/to/project/.venv/bin/python
```

But your editor may run:

```text
/usr/bin/python
```

Then imports work in the terminal but fail in the editor.

Or tests pass in the editor but fail in the terminal.

Fix:

```text
configure the editor to use the project's .venv interpreter
```

The exact steps depend on the editor.

The principle is universal:

```text
all tools for a project should use the same environment
```

That includes:

* Terminal commands.
* Test runner.
* Formatter.
* Linter.
* Type checker.
* Language server.
* Notebook kernel.

---

# `python` vs `python3`

On some systems:

```text
python
```

means Python 3.

On others:

```text
python
```

may not exist or may refer to a different Python.

You may see:

```text
python3
```

used explicitly.

Examples:

```text
python3 -m venv .venv
python3 --version
```

Inside an activated virtual environment, `python` often points to the environment's Python.

Use:

```text
python --version
python -c "import sys; print(sys.executable)"
```

when unsure.

The exact command name is less important than knowing which interpreter is actually running.

Professional habit:

```text
verify the interpreter
do not assume from the command name alone
```

---

# `pip` vs `pip3`

Just like `python` and `python3`, `pip` and `pip3` can point to different environments.

This is why:

```text
python -m pip
```

is valuable.

It avoids guessing which `pip` command your shell found.

Compare:

```text
pip install requests
```

with:

```text
python -m pip install requests
```

The second means:

```text
install using the pip associated with this Python
```

When teaching yourself or debugging, prefer:

```text
python -m pip
```

It makes the relationship explicit.

---

# Creating a Clean Project

A clean beginner project setup might be:

```text
mkdir todo_project
cd todo_project
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Then create files:

```text
todo_project/
    .venv/
    main.py
    README.md
    requirements.txt
```

Add `.venv/` to `.gitignore`.

Install dependencies:

```text
python -m pip install requests
```

Record them:

```text
python -m pip freeze
```

or write a minimal `requirements.txt` manually for simple projects.

This flow is not the only professional flow.

But it teaches the foundation.

Tools can improve the workflow later.

They do not remove the underlying ideas.

---

# Recreating an Environment

Given:

```text
requirements.txt
```

with:

```text
requests==2.32.3
pytest==8.3.3
```

Another developer can run:

```text
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Now their environment has the same listed packages.

This is the point:

```text
do not share the environment directory
share instructions and dependency records
```

The environment can be regenerated.

This makes collaboration possible.

It also makes deployment and continuous integration possible.

We will study those topics later.

---

# Common Mistake: Installing Globally

Bad habit:

```text
pip install package
```

without knowing where it installs.

Possible problems:

* Installs into system Python.
* Requires administrator permissions.
* Conflicts with other projects.
* Breaks operating-system tools.
* Still does not make the package available to your project environment.

Better:

```text
python -m venv .venv
source .venv/bin/activate
python -m pip install package
```

or:

```text
.venv/bin/python -m pip install package
```

The goal is not to memorize one command.

The goal is to know where the package is going.

---

# Common Mistake: Activating the Wrong Environment

It is possible to activate the wrong environment.

Example:

```text
project_a/.venv
project_b/.venv
```

You are working in `project_b`, but your shell still has `project_a/.venv` active.

Then installs and imports behave strangely.

Check:

```text
which python
```

or:

```text
python -c "import sys; print(sys.executable)"
```

On Windows:

```text
where python
```

or the same Python command:

```text
python -c "import sys; print(sys.executable)"
```

Trust inspection over prompt appearance.

The prompt is helpful.

The interpreter path is proof.

---

# Common Mistake: Committing `.venv`

If `.venv` is committed:

* The repository becomes huge.
* Other machines may not use it correctly.
* Paths may be wrong.
* Operating-system-specific files are included.
* Pull requests become noisy.

Fix:

```text
add .venv/ to .gitignore
remove the committed environment from the repository history or index
recreate environments locally
```

Do not store generated dependency installations in source control.

Store the recipe.

That is the durable artifact.

---

# Common Mistake: Moving a Virtual Environment

Bad:

```text
mv project old_project
mv old_project/.venv new_project/.venv
```

The environment may contain absolute paths pointing to the old location.

It may appear to work at first and then fail in strange ways.

Better:

```text
delete .venv
recreate .venv
reinstall dependencies
```

Virtual environments are cheap to recreate.

Treat them as generated infrastructure.

Your source code is the important part.

Your dependency description is the important part.

The `.venv` directory is replaceable.

---

# Common Mistake: Installing Then Importing With Another Python

Example:

```text
python -m pip install rich
python3 script.py
```

If `python` and `python3` are different interpreters, `script.py` may not find `rich`.

Another example:

```text
.venv/bin/python -m pip install rich
/usr/bin/python script.py
```

Package installed into `.venv`.

Script run with system Python.

Import fails.

Correct model:

```text
the Python that installs should be the Python that runs
```

Use:

```text
python -m pip install rich
python script.py
```

after verifying `python` is the environment interpreter.

---

# Common Mistake: Thinking Activation Changes Existing Programs

Activation affects a shell session.

It does not change already-running Python processes.

Example:

* You start a notebook kernel.
* Then you activate `.venv` in a terminal.
* The already-running notebook kernel does not automatically switch environments.

Same for:

* Running servers.
* Long-running REPL sessions.
* Editor language servers.
* Test watchers.

If a tool already started with the wrong Python, restart or reconfigure it.

Activation is not a remote control for every Python process on your machine.

It only affects commands launched from that shell after activation.

---

# Common Mistake: Confusing Dependency Files

You may see:

```text
requirements.txt
pyproject.toml
poetry.lock
uv.lock
Pipfile
Pipfile.lock
```

These files belong to different workflows and tools.

Do not mix them randomly.

At this stage, focus on the core idea:

```text
project declares dependencies
environment installs dependencies
Python imports installed packages
```

Later, we will study packaging and dependency management in more detail.

For now, a simple `requirements.txt` is enough to understand the model.

When joining an existing project, use the dependency tool that project already uses.

Do not invent a parallel system.

---

# Virtual Environments and Tools

Many development tools are installed into virtual environments:

* `pytest`
* `black`
* `ruff`
* `mypy`
* `ipython`
* framework CLIs

Example:

```text
python -m pip install pytest
python -m pytest
```

This runs `pytest` using the active environment.

Why not just:

```text
pytest
```

That can work after activation.

But:

```text
python -m pytest
```

makes it explicit that the tool runs under the current Python.

For learning and debugging, explicit is friendly.

Professional teams often wrap these commands in scripts or task runners later.

The underlying environment idea stays the same.

---

# Virtual Environments and Editors

An editor may need to know the project interpreter.

If it uses the wrong interpreter:

* Autocomplete may miss installed packages.
* Type checking may fail.
* Tests may use wrong dependencies.
* Notebook cells may import different packages than the terminal.

Fix by selecting:

```text
project/.venv/bin/python
```

or on Windows:

```text
project\.venv\Scripts\python.exe
```

The exact UI depends on the editor.

The principle:

```text
editor, terminal, tests, and tools should agree on the environment
```

When they disagree, debugging becomes confusing.

---

# Virtual Environments and Imports

Recall Chapter 40:

```text
import searches locations configured for the running Python
```

Virtual environments change those locations.

Example:

Environment A:

```text
project_a/.venv/site-packages
```

Environment B:

```text
project_b/.venv/site-packages
```

If both have a package named `requests`, they are separate installations.

When running Project A's Python:

```python
import requests
```

loads Project A's installed `requests`.

When running Project B's Python:

```python
import requests
```

loads Project B's installed `requests`.

Same import statement.

Different environment.

Different package location.

That is the whole power of virtual environments.

---

# Virtual Environments and Namespaces

Namespaces are inside running Python.

Virtual environments are outside the program, shaping what can be imported.

Example:

```python
import requests
```

Step one:

```text
environment determines whether requests is installed and on sys.path
```

Step two:

```text
import system loads requests module
```

Step three:

```text
current namespace gets name requests
```

So the full path is:

```text
virtual environment
    -> import search path
    -> loaded module object
    -> namespace binding
```

This connects the last several chapters:

```text
virtual environments decide availability
import system loads code
namespaces bind names
```

That is a professional mental model.

---

# A Complete Setup Walkthrough

Create project:

```text
mkdir weather_app
cd weather_app
```

Create environment:

```text
python -m venv .venv
```

Activate:

```text
source .venv/bin/activate
```

Check Python:

```text
python -c "import sys; print(sys.executable)"
```

Install dependency:

```text
python -m pip install requests
```

Create `main.py`:

```python
import requests

print(requests.__version__)
```

Run:

```text
python main.py
```

Record dependencies:

```text
python -m pip freeze > requirements.txt
```

Project:

```text
weather_app/
    .venv/
    main.py
    requirements.txt
```

Commit `main.py` and `requirements.txt`.

Ignore `.venv/`.

---

# A Complete Debugging Walkthrough

Problem:

```python
import rich
```

fails with:

```text
ModuleNotFoundError: No module named 'rich'
```

Debug:

```text
python -c "import sys; print(sys.executable)"
```

Check environment:

```text
python -m pip show rich
```

If not installed:

```text
python -m pip install rich
```

Run again:

```text
python main.py
```

If it still fails, check whether the command that runs `main.py` uses the same Python:

```text
python -c "import sys; print(sys.executable)"
```

inside the same context.

If using an editor, check the editor interpreter.

Debugging principle:

```text
install and run with the same Python environment
```

Most virtual-environment problems reduce to that.

---

# Professional Habits

Use these habits early:

* Create one virtual environment per project.
* Name it `.venv` unless the project has another convention.
* Do not put source code inside `.venv`.
* Do not commit `.venv`.
* Use `python -m pip` when installing.
* Verify `sys.executable` when confused.
* Keep dependency records in source control.
* Recreate broken environments instead of patching them endlessly.
* Configure your editor to use the project environment.
* Use the project's existing dependency tool.

These habits save hours.

They also make you a better collaborator.

Most Python projects are not only code.

They are code plus environment plus dependencies plus tools.

Virtual environments make that reality manageable.

---

# Exercises

1. Create a virtual environment:

```text
python -m venv .venv
```

Activate it for your operating system.

Then run:

```text
python -c "import sys; print(sys.executable)"
```

What path do you see?

---

2. Inside the environment, run:

```text
python -m pip --version
```

What path appears in the output?

How does it prove which environment `pip` belongs to?

---

3. Install a small package:

```text
python -m pip install rich
```

Then run:

```python
import rich
print(rich)
```

Explain how installation made the import possible.

---

4. Run:

```python
import sys

print(sys.prefix)
print(sys.base_prefix)
```

Are they the same or different inside the virtual environment?

What does that mean?

---

5. Explain the difference between:

```text
pip install requests
```

and:

```text
python -m pip install requests
```

Why is the second often clearer?

---

6. Explain why `.venv/` should not be committed to Git.

List at least three reasons.

---

7. Create a `requirements.txt` file.

Install from it:

```text
python -m pip install -r requirements.txt
```

Explain why this is better than sharing the `.venv` directory.

---

8. Imagine two projects need different versions of the same library.

Draw how two virtual environments solve the conflict.

---

9. Your editor says:

```text
ModuleNotFoundError
```

but your terminal runs the file successfully.

What environment mismatch might be happening?

How would you check?

---

10. In your own words, explain this chain:

```text
virtual environment -> site-packages -> sys.path -> import -> namespace binding
```

Use a package like `requests` or `rich` as the example.

---

# Summary

In this chapter we learned:

* A virtual environment isolates Python packages for a project.
* Virtual environments are commonly created with `python -m venv .venv`.
* A virtual environment has its own Python command, installed packages, scripts, and configuration.
* Activation changes the shell so `python` and `pip` point to the environment first.
* Activation is convenient but not required if you call the environment's Python directly.
* `sys.executable` shows which Python is running.
* `sys.prefix != sys.base_prefix` indicates a virtual environment.
* `python -m pip` installs with the `pip` belonging to that Python interpreter.
* Installed packages live in environment-specific `site-packages`.
* Virtual environments affect import search paths.
* Different projects can use different package versions in different environments.
* Virtual environments are disposable and should be recreated from dependency records.
* `.venv` should not be committed to Git.
* Editors, terminals, tests, and tools should use the same project environment.
* Virtual environments connect dependency management to the import system.

Core model:

```text
project
    source code
    dependency description
    .venv/
        Python interpreter
        site-packages
        installed scripts
```

Runtime model:

```text
selected Python interpreter
    determines environment
    determines sys.path
    determines available installed packages
    imports modules
    binds names in namespaces
```

Virtual environments complete the foundation of program organization.

You now have the chain:

```text
files -> modules
directories -> packages
import system -> loading
namespaces -> name binding
virtual environments -> dependency isolation
```

That chain is the bridge from beginner Python to professional Python engineering.

---

# Preview of Volume II

Volume I built the foundation:

* How computers and operating systems relate to programs.
* How Python executes code.
* How objects, references, mutability, identity, and equality work.
* How primitive types and expressions behave.
* How control flow shapes execution.
* How functions, closures, recursion, and functional patterns work.
* How core data structures store and organize information.
* How Python memory management works.
* How modules, packages, imports, namespaces, and environments organize programs.

Volume II moves into advanced Python and internals.

We will study:

* Object-oriented programming in depth.
* Classes, methods, inheritance, overriding, and MRO.
* Dunder methods and Python's data model.
* Descriptors, properties, and managed attributes.
* Dataclasses and structured object design.
* Iterators, generators, context managers, decorators, and exceptions.
* Files, serialization, concurrency, and asyncio.
* Static typing, protocols, ABCs, metaclasses, and internals.
* CPython architecture, bytecode internals, profiling, and extension concepts.

The transition is deliberate:

```text
Volume I teaches how Python works at the foundation.
Volume II teaches how Python becomes expressive, extensible, and powerful.
```

Now the reader has enough grounding to understand advanced Python without memorizing disconnected tricks.
