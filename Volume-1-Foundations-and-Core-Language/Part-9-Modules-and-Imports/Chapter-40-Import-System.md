# Chapter 40 — Import System

---

# Learning Objectives

By the end of this chapter, you should understand:

* What the import system does.
* What happens when an `import` statement runs.
* Why importing means searching, loading, executing, caching, and binding.
* What `sys.path` is.
* Why the current directory or script directory affects imports.
* What `sys.modules` is.
* Why modules usually execute only once per process.
* How import caching works.
* Why circular imports happen.
* Why partially initialized modules cause confusing errors.
* What finders and loaders do at a beginner level.
* Why packages use `__path__`.
* How `python -m` runs a module by module name.
* How import errors should be read.
* How to design code that imports cleanly.

Chapters 38 and 39 taught us what modules and packages are.

Now we study the machinery that makes them work.

The import system answers this question:

```text
When Python sees import something, how does it find something, load it, and make it available?
```

This chapter does not try to turn you into an import-system implementer.

It gives you the model you need to write, debug, and organize real Python programs.

---

# Concept Overview

An import statement does two broad jobs:

```text
find/load a module
bind a name
```

Example:

```python
import math
```

At a high level, Python:

```text
1. checks whether math is already loaded
2. if not loaded, searches for math
3. creates or obtains a module object
4. initializes and executes module code if needed
5. stores the module in an import cache
6. binds the name math in the current namespace
```

For a user module:

```python
import tasks
```

Python does the same kind of work.

For a package submodule:

```python
import app.tasks.validation
```

Python loads the path step by step:

```text
app
app.tasks
app.tasks.validation
```

The import system is not just file lookup.

It is a runtime system for locating, creating, executing, caching, and binding modules.

---

# Import Is Runtime Behavior

Imports happen while the program runs.

Example:

```python
print("before import")

import math

print("after import")
```

The import statement is executed between the two print calls.

This matters because imports can:

* Execute module code.
* Raise exceptions.
* Bind names.
* Use caches.
* Depend on runtime configuration.

An import is not merely a declaration at the top of a file.

It is an executable statement.

This is why imports can appear inside functions:

```python
def area(radius):
    import math
    return math.pi * radius * radius
```

This works because `import` is a statement.

Still, most imports are placed at the top of files because that makes dependencies visible.

---

# The Two Operations of Import

The import statement combines two operations.

Operation one:

```text
search/load the requested module
```

Operation two:

```text
bind a name in the current namespace
```

Example:

```python
import math
```

After the import:

```text
math -> module object
```

The name `math` exists in the current module namespace.

Example:

```python
from math import sqrt
```

Python still finds and loads the `math` module if needed.

But the bound name is different:

```text
sqrt -> function object from math module
```

The module may be loaded, but the current namespace receives `sqrt`, not necessarily `math`.

This distinction explains many import behaviors.

Importing is not only "make module exist."

It is also "bind names here."

---

# The Import Mental Model

Use this mental model:

```text
import statement
    |
    v
check module cache
    |
    v
search for module
    |
    v
create module object
    |
    v
put module in cache
    |
    v
execute module code
    |
    v
bind name in importing namespace
```

This is simplified.

The real import system has finders, loaders, specs, hooks, built-in modules, frozen modules, extension modules, namespace packages, and path logic.

But the simplified model explains most day-to-day behavior:

* Why import executes top-level code.
* Why later imports usually do not execute the same module again.
* Why circular imports see partially initialized modules.
* Why changing `sys.path` changes what can be imported.
* Why `from module import name` can fail if the name has not been created yet.

The model is not every detail.

It is the map you use before walking into the details.

---

# `sys.path`

`sys.path` is a list of locations Python searches when importing modules.

You can inspect it:

```python
import sys

for entry in sys.path:
    print(entry)
```

The output depends on:

* How Python was started.
* The script location.
* The current working directory.
* The virtual environment.
* Installed packages.
* Environment variables.
* Python configuration.

At a simple level:

```text
sys.path tells Python where to look for importable modules and packages
```

If Python cannot find your module, one common reason is:

```text
the directory containing it is not on sys.path
```

`sys.path` is not the only part of the import system.

But it is the part beginners most often need to understand.

---

# A Simple `sys.path` Example

Project:

```text
project/
    main.py
    tools.py
```

`tools.py`:

```python
def greet(name):
    return f"Hello, {name}"
```

`main.py`:

```python
import tools

print(tools.greet("Ada"))
```

If you run from the project directory:

```text
python main.py
```

Python can usually import `tools`.

Why?

Because the directory containing `main.py` is available for imports.

Conceptually:

```text
project directory is on the import search path
tools.py is in that directory
Python finds tools.py
```

If you move `tools.py` somewhere else that Python is not searching, the import fails.

The import system does not search your entire computer.

It searches configured locations.

---

# Current Directory and Script Directory

Beginners often say:

```text
Python imports from the current directory.
```

That is close, but not always precise.

When running a script:

```text
python path/to/main.py
```

Python usually puts the script's directory near the front of `sys.path`.

When running interactively:

```text
python
```

or:

```text
python -c "..."
```

the current working directory may be used.

When running a module:

```text
python -m package.module
```

the import context is different again.

Practical beginner rule:

```text
where and how you start Python affects imports
```

If an import works in one terminal but not another, check:

```text
working directory
command used to start Python
sys.path
virtual environment
```

This resolves many mysteries.

---

# Import Search Is Ordered

`sys.path` is a list.

Lists have order.

Python searches path entries in order.

That means an earlier location can shadow a later location.

Example:

```text
project/
    random.py
    main.py
```

`main.py`:

```python
import random
```

You may expect Python's standard library `random` module.

But Python might find your local `random.py` first.

This is shadowing.

It can cause confusing errors.

Example:

```text
AttributeError: module 'random' has no attribute 'choice'
```

The problem may be that the imported `random` module is your file, not the standard library module.

Search order matters.

Module names matter.

Avoid naming your files after standard library modules or installed packages.

---

# `sys.modules`

`sys.modules` is a dictionary that stores loaded modules.

You can inspect it:

```python
import sys

print("math" in sys.modules)

import math

print("math" in sys.modules)
print(sys.modules["math"])
```

At a simple level:

```text
sys.modules maps module names to module objects
```

Example:

```text
"math" -> math module object
"sys"  -> sys module object
```

When importing, Python first checks whether the requested module is already in `sys.modules`.

If it is, Python can reuse it.

This is why modules usually execute only once.

The cache is central to the import system.

---

# Why Modules Usually Execute Once

Create:

```python
# noisy.py

print("executing noisy")

value = 10
```

Now:

```python
import noisy
import noisy
import noisy
```

You usually see:

```text
executing noisy
```

only once.

Why?

First import:

```text
not in sys.modules
find module
create module object
execute noisy.py
store in sys.modules
bind name noisy
```

Second import:

```text
already in sys.modules
reuse existing module object
bind name noisy
```

The second import does not rerun the module's top-level code.

This is a feature.

It prevents repeated initialization and keeps module identity stable.

---

# Import Cache and Module State

Because modules are cached, module-level state persists.

Example:

```python
# counter.py

count = 0


def increment():
    global count
    count += 1
    return count
```

Use:

```python
import counter

print(counter.increment())
print(counter.increment())
```

Output:

```text
1
2
```

Now another module imports `counter`.

It gets the same module object from `sys.modules`.

That means it sees the same module-level `count`.

Import cache model:

```text
one process
one loaded module object for a given module name
shared by importers
```

This is why module-level mutable state must be intentional.

It can be shared widely.

---

# Importing the Same Module With Different Names

The cache key is the module's import name.

This matters.

If the same file is imported under different names, Python may treat it as different modules.

Example idea:

```text
project/
    app/
        __init__.py
        settings.py
```

If one part of the program imports:

```python
import app.settings
```

but another somehow imports the same file as:

```python
import settings
```

Python may create two module objects:

```text
sys.modules["app.settings"]
sys.modules["settings"]
```

They can have separate state.

This causes subtle bugs.

Professional guidance:

```text
import modules consistently by their package path
```

Do not mix package imports and direct path tricks.

---

# `import module` Binding

Consider:

```python
import package.submodule
```

What name is bound in the current namespace?

Usually the top-level package name:

```text
package
```

Then you access:

```python
package.submodule
```

Example:

```python
import os.path

print(os.path)
```

The name available directly is:

```text
os
```

not:

```text
path
```

This can surprise beginners.

If you want a shorter local name:

```python
import package.submodule as submodule
```

or:

```python
from package import submodule
```

Import binding depends on the form of the import statement.

Search/loading and name binding are related but separate.

---

# `from module import name` Binding

Example:

```python
from math import sqrt
```

Python loads `math` if necessary.

Then it binds:

```text
sqrt -> math.sqrt object
```

in the current namespace.

The name `math` may not be bound locally.

Example:

```python
from math import sqrt

print(sqrt(9))
print(math.pi)
```

The second line raises:

```text
NameError
```

unless `math` was imported separately.

Why?

Because:

```python
from math import sqrt
```

binds `sqrt`, not `math`.

The module may be loaded in `sys.modules`, but the current namespace does not automatically receive the name `math`.

---

# Importing Packages Step by Step

Consider:

```python
import app.users.models
```

Python does not jump straight to the deepest module.

It works through the path:

```text
app
app.users
app.users.models
```

At each stage:

* The parent package must exist.
* The parent package may be loaded.
* The parent package's `__path__` helps locate submodules.

If `app` cannot be imported, Python cannot import `app.users`.

If `app.users` cannot be imported, Python cannot import `app.users.models`.

This is why an error in a package `__init__.py` can prevent importing submodules.

Example:

```python
import app.users.models
```

may fail because:

```text
app/__init__.py raised an exception
```

even if `models.py` itself is fine.

---

# Package `__path__`

Packages have a special attribute named `__path__`.

At a beginner level:

```text
package.__path__ tells Python where to look for submodules inside that package
```

Example:

```python
import app

print(app.__path__)
```

For a regular package, this usually points to the package directory.

Example:

```text
['.../project/app']
```

When importing:

```python
import app.users
```

Python searches within `app.__path__` for `users`.

This is different from top-level imports, which use `sys.path`.

Simple model:

```text
top-level import -> search sys.path
submodule import -> search parent_package.__path__
```

This is why packages can contain subpackages and modules in a controlled way.

---

# Top-Level Search vs Package Search

Compare:

```python
import users
```

with:

```python
import app.users
```

For:

```python
import users
```

Python searches for a top-level module/package named `users` using the top-level import path.

For:

```python
import app.users
```

Python first imports `app`.

Then it searches for `users` inside `app.__path__`.

These are different requests.

A file at:

```text
app/users.py
```

is not automatically importable as:

```python
import users
```

unless `app` itself is on the import path in a way that makes `users.py` top-level.

Use package-qualified imports consistently:

```python
from app import users
```

or:

```python
import app.users
```

This avoids ambiguity.

---

# Finders and Loaders

The real import system uses finders and loaders.

Beginner model:

```text
finder -> knows how to find a module
loader -> knows how to load and execute it
```

When a module is not already in `sys.modules`, Python asks import finders:

```text
Can you find this module?
```

If a finder can find it, Python gets information about how to load it.

Then a loader performs the loading work.

Different kinds of modules may be found and loaded differently:

* Built-in modules.
* Frozen modules.
* Source `.py` files.
* Compiled extension modules.
* Modules inside zip files.
* Namespace packages.

You do not need to customize finders and loaders yet.

But knowing they exist helps explain why imports are flexible.

Python imports are not limited to reading `.py` files from ordinary directories.

---

# Module Specs

Modern Python represents import information with a module spec.

You can inspect one:

```python
import math

print(math.__spec__)
```

The spec contains import-related information such as:

* Module name.
* Loader.
* Origin.
* Package search locations for packages.

For normal programming, you rarely need to use `__spec__`.

But it explains a useful idea:

```text
before Python loads a module, it needs a plan for loading it
```

That plan is represented by the module spec.

When debugging unusual import behavior, `__spec__` can sometimes reveal where a module came from.

For beginner work, `module.__file__` and `module.__name__` are usually more immediately useful.

---

# Where Did This Module Come From?

When confused, inspect the module.

Example:

```python
import random

print(random)
print(random.__file__)
```

For many Python modules, `__file__` shows the path to the file that was loaded.

This is useful when you suspect shadowing.

Example problem:

```python
import random
print(random.__file__)
```

Output:

```text
/Users/you/project/random.py
```

Now you know Python imported your local file, not the standard library module you expected.

Some modules do not have a normal `__file__`.

Built-in modules may not come from a `.py` file.

But when `__file__` exists, it is one of the easiest import-debugging tools.

---

# Built-In Modules

Some modules are built into Python itself.

Example:

```python
import sys
```

The `sys` module is not loaded from a normal `sys.py` file in your project.

It is provided by the interpreter.

Built-in modules are part of why import is more than file searching.

Python can import:

```text
built-in modules
source files
packages
extension modules
frozen modules
namespace packages
```

The import system provides one interface:

```python
import name
```

but supports multiple sources behind that interface.

This is powerful.

It is also why the import system has finder and loader machinery.

---

# Source Files and Bytecode

When importing a `.py` file, Python executes source code.

It may also create cached bytecode files.

You may see a directory:

```text
__pycache__/
```

Inside it, files like:

```text
module.cpython-314.pyc
```

These are bytecode cache files.

They help Python avoid recompiling unchanged source code every time.

Important:

```text
__pycache__ is an implementation optimization
```

You usually do not edit it.

You usually do not commit it to version control.

You usually do not need to think about it.

The import model remains:

```text
find module
load module
execute module code if needed
cache module object
```

Bytecode caching helps performance.

It does not change the meaning of your imports.

---

# Import Errors

The most common import error is:

```text
ModuleNotFoundError
```

Example:

```python
import does_not_exist
```

Error:

```text
ModuleNotFoundError: No module named 'does_not_exist'
```

This means Python could not find a module with that import name.

Common causes:

* The file does not exist.
* The package is not installed.
* The current environment is wrong.
* The directory is not on `sys.path`.
* The import name is misspelled.
* You are running the program from the wrong place.
* You confused distribution name and import name.

Do not panic.

Ask:

```text
What exact import name did Python try?
Where is the file/package?
Is that location on sys.path?
Which Python environment is running?
```

Those questions usually lead to the answer.

---

# ImportError

Another common error is:

```text
ImportError
```

Example:

```python
from math import not_a_real_name
```

This can produce:

```text
ImportError: cannot import name 'not_a_real_name' from 'math'
```

This means the module was found, but the requested name was not available from that module.

Distinction:

```text
ModuleNotFoundError -> could not find the module
ImportError         -> found something, but import operation failed
```

There are many variations.

Sometimes an `ImportError` is caused by an exception inside the imported module.

Read the full traceback.

The most important line is not always the last line.

Look for where import began and where execution failed.

---

# AttributeError After Import

Sometimes the import succeeds, but attribute access fails.

Example:

```python
import tools

tools.clean_data()
```

Error:

```text
AttributeError: module 'tools' has no attribute 'clean_data'
```

Possible causes:

* `clean_data` is not defined in `tools.py`.
* It is defined under a different name.
* The wrong `tools` module was imported.
* A circular import left the module partially initialized.
* The name is only created under a condition that did not run.

Debug:

```python
import tools

print(tools.__file__)
print(dir(tools))
```

`tools.__file__` tells where the module came from.

`dir(tools)` shows names available on the module object.

This quickly reveals many mistakes.

---

# Circular Imports

A circular import happens when modules depend on each other during import.

Example:

`a.py`:

```python
import b

value_from_a = "A"


def show_a():
    return value_from_a
```

`b.py`:

```python
import a

value_from_b = "B"


def show_b():
    return value_from_b
```

This may work if neither module needs unfinished names too early.

But circular imports often fail when one module tries to use a name before the other module has finished executing.

Circular imports are import-time dependency loops.

They are not always wrong.

But they are a common source of confusing errors.

---

# Partially Initialized Modules

During import, Python creates a module object and puts it in `sys.modules` before executing all of its code.

Why?

To prevent infinite recursion if the module imports itself indirectly.

But this creates a visible behavior:

```text
other modules may see a module before it has finished initializing
```

Example:

`a.py`:

```python
from b import use_a

value = "ready"
```

`b.py`:

```python
import a


def use_a():
    return a.value
```

Depending on exact order, `b` may see `a` before `a.value` exists.

This can produce errors like:

```text
partially initialized module
```

or:

```text
cannot import name ... from partially initialized module ...
```

The phrase "partially initialized" usually points to a circular import.

---

# A Circular Import Failure

`users.py`:

```python
from reports import build_report


def get_user():
    return "Ada"
```

`reports.py`:

```python
from users import get_user


def build_report():
    return f"Report for {get_user()}"
```

Now run:

```python
import users
```

Import flow:

```text
start importing users
users imports build_report from reports
start importing reports
reports imports get_user from users
users has not finished defining get_user yet
failure
```

The problem is not that Python cannot read the files.

The problem is timing.

Each module wants something from the other before initialization is complete.

---

# Fixing Circular Imports

Common fixes:

* Move shared code to a third module.
* Import the module instead of importing a name directly.
* Move an import inside a function if it is only needed at runtime.
* Redesign module boundaries.
* Separate data models from behavior that depends on higher-level modules.

Third module example:

```text
users.py
reports.py
formatting.py
```

`formatting.py`:

```python
def format_user(name):
    return name.title()
```

Now both modules can import `formatting` instead of importing each other.

Dependency graph:

```text
users   ──▶ formatting
reports ──▶ formatting
```

No cycle.

The best circular import fix is usually better design.

---

# Local Imports as a Tool

Sometimes moving an import inside a function is reasonable.

Example:

```python
def build_report():
    from users import get_user
    return f"Report for {get_user()}"
```

Now `users` is imported when `build_report()` runs, not when the module first imports.

This can avoid some circular import timing problems.

But use this carefully.

Local imports can hide dependencies.

They are good when:

* They avoid a real circular import.
* The dependency is optional.
* The import is expensive and rarely needed.
* The import is platform-specific.

They are not good as a default habit.

Top-level imports are clearer when there is no reason to delay them.

---

# `python -m`

The command:

```text
python -m module_name
```

runs a module by module name.

Example:

```text
python -m app.cli
```

This tells Python:

```text
find app.cli using the import system
execute it as the main module
```

This is different from:

```text
python app/cli.py
```

The file-path form runs a file directly.

The `-m` form runs a module in package context.

This matters for relative imports.

If `app/cli.py` contains:

```python
from .tasks import create_task
```

then:

```text
python -m app.cli
```

is usually the correct way to run it from the project root.

---

# `__main__`

When Python runs code as the program entry point, that executing module has:

```python
__name__ == "__main__"
```

This happens for:

```text
python script.py
```

and also for:

```text
python -m package.module
```

But the import context differs.

With:

```text
python -m app.cli
```

Python locates `app.cli` as a module.

Then it executes it as `__main__`.

This lets package-relative imports work correctly because Python understands the package context.

The name `__main__` means:

```text
this is the code currently acting as the program entry point
```

It does not necessarily mean:

```text
this file is not part of a package
```

The `-m` flag is how you run package modules cleanly.

---

# `__package__`

Modules can have a special attribute:

```python
__package__
```

This helps Python resolve explicit relative imports.

Example:

```python
from .tasks import create_task
```

The dot only makes sense if Python knows the current package.

When a module is imported normally as:

```python
import app.cli
```

Python knows the package is:

```text
app
```

When running with:

```text
python -m app.cli
```

Python also gives the module package context.

When running by file path:

```text
python app/cli.py
```

the package context may be missing.

That is why relative imports often fail in directly run package files.

You rarely need to set `__package__` manually.

But knowing it exists explains the behavior.

---

# Relative Import Errors

Common error:

```text
ImportError: attempted relative import with no known parent package
```

This often happens when a file inside a package is run directly:

```text
python app/cli.py
```

and it contains:

```python
from .tasks import create_task
```

Python sees the dot.

The dot means:

```text
current package
```

But the file was run as a script, so Python may not know the parent package.

Fix:

```text
python -m app.cli
```

from the directory that contains `app`.

Another fix is to use absolute imports and run the project consistently.

Do not randomly change imports until the error disappears.

Understand the execution context.

---

# Absolute Imports and Execution Context

Absolute imports are often easier to reason about.

Example:

```python
from app.tasks import create_task
```

This starts from the top-level package:

```text
app
```

For this to work, the directory containing `app` must be on `sys.path`.

Project:

```text
project/
    app/
        __init__.py
        tasks.py
        cli.py
```

Run from `project/`:

```text
python -m app.cli
```

Then:

```python
from app.tasks import create_task
```

can work naturally.

If you run from inside `app/`:

```text
cd app
python cli.py
```

then `app` may no longer be importable as a top-level package.

Again:

```text
where you run from matters
```

---

# The Current Working Directory Trap

Suppose:

```text
project/
    app/
        __init__.py
        tasks.py
        cli.py
```

Inside `cli.py`:

```python
from app.tasks import create_task
```

If you are in:

```text
project/
```

and run:

```text
python -m app.cli
```

Python can find `app`.

If you are in:

```text
project/app/
```

and run:

```text
python cli.py
```

Python may not find top-level package `app`.

Why?

Because the search path now starts from inside the package, not from the directory containing the package.

Practical rule:

```text
run package-based applications from the project root
```

This keeps imports predictable.

---

# Installed Packages

When you install third-party libraries, their modules become available through import paths.

Example:

```text
pip install requests
```

Then:

```python
import requests
```

The installed package lives in an environment-specific location, often called `site-packages`.

You do not normally add that path manually.

Python environment setup handles it.

This is why virtual environments matter.

Different environments can have different installed packages.

Import failure may mean:

```text
the package is installed somewhere,
but not in the Python environment currently running this program
```

We will study virtual environments in Chapter 42.

For now, remember:

```text
imports depend on the active Python environment
```

---

# Distribution Name vs Import Name

Installation names and import names can differ.

Example pattern:

```text
install distribution
import package
```

They often match, but not always.

This means:

```text
pip install something
```

does not guarantee:

```python
import something
```

is the correct import name.

When using third-party libraries, check their documentation for the import name.

This distinction belongs partly to packaging, but it affects import debugging.

If an import fails after installation, ask:

```text
Did I install into the active environment?
Am I using the correct import name?
```

Both questions matter.

---

# Reloading Modules

Sometimes beginners expect:

```python
import module
```

to pick up file changes every time.

It does not.

Once a module is loaded, later imports usually reuse the cached module object.

Python has:

```python
import importlib

importlib.reload(module)
```

This reruns the module code in the existing module object.

Reloading can be useful in interactive exploration.

But it can be tricky because existing objects created from old definitions may still exist.

Example:

```python
from module import SomeClass
```

Reloading `module` does not automatically update every existing reference to the old `SomeClass`.

Professional guidance:

```text
restart the process for clean behavior when changing module code
use reload only when you understand the consequences
```

---

# Modifying `sys.path`

You can modify `sys.path` at runtime:

```python
import sys

sys.path.append("/some/directory")
```

Then Python can search that directory for imports.

This is sometimes useful in quick experiments.

But it is often a poor application design.

Problems:

* It hides project structure.
* It depends on machine-specific paths.
* It can mask packaging problems.
* It can create imports that work only on one computer.

Better solutions usually include:

* Running from the project root.
* Using proper packages.
* Installing the project in a virtual environment.
* Using a standard project layout.

Treat direct `sys.path` edits as a debugging or advanced tool, not the normal way to organize code.

---

# Environment Variables and Import Search

Python can be affected by environment variables.

One important variable is:

```text
PYTHONPATH
```

It can add directories to the import search path.

Example idea:

```text
PYTHONPATH=/path/to/project python main.py
```

This can make modules importable.

But it can also make behavior depend on shell configuration.

If code only works when `PYTHONPATH` is set a certain way, that may be fragile.

For learning, it is enough to know:

```text
environment can affect sys.path
```

For professional projects, prefer reliable packaging and virtual environments over ad hoc environment path changes.

---

# Import-Time Side Effects

Because import executes top-level code, import-time side effects matter.

Bad module:

```python
# dangerous.py

delete_old_files()
send_email()
start_server()
```

Now:

```python
import dangerous
```

does real work immediately.

Good module:

```python
def delete_old_files():
    ...


def send_email():
    ...


def main():
    delete_old_files()
    send_email()


if __name__ == "__main__":
    main()
```

Now importing defines tools.

Running executes behavior.

Import-time side effects are one of the biggest causes of surprising Python programs.

Keep imports boring.

That is a compliment.

---

# Import-Time Configuration

Some modules configure things at import time.

Example:

```python
import logging

logging.basicConfig(level=logging.INFO)
```

If this lives at top level in a widely imported module, it can affect the whole program simply by being imported.

Sometimes this is intended.

Often it is not.

Configuration should usually happen in an entry point:

```python
def main():
    configure_logging()
    run_app()
```

Then:

```python
if __name__ == "__main__":
    main()
```

Libraries should be especially careful.

When someone imports a library, they usually do not want it to configure their whole application.

Modules should define.

Entry points should execute and configure.

---

# Import-Time Performance

Imports take time.

Most imports are fast enough.

But heavy import-time work can slow program startup.

Example:

```python
# reports.py

BIG_DATA = load_large_dataset()
```

Now every import of `reports` may load the dataset the first time.

If most code only needs a small helper from `reports`, that is wasteful.

Better:

```python
def load_reports_data():
    return load_large_dataset()
```

or lazy cache:

```python
_big_data = None


def get_big_data():
    global _big_data

    if _big_data is None:
        _big_data = load_large_dataset()

    return _big_data
```

Keep expensive work out of imports unless there is a clear reason.

---

# Star Imports and Import System

The statement:

```python
from module import *
```

imports many names into the current namespace.

This does not mean:

```text
import every possible object recursively
```

It means import public names from that module according to Python's rules.

If the module defines `__all__`, that list controls which names are imported by star import.

Example:

```python
# tools.py

__all__ = ["clean"]


def clean(value):
    return value.strip()


def _helper(value):
    return value.lower()
```

Then:

```python
from tools import *
```

imports `clean`, not `_helper`.

Star imports are still usually avoided in normal code.

But `__all__` is useful for defining a public API.

---

# `__all__`

`__all__` is a module-level list of public names for star import and documentation intent.

Example:

```python
__all__ = ["create_task", "format_task"]
```

This says:

```text
these are the public names this module intentionally exports
```

It is most useful in:

* Library modules.
* Package `__init__.py` files.
* Modules with many internal helpers.

It does not make other names impossible to access.

Python still allows:

```python
import tools
tools._helper()
```

if the name exists.

`__all__` is an API signal, not a security boundary.

Use it when it clarifies public surface.

---

# Namespace Packages Preview

Chapter 39 mentioned namespace packages.

They are packages that may not have a normal `__init__.py` and can be spread across multiple locations.

This is advanced.

The import system supports them because import search is flexible.

For ordinary projects, use regular packages:

```text
package/
    __init__.py
```

But when you see a package without `__init__.py`, do not immediately assume it is impossible.

Modern Python can support namespace packages.

The practical rule remains:

```text
use __init__.py while learning and for clear regular package boundaries
```

We mention namespace packages only to make the import system feel less mysterious later.

---

# Importing Is Not Dependency Installation

Importing and installing are different operations.

Importing:

```python
import requests
```

means:

```text
load an importable module/package into this running Python process
```

Installing:

```text
pip install requests
```

means:

```text
put a distribution into an environment so it can be imported
```

If import fails, installing may be needed.

But not always.

Maybe:

* The import name is wrong.
* The wrong Python interpreter is running.
* The package is installed in another environment.
* A local file shadows it.
* The project is not installed correctly.

Do not treat every import error as:

```text
just install something
```

First understand what Python is trying to import and where it is looking.

---

# Debugging Import Problems

When an import fails, use a calm checklist.

Question one:

```text
What exact import statement failed?
```

Question two:

```text
Is the module/package name spelled correctly?
```

Question three:

```text
Where is the file or installed package?
```

Question four:

```text
Is that location on sys.path?
```

Question five:

```text
Which Python interpreter/environment is running?
```

Question six:

```text
Is a local file shadowing the intended module?
```

Question seven:

```text
Is this actually a circular import?
```

Useful inspection:

```python
import sys

print(sys.executable)
print(sys.path)
```

For a module that imports:

```python
import module

print(module.__file__)
```

Debugging imports is mostly about making invisible context visible.

---

# A Complete Import Flow Example

Project:

```text
project/
    main.py
    todo/
        __init__.py
        tasks.py
```

`todo/tasks.py`:

```python
print("loading tasks")


def create_task(title):
    return {"title": title, "done": False}
```

`main.py`:

```python
from todo.tasks import create_task

task = create_task("learn imports")
print(task)
```

Run:

```text
python main.py
```

Import flow:

```text
main.py starts
Python sees from todo.tasks import create_task
Python checks sys.modules for todo
Python searches sys.path for package todo
Python finds todo/__init__.py
Python creates todo module object
Python executes todo/__init__.py
Python checks/loads todo.tasks
Python finds todo/tasks.py
Python creates todo.tasks module object
Python stores it in sys.modules
Python executes todo/tasks.py
Python finds create_task in todo.tasks
Python binds create_task in main.py
main.py continues
```

Output includes:

```text
loading tasks
{'title': 'learn imports', 'done': False}
```

The import did real work.

It found, loaded, executed, cached, and bound.

---

# A Complete Circular Import Walkthrough

`a.py`:

```python
from b import value_b

value_a = "A"
```

`b.py`:

```python
from a import value_a

value_b = "B"
```

Run:

```python
import a
```

Step by step:

```text
1. Python starts importing a.
2. Python creates module object for a.
3. Python stores a in sys.modules.
4. Python executes a.py.
5. a.py says from b import value_b.
6. Python starts importing b.
7. Python creates module object for b.
8. Python stores b in sys.modules.
9. Python executes b.py.
10. b.py says from a import value_a.
11. Python finds a in sys.modules.
12. But a.py has not reached value_a = "A" yet.
13. value_a does not exist.
14. Import fails.
```

This is why the module is called partially initialized.

It exists.

It is not done executing.

---

# Better Circular Import Design

Instead of:

```text
a imports b
b imports a
```

move shared values:

```text
shared.py
a.py
b.py
```

`shared.py`:

```python
value_a = "A"
value_b = "B"
```

`a.py`:

```python
from shared import value_b
```

`b.py`:

```python
from shared import value_a
```

Dependency graph:

```text
a ──▶ shared
b ──▶ shared
```

No cycle.

This may look like a small mechanical fix.

It is actually a design improvement.

Shared concepts belong in a shared module.

Modules should not need each other just to reach common definitions.

---

# Good Import Design

Good import design has a few habits:

* Put imports near the top of the file.
* Keep import-time work light.
* Avoid circular dependencies.
* Use package-qualified imports consistently.
* Avoid shadowing standard library names.
* Keep package `__init__.py` small.
* Use `python -m` for package modules.
* Use virtual environments for project dependencies.
* Treat `sys.path` edits as exceptional.
* Read full tracebacks for import errors.

Imports are program architecture made visible.

When imports are clean, the program's dependency structure is easier to understand.

When imports are chaotic, the program becomes harder to reason about before any business logic even runs.

The import system is technical, but import design is human.

It helps the next reader.

---

# Common Mistake: Running From the Wrong Directory

Project:

```text
project/
    app/
        __init__.py
        cli.py
        tasks.py
```

Inside `cli.py`:

```python
from app.tasks import create_task
```

Works from:

```text
project/
```

with:

```text
python -m app.cli
```

May fail from:

```text
project/app/
```

with:

```text
python cli.py
```

Reason:

```text
Python's import search context changed
```

Fix the run command and working directory.

Do not immediately rewrite imports.

---

# Common Mistake: Local File Shadowing

Bad file name:

```text
json.py
```

Then:

```python
import json
```

may import your local file instead of the standard library `json`.

Debug:

```python
import json

print(json.__file__)
```

Fix:

```text
rename your file
remove stale __pycache__ if needed
run again from a clean context
```

Better file names:

```text
json_helpers.py
json_examples.py
data_serialization.py
```

Avoid standard library names for your own modules.

---

# Common Mistake: Expecting Imports to Reload

This does not reload a module:

```python
import settings
import settings
```

It reuses the cached module.

If you changed `settings.py` while a Python process is still running, another import usually does not pick up the change.

In scripts, restart the program.

In interactive sessions, you can use:

```python
import importlib
import settings

importlib.reload(settings)
```

But reload has sharp edges.

Restarting is cleaner when possible.

---

# Common Mistake: Import-Time Actions

Bad:

```python
# backup.py

run_backup()
```

Now:

```python
import backup
```

runs a backup.

Better:

```python
def run_backup():
    ...


def main():
    run_backup()


if __name__ == "__main__":
    main()
```

Now:

```python
import backup
```

only makes `run_backup` available.

Program action happens when explicitly requested.

---

# Common Mistake: Hiding Circular Imports With Local Imports Forever

Local imports can solve timing problems.

But if many functions contain imports like:

```python
def do_work():
    from other_module import something
```

because top-level imports always break, the design may be tangled.

Ask:

```text
Why do these modules depend on each other?
Can shared logic move to a third module?
Should one module own the dependency direction?
Are we mixing layers?
```

Local imports are a tool.

They are not a substitute for clear boundaries.

Use them deliberately.

---

# Common Mistake: Editing `sys.path` as Architecture

This can make code work temporarily:

```python
import sys

sys.path.append("../")
```

But it often creates fragile programs.

Problems:

* Depends on working directory.
* Breaks when files move.
* Hides package layout problems.
* Makes tests and deployment inconsistent.

Better:

* Use packages.
* Run from the project root.
* Use `python -m`.
* Install the project in a virtual environment when appropriate.

`sys.path` is useful to inspect.

It should rarely be your main design tool.

---

# Exercises

1. Create a file:

```python
# noisy.py

print("loading noisy")
value = 10
```

Then import it three times from another file:

```python
import noisy
import noisy
import noisy
```

Explain why the print usually appears once.

---

2. Inspect `sys.path`:

```python
import sys

for entry in sys.path:
    print(entry)
```

Which entry explains how your local module is found?

---

3. Inspect `sys.modules`:

```python
import sys
import math

print(sys.modules["math"])
```

What kind of object is stored there?

---

4. Create a local file named `random.py`.

Then try:

```python
import random
print(random.__file__)
```

Explain why this file name is dangerous.

After the exercise, rename the file to avoid future confusion.

---

5. Build a package:

```text
app/
    __init__.py
    cli.py
    tasks.py
```

Use a relative import inside `cli.py`:

```python
from .tasks import create_task
```

Run it incorrectly with:

```text
python app/cli.py
```

Then run it correctly with:

```text
python -m app.cli
```

Explain the difference.

---

6. Create a circular import with `a.py` and `b.py`.

Make each module import a name from the other before defining its own name.

Read the error message.

Where does it mention partial initialization?

---

7. Fix the circular import by moving shared values to `shared.py`.

Draw the dependency graph before and after.

---

8. Explain the difference between:

```python
import package.submodule
```

and:

```python
from package import submodule
```

What name is bound locally in each case?

---

9. Explain why this is risky at module top level:

```python
connect_to_database()
```

Where should this kind of action usually happen?

---

10. In your own words, describe the full import flow:

```text
check cache
search
load
execute
cache
bind
```

Use a small module example.

---

# Summary

In this chapter we learned:

* Importing is runtime behavior.
* An import statement searches/loads a module and binds a name.
* `sys.path` controls many top-level import search locations.
* The way Python is started affects import behavior.
* Import search is ordered, so shadowing can happen.
* `sys.modules` caches loaded modules by import name.
* Modules usually execute once per process during normal imports.
* Module-level state persists because modules are cached.
* Packages are imported step by step through dotted names.
* Package `__path__` helps Python find submodules.
* Finders locate modules; loaders load and execute them.
* Module specs describe how modules should be loaded.
* `__file__` can help reveal where a module came from.
* Circular imports can expose partially initialized modules.
* `python -m package.module` runs a module by import name with package context.
* Relative imports require known package context.
* Import-time side effects should be minimized.
* Editing `sys.path` is usually not a good architecture strategy.

Core model:

```text
import name
    check sys.modules
    search using import machinery
    create module object
    cache module object
    execute module code
    bind name in current namespace
```

Debugging model:

```text
What did Python try to import?
Where did it search?
What did it find?
What name was bound?
Did a module execute partially?
Is the active environment correct?
```

The import system is the bridge between code organization and running programs.

It turns files, packages, and installed libraries into live module objects.

---

# Preview of Chapter 41

Next we study namespaces.

We have already used the word many times:

```text
local namespace
global namespace
module namespace
package namespace
object attributes
```

Chapter 41 pulls these ideas together.

We will study:

* What a namespace is.
* How names map to objects.
* How local, global, built-in, module, and object namespaces differ.
* How imports bind names into namespaces.
* How attribute access relates to namespaces.
* Why namespaces prevent name collisions.
* How namespace thinking clarifies scope, modules, packages, and objects.

The transition is natural:

```text
the import system loads modules
namespaces explain where imported names live
```

Once namespaces are clear, virtual environments will make more sense because we will separate program names from environment-level dependency locations.
