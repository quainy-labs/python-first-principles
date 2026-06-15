# Chapter 38 — Modules

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a module is.
* Why a Python file can become a module.
* How module-level names work.
* Why modules have their own namespaces.
* What happens at a high level when a module is imported.
* Why importing a module executes code.
* How module objects relate to names and references.
* Why modules help organize larger programs.
* How to split code across files.
* Why module-level side effects should be controlled.
* What `__name__` means at a beginner level.
* Why `if __name__ == "__main__":` is useful.
* How modules prepare us for packages and the import system.

Part VIII taught us how objects live.

Part IX teaches us how programs grow.

So far, most examples have fit inside one file or one snippet.

Real Python programs do not stay that small.

They become groups of files.

Those files need names.

They need boundaries.

They need a way to share functions, classes, constants, and values.

That is what modules begin to solve.

---

# Concept Overview

A module is a Python object that holds Python code and names.

Most commonly, a module comes from a `.py` file.

Example:

```text
calculator.py
```

That file can become a module named:

```text
calculator
```

If `calculator.py` contains:

```python
def add(a, b):
    return a + b
```

another file can use it:

```python
import calculator

result = calculator.add(10, 20)
print(result)
```

Output:

```text
30
```

The core idea:

```text
file -> module
module -> namespace
namespace -> names
names -> objects
```

Modules let code be organized by responsibility.

Instead of one large file, we can create multiple focused files.

---

# Why Modules Exist

Small programs can live in one file.

Example:

```python
def add(a, b):
    return a + b


def subtract(a, b):
    return a - b


print(add(10, 5))
print(subtract(10, 5))
```

This is fine.

But as programs grow, one file becomes hard to read.

Imagine one file containing:

* User input handling.
* Validation.
* Database logic.
* File parsing.
* Logging.
* Business rules.
* Tests.
* Command-line behavior.

The file becomes crowded.

Modules let us separate responsibilities:

```text
input.py
validation.py
database.py
logging_setup.py
reports.py
main.py
```

Each module can hold related code.

This gives the program structure.

Good modules make a program easier to:

* Read.
* Test.
* Reuse.
* Debug.
* Extend.
* Discuss.

Modules are one of the first tools for turning scripts into software.

---

# A Python File as a Module

Suppose you create this file:

```text
math_tools.py
```

Inside it:

```python
def square(number):
    return number * number


def cube(number):
    return number * number * number
```

Now create another file:

```text
main.py
```

Inside `main.py`:

```python
import math_tools

print(math_tools.square(4))
print(math_tools.cube(3))
```

When `main.py` runs, it imports the module `math_tools`.

Then it accesses names inside that module:

```python
math_tools.square
math_tools.cube
```

This dot syntax means:

```text
look inside the module object named math_tools
find the name square
```

The file `math_tools.py` provides code.

The module `math_tools` provides a namespace where that code's names live.

---

# Module Namespaces

A namespace is a mapping from names to objects.

We have already seen namespaces in smaller forms:

* Local function namespaces.
* Global names in a file.
* Object attributes.

A module has a namespace too.

Example file:

```python
# settings.py

APP_NAME = "Task Tracker"
DEBUG = True


def show_settings():
    print(APP_NAME, DEBUG)
```

The module namespace contains:

```text
APP_NAME      -> "Task Tracker"
DEBUG         -> True
show_settings -> function object
```

Another file can access those names:

```python
import settings

print(settings.APP_NAME)
settings.show_settings()
```

The module keeps its names grouped.

This avoids dumping everything into one shared global space.

Instead of:

```python
APP_NAME
```

you write:

```python
settings.APP_NAME
```

That prefix tells the reader where the name comes from.

---

# Modules Are Objects

A module is not just a file.

When imported, it becomes a module object at runtime.

Example:

```python
import settings

print(settings)
print(type(settings))
```

You may see output like:

```text
<module 'settings' from '.../settings.py'>
<class 'module'>
```

The exact path depends on your machine.

The important idea:

```text
settings is a name
settings refers to a module object
the module object has attributes
```

Conceptually:

```text
settings ──▶ module object
               │
               ├── APP_NAME
               ├── DEBUG
               └── show_settings
```

This connects modules back to our object model.

Modules are objects.

Names refer to modules.

Modules contain names that refer to other objects.

---

# Importing a Module

When Python imports a module, it does more than copy text.

At a high level, importing means:

```text
1. find the module
2. create or reuse a module object
3. execute the module's code if needed
4. store names in the module namespace
5. bind a name in the importing code
```

This is a simplified model.

We will study the import system in more detail later in Part IX.

For now, the most important lesson is:

```text
import executes module code
```

Suppose:

```python
# greetings.py

print("loading greetings")


def hello(name):
    return f"Hello, {name}"
```

Then:

```python
import greetings
```

prints:

```text
loading greetings
```

Why?

Because Python executed the top-level code in `greetings.py`.

The function definition also executed.

Executing a `def` statement creates a function object and binds it to a name.

So after import:

```text
greetings.hello -> function object
```

---

# Import Is Not Textual Inclusion

Some languages use mechanisms that behave like inserting one file's text into another file.

Python imports do not work that way.

This is not the right mental model:

```text
copy all text from greetings.py into main.py
```

Better:

```text
load greetings.py as a module object
execute its top-level code
bind the module object to the name greetings
```

Example:

```python
import greetings

message = greetings.hello("Ada")
```

The name `hello` does not appear directly in the importing file's global namespace.

It lives inside the `greetings` module namespace.

That is why we write:

```python
greetings.hello
```

The module boundary remains visible.

This is a good thing.

Visible boundaries help readers understand where names come from.

---

# Top-Level Code

Top-level code is code that is not inside a function, class, or other nested block.

Example:

```python
# report.py

print("creating report tools")

TITLE = "Monthly Report"


def build_report():
    return TITLE
```

All of this is top-level:

```python
print("creating report tools")
TITLE = "Monthly Report"
def build_report():
    ...
```

When the module is imported, top-level code runs.

That includes:

* Assignments.
* Function definitions.
* Class definitions.
* Loops.
* Conditional statements.
* Function calls.
* Print statements.

This is why module design matters.

You usually want imports to define names, not perform surprising work.

Good top-level code:

```python
APP_NAME = "Todo CLI"


def parse_task(text):
    return text.strip()
```

Risky top-level code:

```python
delete_old_files()
send_email()
start_server()
ask_user_for_password()
```

Importing a module should usually be calm and predictable.

---

# Module-Level Names

Names assigned at the top level of a module are module-level names.

Example:

```python
# config.py

MAX_RETRIES = 3
TIMEOUT_SECONDS = 10
```

These names live in the `config` module namespace.

Another file:

```python
import config

print(config.MAX_RETRIES)
```

The name lookup:

```text
config -> module object
MAX_RETRIES -> name inside module object
```

Module-level names are often used for:

* Constants.
* Functions.
* Classes.
* Shared configuration.
* Module-private helper values.

But be careful.

Module-level mutable objects can become global state.

Example:

```python
# store.py

items = []


def add_item(item):
    items.append(item)
```

Any file that imports `store` can affect the same `items` list.

This may be useful.

It may also become hard to reason about.

Module-level state should be intentional.

---

# Accessing Module Attributes

When you write:

```python
import config
```

you can access names with dot syntax:

```python
config.MAX_RETRIES
```

The dot means attribute access.

Modules have attributes just like many other objects.

Example:

```python
import config

print(config.MAX_RETRIES)
config.MAX_RETRIES = 5
print(config.MAX_RETRIES)
```

This can work because module attributes can often be rebound.

But changing imported module attributes from another file can be confusing.

It creates shared mutable program state.

Use module attributes for clear constants and functions.

Be cautious when mutating module attributes from outside the module.

Better design often uses functions:

```python
config.set_timeout(5)
```

or explicit objects:

```python
settings.timeout = 5
```

The right choice depends on the program.

The key is clarity.

---

# Importing Specific Names

Python also allows:

```python
from math_tools import square
```

Then:

```python
print(square(4))
```

This binds the name `square` directly in the importing module.

Conceptually:

```text
math_tools module has name square -> function object
current module gets name square -> same function object
```

There is still one function object.

But now two module namespaces can refer to it:

```text
math_tools.square ──┐
                    ▼
square ─────────▶ function object
```

This form can be convenient.

But it hides the module prefix.

Compare:

```python
math_tools.square(4)
```

with:

```python
square(4)
```

The first tells you where `square` comes from.

The second is shorter.

Both are useful in different contexts.

---

# `import module` vs `from module import name`

There are two common styles.

Style one:

```python
import math_tools

result = math_tools.square(4)
```

Advantages:

* Keeps the module name visible.
* Reduces name collisions.
* Makes code easier to trace.

Style two:

```python
from math_tools import square

result = square(4)
```

Advantages:

* Shorter call sites.
* Convenient for frequently used names.
* Useful when importing one or two clear tools.

Both styles are valid.

Beginner guidance:

```text
Prefer import module when it improves clarity.
Use from module import name when the imported name is specific and obvious.
```

Avoid importing too many names directly.

If a file starts with many direct imports, readers may struggle to know where names came from.

---

# Avoid `from module import *`

Python allows:

```python
from math_tools import *
```

This imports many public names directly into the current namespace.

It is usually a poor choice in normal code.

Why?

Because it hides where names came from.

Example:

```python
from tools import *

result = clean(data)
```

Where did `clean` come from?

The current file?

The imported module?

Another star import?

This makes code harder to read.

It can also cause name collisions.

Example:

```python
from module_a import *
from module_b import *
```

If both modules define `load`, one can overwrite the other in the current namespace.

General rule:

```text
avoid star imports in ordinary application code
```

There are special cases in interactive exploration and some package design, but they are not the default.

---

# Aliasing Imports

You can give an imported module or name a local alias.

Example:

```python
import statistics as stats
```

Then:

```python
mean = stats.mean([10, 20, 30])
```

Alias syntax:

```python
import module_name as local_name
```

You can also alias specific imports:

```python
from math_tools import square as sq
```

Then:

```python
print(sq(5))
```

Aliases are useful when:

* A module name is long.
* A conventional alias exists.
* Two imported names would conflict.
* A local name improves readability.

But aliases should not be cryptic.

Bad:

```python
import customer_reports as crx
```

Maybe `reports` would be clearer:

```python
import customer_reports as reports
```

An alias should help the reader, not merely save characters.

---

# Standard Library Modules

Python includes many modules in its standard library.

The standard library is the set of modules that come with Python.

Examples:

```python
import math
import random
import pathlib
import datetime
```

We have already seen:

```python
import gc
import weakref
```

Those modules provide tools for garbage collection and weak references.

In this chapter, we are not deeply studying each library.

We are studying the module idea.

The module idea is the same whether the module is:

* A file you wrote.
* A standard library module.
* A third-party library module.
* A package module.

Importing gives your code access to names defined somewhere else.

---

# User Modules

A user module is a module you write.

Example project:

```text
project/
    main.py
    tasks.py
```

`tasks.py`:

```python
def normalize_task(text):
    return text.strip()


def is_empty_task(text):
    return normalize_task(text) == ""
```

`main.py`:

```python
import tasks

raw = "  learn modules  "
task = tasks.normalize_task(raw)

if not tasks.is_empty_task(task):
    print(task)
```

This is simple but important.

The `tasks` module owns task-related functions.

The `main` module uses them.

A good module name tells the reader what kind of responsibility lives inside.

---

# Organizing By Responsibility

A module should usually group related ideas.

Good module:

```text
tasks.py
```

contains:

* `create_task`
* `complete_task`
* `format_task`
* `parse_task`

Less clear module:

```text
helpers.py
```

contains:

* `create_task`
* `send_email`
* `parse_date`
* `connect_database`
* `draw_chart`

The name `helpers` often becomes a drawer for unrelated code.

It is not always wrong, but it can become vague.

Prefer modules named after responsibilities:

```text
tasks.py
users.py
reports.py
storage.py
validation.py
formatting.py
```

Good module boundaries reduce mental load.

They let a reader predict where code should live.

---

# A Module as an API

A module exposes names.

Those names form a small API.

API means application programming interface.

At this level, think:

```text
the names other code is expected to use
```

Example:

```python
# temperature.py

def celsius_to_fahrenheit(celsius):
    return celsius * 9 / 5 + 32


def fahrenheit_to_celsius(fahrenheit):
    return (fahrenheit - 32) * 5 / 9
```

The module API is:

```text
temperature.celsius_to_fahrenheit
temperature.fahrenheit_to_celsius
```

Another file can use:

```python
import temperature

print(temperature.celsius_to_fahrenheit(0))
```

Good module APIs are:

* Clear.
* Small enough to understand.
* Named consistently.
* Focused on the module's purpose.

A module is not just a storage place.

It is a boundary.

---

# Private-Looking Names

Python has a convention for names that start with an underscore.

Example:

```python
# temperature.py

def _round_temperature(value):
    return round(value, 1)


def celsius_to_fahrenheit(celsius):
    return _round_temperature(celsius * 9 / 5 + 32)
```

The leading underscore suggests:

```text
this is an internal helper
other modules should usually not use it directly
```

This is a convention, not a strict access lock.

Another file can still do:

```python
import temperature

temperature._round_temperature(12.345)
```

But it should usually avoid doing that.

The underscore tells readers:

```text
this name is not part of the intended public API
```

Python relies heavily on readable conventions.

Respecting those conventions makes code easier to maintain.

---

# Module Constants

Module-level constants are common.

Example:

```python
# limits.py

MAX_USERNAME_LENGTH = 30
MIN_PASSWORD_LENGTH = 12
```

Use:

```python
import limits

if len(username) > limits.MAX_USERNAME_LENGTH:
    print("username too long")
```

Python does not enforce constants.

This can still run:

```python
limits.MAX_USERNAME_LENGTH = 100
```

The uppercase name is a convention.

It means:

```text
this value should be treated as constant
```

Constants belong in modules when they are shared across related code.

But do not turn every value into a module constant.

Use constants for values that have a clear meaning and are reused.

---

# Module State

Module state means values stored in module-level variables that can change.

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

The module remembers `count`.

This can be useful.

It can also create hidden coupling.

Any code importing `counter` shares the same module object and the same `count`.

Stateful modules should be designed carefully.

Before using module state, ask:

```text
Should this value be global to the whole program?
Would passing an object be clearer?
Will tests need to reset this state?
```

Module state is powerful, but it should not be accidental.

---

# Import Caching

When a module is imported, Python usually does not execute it again for every later import.

At a high level, Python caches imported modules.

Example:

```python
# once.py

print("module executed")

value = 10
```

Another file:

```python
import once
import once
```

You usually see:

```text
module executed
```

only once.

Why?

Because Python remembers that the module was already loaded.

Later imports reuse the existing module object.

This matters for module state:

```python
import counter
import counter
```

Both imports refer to the same module object.

We will study the import cache in more detail in Chapter 40.

For now:

```text
importing a module usually executes it once per process
later imports reuse it
```

---

# `__name__`

Every module has a special name called `__name__`.

Example:

```python
print(__name__)
```

If a file is run directly:

```text
python main.py
```

then inside that file:

```text
__name__ == "__main__"
```

If the file is imported as a module:

```python
import main
```

then inside that module:

```text
__name__ == "main"
```

This distinction lets one file behave differently when:

* It is run as the program entry point.
* It is imported as a module.

This is one of the first important module patterns.

---

# The Main Guard

You will often see:

```python
if __name__ == "__main__":
    main()
```

This is called the main guard.

Example:

```python
# app.py

def greet(name):
    return f"Hello, {name}"


def main():
    print(greet("Ada"))


if __name__ == "__main__":
    main()
```

When you run:

```text
python app.py
```

the file's `__name__` is `"__main__"`.

So `main()` runs.

When another file imports it:

```python
import app
```

the function `greet` becomes available:

```python
print(app.greet("Grace"))
```

But `main()` does not run automatically.

This prevents import side effects.

---

# Why the Main Guard Matters

Without a main guard:

```python
# app.py

def greet(name):
    return f"Hello, {name}"


print(greet("Ada"))
```

Now importing `app` prints:

```text
Hello, Ada
```

That may surprise the importing code.

Example:

```python
# test_app.py

import app

def test_greet():
    assert app.greet("Grace") == "Hello, Grace"
```

The test imports the module to access `greet`.

But the module prints during import.

This is noisy and sometimes harmful.

With a main guard:

```python
if __name__ == "__main__":
    print(greet("Ada"))
```

the print only happens when running the file directly.

Importing becomes safe.

---

# Designing Import-Friendly Modules

An import-friendly module can be imported without doing surprising work.

Good import-time behavior:

* Define functions.
* Define classes.
* Define constants.
* Prepare small local lookup tables.

Risky import-time behavior:

* Start servers.
* Open network connections.
* Delete files.
* Prompt for user input.
* Run long computations.
* Write to databases.
* Print large output.

Sometimes import-time work is necessary.

But it should be intentional.

General rule:

```text
importing a module should make tools available
running a program should perform the program's action
```

The main guard helps preserve that difference.

---

# Splitting a Script Into Modules

Start with one script:

```python
def normalize(text):
    return text.strip().lower()


def is_valid(text):
    return normalize(text) != ""


def main():
    task = input("Task: ")
    if is_valid(task):
        print(normalize(task))


main()
```

As it grows, split responsibilities.

`tasks.py`:

```python
def normalize(text):
    return text.strip().lower()


def is_valid(text):
    return normalize(text) != ""
```

`main.py`:

```python
import tasks


def main():
    task = input("Task: ")
    if tasks.is_valid(task):
        print(tasks.normalize(task))


if __name__ == "__main__":
    main()
```

Now task logic is separate from command-line interaction.

This makes `tasks.py` easier to test.

It also makes `main.py` easier to understand.

---

# Testing Modules

Modules make testing easier because logic can be imported without running the whole program.

Example:

```python
# tasks.py

def normalize(text):
    return text.strip().lower()
```

Test:

```python
# test_tasks.py

import tasks


def test_normalize():
    assert tasks.normalize("  Python  ") == "python"
```

The test imports `tasks`.

It calls the function directly.

It does not need to run the command-line program.

This is one reason the main guard matters.

If `tasks.py` asks for input during import, testing becomes painful.

Good modules separate:

```text
definitions
program execution
```

Definitions should be importable.

Program execution should be explicit.

---

# Module Names and File Names

For a simple module file:

```text
math_tools.py
```

the module name is usually:

```text
math_tools
```

Import:

```python
import math_tools
```

Do not include `.py` in the import:

```python
import math_tools.py
```

That is wrong.

The file is named:

```text
math_tools.py
```

The module is imported as:

```text
math_tools
```

Choose file names that are valid module names.

Good:

```text
math_tools.py
user_reports.py
config.py
```

Bad:

```text
math-tools.py
user reports.py
2026-report.py
```

Use lowercase names with underscores.

This keeps imports clean.

---

# Avoid Shadowing Standard Library Modules

Be careful with module file names.

If you create a file named:

```text
random.py
```

and then write:

```python
import random
```

Python may import your file instead of the standard library module you expected.

Similarly risky names:

```text
math.py
json.py
datetime.py
email.py
typing.py
```

This is called shadowing.

It can lead to confusing errors.

Example:

```text
AttributeError: module 'random' has no attribute 'choice'
```

because your local `random.py` was imported, not Python's real `random` module.

Beginner rule:

```text
do not name your files after standard library modules or installed packages
```

Use more specific names:

```text
random_names.py
json_helpers.py
date_formatting.py
```

---

# Circular Imports

A circular import happens when modules import each other.

Example:

```python
# users.py

import reports


def get_user():
    return "Ada"
```

```python
# reports.py

import users


def build_report():
    return users.get_user()
```

Now:

```python
import users
```

Python starts loading `users`.

`users` imports `reports`.

`reports` imports `users`.

But `users` may not be fully loaded yet.

This can create confusing errors.

Circular imports often mean module responsibilities are tangled.

Common fixes:

* Move shared code to a third module.
* Move imports inside functions when appropriate.
* Reconsider module boundaries.
* Reduce top-level work.

We will study this more in the import system chapter.

For now, recognize the smell:

```text
module A needs module B while module B needs module A
```

---

# A Third Module to Break a Cycle

Suppose:

```text
users.py needs format_name
reports.py needs format_name
```

Do not make `users.py` and `reports.py` import each other just to share formatting.

Create:

```text
formatting.py
```

`formatting.py`:

```python
def format_name(first, last):
    return f"{first} {last}"
```

`users.py`:

```python
import formatting


def display_user(user):
    return formatting.format_name(user["first"], user["last"])
```

`reports.py`:

```python
import formatting


def report_title(user):
    return f"Report for {formatting.format_name(user['first'], user['last'])}"
```

Now the dependency direction is clean:

```text
users ─────▶ formatting
reports ───▶ formatting
```

No cycle.

Shared responsibilities often deserve their own module.

---

# Modules and Object References

Modules fit perfectly into the object/reference model.

Example:

```python
import tasks
```

The name `tasks` refers to a module object.

Inside the module:

```python
def normalize(text):
    return text.strip()
```

The module namespace has a name:

```text
normalize -> function object
```

Conceptually:

```text
current module
    tasks ──▶ tasks module object
                  │
                  └── normalize ──▶ function object
```

When you call:

```python
tasks.normalize(" hello ")
```

Python:

```text
looks up tasks in current namespace
finds the module object
looks up normalize in the module namespace
calls the function object
```

Modules are not outside the object model.

They are part of it.

---

# Modules and Memory

Imported modules usually remain alive for the duration of the program.

Why?

Because Python keeps references to loaded modules.

This supports import caching.

It also means module-level objects can live a long time.

Example:

```python
# cache.py

items = {}
```

If this module is imported and `items` grows, the dictionary may live for the entire process.

Again:

```text
module-level state is long-lived state
```

This is useful for constants and shared tools.

It can be dangerous for unbounded mutable collections.

Memory lessons from Part VIII still apply:

```text
objects live while reachable
modules can keep objects reachable
```

---

# Modules and Global Names

The word "global" in Python usually means global to a module, not global to every file.

Example:

```python
# a.py

value = 10
```

```python
# b.py

value = 20
```

These are different names in different module namespaces.

There is:

```text
a.value
b.value
```

They do not collide just because they have the same name.

This is one major benefit of modules.

Each file gets its own global namespace.

Without modules, large programs would have far more name conflicts.

When someone says:

```text
global variable
```

ask:

```text
global in which module?
```

That question is often the key.

---

# The `global` Statement and Modules

Recall that `global` affects names in the current module.

Example:

```python
# counter.py

count = 0


def increment():
    global count
    count += 1
```

Here `global count` means:

```text
use the name count from the counter module namespace
```

It does not mean:

```text
use a universal variable shared by every module
```

If another module has:

```python
# other.py

count = 100
```

that is a different name.

Module namespaces keep globals separated.

This is why modules are fundamental to Python's name model.

---

# Running a Module Directly

When you run a file directly:

```text
python app.py
```

Python executes that file as the main program.

Inside that running file:

```python
__name__
```

is:

```text
"__main__"
```

This gives the file a special role:

```text
entry point of the program
```

An entry point is where program execution begins.

Many projects use a small entry-point file:

```python
# main.py

import tasks


def main():
    tasks.run()


if __name__ == "__main__":
    main()
```

This keeps startup code small and obvious.

The rest of the program lives in importable modules.

---

# Importing a Module for Reuse

When you import a module, you usually want its definitions.

Example:

```python
# conversions.py

def kilometers_to_miles(kilometers):
    return kilometers * 0.621371
```

Use:

```python
import conversions

print(conversions.kilometers_to_miles(5))
```

The module is reusable because it does not perform the conversion at import time.

Bad:

```python
# conversions.py

kilometers = float(input("Kilometers: "))
print(kilometers * 0.621371)
```

This file is a script, not a reusable module.

If another file imports it, it asks for input immediately.

Better:

```python
def kilometers_to_miles(kilometers):
    return kilometers * 0.621371


def main():
    kilometers = float(input("Kilometers: "))
    print(kilometers_to_miles(kilometers))


if __name__ == "__main__":
    main()
```

Now the file can be both:

* Imported as a module.
* Run as a script.

---

# Script vs Module

A script is usually a file meant to be run.

A module is usually a file meant to be imported.

But the same file can be both if designed carefully.

Example:

```python
def useful_function():
    return "result"


def main():
    print(useful_function())


if __name__ == "__main__":
    main()
```

When run:

```text
python tool.py
```

it behaves like a script.

When imported:

```python
import tool
```

it behaves like a reusable module.

The difference is not only the file.

The difference is how the file is used.

Designing files to be import-friendly gives you both options.

---

# Good Module Size

There is no exact line count for a good module.

But a module should have a coherent purpose.

Too small:

```text
one trivial function per file
```

This can create unnecessary navigation.

Too large:

```text
one file containing every idea in the project
```

This becomes difficult to understand.

Good module:

```text
one clear responsibility or closely related group of responsibilities
```

Examples:

```text
validation.py
serializers.py
user_repository.py
task_commands.py
date_formatting.py
```

When a module becomes hard to summarize in one sentence, it may be trying to do too much.

When many modules are impossible to understand without opening all of them, they may be split too finely.

Balance comes with practice.

---

# Import Order

Python code often groups imports near the top of the file.

Example:

```python
import datetime
import pathlib

import tasks
import reports
```

Common ordering:

```text
standard library imports
third-party imports
local application imports
```

At this stage, you do not need to obsess over formatting rules.

But keep imports easy to scan.

Avoid hiding imports deep inside code unless there is a reason.

Top-level imports make dependencies visible.

Sometimes local imports are useful to avoid circular imports or expensive startup work.

But default to clear top-level imports.

---

# Module Documentation

A module can start with a docstring.

Example:

```python
"""Utilities for parsing and formatting task text."""


def normalize_task(text):
    return text.strip()
```

The module docstring explains the module's purpose.

It should answer:

```text
What belongs in this module?
```

Good:

```python
"""Validation helpers for user-submitted task data."""
```

Less useful:

```python
"""This module has functions."""
```

Not every small file needs a long docstring.

But for larger modules, a short purpose statement helps future readers.

---

# Naming Modules

Good module names are:

* Lowercase.
* Short but meaningful.
* Written with underscores when needed.
* Related to responsibility.

Examples:

```text
tasks.py
task_parser.py
user_storage.py
report_builder.py
date_ranges.py
```

Avoid names that are:

* Too vague.
* Too clever.
* Too similar to standard library modules.
* Difficult to type.
* Invalid as import names.

Vague:

```text
stuff.py
misc.py
utils.py
helpers.py
```

Sometimes `utils.py` is acceptable in small projects.

But if it grows, split it into named responsibilities.

Module names are part of the reader's map.

Choose them with care.

---

# Common Mistake: Putting Program Execution at Import Time

Problem:

```python
# email_report.py

def send_report():
    print("sending report")


send_report()
```

Now:

```python
import email_report
```

sends the report.

That is surprising.

Better:

```python
def send_report():
    print("sending report")


def main():
    send_report()


if __name__ == "__main__":
    main()
```

Now importing gives access to `send_report`.

Running the file sends the report.

This is the difference between definition and execution.

Good modules are safe to import.

---

# Common Mistake: Confusing Module Variables With Copies

Suppose:

```python
# settings.py

debug = False
```

Another file:

```python
import settings

settings.debug = True
```

This changes the `debug` name inside the `settings` module object.

Other code that imports `settings` can see that change.

But:

```python
from settings import debug

debug = True
```

This rebinds the local name `debug` in the importing module.

It does not update `settings.debug`.

This difference matters.

`import settings` keeps access through the module object.

`from settings import debug` binds the current value to a local module name.

If you need shared mutable configuration, design it explicitly.

Do not rely on accidental rebinding behavior.

---

# Common Mistake: Circular Imports From Poor Boundaries

If two modules constantly need each other, their responsibilities may be mixed.

Example:

```text
orders.py imports invoices.py
invoices.py imports orders.py
```

Ask:

```text
Is there shared logic that belongs in a third module?
Should one module depend on the other, but not both ways?
Are top-level imports causing unnecessary early loading?
Are we mixing domain logic and formatting?
```

Circular imports are not always fatal.

But they are often a signal that design needs attention.

Clean dependency direction makes programs easier to reason about.

---

# Common Mistake: Overusing Module State

Module state can feel convenient.

Example:

```python
# current_user.py

user = None
```

Then many modules do:

```python
import current_user

print(current_user.user)
```

This creates hidden dependency on global state.

It can make tests difficult.

It can make behavior order-dependent.

It can make concurrent programs harder.

Sometimes module state is appropriate.

Examples:

* Constants.
* Cached immutable data.
* A configured logger.
* A registry with clear ownership.

But if many unrelated modules mutate the same module-level object, pause.

Passing values explicitly may be clearer.

---

# Common Mistake: Naming Everything `utils`

The name `utils` is tempting.

It seems harmless.

But it often hides weak organization.

Example:

```text
utils.py
```

contains:

```python
parse_date()
send_email()
slugify()
connect_database()
resize_image()
```

These functions do not share one responsibility.

Better:

```text
dates.py
emailing.py
text_formatting.py
database.py
images.py
```

Specific modules are easier to navigate.

They also reduce accidental dependencies.

Use module names as design signals.

---

# Common Mistake: Importing Inside Every Function Without Reason

This is usually unnecessary:

```python
def build_report():
    import datetime
    return datetime.date.today()
```

Prefer:

```python
import datetime


def build_report():
    return datetime.date.today()
```

Top-level imports make dependencies visible.

Local imports are useful sometimes:

* To avoid a circular import.
* To delay an expensive import.
* To import an optional dependency.
* To keep platform-specific imports isolated.

But do not hide imports inside functions by default.

Readers should be able to scan the top of a file and see what it depends on.

---

# A Complete Example

Small project:

```text
todo/
    main.py
    tasks.py
    formatting.py
```

`tasks.py`:

```python
def normalize(text):
    return text.strip()


def create_task(text):
    normalized = normalize(text)

    if normalized == "":
        raise ValueError("task cannot be empty")

    return {"title": normalized, "done": False}
```

`formatting.py`:

```python
def format_task(task):
    marker = "x" if task["done"] else " "
    return f"[{marker}] {task['title']}"
```

`main.py`:

```python
import formatting
import tasks


def main():
    task = tasks.create_task("  learn modules  ")
    print(formatting.format_task(task))


if __name__ == "__main__":
    main()
```

Run:

```text
python main.py
```

Output:

```text
[ ] learn modules
```

This example has clean responsibilities:

```text
tasks.py      -> task creation rules
formatting.py -> display formatting
main.py       -> program entry point
```

That is module design in miniature.

---

# How Modules Prepare Us for Packages

A module is usually one file.

A package is a way to organize multiple modules under a shared namespace.

Example idea:

```text
todo/
    __init__.py
    tasks.py
    formatting.py
    storage.py
```

Then imports may look like:

```python
import todo.tasks
```

or:

```python
from todo import tasks
```

We will study packages in the next chapter.

For now, understand:

```text
modules organize names inside files
packages organize modules inside directories
```

Modules are the foundation.

Packages build on them.

---

# Exercises

1. Create a file named `temperature.py` with:

```python
def celsius_to_fahrenheit(celsius):
    return celsius * 9 / 5 + 32
```

Then create `main.py` and import `temperature`.

Call:

```python
temperature.celsius_to_fahrenheit(0)
```

Explain why the module name appears before the function name.

---

2. Explain what happens when this module is imported:

```python
print("loading")

value = 10


def show():
    print(value)
```

Which code runs at import time?

Which names become available in the module namespace?

---

3. Compare:

```python
import settings
```

with:

```python
from settings import DEBUG
```

What name is bound in the importing module in each case?

---

4. Explain why this module is not import-friendly:

```python
name = input("Name: ")
print("Hello", name)
```

Rewrite it using a `main()` function and a main guard.

---

5. Create two modules:

```text
tasks.py
formatting.py
```

Put task creation in `tasks.py`.

Put display formatting in `formatting.py`.

Use both from `main.py`.

Explain why this is clearer than one large file.

---

6. Explain why naming a file `random.py` can cause problems.

What might happen when another file writes:

```python
import random
```

---

7. What is a circular import?

Create a simple explanation using two modules named `a.py` and `b.py`.

Then describe one way to break the cycle.

---

8. Explain this statement:

```text
global variables are global to a module, not global to every file
```

Use two modules that both define a name called `value`.

---

9. Why is this often a bad module name?

```text
utils.py
```

When might it be acceptable?

When should it be split?

---

10. In your own words, explain:

```text
import executes code
```

Why is this important for module design?

---

# Summary

In this chapter we learned:

* A module is a Python object that holds code and names.
* A `.py` file commonly becomes a module.
* Modules have their own namespaces.
* Module-level names live inside the module namespace.
* `import module` binds a module object to a name.
* `from module import name` binds a specific object into the importing module.
* Importing a module executes its top-level code when the module is first loaded.
* Import is not textual inclusion.
* Modules help organize programs by responsibility.
* Module-level mutable state should be intentional.
* Python usually caches imported modules.
* `__name__` tells whether a file is being run directly or imported.
* The main guard prevents program execution from happening during import.
* Good modules are safe and calm to import.
* Module names should be clear, lowercase, and avoid shadowing standard library modules.
* Circular imports often reveal tangled responsibilities.
* Packages build on modules by organizing multiple modules under directories.

Core model:

```text
file.py
    becomes a module object
    module object has a namespace
    namespace maps names to objects
    imports let other code access those names
```

Program organization model:

```text
one file       -> script
focused files  -> modules
module groups  -> packages
larger systems -> many packages with clear boundaries
```

Modules are the first serious step from writing snippets to engineering programs.

---

# Preview of Chapter 39

Next we study packages.

Modules organize names inside files.

Packages organize modules inside directories.

Chapter 39 explains how Python projects use packages to grow beyond a handful of files.

We will study:

* What a package is.
* Why packages are usually directories.
* How packages group related modules.
* What `__init__.py` is for.
* How package imports differ from simple module imports.
* How to think about project layout.
* Why package boundaries matter for maintainability.

The transition is direct:

```text
module -> one file of organized names
package -> directory of organized modules
```

Once modules make sense, packages are the next layer of structure.
