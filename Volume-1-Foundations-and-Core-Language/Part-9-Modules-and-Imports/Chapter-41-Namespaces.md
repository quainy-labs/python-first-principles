# Chapter 41 — Namespaces

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a namespace is.
* Why namespaces map names to objects.
* How local namespaces work.
* How module-level global namespaces work.
* How built-in names are found.
* How imports bind names into namespaces.
* How module and package namespaces relate to attributes.
* How object attributes form namespaces.
* How class namespaces differ from instance namespaces.
* How Python's name lookup rules connect to scope.
* Why namespaces prevent name collisions.
* Why `globals()`, `locals()`, and `vars()` are useful for learning and debugging.
* How namespace thinking clarifies modules, packages, functions, classes, and objects.

We have used the word namespace many times already.

Now we slow down and make it precise.

A namespace is one of the central ideas in Python.

It explains:

* Where variables live.
* Where imported names go.
* Why two files can both define `value`.
* Why `math.sqrt` and `sqrt` are different names.
* Why object attributes use dot syntax.
* Why local variables do not overwrite global variables unless told to.
* Why packages can contain modules with the same short filename.

Once namespaces are clear, many Python behaviors stop feeling like rules to memorize.

They become one model repeated in different places.

---

# Concept Overview

A namespace is a mapping from names to objects.

Simple version:

```text
namespace = place where names are stored
```

More precise version:

```text
namespace = mapping from string names to object references
```

Example:

```python
x = 10
name = "Ada"
items = []
```

Conceptually, the current namespace contains:

```text
x     -> 10
name  -> "Ada"
items -> list object
```

The names are not the objects.

The names point to objects.

This matches the object/reference model from earlier chapters.

Namespaces are where those name-to-object relationships are stored.

---

# Names Are Keys

A name is like a key in a mapping.

Example:

```python
language = "Python"
```

A namespace records:

```text
"language" -> "Python"
```

The name is text.

The object is the value the name refers to.

Now:

```python
language = "Rust"
```

The namespace updates:

```text
"language" -> "Rust"
```

The old string object may still exist if something else refers to it.

The namespace binding changed.

This is rebinding.

The same idea applies to:

* Local variables.
* Module globals.
* Imported names.
* Function names.
* Class names.
* Object attributes.

Python is full of namespaces.

---

# Namespaces Are Not Scopes

The words namespace and scope are related, but not identical.

A namespace is:

```text
where names are stored
```

A scope is:

```text
where a name can be looked up directly
```

Example:

```python
x = 10


def show():
    print(x)
```

The name `x` lives in the module namespace.

Inside `show`, Python can still find `x`.

Why?

Because the function's scope rules allow lookup in the enclosing module global namespace.

So:

```text
namespace -> storage
scope     -> lookup visibility
```

They work together.

Do not worry if this distinction feels subtle at first.

The examples in this chapter will make it natural.

---

# Local Namespaces

A function call creates a local namespace.

Example:

```python
def greet(name):
    message = f"Hello, {name}"
    return message
```

When you call:

```python
greet("Ada")
```

the function call has local names:

```text
name    -> "Ada"
message -> "Hello, Ada"
```

These names exist inside that function call.

When the function returns, the local namespace is no longer active.

That does not always mean every object disappears.

If the function returns an object, that object can survive:

```python
def build_message(name):
    message = f"Hello, {name}"
    return message


result = build_message("Ada")
```

The local name `message` disappears.

The string survives through `result`.

Namespace lifetime and object lifetime are related, but not the same.

---

# Each Function Call Gets Its Own Local Namespace

Every function call has its own local namespace.

Example:

```python
def double(number):
    result = number * 2
    return result
```

Call it:

```python
a = double(10)
b = double(20)
```

The first call has:

```text
number -> 10
result -> 20
```

The second call has:

```text
number -> 20
result -> 40
```

These local namespaces are separate.

The name `number` can be reused for each call because each call gets a fresh local namespace.

This is why functions are reusable.

The same function code can run many times with different local bindings.

---

# Module Namespaces

Every module has a namespace.

Example file:

```python
# settings.py

APP_NAME = "Task Tracker"
DEBUG = True


def show():
    print(APP_NAME, DEBUG)
```

The module namespace contains:

```text
APP_NAME -> "Task Tracker"
DEBUG    -> True
show     -> function object
```

When another file writes:

```python
import settings
```

the name `settings` refers to the module object.

That module object has a namespace.

Access:

```python
settings.APP_NAME
```

means:

```text
look in the settings module namespace for APP_NAME
```

Module namespaces are why two files can both define `DEBUG` without colliding.

They are different module namespaces.

---

# Global Means Module-Level

In Python, a global name is global to a module.

It is not automatically global to every file in the program.

Example:

`a.py`:

```python
value = 10
```

`b.py`:

```python
value = 20
```

These are separate:

```text
a.value -> 10
b.value -> 20
```

Each module has its own global namespace.

This is why the word global can be misleading.

Better beginner translation:

```text
global name = name in the current module namespace
```

When a function uses `global`, it means:

```text
use the module-level namespace for this name
```

not:

```text
use one universal namespace shared by all Python files
```

---

# The `global` Statement

Example:

```python
count = 0


def increment():
    global count
    count += 1
```

Inside `increment`, the statement:

```python
global count
```

tells Python:

```text
when assigning to count, use the module namespace
```

Without `global`, assignment creates or updates a local name:

```python
count = 0


def broken_increment():
    count += 1
```

This fails because Python treats `count` as local due to the assignment, but it tries to read it before assigning a local value.

The error is usually:

```text
UnboundLocalError
```

The namespace issue is:

```text
Python decided count is local in this function
but no local count existed yet
```

`global` changes the assignment target namespace.

---

# Built-In Namespace

Python has a built-in namespace.

It contains names like:

```text
print
len
range
str
int
list
dict
Exception
```

Example:

```python
print(len([1, 2, 3]))
```

You did not import `print` or `len`.

Python finds them in the built-in namespace.

Name lookup roughly follows:

```text
local
enclosing
global/module
built-in
```

This is often called LEGB:

```text
L -> Local
E -> Enclosing
G -> Global
B -> Built-in
```

We studied scope earlier.

Now we can say it in namespace terms:

```text
Python looks through a chain of namespaces to resolve a name
```

---

# Shadowing Built-Ins

Because built-ins are found by name lookup, you can accidentally shadow them.

Bad:

```python
list = [1, 2, 3]
```

Now:

```python
list("abc")
```

fails because `list` refers to your list object, not the built-in `list` type.

Another example:

```python
str = "hello"
int = 42
```

These names hide built-ins in the current namespace.

Python allows this.

But it is usually a mistake.

Avoid naming variables:

```text
list
dict
set
str
int
id
type
sum
min
max
input
file
```

The namespace model explains the bug:

```text
your local or global name is found before the built-in name
```

---

# Enclosing Namespaces

Nested functions create enclosing namespaces.

Example:

```python
def outer():
    message = "hello"

    def inner():
        print(message)

    inner()
```

When `inner` looks for `message`, Python searches:

```text
inner local namespace
outer enclosing namespace
module global namespace
built-in namespace
```

It finds `message` in the enclosing function namespace.

This is how closures work.

If `inner` survives after `outer` returns, the captured namespace information can survive too:

```python
def make_printer():
    message = "hello"

    def inner():
        print(message)

    return inner


printer = make_printer()
printer()
```

The local name `message` from `make_printer` remains available through the closure.

Names and object lifetimes meet again.

---

# The `nonlocal` Statement

`nonlocal` lets a nested function assign to a name in an enclosing function namespace.

Example:

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Here:

```python
nonlocal count
```

means:

```text
count belongs to an enclosing function namespace
not the local namespace of increment
not the module global namespace
```

Without `nonlocal`, assignment would make `count` local to `increment`, causing an error when trying to read it first.

Namespace target:

```text
local assignment       -> current function namespace
nonlocal assignment    -> enclosing function namespace
global assignment      -> module namespace
attribute assignment   -> object namespace
```

This table is worth remembering.

---

# Imports Bind Names Into Namespaces

Imports create bindings in the current namespace.

Example:

```python
import math
```

This binds:

```text
math -> math module object
```

in the current namespace.

Example:

```python
from math import sqrt
```

This binds:

```text
sqrt -> function object from math module
```

in the current namespace.

Example:

```python
import math as m
```

This binds:

```text
m -> math module object
```

The imported module may also live in `sys.modules`.

But the import statement still affects the current namespace by binding a name.

This is why the form of an import matters.

It determines which name appears where.

---

# `import module` Namespace Effect

Example:

```python
import math
```

Namespace after import:

```text
math -> module object
```

Use:

```python
math.sqrt(9)
```

The lookup:

```text
current namespace: math
math module namespace: sqrt
```

This keeps the origin visible.

The name `sqrt` is not directly in the current namespace.

If you write:

```python
sqrt(9)
```

you get:

```text
NameError
```

unless `sqrt` was imported or defined separately.

`import math` gives you the module name.

It does not dump all module names into your namespace.

That is a good thing.

---

# `from module import name` Namespace Effect

Example:

```python
from math import sqrt
```

Namespace after import:

```text
sqrt -> math.sqrt function object
```

Use:

```python
sqrt(9)
```

But:

```python
math.sqrt(9)
```

fails unless `math` was imported too.

Why?

Because the name `math` was not bound in the current namespace by this import form.

The module may be loaded.

It may be in `sys.modules`.

But name binding is separate from module loading.

Namespace model:

```text
import math              -> bind math
from math import sqrt    -> bind sqrt
import math as m         -> bind m
from math import sqrt as s -> bind s
```

This model removes the mystery.

---

# Star Imports and Namespaces

Star import:

```python
from module import *
```

binds many names into the current namespace.

Example:

```python
from math import *
```

Now names like `sqrt`, `sin`, and `cos` may be available directly.

This can feel convenient.

But it makes the namespace harder to understand.

Problems:

* Readers cannot easily tell where names came from.
* Names can collide.
* Later imports can overwrite earlier names.
* Static analysis becomes harder.

Namespace pollution means:

```text
too many names added to a namespace without clear ownership
```

Avoid star imports in ordinary code.

Prefer explicit imports.

Clear namespaces make clear programs.

---

# Module Attributes Are Namespace Entries

Module attributes are names in the module namespace.

Example:

```python
# config.py

DEBUG = True
```

Use:

```python
import config

print(config.DEBUG)
```

The expression:

```python
config.DEBUG
```

means:

```text
look up DEBUG in the namespace of the module object config
```

You can inspect a module namespace:

```python
print(vars(config))
```

This returns a dictionary-like view of the module's names.

At a beginner level:

```text
module namespace = module attributes
```

This is why dot syntax appears for modules:

```python
module.name
```

It is attribute access.

---

# Package Namespaces

Packages are special modules.

They also have namespaces.

Example:

```text
todo/
    __init__.py
    tasks.py
```

If `todo/__init__.py` contains:

```python
APP_NAME = "Todo"
```

then:

```python
import todo

print(todo.APP_NAME)
```

works.

The package namespace contains:

```text
APP_NAME -> "Todo"
```

If you import a submodule:

```python
import todo.tasks
```

then `todo` may have:

```text
tasks -> todo.tasks module object
```

Packages are not just folders.

At runtime, they are module objects with namespaces.

The folder is how Python finds code.

The namespace is how Python exposes names.

---

# Object Attribute Namespaces

Objects can have attributes.

Attributes are also names.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"
user.active = True
```

The object has an attribute namespace:

```text
name   -> "Ada"
active -> True
```

You can inspect many instance namespaces with:

```python
print(vars(user))
```

Output:

```text
{'name': 'Ada', 'active': True}
```

The exact representation is a dictionary.

At a simple level:

```text
object attributes are names stored on an object
```

Dot syntax:

```python
user.name
```

means:

```text
look up name in the object's attribute namespace
```

There are advanced details involving classes, descriptors, and `__getattribute__`, but this model is the starting point.

---

# Instance Namespaces

Each instance can have its own namespace.

Example:

```python
class User:
    pass


ada = User()
grace = User()

ada.name = "Ada"
grace.name = "Grace"
```

Conceptually:

```text
ada instance namespace:
    name -> "Ada"

grace instance namespace:
    name -> "Grace"
```

Same attribute name.

Different objects.

No collision.

This is the same namespace principle again.

A name only collides with another name in the same namespace.

`ada.name` and `grace.name` are separate because they live in separate instance namespaces.

---

# Class Namespaces

Classes have namespaces too.

Example:

```python
class User:
    species = "human"

    def greet(self):
        return "hello"
```

The class namespace contains:

```text
species -> "human"
greet   -> function object
```

Use:

```python
print(User.species)
print(User.greet)
```

Instances can find class attributes:

```python
ada = User()
print(ada.species)
```

Python looks for `species` on the instance first.

If not found there, it can look on the class.

This is attribute lookup, not ordinary local/global name lookup.

Classes and instances have related but distinct namespaces.

We will study this deeply in the object-oriented chapters.

For now, keep the basic model:

```text
class namespace stores class attributes and methods
instance namespace stores per-object attributes
```

---

# Instance Attribute vs Class Attribute

Example:

```python
class User:
    role = "member"


ada = User()
grace = User()

ada.role = "admin"
```

Now:

```python
print(ada.role)
print(grace.role)
print(User.role)
```

Output:

```text
admin
member
member
```

Why?

`ada` has an instance attribute:

```text
ada namespace:
    role -> "admin"
```

`grace` does not have an instance attribute named `role`.

So Python finds `role` on the class:

```text
User namespace:
    role -> "member"
```

Again:

```text
same name
different namespaces
lookup rules decide which one is found
```

That sentence explains a large amount of Python.

---

# Attribute Lookup Is Not Variable Lookup

These look similar:

```python
name
user.name
```

But they use different lookup mechanisms.

Plain name:

```python
name
```

uses scope/name lookup:

```text
local -> enclosing -> global -> built-in
```

Attribute access:

```python
user.name
```

starts with the object:

```text
look up attribute name on user
```

Then Python may consider:

* Instance attributes.
* Class attributes.
* Base classes.
* Descriptors.
* Custom attribute hooks.

Advanced details come later.

For now:

```text
plain name lookup and dot attribute lookup are related ideas,
but they are not the same algorithm
```

This distinction prevents confusion.

---

# Namespaces Prevent Collisions

Without namespaces, every name in a program would compete with every other name.

Imagine:

```python
value = 10
```

in one file and:

```python
value = 20
```

in another file.

Without module namespaces, these would conflict.

With module namespaces:

```text
a.value
b.value
```

both can exist.

Similarly:

```python
user.name
product.name
company.name
```

All can use the attribute name `name`.

They live in different object namespaces.

Namespaces let us reuse good names in different contexts.

This is essential for large programs.

Good naming is not only about unique names.

It is about putting names in the right namespace.

---

# Namespaces and Readability

Compare:

```python
from math import sqrt

result = sqrt(25)
```

with:

```python
import math

result = math.sqrt(25)
```

Both are valid.

The second keeps the namespace visible:

```text
sqrt comes from math
```

Compare:

```python
from formatting import format_task
from reports import format_report
```

with:

```python
import formatting
import reports

formatting.format_task(task)
reports.format_report(report)
```

The second may be more verbose.

It may also be clearer.

Namespace prefixes are not noise when they carry meaning.

Use imports in a way that helps the reader know where names belong.

---

# `globals()`

The function `globals()` returns the current module's global namespace dictionary.

Example:

```python
x = 10

print(globals()["x"])
```

Output:

```text
10
```

You can inspect:

```python
print(globals().keys())
```

This shows names in the module namespace.

Do not use `globals()` as a normal way to manage program state.

But it is useful for learning:

```python
x = 10
print("x" in globals())
```

It reveals that module-level names are stored in a namespace mapping.

The namespace model is not just metaphor.

Python exposes parts of it.

---

# `locals()`

The function `locals()` returns a mapping of local names.

Example:

```python
def show(name):
    message = f"Hello, {name}"
    print(locals())


show("Ada")
```

Possible output:

```text
{'name': 'Ada', 'message': 'Hello, Ada'}
```

This shows the function's local namespace at that moment.

Important caution:

Do not rely on modifying `locals()` inside functions to change local variables.

Example:

```python
def example():
    locals()["x"] = 10
    print(x)
```

This is not reliable as normal code.

Use `locals()` for inspection and debugging, not as a control mechanism.

The useful lesson:

```text
function calls have local namespaces
```

---

# `vars()`

`vars()` returns the `__dict__` namespace for many objects.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"

print(vars(user))
```

Output:

```text
{'name': 'Ada'}
```

For a module:

```python
import math

print(vars(math))
```

This shows the module namespace.

For a class:

```python
print(vars(User))
```

This shows names stored on the class.

`vars()` is a practical namespace inspection tool.

It helps you see:

```text
what names exist on this object?
```

Not every object has a normal `__dict__`.

Some objects use slots or special internal layouts.

But for many objects, `vars()` is an excellent learning tool.

---

# `dir()`

`dir()` lists names available on an object.

Example:

```python
import math

print(dir(math))
```

For an instance:

```python
class User:
    species = "human"

    def greet(self):
        return "hello"


user = User()
user.name = "Ada"

print(dir(user))
```

`dir(user)` may include:

* `name`
* `species`
* `greet`
* many special methods

`dir()` is not exactly the same as `vars()`.

`vars()` shows a namespace dictionary when one exists.

`dir()` tries to list accessible names, including inherited and special names.

Use:

```text
vars() -> what is stored directly here?
dir()  -> what names appear available?
```

Both are useful.

---

# Namespace Dictionaries

Many namespaces are implemented as dictionaries or dictionary-like mappings.

Examples:

```python
globals()
vars(module)
vars(instance)
vars(class)
```

This does not mean every namespace is a plain dictionary in every situation.

Python has optimizations and special cases.

But the mapping idea is right:

```text
name -> object
```

Example:

```python
namespace = {
    "x": 10,
    "name": "Ada",
}
```

Python's actual namespaces are richer than this simple dictionary.

But the dictionary model is powerful enough for reasoning.

When confused, ask:

```text
which namespace contains this name?
what object does it map to?
which lookup rule is being used?
```

---

# Name Lookup Failure

If Python cannot find a plain name, it raises:

```text
NameError
```

Example:

```python
print(missing_name)
```

Error:

```text
NameError: name 'missing_name' is not defined
```

Namespace explanation:

```text
Python searched the relevant namespaces
no binding for missing_name was found
```

For local variables that Python decided are local but not yet assigned, you may see:

```text
UnboundLocalError
```

Example:

```python
x = 10


def show():
    print(x)
    x = 20
```

Python sees assignment to `x` in the function.

So it treats `x` as local.

Then `print(x)` tries to read local `x` before it has a value.

Namespace explanation:

```text
x is local in this function
but local x has not been bound yet
```

---

# Attribute Lookup Failure

If Python cannot find an attribute, it raises:

```text
AttributeError
```

Example:

```python
class User:
    pass


user = User()
print(user.name)
```

Error:

```text
AttributeError
```

Namespace explanation:

```text
Python looked for attribute name on user
it did not find it through attribute lookup
```

This is different from:

```python
print(name)
```

which would use plain name lookup and may raise `NameError`.

Remember:

```text
missing plain name  -> NameError
missing attribute   -> AttributeError
```

This difference helps you read tracebacks.

---

# Namespaces and Mutability

Namespaces map names to objects.

Mutability is about whether objects can change.

Example:

```python
items = []
other = items
```

Namespace:

```text
items -> list object
other -> same list object
```

Mutation:

```python
items.append("Python")
```

The namespace did not need to change.

The object changed.

Rebinding:

```python
items = []
```

Now the namespace changes:

```text
items -> new list object
other -> old list object
```

This distinction appears everywhere.

Namespace operation:

```text
bind or rebind a name
```

Object operation:

```text
mutate an object
```

Do not confuse them.

---

# Namespaces and Aliasing

Aliasing means multiple names refer to the same object.

Example:

```python
a = []
b = a
```

Namespace:

```text
a -> list object
b -> same list object
```

Now:

```python
b.append(1)
print(a)
```

Output:

```text
[1]
```

No magic.

Both names map to the same object.

Namespaces do not copy objects by default.

They store references.

Aliasing is not limited to local names.

Example:

```python
module.value
object.attribute
list_item
dict_value
```

All can refer to the same object.

The object graph can have many paths.

---

# Namespaces and Modules in Memory

When you import a module:

```python
import config
```

the current module namespace gets:

```text
config -> config module object
```

The imported module object has its own namespace:

```text
DEBUG -> True
TIMEOUT -> 10
```

So:

```python
config.DEBUG
```

means:

```text
current namespace finds config
config module namespace finds DEBUG
```

If another file also imports `config`, it gets a reference to the same module object.

Its own namespace has its own name:

```text
config -> same config module object
```

Module caching plus namespaces explain shared module state.

---

# Namespaces and Packages in Memory

Import:

```python
import app.users.models
```

Conceptual namespace chain:

```text
current namespace:
    app -> app package object

app namespace:
    users -> app.users package object

app.users namespace:
    models -> app.users.models module object

app.users.models namespace:
    User -> class object
```

This is why dotted import paths feel natural in Python.

They mirror namespace traversal:

```python
app.users.models.User
```

Each dot moves into another object's attribute namespace.

Again:

```text
dots are namespace navigation
```

At a high level, that is the right intuition.

---

# Namespaces and `sys.modules`

`sys.modules` is also a namespace-like mapping.

It maps module names to module objects.

Example:

```python
import sys
import math

print(sys.modules["math"])
```

Mapping:

```text
"math" -> math module object
```

This is not the same as the current local or global namespace.

Your current module may or may not have a name `math`.

Example:

```python
from math import sqrt
```

Now:

```text
"math" in sys.modules -> True
"math" in globals()   -> maybe False
"sqrt" in globals()   -> True
```

The module is loaded.

But the name bound locally is `sqrt`.

This is a perfect example of why namespace precision matters.

---

# Namespace Pollution

Namespace pollution means adding too many names to a namespace, especially names that are unclear or unnecessary.

Example:

```python
from math import *
from statistics import *
from random import *
```

Now the current namespace contains many names.

Some may collide.

Some may be hard to trace.

Another example:

```python
config_value_1 = ...
config_value_2 = ...
config_value_3 = ...
temporary_result = ...
helper_data = ...
debug_thing = ...
```

Too many module-level names can make a module difficult to understand.

Good namespace hygiene means:

* Import explicitly.
* Keep names meaningful.
* Avoid unnecessary globals.
* Group related behavior into modules, classes, or objects.
* Keep public APIs curated.

Clean namespaces are part of clean design.

---

# Public and Internal Names

Python uses naming conventions to signal public and internal names.

Public:

```python
def create_task(title):
    ...
```

Internal:

```python
def _normalize_title(title):
    ...
```

The leading underscore means:

```text
this name is intended for internal use
```

It does not make the name private in a strict sense.

Other code can still access:

```python
module._normalize_title
```

But convention matters.

Namespaces often contain more names than the public API.

The public API is the subset other code should rely on.

Good modules and packages make this distinction clear.

---

# `__all__` and Public Names

Modules can define:

```python
__all__ = ["create_task", "format_task"]
```

This controls what star import imports:

```python
from module import *
```

It also documents intended public names.

Example:

```python
__all__ = ["create_task"]


def create_task(title):
    return {"title": title}


def _normalize(title):
    return title.strip()
```

Namespace contains both:

```text
create_task
_normalize
```

Public API says:

```text
create_task
```

Again:

```text
namespace contains names
API chooses which names are promises
```

That distinction becomes very important in libraries.

---

# Namespaces and Testing

Testing often imports modules.

Example:

```python
import tasks


def test_create_task():
    task = tasks.create_task("read")
    assert task["title"] == "read"
```

The test module namespace contains:

```text
tasks -> tasks module object
test_create_task -> function object
```

The `tasks` module namespace contains:

```text
create_task -> function object
```

Understanding namespaces helps testing because you know what to patch, import, or call.

Example:

```python
from tasks import create_task
```

binds `create_task` in the test module namespace.

If later code changes `tasks.create_task`, your local imported name may still point to the old object depending on how changes happen.

This matters in mocking, monkey patching, and reload behavior later.

For now, remember:

```text
tests are modules too
their imports are namespace bindings too
```

---

# Namespaces and Debugging

When debugging a name problem, ask:

```text
Which namespace should contain this name?
Is the name actually there?
What object does it refer to?
Is another name shadowing it?
Is this plain name lookup or attribute lookup?
Was the module imported under the expected name?
```

Useful tools:

```python
globals()
locals()
vars(obj)
dir(obj)
obj.__dict__
module.__file__
```

Example:

```python
import config

print(config.__file__)
print(vars(config).keys())
```

This tells you:

* Which module was imported.
* Which names exist in its namespace.

Debugging becomes easier when you stop asking:

```text
Why does Python not know this?
```

and start asking:

```text
Which namespace is Python searching?
```

---

# Common Mistake: Thinking Names Are Boxes

Beginners often imagine:

```text
variable = box containing value
```

This leads to confusion with mutation and aliasing.

Better:

```text
name = label/reference to object
namespace = place where labels are recorded
```

Example:

```python
a = []
b = a
```

There are not two boxes containing two lists.

There is one list object and two names pointing to it.

Namespace:

```text
a -> list object
b -> same list object
```

Mutation through one name is visible through the other because the object is shared.

The namespace model is better than the box model.

---

# Common Mistake: Confusing Module Globals Across Files

`a.py`:

```python
value = 10
```

`b.py`:

```python
value = 20
```

These do not conflict.

They are:

```text
a.value
b.value
```

If `b.py` wants `a.value`, it must import `a`:

```python
import a

print(a.value)
```

The name `value` in `b.py` remains separate.

Global does not mean universal.

It means module-level.

This is one of the most important namespace corrections in Python.

---

# Common Mistake: Rebinding an Imported Name

`settings.py`:

```python
DEBUG = False
```

`main.py`:

```python
from settings import DEBUG

DEBUG = True
```

This does not change:

```python
settings.DEBUG
```

It rebinds the name `DEBUG` in `main.py`.

Namespace explanation:

```text
settings namespace:
    DEBUG -> False

main namespace after from import:
    DEBUG -> False

main namespace after rebinding:
    DEBUG -> True

settings namespace unchanged
```

If you want to modify the module attribute:

```python
import settings

settings.DEBUG = True
```

Even then, shared mutable configuration should be designed carefully.

---

# Common Mistake: Assuming Attribute Names Are Local Variables

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Inside `__init__`:

```python
name
```

is a local parameter.

```python
self.name
```

is an attribute on the object referred to by `self`.

These are different namespaces.

Conceptually:

```text
local namespace:
    self -> user object
    name -> "Ada"

user object namespace:
    name -> "Ada"
```

The same text `name` appears in both places.

But:

```python
name
```

and:

```python
self.name
```

are not the same lookup.

This distinction is essential for object-oriented Python.

---

# Common Mistake: Too Many Direct Imports

This can become hard to read:

```python
from users import create, delete, update, validate, format_name
from reports import create_report, delete_report, format_report
from storage import save, load, delete_record
```

Later:

```python
delete(user)
save(report)
format_name(user)
```

Where did each name come from?

Sometimes direct imports are fine.

But too many direct imports crowd the current namespace.

Alternative:

```python
import users
import reports
import storage

users.delete(user)
storage.save(report)
users.format_name(user)
```

The prefixes are useful context.

Shorter code is not always clearer code.

---

# Common Mistake: Ignoring Namespace Boundaries in Packages

Package:

```text
app/
    users/
        models.py
    tasks/
        models.py
```

Both modules are named:

```text
models
```

This is fine because their full names differ:

```text
app.users.models
app.tasks.models
```

But problems happen if code imports inconsistently:

```python
import models
```

from one location and:

```python
import app.users.models
```

from another.

You may accidentally create confusion about which namespace is being used.

Use package-qualified imports consistently.

Package namespaces exist to make context clear.

Use them.

---

# Namespace Design Guidelines

Good namespace design asks:

```text
Where should this name live?
Who should access it?
Is it local, module-level, object-level, class-level, or package-level?
Should it be public or internal?
Will this name collide with another useful name?
Does an import prefix improve readability?
Is this mutable state in a long-lived namespace?
```

General guidance:

* Keep temporary names local.
* Keep reusable functions in modules.
* Keep related modules in packages.
* Keep per-object data on instances.
* Keep shared class data on classes.
* Keep public APIs intentional.
* Avoid unnecessary module-level mutable state.
* Avoid star imports.
* Avoid shadowing built-ins.

Namespaces are not only a technical mechanism.

They are design surfaces.

They shape how readers understand a program.

---

# A Complete Namespace Walkthrough

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
DEFAULT_DONE = False


def create_task(title):
    task = {"title": title, "done": DEFAULT_DONE}
    return task
```

`main.py`:

```python
from todo import tasks


def main():
    task = tasks.create_task("learn namespaces")
    print(task)


if __name__ == "__main__":
    main()
```

Namespace walkthrough:

```text
main module namespace:
    tasks -> todo.tasks module object
    main  -> function object

todo.tasks module namespace:
    DEFAULT_DONE -> False
    create_task  -> function object

main() local namespace while running:
    task -> dictionary object

task dictionary:
    "title" -> "learn namespaces"
    "done"  -> False
```

Notice that names live at different levels.

Understanding those levels makes the program easy to reason about.

---

# Another Walkthrough With Objects

Code:

```python
class User:
    species = "human"

    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello, {self.name}"


ada = User("Ada")
```

Namespaces:

```text
module namespace:
    User -> class object
    ada  -> User instance

User class namespace:
    species -> "human"
    __init__ -> function object
    greet -> function object

ada instance namespace:
    name -> "Ada"

__init__ local namespace during call:
    self -> ada instance
    name -> "Ada"
```

When:

```python
ada.greet()
```

Python finds `greet` through attribute lookup.

When `greet` runs, it has a local namespace containing `self`.

Then:

```python
self.name
```

looks in the instance namespace.

This is namespaces all the way down.

---

# Exercises

1. For this code:

```python
x = 10


def show():
    y = 20
    print(x, y)
```

Identify which namespace contains `x` and which namespace contains `y`.

---

2. Explain why this raises an error:

```python
count = 0


def increment():
    count += 1
```

What namespace does Python think `count` belongs to inside the function?

---

3. Compare:

```python
import math
```

with:

```python
from math import sqrt
```

What name is bound in the current namespace in each case?

---

4. Create a module named `config.py`:

```python
DEBUG = False
```

Then in another file try:

```python
from config import DEBUG
DEBUG = True
```

Does this change `config.DEBUG`?

Explain using namespaces.

---

5. Use `locals()` inside a function:

```python
def example(name):
    message = f"Hello, {name}"
    print(locals())
```

What names appear?

---

6. Use `vars()` on an object:

```python
class User:
    pass


user = User()
user.name = "Ada"
print(vars(user))
```

What namespace are you seeing?

---

7. Explain the difference between:

```python
name
```

and:

```python
self.name
```

inside a method.

---

8. Why is this a bad idea?

```python
list = [1, 2, 3]
```

Explain using built-in namespace lookup.

---

9. Create two modules that both define:

```python
value = 10
```

or:

```python
value = 20
```

Import both and access:

```python
a.value
b.value
```

Explain why there is no collision.

---

10. In your own words, explain:

```text
dots are namespace navigation
```

Use examples from modules, packages, and objects.

---

# Summary

In this chapter we learned:

* A namespace maps names to objects.
* Names are bindings, not boxes.
* Local namespaces exist during function calls.
* Module namespaces store module-level global names.
* In Python, global usually means global to a module.
* Built-in names live in the built-in namespace.
* Python resolves plain names through scope-based namespace lookup.
* Imports bind names into the current namespace.
* Different import forms bind different names.
* Modules and packages expose namespaces through attributes.
* Objects have attribute namespaces.
* Instances and classes have different namespaces.
* Plain name lookup and attribute lookup are different.
* Namespaces prevent collisions by giving names context.
* `globals()`, `locals()`, `vars()`, and `dir()` help inspect namespaces.
* Clean namespaces improve readability and design.

Core model:

```text
namespace:
    name -> object

plain name lookup:
    local -> enclosing -> global/module -> built-in

attribute lookup:
    object.name
    module.name
    package.name
    class.name
```

Design model:

```text
temporary value -> local namespace
shared module tool -> module namespace
organized file group -> package namespace
per-object data -> instance namespace
shared object behavior -> class namespace
```

Namespaces are the map of a Python program.

They tell us where names live, what they refer to, and how different parts of a program avoid stepping on each other.

---

# Preview of Chapter 42

Next we study virtual environments.

Namespaces explain where names live inside a running Python program.

Virtual environments explain where installed packages live outside the program, at the environment level.

Chapter 42 will study:

* What a virtual environment is.
* Why Python projects need isolated environments.
* How installed packages relate to imports.
* Why different projects can require different dependency versions.
* How virtual environments affect `python`, `pip`, and import search paths.
* Why environment isolation prevents dependency conflicts.
* How virtual environments prepare us for professional packaging and tooling.

The transition is practical:

```text
namespaces organize names inside Python
virtual environments organize dependencies around Python
```

This will complete Part IX and prepare us to move into Volume II with a clean understanding of code organization.
