# Chapter 57 — Metaclasses

---

# Learning Objectives

By the end of this chapter, you should understand:

* Why classes are objects.
* What `type` does.
* What a metaclass is.
* Why most classes have `type` as their metaclass.
* How Python creates a class.
* How the `metaclass=` keyword works.
* How metaclasses are inherited.
* How metaclass `__new__` and `__init__` differ.
* What `__prepare__` does.
* How class namespaces are built.
* How `__set_name__` and `__init_subclass__` fit into class creation.
* Why `ABCMeta` is a real metaclass you have already used.
* What metaclass conflicts are.
* When class decorators are simpler.
* When `__init_subclass__` is simpler.
* When descriptors are simpler.
* When metaclasses are justified.
* Why custom metaclasses should be rare.

Chapter 56 studied `__slots__`.

Slots customize instance storage.

Metaclasses customize class creation.

That is one level higher.

So far, we have customized:

* how objects store attributes
* how attributes are accessed
* how methods bind
* how operators behave
* how objects compare
* how classes validate values

Metaclasses ask a deeper question:

```text
how is the class object itself created?
```

In Python, classes are objects.

Because classes are objects, they also have types.

The type of a class is called its metaclass.

Most of the time, that metaclass is `type`.

---

# Classes Are Objects

Start with an ordinary class:

```python
class User:
    pass
```

`User` is an object.

You can assign it to another name:

```python
Account = User
```

You can pass it to a function:

```python
def create(cls):
    return cls()


user = create(User)
```

You can store it in a list:

```python
models = [User]
```

You can inspect it:

```python
print(User.__name__)
print(User.__dict__)
```

You can ask for its type:

```python
print(type(User))
```

For ordinary classes, the output is:

```python
<class 'type'>
```

This means:

```text
User is an object whose type is type
```

An instance of `User` has type `User`.

The class `User` itself has type `type`.

That is the first metaclass idea.

---

# Instances, Classes, and Metaclasses

Consider:

```python
user = User()
```

Then:

```python
type(user)
```

is:

```python
User
```

And:

```python
type(User)
```

is:

```python
type
```

The levels are:

```text
user  -> instance
User  -> class
type  -> metaclass
```

Another way:

```text
user is an instance of User
User is an instance of type
```

This feels strange at first because classes usually feel like definitions, not objects.

But in Python, a class definition creates a class object.

That class object can then create instances.

The metaclass creates the class object.

---

# What `type` Does

You already know `type` as an inspection function:

```python
type(10)
type("hello")
type(User())
```

But `type` can also create classes.

This class definition:

```python
class User:
    role = "member"

    def greet(self):
        return "hello"
```

is conceptually similar to:

```python
def greet(self):
    return "hello"


User = type(
    "User",
    (),
    {
        "role": "member",
        "greet": greet,
    },
)
```

The three arguments are:

```text
name       -> class name
bases      -> tuple of base classes
namespace  -> dictionary of class attributes
```

So:

```python
type("User", (), namespace)
```

creates a class object.

This is not how you should usually write classes.

Class syntax is clearer.

But this example proves an important truth:

```text
classes are created by calling a metaclass
```

For normal classes, the metaclass is `type`.

---

# What a Metaclass Is

A metaclass is the class of a class.

Ordinary instance:

```python
user = User()
```

Class of the instance:

```python
type(user) is User
```

Class object:

```python
User
```

Class of the class:

```python
type(User) is type
```

So:

```text
type is the default metaclass
```

You can define your own metaclass by subclassing `type`:

```python
class Meta(type):
    pass
```

Then use it:

```python
class User(metaclass=Meta):
    pass
```

Now:

```python
type(User)
```

is:

```python
Meta
```

And:

```python
isinstance(User, Meta)
```

is:

```python
True
```

The instance of `User` is still a `User`.

But the class object `User` is an instance of `Meta`.

---

# Why Metaclasses Exist

Metaclasses exist because sometimes you want to customize class creation.

Not instance creation.

Class creation.

Examples:

* validate class definitions
* register subclasses automatically
* collect fields declared in a class body
* modify class attributes before the class is created
* enforce naming rules
* create framework-level declarative APIs
* customize `isinstance` or `issubclass`
* build abstract base class behavior

You have already used a metaclass indirectly:

```python
from abc import ABC


class Service(ABC):
    ...
```

`ABC` uses `ABCMeta`.

That metaclass helps enforce abstract methods and virtual subclass behavior.

Metaclasses are not just theoretical.

They power real standard-library behavior.

But most application code does not need custom metaclasses.

Python gives you simpler tools for many class-customization problems.

---

# The Class Creation Pipeline

When Python executes a class definition, it does more than collect methods.

The official process includes steps like:

```text
resolve special base entries
determine the appropriate metaclass
prepare the class namespace
execute the class body
create the class object
call descriptor __set_name__ hooks
call __init_subclass__ hooks
apply class decorators
bind the final class object to the class name
```

You do not need to memorize every detail immediately.

But you should understand the broad shape:

```text
class statement -> namespace -> metaclass call -> class object
```

Example:

```python
class User(Base, metaclass=Meta):
    role = "member"

    def greet(self):
        return "hello"
```

Python roughly:

1. Determines the base classes.
2. Determines the metaclass.
3. Prepares a namespace for the class body.
4. Executes the class body into that namespace.
5. Calls the metaclass to create the class object.
6. Runs class-creation hooks.
7. Assigns the resulting class object to `User`.

Metaclasses let you customize parts of this process.

---

# A Metaclass That Logs Class Creation

Start with a visible example:

```python
class LoggingMeta(type):
    def __new__(mcls, name, bases, namespace):
        print(f"creating class {name}")
        return super().__new__(mcls, name, bases, namespace)
```

Use it:

```python
class User(metaclass=LoggingMeta):
    pass
```

When Python executes the class definition, it prints:

```text
creating class User
```

This happens when the class is created, not when an instance is created.

Later:

```python
user = User()
```

does not create the class again.

The metaclass affected class creation.

The class creates instances afterward.

This distinction is central:

```text
metaclass __new__ creates the class object
class __new__ creates instance objects
```

---

# Metaclass `__new__`

Metaclass `__new__` creates the class object.

Signature:

```python
def __new__(mcls, name, bases, namespace, **kwargs):
    ...
```

Common parameters:

```text
mcls       -> the metaclass being called
name       -> class name as a string
bases      -> tuple of base classes
namespace  -> class body namespace
kwargs     -> extra class-definition keywords
```

Example:

```python
class RequireDocstringMeta(type):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        if cls.__doc__ is None:
            raise TypeError(f"{name} must have a docstring")
        return cls
```

Use it:

```python
class Documented(metaclass=RequireDocstringMeta):
    """This class is allowed."""
```

This fails:

```python
class Undocumented(metaclass=RequireDocstringMeta):
    pass
```

because the class object is rejected during creation.

This kind of validation is one metaclass use case.

But ask whether a simpler tool would do.

For many projects, tests or linters are better than runtime metaclass enforcement.

---

# Metaclass `__init__`

Metaclass `__new__` creates the class object.

Metaclass `__init__` initializes it after creation.

Example:

```python
class LoggingMeta(type):
    def __new__(mcls, name, bases, namespace):
        print(f"__new__ for {name}")
        return super().__new__(mcls, name, bases, namespace)

    def __init__(cls, name, bases, namespace):
        print(f"__init__ for {name}")
        super().__init__(name, bases, namespace)
```

Use:

```python
class User(metaclass=LoggingMeta):
    pass
```

Output:

```text
__new__ for User
__init__ for User
```

Use `__new__` when you need to control or modify creation.

Use `__init__` when the class object already exists and you want to initialize or register it.

Many metaclasses use `__new__` because they need to inspect or change the namespace before the final class is fully formed.

---

# Extra Class Keywords

Class definitions can pass extra keywords:

```python
class Model(table="users", metaclass=ModelMeta):
    pass
```

The metaclass can receive them:

```python
class ModelMeta(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        table = kwargs.pop("table", None)
        cls = super().__new__(mcls, name, bases, namespace)
        cls.table = table
        return cls
```

Use:

```python
class User(metaclass=ModelMeta, table="users"):
    pass
```

Now:

```python
User.table
```

is:

```python
"users"
```

This pattern also works with `__init_subclass__`, which is often simpler.

We will compare them soon.

---

# `__prepare__`

Before Python executes the class body, it needs a namespace.

Usually that namespace behaves like a dictionary.

A metaclass can customize namespace preparation with:

```python
__prepare__
```

Example:

```python
class PrepareMeta(type):
    @classmethod
    def __prepare__(mcls, name, bases):
        print(f"preparing namespace for {name}")
        return {}

    def __new__(mcls, name, bases, namespace):
        print(f"namespace contains: {list(namespace)}")
        return super().__new__(mcls, name, bases, namespace)
```

Use:

```python
class User(metaclass=PrepareMeta):
    role = "member"

    def greet(self):
        return "hello"
```

The namespace receives names created by the class body:

```text
__module__
__qualname__
role
greet
```

Historically, `__prepare__` was important for preserving class attribute order before normal dictionaries preserved insertion order.

It is still useful when a framework needs a custom mapping during class body execution.

Most code should not need it.

---

# A Namespace That Rejects Duplicate Names

Here is a teaching example.

Suppose we want to reject duplicate names in a class body.

Custom namespace:

```python
class NoDuplicateDict(dict):
    def __setitem__(self, key, value):
        if key in self:
            raise TypeError(f"duplicate class attribute: {key}")
        super().__setitem__(key, value)
```

Metaclass:

```python
class NoDuplicateMeta(type):
    @classmethod
    def __prepare__(mcls, name, bases):
        return NoDuplicateDict()
```

Use:

```python
class Example(metaclass=NoDuplicateMeta):
    x = 1
    y = 2
```

This is fine.

But:

```python
class Broken(metaclass=NoDuplicateMeta):
    x = 1
    x = 2
```

raises an error.

This example shows what `__prepare__` can do:

```text
control the namespace while the class body executes
```

But this is advanced.

Most projects should not enforce style through metaclasses unless there is a strong framework reason.

---

# Class Body Execution

A class body is executable code.

Example:

```python
class Example:
    print("inside class body")
    value = 10
```

The print runs when the class is defined.

Not when an instance is created.

Class body execution fills the namespace that will become the class dictionary.

Example:

```python
class Example:
    x = 1
    y = x + 1
```

The class body creates:

```python
x
y
```

in the class namespace.

Then the metaclass receives that namespace and creates the class object.

This helps explain why metaclasses can inspect class declarations.

The class body has already run.

The namespace contains the results.

---

# Class Dictionaries Are Not Ordinary Writable Dictionaries

After a class is created, its `__dict__` is exposed through a read-only proxy.

Example:

```python
class User:
    name = "Maya"
```

Then:

```python
User.__dict__
```

is not a normal mutable dictionary.

You cannot directly assign:

```python
User.__dict__["role"] = "admin"
```

But you can assign attributes normally:

```python
User.role = "admin"
```

The metaclass receives the namespace during creation.

After creation, Python wraps the class dictionary view.

This is one reason class creation is a special moment.

Frameworks often use metaclasses or class hooks to inspect declarations at that moment.

---

# `__set_name__` During Class Creation

Chapter 54 introduced `__set_name__`.

When Python creates a class, it scans class attributes for objects with `__set_name__`.

Example:

```python
class Field:
    def __set_name__(self, owner, name):
        self.owner = owner
        self.name = name
```

Use:

```python
class User:
    email = Field()
```

During class creation, Python calls:

```python
User.__dict__["email"].__set_name__(User, "email")
```

This happens as part of class object creation.

Descriptors often remove the need for metaclasses because `__set_name__` gives each descriptor enough class-creation awareness.

Before writing a metaclass to tell fields their names, ask:

```text
can __set_name__ solve this?
```

Often, yes.

---

# `__init_subclass__`

`__init_subclass__` is called on a parent class when a subclass is created.

Example:

```python
class Base:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        print(f"created subclass {cls.__name__}")
```

Use:

```python
class Child(Base):
    pass
```

Output:

```text
created subclass Child
```

This is much simpler than a metaclass for many subclass-registration tasks.

Example:

```python
class Plugin:
    registry = {}

    def __init_subclass__(cls, name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if name is not None:
            Plugin.registry[name] = cls
```

Use:

```python
class JsonPlugin(Plugin, name="json"):
    pass
```

Now:

```python
Plugin.registry["json"] is JsonPlugin
```

is:

```python
True
```

No metaclass needed.

---

# Metaclass Versus `__init_subclass__`

Use `__init_subclass__` when a base class wants to react to future subclasses.

Example:

```python
class Plugin:
    def __init_subclass__(cls, **kwargs):
        ...
```

This is often enough for:

* subclass registration
* subclass validation
* setting subclass defaults
* collecting subclass metadata

Use a metaclass when you need to customize class creation more deeply:

* prepare a custom namespace before the class body runs
* control the actual class object creation
* customize class-level operators or attribute access
* implement behavior like `ABCMeta`
* coordinate multiple class-creation hooks at framework level

The rule:

```text
if __init_subclass__ solves it, prefer __init_subclass__
```

Metaclasses should not be your first tool.

---

# Class Decorators

A class decorator receives a class object after it is created and returns a class object.

Example:

```python
def add_table_name(cls):
    cls.table_name = cls.__name__.lower()
    return cls
```

Use:

```python
@add_table_name
class User:
    pass
```

Now:

```python
User.table_name
```

is:

```python
"user"
```

Class decorators are often simpler than metaclasses.

They work well when:

* you only need to modify one class
* you do not need to affect future subclasses automatically
* you do not need a custom namespace before class body execution
* you want explicit opt-in at the class definition

Metaclasses are more invasive.

Class decorators are direct and local.

---

# Metaclass Versus Class Decorator

Suppose you want to add a `table_name` attribute.

Decorator:

```python
def table(name):
    def decorate(cls):
        cls.table_name = name
        return cls
    return decorate


@table("users")
class User:
    pass
```

Metaclass:

```python
class TableMeta(type):
    def __new__(mcls, name, bases, namespace, table_name=None):
        cls = super().__new__(mcls, name, bases, namespace)
        cls.table_name = table_name
        return cls


class User(metaclass=TableMeta, table_name="users"):
    pass
```

The decorator is easier to understand.

The metaclass may be justified if many related class-creation behaviors need to be centralized.

Prefer the simpler tool unless you need the deeper hook.

---

# Descriptors Instead of Metaclasses

Suppose you want fields to know their names:

```python
class User:
    email = Field()
```

Old framework designs often used metaclasses to scan fields and assign names.

Modern Python gives descriptors `__set_name__`:

```python
class Field:
    def __set_name__(self, owner, name):
        self.owner = owner
        self.name = name
```

This may remove the need for a metaclass entirely.

Descriptors are ideal when the customization belongs to attributes.

Metaclasses are for class creation as a whole.

Ask:

```text
am I customizing one attribute's behavior?
```

Use a descriptor or property.

Ask:

```text
am I customizing how the class itself is created?
```

Maybe a metaclass.

---

# A Registry Metaclass

Here is a classic metaclass example: automatic registration.

```python
class RegistryMeta(type):
    registry = {}

    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        if name != "BasePlugin":
            mcls.registry[name] = cls
        return cls
```

Use:

```python
class BasePlugin(metaclass=RegistryMeta):
    pass


class JsonPlugin(BasePlugin):
    pass


class CsvPlugin(BasePlugin):
    pass
```

Now:

```python
RegistryMeta.registry
```

contains:

```python
{
    "JsonPlugin": JsonPlugin,
    "CsvPlugin": CsvPlugin,
}
```

This works.

But `__init_subclass__` can often do the same thing more simply:

```python
class BasePlugin:
    registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        BasePlugin.registry[cls.__name__] = cls
```

Use the metaclass version only if you need metaclass-level behavior beyond subclass notification.

---

# A Validation Metaclass

Suppose every command class must define a `name` attribute and an `execute` method.

Metaclass:

```python
class CommandMeta(type):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)

        if bases:
            if not hasattr(cls, "name"):
                raise TypeError(f"{name} must define name")
            if "execute" not in namespace:
                raise TypeError(f"{name} must define execute")

        return cls
```

Use:

```python
class Command(metaclass=CommandMeta):
    pass


class SaveCommand(Command):
    name = "save"

    def execute(self):
        print("saving")
```

This enforces rules at class creation time.

But an ABC may be better:

```python
from abc import ABC, abstractmethod


class Command(ABC):
    @abstractmethod
    def execute(self):
        raise NotImplementedError
```

ABCs are clearer for required behavior.

Metaclasses are often too blunt for ordinary interface enforcement.

---

# `ABCMeta`

Abstract base classes use a metaclass named `ABCMeta`.

Example:

```python
from abc import ABC, abstractmethod


class Shape(ABC):
    @abstractmethod
    def area(self):
        raise NotImplementedError
```

`ABC` is a helper base class whose metaclass is `ABCMeta`.

That metaclass tracks abstract methods.

It prevents instantiation until abstract methods are implemented:

```python
Shape()
```

fails.

Concrete subclass:

```python
class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2
```

Now:

```python
Circle(10)
```

works.

This is a good metaclass use case because abstract-class behavior is class-level behavior.

It affects whether a class can be instantiated and how subclass relationships are recognized.

---

# Metaclass Inheritance

If a base class uses a metaclass, subclasses usually use a compatible metaclass too.

Example:

```python
class Meta(type):
    pass


class Base(metaclass=Meta):
    pass


class Child(Base):
    pass
```

Now:

```python
type(Child)
```

is:

```python
Meta
```

The subclass inherits the metaclass behavior.

This is powerful.

It is also one reason metaclasses are invasive.

Once a base class uses a metaclass, its subclasses are affected.

Class decorators affect only the decorated class unless they deliberately modify inheritance behavior.

Metaclasses flow through class hierarchies.

---

# Metaclass Conflicts

Metaclass conflicts happen when a class inherits from bases with incompatible metaclasses.

Example:

```python
class MetaA(type):
    pass


class MetaB(type):
    pass


class A(metaclass=MetaA):
    pass


class B(metaclass=MetaB):
    pass
```

Now:

```python
class C(A, B):
    pass
```

may fail with a `TypeError`.

Python needs one metaclass for `C`.

That metaclass must be compatible with the metaclasses of all bases.

If no suitable most-derived metaclass exists, class creation fails.

One fix is to define a combined metaclass:

```python
class MetaAB(MetaA, MetaB):
    pass
```

Then:

```python
class C(A, B, metaclass=MetaAB):
    pass
```

But this can become complicated.

Metaclass conflicts are another reason custom metaclasses should be rare.

They affect composition.

---

# Metaclasses Can Customize Class Attribute Access

A metaclass is the class of a class.

So methods on the metaclass affect the class object.

Example:

```python
class Meta(type):
    def __getattr__(cls, name):
        if name == "dynamic":
            return f"dynamic value for {cls.__name__}"
        raise AttributeError(name)
```

Use:

```python
class User(metaclass=Meta):
    pass
```

Now:

```python
User.dynamic
```

returns:

```python
"dynamic value for User"
```

This customizes missing attributes on the class object.

It does not customize missing attributes on `User` instances.

For instances, define `__getattr__` on `User`.

The level matters:

```text
instance attribute behavior -> methods on the class
class attribute behavior    -> methods on the metaclass
```

---

# Metaclasses Can Customize Class Calls

Calling a class creates an instance:

```python
user = User()
```

But `User` is an object.

Calling an object uses its type's `__call__`.

For a class object, the type is its metaclass.

So a metaclass can customize class calls:

```python
class SingletonMeta(type):
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, "_instance"):
            cls._instance = super().__call__(*args, **kwargs)
        return cls._instance
```

Use:

```python
class Settings(metaclass=SingletonMeta):
    pass
```

Now:

```python
Settings() is Settings()
```

is:

```python
True
```

This demonstrates power.

It is not automatically good design.

Singletons can create hidden global state and testing problems.

The example shows that metaclasses can intercept class calls.

Use that power carefully.

---

# The Instance Creation Chain

Normally, calling a class:

```python
obj = User()
```

uses:

```text
type.__call__
```

which roughly:

1. calls `User.__new__`
2. calls `User.__init__`
3. returns the instance

If `User` has a custom metaclass, that metaclass's `__call__` can wrap or change this process.

Example:

```python
class TraceMeta(type):
    def __call__(cls, *args, **kwargs):
        print(f"creating instance of {cls.__name__}")
        return super().__call__(*args, **kwargs)
```

Use:

```python
class User(metaclass=TraceMeta):
    def __init__(self, name):
        self.name = name
```

Now:

```python
User("Maya")
```

prints:

```text
creating instance of User
```

This is class-call customization.

It is different from class-object creation.

Metaclass `__new__` creates the class.

Metaclass `__call__` affects calling the class later.

---

# A Declarative Field Example

Frameworks often use class declarations like this:

```python
class User(Model):
    id = Field("INTEGER")
    email = Field("TEXT")
```

A metaclass can collect fields:

```python
class Field:
    def __init__(self, column_type):
        self.column_type = column_type

    def __set_name__(self, owner, name):
        self.name = name
```

Metaclass:

```python
class ModelMeta(type):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)

        fields = {}
        for base in reversed(cls.__mro__):
            for attr_name, value in vars(base).items():
                if isinstance(value, Field):
                    fields[attr_name] = value

        cls.fields = fields
        return cls
```

Base:

```python
class Model(metaclass=ModelMeta):
    pass
```

Use:

```python
class User(Model):
    id = Field("INTEGER")
    email = Field("TEXT")
```

Now:

```python
User.fields
```

contains field metadata.

This is a realistic framework pattern.

Even here, modern Python can often use `__init_subclass__` plus descriptors instead.

Metaclasses are one option, not the only option.

---

# A Similar Design with `__init_subclass__`

The field collection can be done without a custom metaclass:

```python
class Model:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)

        fields = {}
        for base in reversed(cls.__mro__):
            for name, value in vars(base).items():
                if isinstance(value, Field):
                    fields[name] = value

        cls.fields = fields
```

Then:

```python
class User(Model):
    id = Field("INTEGER")
    email = Field("TEXT")
```

works similarly.

This version is usually easier to compose with other classes.

It avoids metaclass conflicts.

It is simpler to explain.

The lesson:

```text
many old metaclass examples are now better solved with __init_subclass__ and __set_name__
```

Learn metaclasses so you understand them.

Reach for simpler hooks first.

---

# `__classcell__` and `super()`

There is a subtle implementation detail around zero-argument `super()`.

Example:

```python
class Child(Base):
    def method(self):
        return super().method()
```

Python needs an implicit reference to the class being defined.

During class creation, the compiler may create a special `__class__` closure cell.

In CPython, this can appear in the namespace as `__classcell__`.

If a metaclass rewrites the namespace and fails to pass `__classcell__` to `type.__new__`, zero-argument `super()` can break.

The practical rule:

```python
class SafeMeta(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        return super().__new__(mcls, name, bases, namespace)
```

Do not casually drop namespace entries.

If you copy or filter the namespace, preserve special entries unless you know exactly what you are doing.

Metaclasses are deep enough that small mistakes can break ordinary language features.

---

# Class Creation Order Example

Here is a noisy example that shows order.

```python
class Descriptor:
    def __set_name__(self, owner, name):
        print(f"__set_name__ {owner.__name__}.{name}")


class Meta(type):
    @classmethod
    def __prepare__(mcls, name, bases):
        print(f"prepare {name}")
        return {}

    def __new__(mcls, name, bases, namespace):
        print(f"new {name}")
        return super().__new__(mcls, name, bases, namespace)

    def __init__(cls, name, bases, namespace):
        print(f"init {name}")
        super().__init__(name, bases, namespace)


class Base(metaclass=Meta):
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        print(f"init_subclass {cls.__name__}")


class Child(Base):
    field = Descriptor()
```

You do not need to memorize exact print order for daily coding.

But the example shows that class creation has phases.

Descriptors, metaclasses, and subclass hooks participate in those phases.

Understanding the phases prevents mystery.

---

# Common Mistake: Using a Metaclass for Instance Validation

Suppose you want a positive price.

Do not start with a metaclass.

Use a property:

```python
class Product:
    def __init__(self, price):
        self.price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")
        self._price = value
```

Or a descriptor if reusable:

```python
class Product:
    price = PositiveNumber()
```

Metaclasses are for class creation.

Instance value validation is usually not a metaclass problem.

---

# Common Mistake: Using a Metaclass for One Class

If only one class needs modification, a class decorator may be clearer.

Metaclass:

```python
class AddNameMeta(type):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        cls.slug = name.lower()
        return cls
```

Decorator:

```python
def add_slug(cls):
    cls.slug = cls.__name__.lower()
    return cls
```

Use:

```python
@add_slug
class User:
    pass
```

The decorator is more direct.

Metaclasses are inherited and affect subclass creation.

If you do not need that, use a simpler tool.

---

# Common Mistake: Forgetting Metaclass Conflicts

Metaclasses make inheritance harder.

If your library base class uses a custom metaclass, users combining it with another framework base class may hit conflicts.

Example:

```python
class MyModel(SomeFrameworkBase, AnotherFrameworkBase):
    pass
```

If both bases have incompatible metaclasses, class creation can fail.

This is one reason library authors should avoid custom metaclasses unless they are truly needed.

Every metaclass becomes part of the inheritance contract.

---

# Common Mistake: Hiding Too Much Work at Class Definition Time

Class bodies execute at import time.

Metaclass code also runs at import time.

If a metaclass does expensive work:

* network calls
* database queries
* file scans
* large computations
* global registration with side effects

then importing a module can become slow or fragile.

Class creation should usually be lightweight.

Frameworks may do some registration at import time, but it should be deliberate.

Avoid surprising import-time side effects.

---

# Common Mistake: Not Calling `super()`

When writing metaclasses, call `super()` unless you have a precise reason not to.

Bad:

```python
class Meta(type):
    def __new__(mcls, name, bases, namespace):
        return type.__new__(mcls, name, bases, namespace)
```

This may skip behavior from other metaclasses in a hierarchy.

Better:

```python
class Meta(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        return super().__new__(mcls, name, bases, namespace)
```

In cooperative multiple inheritance, `super()` matters.

Metaclasses are classes too.

Their inheritance should be cooperative when possible.

---

# Common Mistake: Dropping Unknown Keyword Arguments

Class customization hooks may receive keywords.

In cooperative designs, consume what you need and pass the rest along.

Example:

```python
class Base:
    def __init_subclass__(cls, table=None, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.table = table
```

This allows other base classes to receive their own keywords.

The same idea applies to metaclasses:

```python
class Meta(type):
    def __new__(mcls, name, bases, namespace, table=None, **kwargs):
        cls = super().__new__(mcls, name, bases, namespace)
        cls.table = table
        return cls
```

Be careful with keyword handling.

Class creation often involves multiple participants.

---

# When Metaclasses Are Appropriate

Metaclasses may be appropriate when:

* you are building a framework
* class declarations form a domain-specific language
* class creation must be validated centrally
* class namespaces need custom behavior before execution
* class-level behavior must be inherited automatically
* `isinstance` or `issubclass` behavior must be customized
* simpler tools cannot express the requirement cleanly

Examples:

* abstract base classes
* enum-like systems
* ORM model systems
* schema declaration systems
* plugin frameworks with strict class contracts
* advanced proxy systems

Even then, consider:

* class decorators
* descriptors
* `__set_name__`
* `__init_subclass__`
* ordinary base classes
* registries
* explicit factory functions

Metaclasses should survive comparison with simpler alternatives.

---

# When to Avoid Metaclasses

Avoid metaclasses when:

* you only need instance validation
* one class needs a small modification
* a decorator would be clearer
* `__init_subclass__` is enough
* descriptors can handle field behavior
* the team will struggle to debug it
* inheritance composition matters
* import-time side effects are risky
* the design is not yet stable

Metaclasses are not bad.

They are just high-leverage machinery.

High leverage means small mistakes can move a lot.

Use them when the problem is truly class-creation-level.

---

# A Decision Ladder

When you think you need a metaclass, climb this ladder first.

Need a simple value check?

```text
use __init__, __post_init__, property, or descriptor
```

Need reusable attribute behavior?

```text
use a descriptor
```

Need a class to react when subclassed?

```text
use __init_subclass__
```

Need to modify one class after creation?

```text
use a class decorator
```

Need named alternate constructors?

```text
use classmethod
```

Need class body declarations to collect metadata?

```text
try descriptors + __set_name__ + __init_subclass__
```

Need to control namespace preparation or class object creation itself?

```text
consider a metaclass
```

This ladder keeps designs simpler.

---

# Practice: Inspect Class Types

Run:

```python
class User:
    pass


user = User()

print(type(user))
print(type(User))
print(type(type))
```

Expected ideas:

```text
user is an instance of User
User is an instance of type
type is also an object
```

The last line may feel strange:

```python
type(type)
```

is:

```python
type
```

Python's object model eventually loops at `type`.

You do not need to meditate on that too long.

Just remember:

```text
classes are objects
```

---

# Practice: Create a Class with `type`

Create a class dynamically:

```python
def greet(self):
    return f"hello {self.name}"


User = type(
    "User",
    (),
    {
        "__init__": lambda self, name: setattr(self, "name", name),
        "greet": greet,
    },
)
```

Use:

```python
user = User("Maya")
assert user.greet() == "hello Maya"
```

This exercise shows that class syntax creates the same kind of object that `type(...)` can create manually.

Use normal class syntax in real code unless dynamic class creation is truly needed.

---

# Practice: Logging Metaclass

Write a metaclass that logs class creation.

Solution:

```python
class LoggingMeta(type):
    def __new__(mcls, name, bases, namespace):
        print(f"creating {name}")
        return super().__new__(mcls, name, bases, namespace)
```

Use:

```python
class Example(metaclass=LoggingMeta):
    pass
```

The message appears when the class definition runs.

Not when instances are created.

---

# Practice: Replace a Metaclass with `__init_subclass__`

Metaclass version:

```python
class RegistryMeta(type):
    registry = {}

    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        if bases:
            mcls.registry[name] = cls
        return cls
```

Rewrite with `__init_subclass__`:

```python
class Plugin:
    registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin.registry[cls.__name__] = cls
```

Use:

```python
class JsonPlugin(Plugin):
    pass
```

Now:

```python
Plugin.registry["JsonPlugin"] is JsonPlugin
```

should be true.

This is often the simpler design.

---

# Practice: Use `__set_name__` Instead of a Metaclass

Create a field descriptor:

```python
class Field:
    def __set_name__(self, owner, name):
        self.owner = owner
        self.name = name
```

Use:

```python
class User:
    email = Field()
```

Check:

```python
assert User.email.name == "email"
assert User.email.owner is User
```

This shows that field-name awareness does not require a metaclass.

Descriptors can receive class-creation information directly.

---

# Practice: Choose the Tool

Choose the simplest likely tool:

```text
Validate that price is non-negative
Create objects from JSON
Register subclasses automatically
Make a field know its class attribute name
Modify one class after creation
Control class namespace before body execution
Prevent instantiation of abstract classes
Collect ORM fields across inheritance
```

Likely answers:

```text
Validate price -> property or descriptor
Create objects from JSON -> classmethod
Register subclasses -> __init_subclass__
Field knows name -> __set_name__
Modify one class -> class decorator
Control namespace before body execution -> metaclass
Prevent abstract instantiation -> ABC / ABCMeta
Collect ORM fields -> descriptors + __init_subclass__, possibly metaclass for framework-level needs
```

The point is not that metaclasses are never right.

The point is that they are rarely the first right answer.

---

# Summary

Classes are objects.

Most classes are instances of `type`.

The class of a class is called its metaclass.

`type` is the default metaclass and can create classes from a name, bases, and namespace.

A custom metaclass is usually created by subclassing `type`.

Metaclasses customize class creation, not ordinary instance attribute access.

Python class creation includes determining the metaclass, preparing a namespace, executing the class body, creating the class object, calling `__set_name__`, calling `__init_subclass__`, applying class decorators, and binding the final class object to its name.

Metaclass `__new__` creates the class object.

Metaclass `__init__` initializes the class object after creation.

Metaclass `__prepare__` can provide a custom namespace before the class body executes.

Metaclass `__call__` can affect what happens when the class is called to create instances.

Metaclasses are inherited and can cause conflicts in multiple inheritance.

`ABCMeta` is an important standard-library metaclass used by abstract base classes.

Many problems that once required metaclasses can now be solved with descriptors, `__set_name__`, `__init_subclass__`, class decorators, or ordinary base classes.

The design principle is:

```text
use metaclasses only when the problem is truly about class creation itself
```

Metaclasses are powerful.

They are also contagious through inheritance and harder to compose.

Understand them deeply.

Use them sparingly.

---

# Preview of Chapter 58

Chapter 57 completes Volume II Part II: the Python data model.

We have studied:

* dunder methods
* operator overloading
* descriptors
* properties
* static methods
* class methods
* slots
* metaclasses

Next we enter Part III: Pythonic Abstractions.

Chapter 58 begins with iterators.

Iteration is one of Python's most important protocols.

It powers:

* `for` loops
* comprehensions
* unpacking
* many built-in functions
* streaming data processing
* generators
* custom containers

The transition is:

```text
the data model explains how protocols work
iterators are one of Python's most important protocols
```

After metaclasses, iterators may feel refreshingly concrete.

But they are just as fundamental.

Once you understand iteration deeply, loops, generators, comprehensions, files, lazy pipelines, and many standard-library tools become easier to reason about.

