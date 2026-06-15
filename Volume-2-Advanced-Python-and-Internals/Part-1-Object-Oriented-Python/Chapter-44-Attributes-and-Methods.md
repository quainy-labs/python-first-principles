# Chapter 44 — Attributes and Methods

---

# Learning Objectives

By the end of this chapter, you should understand:

* What an attribute is.
* How instance attributes are stored.
* How class attributes are stored.
* How Python looks up attributes.
* Why attribute lookup and attribute assignment are different.
* How instance attributes can shadow class attributes.
* How methods are functions stored on classes.
* What a bound method is.
* How `self` is passed automatically.
* Why calling `obj.method()` differs from calling `Class.method(obj)`.
* How `getattr()`, `setattr()`, `hasattr()`, and `delattr()` work.
* Why mutable class attributes are dangerous when used as per-instance state.
* How attribute lookup prepares us for properties, descriptors, inheritance, and MRO.

Chapter 43 introduced classes and instances.

Now we zoom in on the most important operation in object-oriented Python:

```text
attribute access
```

Every time you write:

```python
user.name
user.greet()
config.DEBUG
math.sqrt
```

Python is doing attribute lookup.

Understanding this one idea unlocks a surprising amount of advanced Python.

Methods, class attributes, instance attributes, properties, descriptors, inheritance, `super()`, and the data model all depend on attribute lookup.

This chapter is where the machinery starts to become visible.

---

# Concept Overview

An attribute is a name attached to an object.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Create an instance:

```python
user = User("Ada")
```

Access:

```python
print(user.name)
```

Here:

```text
name
```

is an attribute of the `user` object.

Conceptually:

```text
user instance namespace:
    name -> "Ada"
```

The expression:

```python
user.name
```

means:

```text
look up attribute name on object user
```

Attributes are how objects expose state and behavior.

State:

```python
user.name
```

Behavior:

```python
user.greet()
```

Both use attribute lookup.

---

# Attribute Access Uses Dot Syntax

Dot syntax:

```python
object.attribute
```

means:

```text
find attribute on object
```

Examples:

```python
user.name
account.balance
task.done
math.sqrt
settings.DEBUG
```

The object before the dot can be:

* An instance.
* A class.
* A module.
* A package.
* A function.
* Almost any Python object.

Dot syntax is everywhere because Python's object model is everywhere.

Example:

```python
import math

print(math.pi)
print(math.sqrt(25))
```

`math` is a module object.

`pi` and `sqrt` are attributes of that module.

Same syntax:

```python
module.attribute
instance.attribute
class.attribute
```

The details differ, but the idea is the same:

```text
attribute lookup on an object
```

---

# Instance Attributes

Instance attributes belong to individual instances.

Example:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
```

Create:

```python
ada = User("Ada", "ada@example.com")
grace = User("Grace", "grace@example.com")
```

Each instance has its own attributes:

```text
ada:
    name  -> "Ada"
    email -> "ada@example.com"

grace:
    name  -> "Grace"
    email -> "grace@example.com"
```

Use:

```python
print(ada.name)
print(grace.name)
```

Output:

```text
Ada
Grace
```

Same attribute name.

Different instance namespaces.

This is why classes can create many objects with the same structure but different state.

---

# Inspecting Instance Attributes

For ordinary Python instances, instance attributes are stored in a dictionary called `__dict__`.

Example:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email


user = User("Ada", "ada@example.com")

print(user.__dict__)
```

Output:

```text
{'name': 'Ada', 'email': 'ada@example.com'}
```

You can also use:

```python
print(vars(user))
```

Output:

```text
{'name': 'Ada', 'email': 'ada@example.com'}
```

For many ordinary classes:

```text
vars(user) shows the instance namespace
```

This is not true for every possible object.

Some classes use `__slots__`.

Some objects are implemented in C.

Some objects customize attribute access.

But for normal beginner-defined classes, `__dict__` is the right model.

---

# Class Attributes

Class attributes belong to the class object.

Example:

```python
class User:
    role = "member"

    def __init__(self, name):
        self.name = name
```

The class namespace contains:

```text
role -> "member"
__init__ -> function object
```

Access through the class:

```python
print(User.role)
```

Output:

```text
member
```

Access through an instance:

```python
ada = User("Ada")
print(ada.role)
```

Output:

```text
member
```

Why does `ada.role` work?

Because if Python does not find `role` on the instance, it can look on the class.

This is attribute lookup.

---

# Inspecting Class Attributes

Class attributes are stored in the class namespace.

Example:

```python
class User:
    role = "member"

    def greet(self):
        return "hello"
```

Inspect:

```python
print(User.__dict__)
```

You will see a mapping containing names such as:

```text
role
greet
__module__
__dict__
__weakref__
__doc__
```

Some names are added by Python automatically.

You can also use:

```python
print(vars(User))
```

The important part:

```text
role -> "member"
greet -> function object
```

A class is an object with a namespace.

Class attributes live there.

Instance attributes live on instances.

---

# Basic Attribute Lookup

For a simple instance attribute lookup:

```python
obj.name
```

Python roughly asks:

```text
1. Does the instance have this attribute?
2. If not, does the class have this attribute?
3. If not, do parent classes have this attribute?
4. If not, raise AttributeError.
```

This is simplified.

Descriptors can change the order.

We will study descriptors later.

For now, the beginner model is:

```text
instance first
class next
inherited classes later
```

Example:

```python
class User:
    role = "member"


ada = User()
ada.name = "Ada"
```

Lookup:

```python
ada.name
```

finds `name` on the instance.

Lookup:

```python
ada.role
```

does not find `role` on the instance.

Then it finds `role` on the class.

---

# Attribute Lookup Example

Code:

```python
class User:
    role = "member"

    def __init__(self, name):
        self.name = name


ada = User("Ada")
```

Namespaces:

```text
ada instance:
    name -> "Ada"

User class:
    role -> "member"
    __init__ -> function object
```

Expression:

```python
ada.name
```

Lookup:

```text
find name on ada instance -> yes -> "Ada"
```

Expression:

```python
ada.role
```

Lookup:

```text
find role on ada instance -> no
find role on User class -> yes -> "member"
```

Expression:

```python
ada.missing
```

Lookup:

```text
find missing on ada instance -> no
find missing on User class -> no
raise AttributeError
```

---

# Attribute Assignment Is Different From Lookup

This is crucial.

Attribute lookup:

```python
ada.role
```

can search the instance and then the class.

Attribute assignment:

```python
ada.role = "admin"
```

usually writes to the instance.

It does not search for `role` on the class and modify it.

Example:

```python
class User:
    role = "member"


ada = User()
grace = User()

ada.role = "admin"

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

Assignment created:

```text
ada instance:
    role -> "admin"
```

The class still has:

```text
User class:
    role -> "member"
```

Lookup and assignment are different operations.

Do not blur them.

---

# Shadowing Class Attributes

When an instance attribute has the same name as a class attribute, it shadows the class attribute for that instance.

Example:

```python
class User:
    role = "member"


ada = User()
grace = User()

ada.role = "admin"
```

Now:

```text
ada.role
```

finds the instance attribute first:

```text
admin
```

But:

```text
grace.role
```

does not find `role` on `grace`, so it finds the class attribute:

```text
member
```

The class attribute still exists:

```python
print(User.role)
```

Output:

```text
member
```

Shadowing means:

```text
a nearer namespace has a name that hides a farther namespace's name during lookup
```

This is the same concept we saw with local variables shadowing globals and built-ins.

---

# Deleting an Instance Attribute Can Reveal a Class Attribute

Example:

```python
class User:
    role = "member"


ada = User()
ada.role = "admin"

print(ada.role)

del ada.role

print(ada.role)
```

Output:

```text
admin
member
```

What happened?

Before deletion:

```text
ada instance:
    role -> "admin"

User class:
    role -> "member"
```

After:

```python
del ada.role
```

the instance attribute is removed.

Now lookup:

```python
ada.role
```

does not find `role` on the instance.

So it finds `role` on the class.

Deletion did not create the class attribute.

It revealed the one that was already there.

---

# Mutating a Class Attribute Through an Instance

Assignment and mutation are different.

Consider:

```python
class Team:
    members = []


red = Team()
blue = Team()

red.members.append("Ada")

print(blue.members)
```

Output:

```text
['Ada']
```

Why?

Lookup:

```python
red.members
```

does not find `members` on `red`.

It finds `members` on the class.

That object is a list.

Then:

```python
.append("Ada")
```

mutates the list.

It does not assign a new `members` attribute to `red`.

So both instances still see the shared class list.

This is the classic mutable class attribute trap.

---

# Assignment Creates an Instance Attribute

Compare:

```python
class Team:
    members = []


red = Team()
blue = Team()

red.members = ["Ada"]
```

Now:

```python
print(red.members)
print(blue.members)
print(Team.members)
```

Output:

```text
['Ada']
[]
[]
```

Why?

Assignment:

```python
red.members = ["Ada"]
```

creates an instance attribute on `red`.

It does not mutate `Team.members`.

Namespaces:

```text
red instance:
    members -> ["Ada"]

blue instance:
    no members

Team class:
    members -> []
```

`blue.members` still finds the class list.

This contrast is vital:

```text
red.members.append(...) -> lookup then mutate found object
red.members = ...       -> assign attribute on red
```

---

# Correct Per-Instance Mutable Attributes

If each instance needs its own list, create the list in `__init__`.

Correct:

```python
class Team:
    def __init__(self, name):
        self.name = name
        self.members = []

    def add_member(self, member):
        self.members.append(member)
```

Use:

```python
red = Team("Red")
blue = Team("Blue")

red.add_member("Ada")

print(red.members)
print(blue.members)
```

Output:

```text
['Ada']
[]
```

Each instance has its own namespace:

```text
red:
    name -> "Red"
    members -> red's list

blue:
    name -> "Blue"
    members -> blue's list
```

Rule:

```text
mutable state that belongs to one object should usually be created on that object
```

---

# Methods Are Attributes Too

A method is found through attribute lookup.

Example:

```python
class User:
    def greet(self):
        return "hello"


user = User()
```

Call:

```python
user.greet()
```

Before Python calls anything, it must evaluate:

```python
user.greet
```

That is attribute lookup.

Where is `greet` stored?

In the class namespace:

```text
User class:
    greet -> function object
```

When accessed through the instance, Python returns a bound method:

```python
method = user.greet
print(method)
```

The bound method remembers both:

```text
the function
the instance
```

Then:

```python
method()
```

passes the instance as `self`.

---

# Functions Stored on Classes

Inside a class body:

```python
class User:
    def greet(self):
        return "hello"
```

the `def` statement creates a function object.

That function object is stored in the class namespace under the name `greet`.

Conceptually:

```text
User class namespace:
    greet -> function object
```

Access through the class:

```python
print(User.greet)
```

You see a function.

Access through an instance:

```python
user = User()
print(user.greet)
```

You see a bound method.

The function did not change permanently.

Attribute access transformed it for this access.

That transformation is part of the descriptor protocol.

We will study descriptors later.

For now:

```text
class access gives function
instance access gives bound method
```

---

# Bound Methods

A bound method is a callable object that remembers an instance.

Example:

```python
class User:
    def greet(self):
        return f"hello from {self.name}"

    def __init__(self, name):
        self.name = name


user = User("Ada")
method = user.greet
```

Now:

```python
print(method())
```

Output:

```text
hello from Ada
```

The method object remembers:

```text
function -> User.greet
self     -> user
```

You can inspect:

```python
print(method.__self__)
print(method.__func__)
```

`__self__` is the bound instance.

`__func__` is the original function.

These details are not used every day.

But they prove the model:

```text
bound method = function + instance
```

---

# `obj.method()` vs `Class.method(obj)`

These are usually equivalent for ordinary instance methods:

```python
obj.method()
```

and:

```python
Class.method(obj)
```

Example:

```python
class User:
    def greet(self):
        return f"hello {self.name}"

    def __init__(self, name):
        self.name = name


ada = User("Ada")

print(ada.greet())
print(User.greet(ada))
```

Both output:

```text
hello Ada
```

Why?

```python
ada.greet()
```

gets a bound method that already remembers `ada`.

```python
User.greet(ada)
```

gets the raw function from the class and passes `ada` manually.

This explains `self`.

`self` is not magic inside the function.

It is just the first parameter.

The method call syntax supplies it automatically.

---

# Method Lookup Uses Attributes

Method lookup follows attribute lookup rules.

Example:

```python
class User:
    def describe(self):
        return "user"


user = User()
```

When you call:

```python
user.describe()
```

Python:

```text
1. looks for describe on user instance
2. if not found, looks on User class
3. finds function object
4. binds it to user
5. calls bound method
```

This means methods are not a separate lookup category.

They are attributes that happen to be callable and bind specially when they are functions stored on classes.

This is one of Python's elegant unifications:

```text
data attribute lookup
method lookup
module attribute lookup
class attribute lookup
```

all use the attribute model.

---

# Overriding a Method on One Instance

Because methods are attributes, you can shadow a method on a single instance.

Example:

```python
class User:
    def greet(self):
        return "hello"


user = User()
user.greet = lambda: "custom hello"

print(user.greet())
```

Output:

```text
custom hello
```

What happened?

The instance now has an attribute named `greet`.

Lookup finds the instance attribute before the class method.

Namespace:

```text
user instance:
    greet -> lambda function

User class:
    greet -> original function
```

This is flexible.

It can also be confusing.

Do not usually replace methods on individual instances in ordinary application code.

But knowing it is possible helps explain how attribute lookup works.

---

# Callable Attributes Are Not Always Methods

If an instance attribute is callable, you can call it.

Example:

```python
class Button:
    pass


button = Button()
button.action = lambda: "clicked"

print(button.action())
```

Output:

```text
clicked
```

But `action` is not a normal bound method.

It is just a function object stored directly on the instance.

It does not automatically receive `self`.

Example:

```python
button.action = lambda self: "clicked"
button.action()
```

This fails because no `self` is passed automatically for plain functions stored on instances.

Automatic method binding happens for functions found on classes.

This distinction matters.

---

# Attribute Assignment

Attribute assignment:

```python
obj.name = value
```

usually stores a name on the object.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"
```

Now:

```python
print(user.__dict__)
```

Output:

```text
{'name': 'Ada'}
```

For ordinary instances, assignment writes to the instance namespace.

But there are advanced exceptions:

* Properties can intercept assignment.
* Descriptors can intercept assignment.
* `__setattr__` can customize assignment.
* `__slots__` can restrict available attributes.

We will study these later.

For now:

```text
obj.name = value
```

usually creates or updates an instance attribute.

---

# Attribute Deletion

Attribute deletion:

```python
del obj.name
```

usually removes an attribute from the object.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"

print(user.name)

del user.name

print(user.name)
```

The last line raises:

```text
AttributeError
```

because the instance attribute no longer exists.

If a class attribute with the same name exists, deletion can reveal it:

```python
class User:
    role = "member"


user = User()
user.role = "admin"

del user.role

print(user.role)
```

Output:

```text
member
```

Again:

```text
delete instance attribute
then lookup can find class attribute
```

---

# `getattr()`

`getattr()` gets an attribute by name.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


user = User("Ada")

print(getattr(user, "name"))
```

Output:

```text
Ada
```

This is equivalent to:

```python
user.name
```

when the attribute name is known in code.

`getattr()` is useful when the attribute name is dynamic:

```python
field = "name"
print(getattr(user, field))
```

You can provide a default:

```python
print(getattr(user, "email", "missing"))
```

Output:

```text
missing
```

Without a default, missing attributes raise `AttributeError`.

---

# `setattr()`

`setattr()` sets an attribute by name.

Example:

```python
class User:
    pass


user = User()

setattr(user, "name", "Ada")

print(user.name)
```

Output:

```text
Ada
```

This is equivalent to:

```python
user.name = "Ada"
```

when the attribute name is known in code.

`setattr()` is useful for dynamic names:

```python
field = "email"
value = "ada@example.com"

setattr(user, field, value)
```

Use it carefully.

Dynamic attribute creation can make objects harder to understand if overused.

Clear explicit attributes are usually better.

---

# `hasattr()`

`hasattr()` checks whether an object has an attribute.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"

print(hasattr(user, "name"))
print(hasattr(user, "email"))
```

Output:

```text
True
False
```

At a high level:

```python
hasattr(obj, name)
```

tries to get the attribute.

If lookup succeeds, it returns `True`.

If lookup raises `AttributeError`, it returns `False`.

This means `hasattr()` can trigger custom attribute logic.

For ordinary objects, that is fine.

For advanced objects with properties or custom `__getattr__`, remember that attribute checks may run code.

---

# `delattr()`

`delattr()` deletes an attribute by name.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"

delattr(user, "name")

print(hasattr(user, "name"))
```

Output:

```text
False
```

This is equivalent to:

```python
del user.name
```

when the attribute name is known.

Use `delattr()` for dynamic names:

```python
field = "name"
delattr(user, field)
```

Again, dynamic attribute manipulation is powerful.

It should be used when the design calls for it, not as a default habit.

---

# `dir()`

`dir()` lists names available on an object.

Example:

```python
class User:
    role = "member"

    def __init__(self, name):
        self.name = name

    def greet(self):
        return "hello"


user = User("Ada")

print(dir(user))
```

You may see names such as:

```text
name
role
greet
__class__
__dict__
...
```

`dir()` includes more than instance attributes.

It includes names available through the object's type and inheritance.

Use:

```python
vars(user)
```

to see direct instance attributes.

Use:

```python
dir(user)
```

to inspect available names more broadly.

---

# Missing Attributes

If an attribute is not found, Python raises `AttributeError`.

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
Python looked for name on the instance
then on the class
then inherited classes
it did not find name
```

Compare with:

```python
print(name)
```

That raises:

```text
NameError
```

because plain name lookup failed.

Different operation:

```text
name       -> plain name lookup
user.name  -> attribute lookup
```

Different error:

```text
NameError
AttributeError
```

---

# Attribute Lookup and Inheritance Preview

We have mostly discussed one class.

Inheritance adds parent classes.

Example:

```python
class Animal:
    def speak(self):
        return "sound"


class Dog(Animal):
    pass


dog = Dog()
print(dog.speak())
```

Output:

```text
sound
```

Where did `speak` come from?

Not from the `dog` instance.

Not from the `Dog` class.

It came from the parent class `Animal`.

Inheritance extends attribute lookup:

```text
instance
class
parent classes
```

For multiple inheritance, the order becomes more interesting.

That is MRO.

We will study it later.

---

# Attribute Lookup and Descriptors Preview

Our simplified model says:

```text
instance first
class next
```

But descriptors can alter this.

Properties, methods, static methods, class methods, and many advanced Python features rely on descriptors.

Example:

```python
class User:
    @property
    def name(self):
        return "Ada"
```

Here:

```python
user.name
```

looks like ordinary attribute access.

But it runs a method behind the scenes.

That is descriptor behavior.

We are not studying descriptors fully yet.

But remember:

```text
attribute lookup is customizable
```

This is why attribute lookup is central to advanced Python.

---

# Attribute Lookup and Properties Preview

Properties let method logic appear as attribute access.

Example:

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height
```

Use:

```python
rectangle = Rectangle(10, 5)
print(rectangle.area)
```

No parentheses.

It looks like data.

But Python calls the property's getter method.

This is possible because attribute lookup has hooks.

Chapter 45 will introduce managed attributes and encapsulation.

Chapter 54 will go deeper into descriptors.

For now, see properties as a preview of the power hidden behind dot syntax.

---

# Attribute Assignment and Properties Preview

Properties can also manage assignment.

Example idea:

```python
class User:
    @property
    def age(self):
        ...

    @age.setter
    def age(self, value):
        ...
```

Then:

```python
user.age = 30
```

may run validation code.

This is different from ordinary instance attribute assignment.

So far, our simple rule has been:

```text
obj.name = value writes to obj
```

That is true for ordinary attributes.

But managed attributes can intercept assignment.

This is why we are moving from:

```text
basic attributes
```

to:

```text
managed attributes
```

in the next chapter.

---

# Attribute Access Is Part of API Design

When you expose an attribute, you expose part of an object's interface.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Other code may use:

```python
user.name
```

That becomes part of how the object is used.

If later you rename it:

```python
user.full_name
```

you may break callers.

Attributes are not just implementation details once other code depends on them.

Object API includes:

* Attributes callers read.
* Attributes callers assign.
* Methods callers call.
* Exceptions methods raise.
* Behavior methods promise.

Designing attributes is designing an interface.

---

# Public and Internal Attributes

Python uses naming conventions.

Public attribute:

```python
user.name
```

Internal-looking attribute:

```python
user._name
```

The leading underscore means:

```text
this is intended for internal use
```

It is not strict privacy.

Other code can still access:

```python
user._name
```

But the convention says:

```text
do not rely on this from outside the class
```

This convention matters because Python does not force heavy access restrictions.

Python relies on clear interfaces and responsible usage.

Chapter 45 will discuss encapsulation more carefully.

---

# A Complete Example: User

Class:

```python
class User:
    role = "member"

    def __init__(self, name, email):
        self.name = name
        self.email = email

    def display(self):
        return f"{self.name} <{self.email}>"
```

Use:

```python
ada = User("Ada", "ada@example.com")
```

Namespaces:

```text
ada instance:
    name  -> "Ada"
    email -> "ada@example.com"

User class:
    role    -> "member"
    __init__ -> function object
    display -> function object
```

Lookups:

```python
ada.name
```

finds instance attribute.

```python
ada.role
```

finds class attribute.

```python
ada.display
```

finds function on class and returns bound method.

```python
ada.display()
```

calls the bound method.

This one example shows the whole chapter in miniature.

---

# A Complete Example: Counter

Class:

```python
class Counter:
    step = 1

    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += self.step
        return self.value
```

Use:

```python
counter = Counter()

print(counter.increment())
print(counter.increment())
```

Output:

```text
1
2
```

Lookup inside `increment`:

```python
self.value
```

finds instance attribute.

```python
self.step
```

does not find `step` on the instance, so it finds class attribute.

Now:

```python
counter.step = 5
```

This creates an instance attribute.

Next:

```python
print(counter.increment())
```

uses instance `step`, not class `step`.

This is a practical use of class defaults that instances can override.

---

# A Complete Example: Plugin Registry

Class attributes can represent shared data when sharing is intentional.

Example:

```python
class PluginRegistry:
    plugins = {}

    @classmethod
    def register(cls, name, plugin):
        cls.plugins[name] = plugin
```

We have not deeply studied `@classmethod` yet.

But the idea is:

```text
plugins belongs to the class, not one instance
```

Shared class attributes are appropriate when the data is truly shared.

Bad:

```text
per-team members list stored on Team class
```

Good:

```text
one registry shared by the registry class
```

The issue is not mutability itself.

The issue is accidental sharing.

Mutable class attributes are fine when shared mutation is the design.

They are dangerous when each instance should have its own state.

---

# Common Mistake: Thinking `obj.attr = value` Updates the Class

Example:

```python
class User:
    role = "member"


user = User()
user.role = "admin"
```

This does not update:

```python
User.role
```

It creates or updates:

```python
user.role
```

Check:

```python
print(User.role)
print(user.role)
```

Output:

```text
member
admin
```

If you want to update the class attribute:

```python
User.role = "admin"
```

Then instances without their own `role` will see the new class value.

Attribute assignment target matters.

---

# Common Mistake: Mutating Shared Class Data Accidentally

Bad:

```python
class Cart:
    items = []

    def add(self, item):
        self.items.append(item)
```

Every cart shares the same list.

Correct:

```python
class Cart:
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)
```

Now each cart has its own list.

Ask:

```text
Should this data be shared by all instances?
```

If yes, class attribute may be right.

If no, use instance attribute.

---

# Common Mistake: Forgetting Methods Are Attributes

Beginners often think methods are stored on each instance.

For normal methods, they are not.

Example:

```python
class User:
    def greet(self):
        return "hello"
```

The function `greet` is stored on the class.

Instances access it through attribute lookup.

This means:

```python
User.greet
```

and:

```python
user.greet
```

are related but not identical.

Class access gives the function.

Instance access gives a bound method.

This is why `self` appears automatically in normal method calls.

---

# Common Mistake: Calling a Class Function Without an Instance

Example:

```python
class User:
    def greet(self):
        return "hello"


User.greet()
```

This fails because `self` is missing.

Correct:

```python
user = User()
user.greet()
```

or:

```python
User.greet(user)
```

The method needs an instance.

Access through an instance binds the instance automatically.

Access through the class gives the raw function, so you must pass the instance yourself.

---

# Common Mistake: Using `hasattr()` Without Realizing It Runs Lookup

Example:

```python
hasattr(obj, "name")
```

This performs attribute lookup.

For simple objects, that is harmless.

But for objects with properties or custom attribute access, lookup can run code.

Example:

```python
class Example:
    @property
    def value(self):
        print("checking value")
        return 10
```

Then:

```python
hasattr(Example(), "value")
```

can print:

```text
checking value
```

This is not a reason to avoid `hasattr()`.

It is a reason to understand it.

It asks:

```text
can this attribute be looked up without AttributeError?
```

not:

```text
is there a simple key in __dict__?
```

---

# Common Mistake: Overusing Dynamic Attributes

Dynamic attribute tools:

```python
getattr()
setattr()
delattr()
```

are powerful.

But code like this can become hard to understand:

```python
for key, value in data.items():
    setattr(obj, key, value)
```

Now object attributes depend on external data.

Maybe that is exactly right.

Maybe it is dangerous.

Ask:

```text
Which attributes should this object have?
Are they known and documented?
Could unexpected keys overwrite important attributes?
Should this data stay in a dictionary instead?
```

Classes should make structure clearer.

Do not use dynamic attributes in a way that hides structure.

---

# Design Guidance

When designing attributes and methods, ask:

```text
What state belongs to each instance?
What data should be shared by the class?
Which attributes are public?
Which attributes are internal?
Which behavior belongs as a method?
Should callers read this as an attribute or call it as a method?
Could a class attribute accidentally be mutated by all instances?
Does assignment mean per-instance override or shared class change?
```

Basic rules:

* Store per-object state on instances.
* Store truly shared defaults or constants on classes.
* Avoid mutable class attributes unless sharing is intentional.
* Use methods for behavior.
* Keep public attributes clear and stable.
* Use leading underscores for internal implementation details.
* Prefer explicit attributes over mysterious dynamic ones.

Good object design is mostly clear ownership of names and behavior.

---

# Exercises

1. Create:

```python
class User:
    role = "member"

    def __init__(self, name):
        self.name = name
```

Create a user.

Print:

```python
vars(user)
vars(User)
```

Which namespace contains `name`?

Which namespace contains `role`?

---

2. Predict the output:

```python
class User:
    role = "member"


ada = User()
grace = User()

ada.role = "admin"

print(ada.role)
print(grace.role)
print(User.role)
```

Explain using attribute lookup and assignment.

---

3. Predict the output:

```python
class Team:
    members = []


red = Team()
blue = Team()

red.members.append("Ada")

print(blue.members)
```

Why does this happen?

Rewrite the class so each team has its own members list.

---

4. Create a class with a method:

```python
class Greeter:
    def greet(self):
        return "hello"
```

Inspect:

```python
print(Greeter.greet)
print(Greeter().greet)
```

What is the difference?

---

5. Show that these are equivalent:

```python
obj.method()
Class.method(obj)
```

using a small class of your own.

---

6. Use `getattr()` to read an attribute whose name is stored in a variable:

```python
field = "name"
```

Then use `setattr()` to set another attribute dynamically.

When is this useful?

When might it be dangerous?

---

7. Explain the difference between:

```python
obj.attr = value
```

and:

```python
obj.attr.append(value)
```

Use a class attribute list example.

---

8. Create an instance attribute that shadows a class attribute.

Then delete the instance attribute.

Show that the class attribute becomes visible again.

---

9. Explain why this fails:

```python
class User:
    def greet(self):
        return "hello"


User.greet()
```

How can you call it correctly?

---

10. In your own words, explain:

```text
methods are attributes found through lookup
```

Use the terms:

```text
class namespace
function object
bound method
self
```

---

# Summary

In this chapter we learned:

* Attributes are names attached to objects.
* Dot syntax performs attribute access.
* Instance attributes live on individual instances.
* Class attributes live on class objects.
* Ordinary instances often store attributes in `__dict__`.
* Class objects have namespaces too.
* Attribute lookup can find names on instances and classes.
* Attribute assignment usually writes to the instance.
* Instance attributes can shadow class attributes.
* Deleting an instance attribute can reveal a class attribute.
* Mutating a found class attribute can affect all instances.
* Methods are functions stored on classes.
* Accessing a method through an instance creates a bound method.
* A bound method remembers the function and the instance.
* `obj.method()` usually behaves like `Class.method(obj)`.
* `getattr()`, `setattr()`, `hasattr()`, and `delattr()` provide dynamic attribute access tools.
* Attribute lookup prepares us for inheritance, properties, descriptors, and MRO.

Core model:

```text
obj.attr lookup:
    look on object
    look on class
    look on parent classes
    maybe invoke descriptor behavior

obj.attr = value:
    usually assign on object
```

Method model:

```text
class namespace:
    method_name -> function object

instance lookup:
    instance.method_name -> bound method

bound method:
    function + instance
```

Attribute lookup is the engine under object-oriented Python.

Once you understand it, the advanced topics become far less mysterious.

---

# Preview of Chapter 45

Next we study encapsulation and managed attributes.

So far, attributes have mostly been direct:

```python
user.name
user.email
account.balance
```

But real programs often need rules around attributes.

Chapter 45 explains how Python handles that without abandoning its object model.

We will study:

* What encapsulation means in Python.
* Public and internal attributes.
* Leading underscore conventions.
* Name mangling with double underscores.
* Why Python does not use strict private fields by default.
* How properties manage attribute access.
* How validation can happen during assignment.
* Why methods and properties are API design tools.
* How managed attributes prepare us for descriptors.

The transition is direct:

```text
attribute lookup explains how attributes are found
managed attributes explain how attribute access can be controlled
```

This is where Python's practical approach to encapsulation begins.
