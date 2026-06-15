# Chapter 43 — Classes and Instances

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a class is in Python.
* What an instance is.
* Why classes are objects too.
* How class definitions create class objects.
* How instances are created from classes.
* What `self` means.
* How `__init__` initializes a new instance.
* Why instance attributes belong to individual objects.
* Why class attributes belong to the class object.
* How classes connect to namespaces.
* How classes connect to the object/reference model from Volume I.
* How `type()` and `isinstance()` help inspect objects.
* Why object-oriented programming is about modeling behavior and state together.
* How this chapter prepares us for attributes, methods, inheritance, MRO, descriptors, and the Python data model.

Welcome to Volume II.

Volume I built the ground:

```text
objects
references
mutability
functions
data structures
memory
modules
imports
namespaces
```

Volume II begins by asking:

```text
How do we define our own kinds of objects?
```

That is what classes do.

A class lets you define a new kind of object.

An instance is one object created from that class.

This sounds simple.

But in Python, classes are not just syntax for grouping functions.

Classes are runtime objects.

Instances are runtime objects.

Methods are function objects connected through attribute lookup.

Inheritance, descriptors, properties, dataclasses, MRO, and metaclasses all build on this foundation.

So we will go slowly.

Not because classes are impossible.

Because classes are the doorway to Python's advanced object model.

---

# Concept Overview

A class defines a kind of object.

Example:

```python
class User:
    pass
```

This creates a class object named `User`.

You can create an instance:

```python
ada = User()
```

Now:

```text
User -> class object
ada  -> User instance
```

Conceptually:

```text
User ──▶ class object

ada  ──▶ instance of User
```

The class is the template-like object.

The instance is an individual object created from the class.

But be careful with the word template.

In Python, a class is not only a static blueprint.

It is a live object with a namespace.

It can be passed around, stored in variables, inspected, modified, and used to create instances.

This is why Python's object model is so flexible.

---

# Why Classes Exist

Functions let us group behavior.

Dictionaries let us group data.

Classes let us group data and behavior together.

Suppose we represent a user with a dictionary:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
}
```

We can write functions:

```python
def display_name(user):
    return user["name"].title()
```

Use:

```python
print(display_name(user))
```

This works.

But as behavior grows, the connection between the data and the functions can become scattered.

Classes let us express:

```text
this data and this behavior belong together
```

Example:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def display_name(self):
        return self.name.title()
```

Use:

```python
user = User("Ada", "ada@example.com")
print(user.display_name())
```

Now the object carries both:

```text
state    -> name and email
behavior -> display_name
```

Object-oriented programming begins there.

---

# Class Definitions Create Objects

When Python executes a class statement, it creates a class object.

Example:

```python
class User:
    pass
```

This is executable code.

It is not merely a declaration read by a compiler long before runtime.

Python executes the class body and creates a class object.

Then it binds the class name:

```text
User -> class object
```

You can prove the class is an object:

```python
class User:
    pass


print(User)
print(type(User))
```

Output looks like:

```text
<class '__main__.User'>
<class 'type'>
```

The class `User` is itself an object.

Its type is `type`.

We will study metaclasses later.

For now, remember:

```text
classes are objects that create instances
```

---

# Instances Are Objects Created From Classes

After defining:

```python
class User:
    pass
```

you can call the class:

```python
ada = User()
```

This creates a new instance.

The name `ada` refers to that instance.

Check:

```python
print(type(ada))
```

Output:

```text
<class '__main__.User'>
```

The instance's type is `User`.

Conceptually:

```text
User() creates a new User instance

ada ──▶ User instance
```

Each call creates a separate instance:

```python
ada = User()
grace = User()

print(ada is grace)
```

Output:

```text
False
```

Both are `User` instances.

They are not the same object.

This follows the identity model from Volume I.

---

# Class Names Are Normal Names

The name of a class is a normal name binding.

Example:

```python
class User:
    pass
```

This binds:

```text
User -> class object
```

You can assign another name to the same class:

```python
Person = User
```

Now:

```text
User   ──┐
         ▼
Person ─▶ class object
```

Use:

```python
ada = Person()
print(type(ada))
```

Output:

```text
<class '__main__.User'>
```

The class object still has its original class name for representation.

But `Person` refers to the same class object.

This matters because classes obey the same reference rules as other objects:

* They can be assigned to names.
* They can be stored in containers.
* They can be passed to functions.
* They can be returned from functions.
* They can have attributes.

Python is consistent here.

Classes are first-class objects.

---

# Calling a Class

When you write:

```python
user = User()
```

you are calling the class object.

At a beginner level, class calling means:

```text
create a new instance
initialize it
return it
```

This process involves advanced hooks:

```text
__new__
__init__
metaclass __call__
```

We will study those later.

For now, the practical model is:

```text
ClassName(arguments) -> new initialized instance
```

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


ada = User("Ada")
```

The call:

```python
User("Ada")
```

creates an instance and runs initialization code.

The result is bound to `ada`.

---

# `__init__`

`__init__` is the initializer method.

Example:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
```

When you call:

```python
user = User("Ada", "ada@example.com")
```

Python creates a new `User` instance and calls:

```python
user.__init__("Ada", "ada@example.com")
```

Conceptually.

Inside `__init__`, `self` refers to the new instance.

The assignments:

```python
self.name = name
self.email = email
```

store attributes on that instance.

After initialization:

```python
print(user.name)
print(user.email)
```

Output:

```text
Ada
ada@example.com
```

`__init__` prepares the object for use.

It should usually leave the object in a valid starting state.

---

# `__init__` Does Not Create the Object

Beginners often say:

```text
__init__ creates the object
```

That is close enough for casual speech, but not technically correct.

More precise:

```text
__init__ initializes an object that has already been created
```

Object creation involves `__new__`.

Initialization involves `__init__`.

Example:

```text
__new__  -> create object
__init__ -> initialize object
```

Most classes do not need to define `__new__`.

Most classes define `__init__`.

For now, use this model:

```text
User("Ada")
    creates a User instance
    passes it to __init__
    returns the initialized instance
```

We will revisit creation and initialization when we study the data model and metaclasses.

---

# `self`

`self` is the conventional name for the current instance.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello, {self.name}"
```

When you call:

```python
ada = User("Ada")
ada.greet()
```

inside `greet`, `self` refers to `ada`.

Conceptually:

```text
self -> ada instance
```

`self` is not a keyword.

Python does not require the name `self`.

This works but is bad style:

```python
class User:
    def __init__(this_object, name):
        this_object.name = name
```

Use `self`.

Every Python reader expects it.

Conventions are part of readability.

---

# Why Methods Take `self`

Look at:

```python
class User:
    def greet(self):
        return "hello"
```

Use:

```python
user = User()
print(user.greet())
```

It looks like `greet` is called with no arguments.

But the method definition expects one parameter:

```python
def greet(self):
```

What happened?

When you access a function through an instance:

```python
user.greet
```

Python creates a bound method.

That bound method remembers:

```text
function -> User.greet
instance -> user
```

When you call:

```python
user.greet()
```

Python passes the instance as the first argument automatically.

Conceptually:

```python
User.greet(user)
```

This is a key piece of Python's object model.

We will study methods more deeply in Chapter 44.

---

# Instance Attributes

Instance attributes are attributes stored on individual instances.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Create two users:

```python
ada = User("Ada")
grace = User("Grace")
```

Each instance has its own `name` attribute:

```text
ada instance namespace:
    name -> "Ada"

grace instance namespace:
    name -> "Grace"
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

This is exactly the namespace model from Chapter 41.

Classes make that model practical for custom objects.

---

# Inspecting Instance Attributes

For many normal Python objects, you can inspect instance attributes with `vars()`.

Example:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email


user = User("Ada", "ada@example.com")

print(vars(user))
```

Output:

```text
{'name': 'Ada', 'email': 'ada@example.com'}
```

This shows the instance namespace.

At a simple level:

```text
user.name is stored in user.__dict__["name"]
```

Do not rely on this for every object.

Some classes use `__slots__` or custom attribute behavior.

But for ordinary beginner classes, `vars(instance)` is a clear way to see instance state.

This connects directly to namespaces.

---

# Class Attributes

Class attributes are attributes stored on the class object.

Example:

```python
class User:
    species = "human"

    def __init__(self, name):
        self.name = name
```

Here:

```python
species = "human"
```

is in the class body.

It becomes a class attribute.

Access:

```python
print(User.species)
```

Output:

```text
human
```

Instances can also access class attributes:

```python
ada = User("Ada")
print(ada.species)
```

Output:

```text
human
```

Python does not copy `species` into each instance.

It finds the attribute on the class when it is not found on the instance.

We will study attribute lookup in detail in Chapter 44.

---

# Class Attribute vs Instance Attribute

Example:

```python
class User:
    role = "member"

    def __init__(self, name):
        self.name = name
```

Create:

```python
ada = User("Ada")
grace = User("Grace")
```

Access:

```python
print(ada.role)
print(grace.role)
```

Both output:

```text
member
```

Now:

```python
ada.role = "admin"
```

This creates an instance attribute on `ada`.

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

Namespace explanation:

```text
ada instance:
    role -> "admin"

grace instance:
    no role

User class:
    role -> "member"
```

Same name.

Different namespaces.

Lookup rules decide what is found.

---

# Be Careful With Mutable Class Attributes

Mutable class attributes can surprise beginners.

Example:

```python
class Team:
    members = []

    def __init__(self, name):
        self.name = name
```

Create:

```python
red = Team("Red")
blue = Team("Blue")

red.members.append("Ada")

print(blue.members)
```

Output:

```text
['Ada']
```

Why?

`members` is stored on the class.

Both instances find the same list through the class.

Conceptually:

```text
Team.members -> one shared list
red.members  -> same list through class lookup
blue.members -> same list through class lookup
```

If each instance should have its own list, create it in `__init__`:

```python
class Team:
    def __init__(self, name):
        self.name = name
        self.members = []
```

Now each instance gets its own list.

This is one of the most important beginner class mistakes.

---

# The Class Body

A class body is executed when Python creates the class.

Example:

```python
class Example:
    print("inside class body")
    value = 10
```

When the class statement runs, it prints:

```text
inside class body
```

The class body creates a namespace.

Names assigned in the class body become class attributes.

Example:

```python
class Example:
    value = 10

    def show(self):
        return self.value
```

Class namespace:

```text
value -> 10
show  -> function object
```

After the class object is created, the name `Example` is bound to it.

This is a runtime process.

It is not just static description.

---

# Functions Inside Classes

When you define a function inside a class body:

```python
class User:
    def greet(self):
        return "hello"
```

the class namespace gets:

```text
greet -> function object
```

You can inspect:

```python
print(User.greet)
```

You will see a function-like representation.

When accessed through an instance:

```python
user = User()
print(user.greet)
```

you see a bound method.

This transformation is part of Python's descriptor machinery.

We will study descriptors later.

For now:

```text
function stored on class
access through instance
becomes bound method
```

That is why `self` is passed automatically.

---

# Classes Connect Data and Behavior

Without classes:

```python
def area(rectangle):
    return rectangle["width"] * rectangle["height"]


rectangle = {"width": 10, "height": 5}
print(area(rectangle))
```

With classes:

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height


rectangle = Rectangle(10, 5)
print(rectangle.area())
```

Now the `area` behavior lives with the rectangle object.

This can improve readability:

```python
rectangle.area()
```

means:

```text
ask this rectangle for its area
```

Object-oriented design is not always better.

But it is powerful when data and behavior naturally belong together.

---

# A Class Is Not Just a Dictionary Replacement

It is tempting to replace every dictionary with a class.

Do not.

Dictionaries are excellent for flexible key-value data.

Example:

```python
settings = {
    "theme": "dark",
    "language": "en",
}
```

This may not need a class.

Classes are useful when:

* Objects have meaningful behavior.
* Objects have consistent structure.
* You want clear construction rules.
* You want methods attached to data.
* You want type-based behavior.
* You want inheritance or protocols later.

Dictionary:

```text
flexible mapping of keys to values
```

Class:

```text
defined kind of object with state and behavior
```

Choose based on the problem.

Professional Python uses both.

---

# A Class Creates a Type

When you define:

```python
class User:
    pass
```

you create a new type of object.

Check:

```python
user = User()

print(type(user))
```

Output:

```text
<class '__main__.User'>
```

The class `User` is the type of `user`.

You can ask:

```python
print(isinstance(user, User))
```

Output:

```text
True
```

This means:

```text
user is an instance of User
```

Types are central to Python's runtime behavior.

Even though Python is dynamically typed, objects still have types.

Dynamic typing means names do not have fixed declared types.

It does not mean objects lack types.

---

# `type()`

`type()` returns the type of an object.

Examples:

```python
print(type(10))
print(type("Ada"))
print(type([]))
```

Output:

```text
<class 'int'>
<class 'str'>
<class 'list'>
```

For custom classes:

```python
class User:
    pass


user = User()
print(type(user))
```

Output:

```text
<class '__main__.User'>
```

Use `type()` for inspection.

But do not overuse exact type checks in design.

Later we will study polymorphism and duck typing.

Often, what an object can do matters more than its exact type.

For now, `type()` helps us see the class-instance relationship.

---

# `isinstance()`

`isinstance()` checks whether an object is an instance of a class or a compatible class through inheritance.

Example:

```python
class User:
    pass


user = User()

print(isinstance(user, User))
```

Output:

```text
True
```

With built-in types:

```python
print(isinstance(10, int))
print(isinstance("Ada", str))
print(isinstance([], list))
```

Output:

```text
True
True
True
```

`isinstance()` is more flexible than:

```python
type(obj) is SomeClass
```

because `isinstance()` understands inheritance.

We will study inheritance soon.

For now, remember:

```text
isinstance(obj, Class) asks whether obj belongs to that class family
```

---

# Multiple Instances

A class can create many instances.

Example:

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author


book_one = Book("Python Basics", "Ada")
book_two = Book("Advanced Python", "Grace")
```

Each instance has its own state:

```text
book_one:
    title -> "Python Basics"
    author -> "Ada"

book_two:
    title -> "Advanced Python"
    author -> "Grace"
```

Use:

```python
print(book_one.title)
print(book_two.title)
```

Output:

```text
Python Basics
Advanced Python
```

This is one of the main reasons classes exist.

They let us create many objects with the same structure and behavior but different state.

---

# Behavior Shared Through the Class

Methods are shared through the class.

Example:

```python
class Book:
    def __init__(self, title):
        self.title = title

    def describe(self):
        return f"Book: {self.title}"
```

Create:

```python
a = Book("Python Basics")
b = Book("Advanced Python")
```

Each instance has its own `title`.

But the method function `describe` lives on the class.

Conceptually:

```text
Book class:
    describe -> function object

a instance:
    title -> "Python Basics"

b instance:
    title -> "Advanced Python"
```

When called:

```python
a.describe()
b.describe()
```

the same method behavior runs with different `self` objects.

This is efficient and expressive.

State varies by instance.

Behavior is usually defined once on the class.

---

# Instance State

Instance state is the data stored on a particular instance.

Example:

```python
class Counter:
    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += 1
```

Use:

```python
a = Counter()
b = Counter()

a.increment()
a.increment()
b.increment()

print(a.value)
print(b.value)
```

Output:

```text
2
1
```

Both objects are counters.

Each has its own `value`.

State is not shared unless you explicitly store it somewhere shared.

This is the heart of instance-based object-oriented programming:

```text
same class
same behavior
different instance state
```

---

# Valid Starting State

`__init__` should usually create a valid starting state.

Bad:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Maybe this is incomplete if every user must have an email too.

Better:

```python
class User:
    def __init__(self, name, email):
        if name == "":
            raise ValueError("name cannot be empty")

        if email == "":
            raise ValueError("email cannot be empty")

        self.name = name
        self.email = email
```

Now a `User` object cannot be created without required information.

This is one advantage of classes over loose dictionaries.

Construction can enforce rules.

An object should not have to wander through the program half-formed.

Initialize it clearly.

---

# Methods Can Use Instance State

Example:

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def describe(self):
        return f"{self.owner}: {self.balance}"
```

Use:

```python
account = BankAccount("Ada", 100)
account.deposit(50)
print(account.describe())
```

Output:

```text
Ada: 150
```

Methods can read and modify instance state through `self`.

The method does not need the account passed explicitly:

```python
deposit(account, 50)
```

Instead:

```python
account.deposit(50)
```

The object is the receiver of the operation.

That is object-oriented style.

---

# Avoid Objects That Are Only Data Bags

Sometimes a class only holds data:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

This can be fine.

But if a class only stores data and has no behavior, ask whether another structure is better:

* A tuple.
* A dictionary.
* A dataclass.
* A named tuple.
* A simple object class.

Later, Chapter 51 will study dataclasses.

For now, writing simple classes by hand is useful because it teaches the object model.

But professional Python often uses dataclasses for structured data.

Classes are not only for data.

They shine when they express behavior, invariants, and relationships.

---

# Class Naming

Class names usually use CapWords.

Examples:

```python
class User:
    pass


class BankAccount:
    pass


class JsonParser:
    pass
```

Function and variable names usually use lowercase with underscores:

```python
def create_user():
    ...


user_name = "Ada"
```

The naming difference helps readers.

When you see:

```python
User()
```

you expect a class being called to create an instance.

When you see:

```python
create_user()
```

you expect a function.

Naming conventions make code easier to scan.

---

# Empty Classes

An empty class can be written with `pass`:

```python
class Marker:
    pass
```

This creates a class object.

You can create instances:

```python
marker = Marker()
```

You can attach attributes:

```python
marker.name = "important"
```

This works because ordinary instances can have dynamic attributes.

But empty classes are not common in well-designed application code unless they serve a clear purpose.

They may be used for:

* Simple markers.
* Experiments.
* Tests.
* Dynamic attribute containers.
* Teaching.

In real design, a class usually defines meaningful initialization or behavior.

---

# Dynamic Attributes

Ordinary Python objects can often receive new attributes after creation.

Example:

```python
class User:
    pass


user = User()
user.name = "Ada"
user.email = "ada@example.com"
```

This flexibility is powerful.

It can also become messy.

If every part of the program adds attributes whenever it wants, objects become hard to reason about.

Better:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
```

Now the expected attributes are visible at construction.

Dynamic attributes are part of Python's flexibility.

Clear initialization is part of Python's maintainability.

Use both with judgment.

---

# Classes and Modules

Classes usually live in modules.

Example:

`users.py`:

```python
class User:
    def __init__(self, name):
        self.name = name
```

`main.py`:

```python
from users import User

ada = User("Ada")
```

Namespace model:

```text
users module namespace:
    User -> class object

main module namespace:
    User -> same class object
    ada  -> User instance
```

The class object can be imported like any other object.

This connects classes back to modules and namespaces.

Classes do not live in a separate universe.

They are objects bound to names in namespaces.

---

# Classes and Packages

In larger projects, classes are often organized inside package modules.

Example:

```text
app/
    __init__.py
    users/
        __init__.py
        models.py
```

`app/users/models.py`:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Import:

```python
from app.users.models import User
```

Use:

```python
ada = User("Ada")
```

Full namespace path:

```text
app.users.models.User
```

This tells us:

```text
application package
users package
models module
User class
```

Good package organization makes class locations predictable.

---

# Classes and Memory

Classes and instances are objects in memory.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


ada = User("Ada")
```

Conceptual graph:

```text
module namespace:
    User ──▶ class object
    ada  ──▶ instance object

instance object:
    name ──▶ "Ada"

class object:
    __init__ ──▶ function object
```

The instance knows its class.

The class stores methods.

The module stores the class name.

The instance stores per-object state.

All the Volume I ideas still apply:

* Names.
* References.
* Objects.
* Namespaces.
* Memory.
* Attribute lookup.

Classes combine those ideas.

---

# A Complete Example: Task

Suppose we want to model a task.

Class:

```python
class Task:
    def __init__(self, title):
        if title.strip() == "":
            raise ValueError("title cannot be empty")

        self.title = title.strip()
        self.done = False

    def complete(self):
        self.done = True

    def display(self):
        marker = "x" if self.done else " "
        return f"[{marker}] {self.title}"
```

Use:

```python
task = Task("learn classes")

print(task.display())

task.complete()

print(task.display())
```

Output:

```text
[ ] learn classes
[x] learn classes
```

What the class defines:

```text
construction rule:
    title cannot be empty

state:
    title
    done

behavior:
    complete
    display
```

This is a good beginner example of object-oriented design.

The object owns state and exposes behavior that changes or reads that state.

---

# A Complete Example: Shopping Cart

Class:

```python
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, name, price):
        self.items.append({"name": name, "price": price})

    def total(self):
        amount = 0

        for item in self.items:
            amount += item["price"]

        return amount
```

Use:

```python
cart = ShoppingCart()
cart.add_item("Book", 30)
cart.add_item("Pen", 5)

print(cart.total())
```

Output:

```text
35
```

Why `self.items = []` belongs in `__init__`:

```text
each cart should have its own list
```

If `items` were a mutable class attribute, carts would accidentally share one list.

This example reinforces:

```text
per-instance mutable state belongs on the instance
```

---

# A Complete Example: Temperature

Class:

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    def fahrenheit(self):
        return self.celsius * 9 / 5 + 32

    def is_freezing(self):
        return self.celsius <= 0
```

Use:

```python
today = Temperature(12)

print(today.fahrenheit())
print(today.is_freezing())
```

Output:

```text
53.6
False
```

This class groups:

```text
state:
    celsius

behavior:
    fahrenheit conversion
    freezing check
```

Could this be simple functions?

Yes.

Example:

```python
def fahrenheit(celsius):
    return celsius * 9 / 5 + 32
```

For a tiny conversion, a function may be enough.

Classes become more useful when the object has multiple operations, invariants, or relationships.

Do not force classes where functions are clearer.

---

# Object-Oriented Design Starts Small

Object-oriented programming can sound grand.

At the beginning, think simply:

```text
What kind of thing am I modeling?
What data does one thing need?
What behavior belongs to that thing?
How should one thing be initialized?
What should be impossible for this thing?
```

Example:

```text
Task
    data:
        title
        done
    behavior:
        complete
        display
    rules:
        title cannot be empty
```

That is enough to start a class.

Do not begin with inheritance.

Do not begin with patterns.

Do not begin with metaclasses.

Begin with:

```text
state + behavior + rules
```

The advanced machinery exists to support good models, not replace them.

---

# Common Mistake: Forgetting `self`

Bad:

```python
class User:
    def greet():
        return "hello"
```

Use:

```python
user = User()
user.greet()
```

This raises an error because Python passes the instance automatically, but the method does not accept it.

Correct:

```python
class User:
    def greet(self):
        return "hello"
```

Remember:

```text
instance.method()
```

conceptually passes the instance as the first argument.

That first parameter is conventionally called `self`.

---

# Common Mistake: Returning From `__init__`

Bad:

```python
class User:
    def __init__(self, name):
        self.name = name
        return self
```

`__init__` should return `None`.

Do not return the instance from `__init__`.

Python handles returning the new object from the class call.

Correct:

```python
class User:
    def __init__(self, name):
        self.name = name
```

If `__init__` explicitly returns a non-`None` value, Python raises an error.

Mental model:

```text
__init__ initializes
class call returns the instance
```

---

# Common Mistake: Shared Mutable Class State

Bad:

```python
class Notebook:
    notes = []

    def add(self, note):
        self.notes.append(note)
```

Use:

```python
a = Notebook()
b = Notebook()

a.add("one")
print(b.notes)
```

Output:

```text
['one']
```

Both notebooks share the class-level list.

Correct:

```python
class Notebook:
    def __init__(self):
        self.notes = []

    def add(self, note):
        self.notes.append(note)
```

Now each notebook has its own notes list.

Rule:

```text
mutable per-object data belongs in __init__
shared class-level data belongs on the class
```

---

# Common Mistake: Making Everything a Class

Not every problem needs a class.

Good function:

```python
def slugify(text):
    return text.lower().replace(" ", "-")
```

Unnecessary class:

```python
class Slugifier:
    def slugify(self, text):
        return text.lower().replace(" ", "-")
```

If there is no meaningful state, no clear object identity, and no group of related behavior, a function may be clearer.

Classes are powerful.

Functions are powerful too.

Professional Python is not "classes everywhere."

It is choosing the simplest structure that expresses the idea well.

---

# Common Mistake: Hiding Too Much in `__init__`

`__init__` should initialize the object.

It should not usually perform surprising heavy work.

Risky:

```python
class Report:
    def __init__(self):
        self.data = download_large_dataset()
        send_tracking_email()
        connect_to_database()
```

Now creating a `Report` object does network work, sends email, and opens a database connection.

Sometimes constructors need resources.

But be deliberate.

Better:

```python
class Report:
    def __init__(self, data):
        self.data = data
```

Then:

```python
data = download_large_dataset()
report = Report(data)
```

Clear construction is easier to test and reason about.

---

# Common Mistake: Confusing Class and Instance

Example:

```python
class User:
    pass
```

`User` is the class.

```python
ada = User()
```

`ada` is an instance.

This fails:

```python
User.name
```

unless the class has a class attribute named `name`.

This works after initialization:

```python
ada.name
```

if the instance has an instance attribute named `name`.

Class:

```text
defines shared behavior and class-level data
```

Instance:

```text
individual object with per-object state
```

Keep the distinction sharp.

It will matter even more when we study inheritance and descriptors.

---

# Common Mistake: Assuming Type Hints Create Attributes

You may see:

```python
class User:
    name: str
    email: str
```

This declares annotations.

It does not automatically create instance attributes.

This will fail:

```python
user = User()
print(user.name)
```

unless something assigned `self.name`.

Correct initialization:

```python
class User:
    name: str
    email: str

    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email
```

Annotations describe expected types.

Assignments create bindings.

This distinction will matter when we study static typing and dataclasses.

---

# How This Prepares Us for Advanced Python

Classes are the root of many advanced topics.

Attributes and methods:

```text
how names are found on objects and classes
```

Encapsulation and properties:

```text
how access to attributes can be managed
```

Inheritance:

```text
how classes reuse and specialize behavior
```

MRO:

```text
how Python decides where inherited methods come from
```

Duck typing:

```text
how behavior can matter more than exact class
```

Descriptors:

```text
how attribute access can be customized deeply
```

Dunder methods:

```text
how objects participate in Python syntax and protocols
```

Metaclasses:

```text
how classes themselves are created and controlled
```

Everything starts with:

```text
class object
instance object
attribute lookup
self
```

This chapter is the foundation.

---

# Exercises

1. Create a class:

```python
class User:
    pass
```

Create two instances:

```python
ada = User()
grace = User()
```

Check:

```python
print(ada is grace)
print(type(ada))
print(isinstance(ada, User))
```

Explain each result.

---

2. Write a `Book` class with `title` and `author` instance attributes.

Create two books.

Print their titles.

Explain why each book has its own title.

---

3. Write a `Counter` class:

```python
counter = Counter()
counter.increment()
counter.increment()
print(counter.value)
```

The output should be:

```text
2
```

Use `__init__` to initialize `value`.

---

4. Explain the difference between:

```python
class User:
    role = "member"
```

and:

```python
class User:
    def __init__(self):
        self.role = "member"
```

Where is `role` stored in each case?

---

5. Fix this class:

```python
class Team:
    members = []

    def add(self, member):
        self.members.append(member)
```

Why is the original version dangerous?

---

6. Explain what `self` refers to here:

```python
class Task:
    def __init__(self, title):
        self.title = title

    def complete(self):
        self.done = True
```

What object is `self` when you call:

```python
task.complete()
```

---

7. Why should `__init__` not return `self`?

What is the role of `__init__`?

---

8. Create a `Rectangle` class with:

* `width`
* `height`
* `area()`
* `perimeter()`

Then create two rectangles and show that each has different state but shared behavior.

---

9. Explain why this annotation does not create an attribute:

```python
class User:
    name: str
```

What line actually creates the instance attribute?

---

10. In your own words, explain:

```text
classes are objects
instances are objects
classes create instances
```

Use a diagram with arrows from names to objects.

---

# Summary

In this chapter we learned:

* A class defines a kind of object.
* A class statement creates a class object.
* A class name is a normal name bound to a class object.
* Calling a class creates a new instance.
* Instances are individual objects created from classes.
* `__init__` initializes a new instance.
* `self` conventionally refers to the current instance.
* Instance attributes belong to individual instances.
* Class attributes belong to the class object.
* Mutable per-instance state should usually be created in `__init__`.
* Methods are functions stored on classes and bound to instances when accessed through instances.
* `type()` shows an object's type.
* `isinstance()` checks whether an object is an instance of a class family.
* Classes connect state, behavior, and rules.
* Classes are part of the same object/reference/namespace model from Volume I.

Core model:

```text
module namespace:
    User -> class object
    ada  -> instance object

class object:
    methods
    class attributes

instance object:
    per-object attributes
```

Object-oriented model:

```text
class:
    defines structure and behavior

instance:
    carries individual state

self:
    the instance currently receiving a method call
```

This is the base layer for advanced Python.

Everything that follows in Part I builds on this.

---

# Preview of Chapter 44

Next we study attributes and methods in depth.

This chapter introduced instance attributes, class attributes, and methods.

Chapter 44 explains how Python finds them.

We will study:

* Attribute lookup order.
* Instance namespaces.
* Class namespaces.
* Method binding.
* Bound methods.
* How `self` is passed.
* Why class attributes can be read through instances.
* Why instance attributes can shadow class attributes.
* How attribute assignment differs from attribute lookup.
* How this prepares us for properties and descriptors.

The transition is direct:

```text
classes create objects
attributes and methods define how those objects expose names and behavior
```

Once attribute lookup is clear, inheritance, descriptors, properties, and MRO become much easier to understand.
