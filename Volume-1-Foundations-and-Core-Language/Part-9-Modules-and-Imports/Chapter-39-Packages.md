# Chapter 39 — Packages

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a package is.
* How packages relate to modules.
* Why packages are usually directories.
* How packages group related modules.
* What `__init__.py` is for.
* How a package gets its own namespace.
* How to import modules from packages.
* How absolute imports work.
* How relative imports work at a beginner level.
* Why package layout affects maintainability.
* Why packages are not the same thing as installable distributions.
* How packages prepare us for the import system and virtual environments.

Chapter 38 introduced modules.

A module usually starts as one `.py` file.

Packages are the next layer.

They organize multiple modules under one shared name.

If a module is a named file of Python code, a package is a named collection of modules.

That small idea is how Python projects grow from:

```text
one file
```

to:

```text
many files with structure
```

---

# Concept Overview

A package is a way to organize related modules.

Most commonly, a package is a directory that contains Python files.

Example:

```text
todo/
    __init__.py
    tasks.py
    storage.py
    formatting.py
```

Here:

```text
todo
```

is a package.

Inside it:

```text
tasks.py
storage.py
formatting.py
```

are modules.

They can be imported with package-qualified names:

```python
import todo.tasks
import todo.storage
import todo.formatting
```

The package gives related modules a common namespace.

Instead of several unrelated names:

```text
tasks
storage
formatting
```

you get:

```text
todo.tasks
todo.storage
todo.formatting
```

That prefix tells the reader:

```text
these modules belong together
```

---

# Modules vs Packages

A module is usually one file.

Example:

```text
tasks.py
```

Import:

```python
import tasks
```

A package is usually one directory containing modules.

Example:

```text
todo/
    __init__.py
    tasks.py
```

Import:

```python
import todo.tasks
```

Simple model:

```text
module  -> file
package -> directory of modules
```

This model is good enough for now.

There are advanced details, such as namespace packages, that make the full story more flexible.

But the beginner model is:

```text
packages organize modules
```

That is the point to keep.

---

# Why Packages Exist

Modules solve the problem of large files.

Packages solve the problem of too many modules.

Imagine a small project:

```text
main.py
tasks.py
storage.py
formatting.py
validation.py
```

This is manageable.

Now imagine a larger project:

```text
main.py
users.py
user_storage.py
user_validation.py
tasks.py
task_storage.py
task_validation.py
reports.py
report_formatting.py
report_storage.py
auth.py
auth_tokens.py
auth_passwords.py
```

The root directory becomes crowded.

Packages let us group by responsibility:

```text
app/
    __init__.py
    users/
        __init__.py
        models.py
        storage.py
        validation.py
    tasks/
        __init__.py
        models.py
        storage.py
        validation.py
    reports/
        __init__.py
        formatting.py
        storage.py
```

Now the structure tells a story.

User-related code lives under `app.users`.

Task-related code lives under `app.tasks`.

Report-related code lives under `app.reports`.

Packages turn file organization into program architecture.

---

# A First Package

Create this structure:

```text
todo/
    __init__.py
    tasks.py
    formatting.py
main.py
```

`todo/tasks.py`:

```python
def create_task(title):
    return {"title": title.strip(), "done": False}
```

`todo/formatting.py`:

```python
def format_task(task):
    marker = "x" if task["done"] else " "
    return f"[{marker}] {task['title']}"
```

`main.py`:

```python
import todo.tasks
import todo.formatting


task = todo.tasks.create_task("learn packages")
print(todo.formatting.format_task(task))
```

Output:

```text
[ ] learn packages
```

The package name appears first:

```text
todo.tasks
todo.formatting
```

That tells us both modules belong to the `todo` package.

---

# What `__init__.py` Means

The file:

```text
__init__.py
```

marks a directory as a regular Python package.

Example:

```text
todo/
    __init__.py
    tasks.py
```

The name is unusual at first.

It has double underscores on both sides.

Pronounce it:

```text
dunder init
```

At a simple level:

```text
__init__.py says "this directory is a package"
```

It can be empty:

```python
# todo/__init__.py
```

An empty `__init__.py` is common.

It still matters because it defines the package boundary.

When Python imports the package, `__init__.py` can also execute.

So it should stay calm and intentional, just like module top-level code.

---

# Package Initialization

When you import a package, Python may execute the package's `__init__.py`.

Example:

```text
todo/
    __init__.py
    tasks.py
```

`todo/__init__.py`:

```python
print("loading todo package")
```

`main.py`:

```python
import todo
```

Output:

```text
loading todo package
```

The package itself is also a module object.

That object has a namespace.

Conceptually:

```text
todo ──▶ package module object
```

When you import:

```python
import todo.tasks
```

Python loads the package and then the submodule.

At a high level:

```text
load todo package
load todo.tasks module
bind names so todo.tasks can be accessed
```

We will study this in more detail in Chapter 40.

For now, remember:

```text
packages are imported too
```

---

# Packages Have Namespaces

A package has its own namespace.

Example:

```text
todo/
    __init__.py
    tasks.py
```

`todo/__init__.py`:

```python
APP_NAME = "Todo"
```

`main.py`:

```python
import todo

print(todo.APP_NAME)
```

Output:

```text
Todo
```

The package namespace contains `APP_NAME`.

If we also import a submodule:

```python
import todo.tasks
```

then the package can expose an attribute named `tasks`:

```python
todo.tasks
```

Conceptually:

```text
todo package object
    APP_NAME -> "Todo"
    tasks    -> todo.tasks module object
```

Packages fit the same object model as modules.

Names refer to package objects.

Package objects have attributes.

Those attributes can refer to submodules, functions, classes, constants, and other objects.

---

# Importing a Submodule

Given:

```text
todo/
    __init__.py
    tasks.py
```

You can write:

```python
import todo.tasks
```

Then use:

```python
task = todo.tasks.create_task("read")
```

The full name:

```text
todo.tasks.create_task
```

means:

```text
inside package todo
inside module tasks
find name create_task
```

This is explicit.

It is also long.

For frequently used modules, you can import with an alias:

```python
import todo.tasks as tasks

task = tasks.create_task("read")
```

This keeps the module boundary but shortens the call site.

Use aliases when they improve readability.

Do not use aliases that hide meaning.

---

# Importing Names From a Submodule

You can import a specific name:

```python
from todo.tasks import create_task
```

Then:

```python
task = create_task("read")
```

This binds `create_task` directly in the current module.

Conceptually:

```text
todo.tasks.create_task ──┐
                         ▼
create_task ─────────▶ function object
```

This is convenient.

But it removes the package/module prefix from the call site.

Compare:

```python
todo.tasks.create_task("read")
```

with:

```python
create_task("read")
```

The first is more explicit.

The second is shorter.

Beginner guidance:

```text
use explicit imports until the code becomes noisy
then import specific names when they are truly central
```

Readable code is the goal.

---

# Importing From a Package

Sometimes you want:

```python
from todo import tasks
```

Then:

```python
task = tasks.create_task("read")
```

This means:

```text
from package todo, import the submodule or attribute tasks
```

Depending on package structure and import behavior, Python may load the submodule or find an attribute exposed by the package.

At a beginner level, use this when it reads naturally:

```python
from todo import tasks
from todo import formatting
```

Then:

```python
task = tasks.create_task("read")
print(formatting.format_task(task))
```

This style keeps module names visible:

```text
tasks.create_task
formatting.format_task
```

but avoids repeating:

```text
todo.
```

It is often a nice balance inside application code.

---

# Absolute Imports

An absolute import starts from a top-level package or module name.

Example:

```python
from todo.tasks import create_task
```

This starts at:

```text
todo
```

Then goes to:

```text
tasks
```

Then imports:

```text
create_task
```

Another example:

```python
import todo.formatting
```

Absolute imports are clear because they show the full path from the package root.

Inside a project, absolute imports often make dependencies easy to understand.

Example:

```python
from app.users.validation import validate_user
```

This tells the reader exactly where `validate_user` comes from.

Absolute imports are usually the best default for application code.

---

# Relative Imports

A relative import starts from the current package location.

Example package:

```text
todo/
    __init__.py
    tasks.py
    formatting.py
```

Inside `todo/formatting.py`, you might write:

```python
from .tasks import create_task
```

The dot means:

```text
current package
```

So:

```python
from .tasks import create_task
```

means:

```text
from the tasks module next to me in this package, import create_task
```

Relative imports are useful inside packages.

They make it clear that modules are part of the same package.

But they only work in package context.

Do not use relative imports casually in standalone scripts.

We will explain the deeper rules in Chapter 40.

For now:

```text
absolute import -> starts from package root
relative import -> starts from current package position
```

---

# One Dot and Two Dots

Relative imports use dots.

One dot:

```python
from . import tasks
```

means:

```text
from the current package
```

Two dots:

```python
from .. import shared
```

means:

```text
from the parent package
```

Example:

```text
app/
    __init__.py
    shared.py
    users/
        __init__.py
        validation.py
```

Inside `app/users/validation.py`:

```python
from .. import shared
```

means:

```text
go up from app.users to app
import shared
```

Use relative imports carefully.

Too many dots can make code harder to read.

If a relative import feels like a maze, an absolute import may be clearer.

---

# Package Layout

Package layout is the directory structure of your Python code.

Small layout:

```text
project/
    main.py
    todo/
        __init__.py
        tasks.py
        formatting.py
```

Larger layout:

```text
project/
    app/
        __init__.py
        users/
            __init__.py
            models.py
            validation.py
        tasks/
            __init__.py
            models.py
            validation.py
        storage/
            __init__.py
            files.py
            database.py
```

Good layout answers:

```text
Where should this code live?
Where should I look for related behavior?
What depends on what?
```

Poor layout hides those answers.

There is no single perfect package layout for every project.

But there is a consistent principle:

```text
group code by responsibility
```

---

# Nested Packages

Packages can contain packages.

Example:

```text
app/
    __init__.py
    users/
        __init__.py
        models.py
        validation.py
```

Here:

```text
app
```

is a package.

```text
app.users
```

is also a package.

```text
app.users.models
```

is a module.

Import:

```python
from app.users.models import User
```

This path tells a story:

```text
application package
users area
models module
User name
```

Nested packages are useful when a project has multiple domains or layers.

But nesting too deeply can become painful.

If imports become:

```python
from company.product.backend.services.users.profiles.validation.rules import check_name
```

the structure may be too deep or too fragmented.

Package layout should help readers, not test their patience.

---

# `__init__.py` Can Re-Export Names

Sometimes a package's `__init__.py` exposes names from submodules.

Example:

```text
todo/
    __init__.py
    tasks.py
```

`todo/tasks.py`:

```python
def create_task(title):
    return {"title": title, "done": False}
```

`todo/__init__.py`:

```python
from .tasks import create_task
```

Now users can write:

```python
from todo import create_task
```

instead of:

```python
from todo.tasks import create_task
```

This is called re-exporting.

It can create a cleaner public API.

But do not re-export everything automatically.

Use it when the package should present a simple surface.

Ask:

```text
What names should users of this package reach for first?
```

Those names may belong in `__init__.py`.

---

# Keep `__init__.py` Light

Because `__init__.py` runs when the package is imported, keep it light.

Good:

```python
"""Tools for working with todo tasks."""

from .tasks import create_task
```

Risky:

```python
connect_to_database()
load_large_dataset()
start_background_worker()
```

Importing a package should usually not perform heavy work.

This is the same lesson from modules:

```text
imports should make tools available
program execution should be explicit
```

If `__init__.py` becomes large, it may be doing too much.

Most packages have a small `__init__.py`.

Some have an empty one.

Both are normal.

---

# Public Package API

A package can hide its internal layout behind a simpler public API.

Example:

```text
todo/
    __init__.py
    tasks.py
    formatting.py
    storage.py
```

Internal modules:

```python
# todo/tasks.py
def create_task(title):
    ...
```

```python
# todo/formatting.py
def format_task(task):
    ...
```

Package API:

```python
# todo/__init__.py
from .tasks import create_task
from .formatting import format_task
```

User code:

```python
from todo import create_task, format_task
```

This is convenient if those are the main features of the package.

But if a package exposes too many names from `__init__.py`, it becomes cluttered.

Good public APIs are curated.

They do not expose every internal helper.

They expose what users should rely on.

---

# Internal Modules

Some modules inside a package are internal implementation details.

Conventionally, names starting with underscore are internal.

Example:

```text
todo/
    __init__.py
    tasks.py
    _parsing.py
```

`_parsing.py` suggests:

```text
this module is for internal package use
```

Other code can still import it:

```python
import todo._parsing
```

Python does not strictly forbid it.

But the underscore tells readers:

```text
do not treat this as stable public API
```

This convention helps packages evolve.

Public modules are promises.

Internal modules are implementation details.

---

# Package Boundaries

A package boundary says:

```text
these modules belong together
```

It also suggests:

```text
code outside this package should interact through selected public names
```

Example:

```text
app/
    users/
        __init__.py
        models.py
        service.py
        _password_rules.py
```

Code outside `users` might use:

```python
from app.users.service import create_user
```

It probably should not use:

```python
from app.users._password_rules import _score_password
```

Package boundaries are not walls.

They are agreements.

They keep large programs understandable.

Without boundaries, every module reaches into every other module's details.

That becomes fragile.

---

# Packages and Object References

Packages fit the object/reference model.

Import:

```python
import todo.tasks
```

Conceptually:

```text
todo ──▶ package module object
          │
          └── tasks ──▶ module object
                         │
                         └── create_task ──▶ function object
```

When code calls:

```python
todo.tasks.create_task("read")
```

Python follows references:

```text
name todo
attribute tasks
attribute create_task
call function
```

Packages are not magic containers outside Python's object model.

They are module objects arranged in a hierarchy.

This is why the earlier chapters about names, references, and namespaces continue to matter.

---

# Packages and Namespaces

A package name prevents collisions.

Imagine two modules:

```text
users/models.py
tasks/models.py
```

Both files are named:

```text
models.py
```

Without packages, that would be confusing.

With packages:

```text
app.users.models
app.tasks.models
```

These are distinct module names.

They can both define a class named `Model`:

```text
app.users.models.Model
app.tasks.models.Model
```

The full package path disambiguates them.

This is one reason packages are essential in larger programs.

They give names context.

---

# Packages and Dependencies

Packages also show dependencies.

Example:

```python
from app.users.validation import validate_user
from app.storage.database import save_record
```

This tells us the current module depends on:

```text
app.users.validation
app.storage.database
```

Clean package design often has clear dependency direction.

Example:

```text
app.users -> app.storage
app.tasks -> app.storage
app.reports -> app.users and app.tasks
```

Messy design often has circular dependencies:

```text
app.users -> app.reports
app.reports -> app.users
```

Some dependency cycles are manageable.

Many are signs that responsibilities need to be separated.

Packages make dependency structure visible.

That visibility is part of their value.

---

# Package Names and Project Names

The package name is the name used in imports.

Example:

```text
quainy_tools/
    __init__.py
```

Import:

```python
import quainy_tools
```

The project directory may have a different name:

```text
python-you-beauty/
    quainy_tools/
        __init__.py
```

The project name:

```text
python-you-beauty
```

is the folder containing the project.

The package name:

```text
quainy_tools
```

is what Python imports.

These can differ.

This becomes especially important when packaging code for installation.

For now, remember:

```text
directory containing project != always import package name
```

---

# Package vs Distribution

The word package can be confusing.

In everyday Python, people use "package" in two related ways.

Meaning one:

```text
import package
```

This is a Python package in code:

```text
todo/
    __init__.py
```

Meaning two:

```text
install package
```

This often means an installable distribution from a package index.

Example:

```text
pip install requests
```

The installed distribution provides importable packages and modules.

The distribution name and import name are often the same, but not always.

Example idea:

```text
install name -> something on package index
import name  -> Python package/module name
```

We will study packaging later in the book.

For this chapter:

```text
package means importable code organization
```

---

# Regular Packages and Namespace Packages

Most beginner packages use `__init__.py`.

Example:

```text
todo/
    __init__.py
    tasks.py
```

This is a regular package.

Python also supports namespace packages, which can exist without `__init__.py`.

Namespace packages are useful for advanced cases where one package's modules can be spread across multiple directories or distributions.

You do not need them for ordinary beginner projects.

Practical guidance:

```text
use __init__.py for your packages while learning and for most normal project layouts
```

This makes package boundaries explicit.

We mention namespace packages here only so you are not surprised later if you see a directory imported without `__init__.py`.

The core model remains:

```text
packages group modules under a shared name
```

---

# The `src` Layout Preview

Many professional Python projects use a `src` layout.

Example:

```text
project/
    pyproject.toml
    src/
        todo/
            __init__.py
            tasks.py
    tests/
        test_tasks.py
```

The import package is:

```text
todo
```

But the code lives under:

```text
src/todo
```

This layout helps avoid accidentally importing code from the current working directory instead of the installed package.

That concern is more advanced.

We will return to it in the software engineering and packaging parts of the book.

For now, know that both layouts exist:

Simple learning layout:

```text
project/
    todo/
        __init__.py
```

Professional packaging layout:

```text
project/
    src/
        todo/
            __init__.py
```

Do not worry if `src` layout feels early.

The important idea is still the package path:

```text
todo
```

---

# Running Code Inside a Package

A common beginner mistake is running a module inside a package directly by file path and then being surprised by imports.

Example:

```text
todo/
    __init__.py
    tasks.py
    cli.py
```

Inside `todo/cli.py`:

```python
from .tasks import create_task
```

If you run:

```text
python todo/cli.py
```

the relative import may fail because Python is treating `cli.py` as a script, not as part of the package in the expected way.

A package-aware way is:

```text
python -m todo.cli
```

The `-m` flag runs a module by module name.

This lets Python understand the package context.

We will study this more in Chapter 40.

For now:

```text
package modules are best run by module name, not by raw file path, when relative imports are involved
```

---

# `__main__.py`

Packages can define a special file:

```text
__main__.py
```

Example:

```text
todo/
    __init__.py
    __main__.py
    tasks.py
```

Then you can run:

```text
python -m todo
```

Python executes:

```text
todo/__main__.py
```

This lets a package behave like a command.

Simple `todo/__main__.py`:

```python
from .tasks import create_task


def main():
    task = create_task("learn __main__")
    print(task)


if __name__ == "__main__":
    main()
```

This is not needed for every package.

It is useful when a package should be runnable.

Example:

```text
python -m package_name
```

can become a clean entry point.

---

# Package Data Preview

Packages often contain Python code.

Sometimes they also contain data files.

Examples:

* Templates.
* Configuration defaults.
* Static assets.
* Example data.
* Text resources.

Example structure:

```text
reports/
    __init__.py
    builder.py
    templates/
        summary.txt
```

Handling package data correctly is an advanced packaging topic.

Do not assume a data file can always be opened by a simple relative path.

That may work in a local script and fail after installation.

We will study this later.

For this chapter, simply know:

```text
packages can organize code and sometimes related data
```

But Python code organization is the main focus here.

---

# A Complete Small Package

Project:

```text
todo_project/
    main.py
    todo/
        __init__.py
        tasks.py
        formatting.py
        storage.py
```

`todo/tasks.py`:

```python
def create_task(title):
    title = title.strip()

    if title == "":
        raise ValueError("title cannot be empty")

    return {"title": title, "done": False}
```

`todo/formatting.py`:

```python
def format_task(task):
    marker = "x" if task["done"] else " "
    return f"[{marker}] {task['title']}"
```

`todo/storage.py`:

```python
def save_task(task):
    print("saving", task["title"])
```

`todo/__init__.py`:

```python
"""Todo package."""
```

`main.py`:

```python
from todo import formatting
from todo import storage
from todo import tasks


def main():
    task = tasks.create_task("learn packages")
    print(formatting.format_task(task))
    storage.save_task(task)


if __name__ == "__main__":
    main()
```

Output:

```text
[ ] learn packages
saving learn packages
```

The package groups task-related modules.

The main program coordinates them.

This is the beginning of application structure.

---

# A Package With a Public API

We can make the package easier to use by re-exporting selected names.

`todo/__init__.py`:

```python
"""Todo package."""

from .formatting import format_task
from .tasks import create_task
```

Now `main.py` can write:

```python
from todo import create_task, format_task
from todo.storage import save_task


def main():
    task = create_task("learn packages")
    print(format_task(task))
    save_task(task)


if __name__ == "__main__":
    main()
```

This makes the common API shorter:

```text
todo.create_task
todo.format_task
```

But it also creates a responsibility:

```text
__init__.py now defines part of the public package surface
```

If other code relies on:

```python
from todo import create_task
```

then removing that re-export later can break users.

Public APIs are promises.

Make them intentionally.

---

# Common Mistake: Treating Packages as Random Folders

A package should not be just a folder full of unrelated files.

Bad:

```text
app/
    __init__.py
    database.py
    image_resizing.py
    invoice_pdf.py
    password_hashing.py
    weather_api.py
```

Maybe this is still too broad.

Better:

```text
app/
    __init__.py
    auth/
        __init__.py
        passwords.py
    billing/
        __init__.py
        invoices.py
        pdf.py
    media/
        __init__.py
        images.py
    weather/
        __init__.py
        client.py
```

The better layout groups by responsibility.

Packages should help a reader predict where code belongs.

If a package name does not help prediction, reconsider it.

---

# Common Mistake: Heavy `__init__.py`

This is risky:

```python
# app/__init__.py

connect_database()
load_config_from_network()
start_scheduler()
```

Why?

Because importing `app` now performs major work.

Any code that simply wants one utility from the package may accidentally start the whole application.

Better:

```python
# app/__init__.py

"""Application package."""
```

Then:

```python
# app/main.py

def main():
    connect_database()
    start_scheduler()
```

Keep package initialization lightweight.

Execution belongs in explicit entry points.

---

# Common Mistake: Re-Exporting Too Much

It can be tempting to put everything into `__init__.py`:

```python
from .a import *
from .b import *
from .c import *
```

This makes the package surface unclear.

Problems:

* Name collisions.
* Slow imports.
* Hidden dependencies.
* Harder documentation.
* Public API becomes accidental.

Better:

```python
from .tasks import create_task
from .formatting import format_task
```

Curate the package API.

Expose the names users should reach for.

Leave internal helpers inside internal modules.

---

# Common Mistake: Deep Package Nesting Too Early

Some projects begin with too much structure:

```text
app/
    core/
        domain/
            services/
                users/
                    validation/
                        rules.py
```

This may be appropriate in a very large system.

But for a small project, it creates navigation overhead.

Start with structure that fits the problem.

Small project:

```text
todo/
    tasks.py
    storage.py
    formatting.py
```

Growing project:

```text
todo/
    tasks/
        models.py
        service.py
    storage/
        files.py
        database.py
```

Let structure grow with real complexity.

Do not create deep folders just to feel professional.

Professional structure is useful structure.

---

# Common Mistake: Relative Import Maze

This is hard to read:

```python
from ....shared.formatting.names import format_name
```

Many dots make the reader count directories mentally.

Sometimes relative imports are elegant:

```python
from .tasks import create_task
```

Sometimes absolute imports are clearer:

```python
from app.shared.formatting.names import format_name
```

Rule of thumb:

```text
use relative imports for nearby package siblings
use absolute imports when the path communicates architecture better
```

If imports feel confusing, the package layout may need attention.

---

# Common Mistake: Running Package Files Directly

Suppose:

```text
app/
    __init__.py
    cli.py
    tasks.py
```

`app/cli.py`:

```python
from .tasks import create_task
```

Running:

```text
python app/cli.py
```

can fail with an import error because relative imports need package context.

Use:

```text
python -m app.cli
```

This tells Python:

```text
run the module app.cli
```

not merely:

```text
run this file path
```

This distinction becomes important as soon as packages and relative imports enter the program.

---

# Common Mistake: Confusing Install Name and Import Name

Later you will install third-party code.

Sometimes the install command name and import name differ.

Example idea:

```text
pip install some-project-name
```

but:

```python
import some_package_name
```

This can confuse beginners.

The install name identifies a distribution.

The import name identifies a module or package inside Python.

They often match.

They do not have to.

This book will explain packaging and installation later.

For now:

```text
package in imports
distribution in installation
```

Do not treat those words as always identical.

---

# Design Guidance

When creating packages, ask:

```text
What responsibility does this package have?
What modules belong inside it?
Which names are public?
Which modules are internal?
Should __init__.py be empty or expose selected names?
Are imports easy to understand?
Is the package too broad?
Is the package too deeply nested?
Can this package be imported without side effects?
```

Good package design is not about making many folders.

It is about creating useful boundaries.

A useful boundary helps readers answer:

```text
where does this behavior belong?
what can other code depend on?
what is implementation detail?
```

Those questions are the beginning of architecture.

---

# Exercises

1. Create this package:

```text
todo/
    __init__.py
    tasks.py
```

Put this in `tasks.py`:

```python
def create_task(title):
    return {"title": title, "done": False}
```

Import it from `main.py` using:

```python
import todo.tasks
```

Then call:

```python
todo.tasks.create_task("read")
```

Explain each part of the dotted name.

---

2. Explain what `__init__.py` does at a beginner level.

Why can it be empty and still useful?

---

3. Compare these imports:

```python
import todo.tasks
```

```python
from todo import tasks
```

```python
from todo.tasks import create_task
```

What name becomes available in the current module in each case?

---

4. Create a package:

```text
reports/
    __init__.py
    formatting.py
    builder.py
```

Inside `builder.py`, import from `formatting.py` using a relative import.

Then rewrite it using an absolute import.

Which version is clearer?

---

5. Explain why heavy work in `__init__.py` can be dangerous.

Use an example involving a database connection or network call.

---

6. What is the difference between a package and a distribution?

Explain using:

```text
import name
```

and:

```text
install name
```

---

7. Why might `python app/cli.py` fail when `cli.py` uses relative imports?

What command is usually better for running a package module?

---

8. Design a package layout for a small note-taking application.

Include modules for:

* Notes.
* Storage.
* Formatting.
* Command-line behavior.

Explain why each file belongs where it does.

---

9. Explain why this package might be too vague:

```text
utils/
    __init__.py
    dates.py
    emails.py
    images.py
    database.py
```

How could you reorganize it by responsibility?

---

10. In your own words, explain:

```text
module -> one file of names
package -> directory of modules
```

Then describe one situation where a program should move from modules to packages.

---

# Summary

In this chapter we learned:

* A package organizes related modules.
* A package is usually a directory.
* A regular package usually contains `__init__.py`.
* `__init__.py` marks a package and can run package initialization code.
* Packages have namespaces.
* Submodules can be imported with dotted names.
* Absolute imports start from a top-level package or module name.
* Relative imports start from the current package position.
* Package layout should reflect responsibility.
* Nested packages help organize larger systems.
* `__init__.py` can re-export selected names as a public API.
* `__init__.py` should usually stay lightweight.
* Internal modules can use leading underscores by convention.
* Packages define boundaries and dependency structure.
* A Python package is not exactly the same thing as an installable distribution.
* Package modules with relative imports are often best run with `python -m package.module`.

Core model:

```text
module:
    one file
    one namespace of names

package:
    directory of modules
    package namespace
    dotted import paths
```

Architecture model:

```text
files become modules
related modules become packages
related packages become applications and libraries
```

Packages are the second major step from scripts to professional Python structure.

---

# Preview of Chapter 40

Next we study the import system.

Modules and packages tell us what Python code can be organized into.

The import system explains how Python finds and loads that code.

Chapter 40 will study:

* What happens when an import statement runs.
* How Python searches for modules.
* What `sys.path` means.
* Why the current directory matters.
* How import caching works.
* What `sys.modules` stores.
* Why circular imports happen.
* Why module execution happens only once in normal imports.
* How `python -m` relates to module execution.

The transition is direct:

```text
modules and packages define code structure
the import system loads that structure into a running program
```

Now that we know what modules and packages are, we can understand how Python actually finds them.
