# Chapter 47 — Inheritance and Method Overriding

---

# Learning Objectives

By the end of this chapter, you should understand:

* What inheritance means in Python.
* What parent classes and child classes are.
* How inherited attributes and methods are found.
* Why inheritance models is-a relationships.
* How inheritance differs from composition.
* What method overriding means.
* How child classes specialize parent behavior.
* How subclasses can reuse inherited behavior.
* Why `super()` is used when extending parent behavior.
* What inherited initialization means.
* How to avoid common inheritance mistakes.
* Why inheritance should be used carefully.
* How this chapter prepares us for MRO and `super()` in depth.

Chapter 46 taught composition.

Composition says:

```text
one object has or uses another object
```

Inheritance says:

```text
one class is a specialized kind of another class
```

That distinction matters.

Inheritance is powerful.

It lets classes reuse and specialize behavior.

But it also creates tight relationships between classes.

Used well, inheritance makes code expressive.

Used poorly, it creates brittle hierarchies that are hard to change.

This chapter teaches inheritance carefully, with composition still fresh in mind.

---

# Concept Overview

Inheritance lets one class receive behavior from another class.

Example:

```python
class Animal:
    def speak(self):
        return "some sound"


class Dog(Animal):
    pass
```

Use:

```python
dog = Dog()
print(dog.speak())
```

Output:

```text
some sound
```

`Dog` does not define `speak`.

Python finds `speak` on `Animal`.

Relationship:

```text
Dog inherits from Animal
Dog is a child class
Animal is a parent class
```

Another set of terms:

```text
Dog is a subclass of Animal
Animal is a superclass of Dog
```

These terms mean the same broad relationship.

---

# Parent and Child Classes

Parent class:

```python
class Animal:
    def eat(self):
        return "eating"
```

Child class:

```python
class Dog(Animal):
    pass
```

The parentheses:

```python
class Dog(Animal):
```

mean:

```text
Dog inherits from Animal
```

Create:

```python
dog = Dog()
```

Call inherited method:

```python
print(dog.eat())
```

Output:

```text
eating
```

Python's attribute lookup includes parent classes.

Simplified lookup:

```text
instance
child class
parent class
parent's parent classes
```

This is how inherited methods are found.

---

# Inheritance Is Attribute Lookup

Inheritance works through attribute lookup.

Example:

```python
class Animal:
    def speak(self):
        return "sound"


class Dog(Animal):
    pass


dog = Dog()
```

When you call:

```python
dog.speak()
```

Python searches:

```text
dog instance namespace
Dog class namespace
Animal class namespace
```

It finds `speak` in `Animal`.

Then it binds the method to `dog`.

This is not a separate magic system.

It extends the attribute lookup model from Chapter 44.

Inheritance is mostly about where Python searches when looking for attributes.

That idea will become essential in Chapter 48, where we study MRO.

---

# Is-A Relationships

Inheritance should usually model an is-a relationship.

Examples:

```text
Dog is an Animal.
Car is a Vehicle.
SavingsAccount is an Account.
CsvReport is a Report.
AdminUser is a User.
```

If you can naturally say:

```text
child is a kind of parent
```

inheritance may fit.

If the relationship is:

```text
object has another object
object uses another object
object needs another object
```

composition may fit better.

Bad inheritance:

```text
Car is an Engine.
UserService is a Database.
Report is a Formatter.
```

These are probably has-a or uses-a relationships.

Use composition for those.

Inheritance should express meaning, not merely code reuse.

---

# A Simple Inheritance Example

Parent:

```python
class Vehicle:
    def __init__(self, name):
        self.name = name

    def describe(self):
        return f"Vehicle: {self.name}"
```

Child:

```python
class Car(Vehicle):
    pass
```

Use:

```python
car = Car("Sedan")
print(car.describe())
```

Output:

```text
Vehicle: Sedan
```

`Car` inherits:

* `__init__`
* `describe`

from `Vehicle`.

Because `Car` does not define its own `__init__`, Python uses the inherited one.

Because `Car` does not define its own `describe`, Python uses the inherited one.

This is basic inheritance:

```text
child receives behavior from parent
```

---

# Method Overriding

Method overriding means a child class defines a method with the same name as a parent method.

Example:

```python
class Animal:
    def speak(self):
        return "some sound"


class Dog(Animal):
    def speak(self):
        return "woof"
```

Use:

```python
animal = Animal()
dog = Dog()

print(animal.speak())
print(dog.speak())
```

Output:

```text
some sound
woof
```

`Dog.speak` overrides `Animal.speak`.

Lookup for `dog.speak`:

```text
dog instance -> no speak
Dog class -> speak found
Animal class -> not needed
```

The child method is found first.

That is overriding.

---

# Overriding Specializes Behavior

Overriding lets a subclass specialize parent behavior.

Example:

```python
class Report:
    def render(self):
        return "generic report"


class HtmlReport(Report):
    def render(self):
        return "<p>html report</p>"


class TextReport(Report):
    def render(self):
        return "text report"
```

Use:

```python
reports = [Report(), HtmlReport(), TextReport()]

for report in reports:
    print(report.render())
```

Output:

```text
generic report
<p>html report</p>
text report
```

Each class has a `render` method.

The same call:

```python
report.render()
```

can produce different behavior depending on the object's class.

This is a foundation for polymorphism.

We will study that more deeply in Chapter 49.

---

# Inheriting Initialization

If a child class does not define `__init__`, it inherits the parent's `__init__`.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


class AdminUser(User):
    pass
```

Use:

```python
admin = AdminUser("Ada")
print(admin.name)
```

Output:

```text
Ada
```

`AdminUser` inherited `User.__init__`.

Lookup for `__init__` during construction finds it on `User`.

This can be useful when the child class needs the same initialization as the parent.

But if the child needs extra state, it may define its own `__init__`.

Then we need to decide whether and how to call the parent initializer.

That is where `super()` enters.

---

# Overriding `__init__`

A child class can define its own `__init__`.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


class AdminUser(User):
    def __init__(self, name, permissions):
        self.name = name
        self.permissions = permissions
```

Use:

```python
admin = AdminUser("Ada", ["manage_users"])
```

This works.

But it duplicates:

```python
self.name = name
```

from `User.__init__`.

Duplication can be small at first.

Later, if `User.__init__` changes, `AdminUser.__init__` may become inconsistent.

Better:

```python
class AdminUser(User):
    def __init__(self, name, permissions):
        super().__init__(name)
        self.permissions = permissions
```

This calls the parent initializer.

Then the child adds its own state.

---

# `super()` Preview

`super()` gives access to the next class in the method resolution order.

For simple single inheritance, beginners can read:

```python
super().__init__(name)
```

as:

```text
call the parent class's __init__
```

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


class AdminUser(User):
    def __init__(self, name, permissions):
        super().__init__(name)
        self.permissions = permissions
```

This avoids duplicating parent initialization.

But `super()` is deeper than "call parent."

In multiple inheritance, it follows MRO.

Chapter 48 will study `super()` properly.

For now, the practical model is enough:

```text
use super() when extending inherited behavior
```

---

# Extending a Parent Method

Sometimes a child wants to add behavior before or after parent behavior.

Example:

```python
class Report:
    def render(self):
        return "body"


class TitledReport(Report):
    def render(self):
        body = super().render()
        return f"Title\n{body}"
```

Use:

```python
report = TitledReport()
print(report.render())
```

Output:

```text
Title
body
```

The child did not replace parent behavior entirely.

It extended it.

Pattern:

```python
def method(self):
    result = super().method()
    return modified_result
```

This is common.

It keeps parent behavior reusable while allowing child specialization.

---

# Replacing vs Extending

Overriding can replace behavior completely.

Example:

```python
class Animal:
    def speak(self):
        return "sound"


class Dog(Animal):
    def speak(self):
        return "woof"
```

`Dog.speak` replaces `Animal.speak`.

Overriding can also extend behavior.

Example:

```python
class Logger:
    def log(self, message):
        print(message)


class TimestampLogger(Logger):
    def log(self, message):
        super().log(f"[time] {message}")
```

`TimestampLogger.log` uses parent behavior but changes the message.

Ask:

```text
Do I want to replace the parent behavior?
Or build on it?
```

If building on it, use `super()` rather than copying the parent method's code.

---

# Inherited Class Attributes

Class attributes are inherited too.

Example:

```python
class User:
    role = "user"


class AdminUser(User):
    pass
```

Use:

```python
print(AdminUser.role)
print(AdminUser().role)
```

Output:

```text
user
user
```

Python finds `role` on the parent class.

A child can override class attributes:

```python
class AdminUser(User):
    role = "admin"
```

Now:

```python
print(AdminUser.role)
print(User.role)
```

Output:

```text
admin
user
```

The child class has its own class attribute named `role`.

It shadows the parent class attribute.

---

# Inherited Instance Behavior

Instance attributes are usually created by methods.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Child:

```python
class AdminUser(User):
    pass
```

Use:

```python
admin = AdminUser("Ada")
```

The instance attribute `name` is not inherited as a stored value.

It is created when inherited `User.__init__` runs on the `AdminUser` instance.

This distinction matters.

Class methods and attributes are inherited through lookup.

Instance attributes are per-object state created at runtime.

Better wording:

```text
AdminUser inherits the method that creates name.
Each AdminUser instance gets its own name when initialized.
```

Not:

```text
AdminUser inherits the name value.
```

---

# Subclasses Are Instances of Parent Classes

`isinstance()` understands inheritance.

Example:

```python
class Animal:
    pass


class Dog(Animal):
    pass


dog = Dog()
```

Check:

```python
print(isinstance(dog, Dog))
print(isinstance(dog, Animal))
```

Output:

```text
True
True
```

The dog is an instance of `Dog`.

It is also considered an instance of `Animal`.

Why?

Because `Dog` is a subclass of `Animal`.

This is one reason inheritance should model is-a relationships.

If:

```python
isinstance(obj, Parent)
```

is true, it should make conceptual sense.

---

# `issubclass()`

`issubclass()` checks class relationships.

Example:

```python
class Animal:
    pass


class Dog(Animal):
    pass
```

Check:

```python
print(issubclass(Dog, Animal))
print(issubclass(Animal, Dog))
```

Output:

```text
True
False
```

`Dog` is a subclass of `Animal`.

`Animal` is not a subclass of `Dog`.

Also:

```python
print(issubclass(Dog, Dog))
```

Output:

```text
True
```

A class is considered a subclass of itself for this check.

Use `issubclass()` when you are reasoning about class relationships.

Use `isinstance()` when you are reasoning about objects.

---

# Base Class `object`

In Python 3, all classes inherit from `object` eventually.

These are equivalent:

```python
class User:
    pass
```

and:

```python
class User(object):
    pass
```

You usually write the shorter form:

```python
class User:
    pass
```

Check:

```python
print(issubclass(User, object))
```

Output:

```text
True
```

`object` provides basic behavior common to Python objects.

This includes methods like:

```text
__str__
__repr__
__eq__
__getattribute__
```

We will study many dunder methods later.

For now, remember:

```text
every normal class participates in a larger inheritance chain ending at object
```

---

# Overriding `__str__`

Inheritance is not only about your own parent classes.

You can override behavior inherited from `object`.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name
```

Use:

```python
user = User("Ada")
print(str(user))
print(user)
```

Output:

```text
Ada
Ada
```

`User` overrides `object.__str__`.

This lets your object participate in Python's string conversion protocol.

Dunder methods are a major part of Python's data model.

We will study them in Part II.

This example shows that overriding can customize built-in behavior too.

---

# Specialization Example: Employees

Parent:

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def annual_pay(self):
        return self.salary
```

Child:

```python
class Manager(Employee):
    def __init__(self, name, salary, bonus):
        super().__init__(name, salary)
        self.bonus = bonus

    def annual_pay(self):
        return self.salary + self.bonus
```

Use:

```python
employee = Employee("Ada", 100_000)
manager = Manager("Grace", 120_000, 20_000)

print(employee.annual_pay())
print(manager.annual_pay())
```

Output:

```text
100000
140000
```

This is a reasonable inheritance example if:

```text
Manager is a kind of Employee
```

and manager-specific behavior specializes employee behavior.

---

# Specialization Example: Shapes

Parent:

```python
class Shape:
    def area(self):
        raise NotImplementedError("subclasses must implement area")
```

Children:

```python
class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius * self.radius
```

Use:

```python
shapes = [Rectangle(10, 5), Circle(3)]

for shape in shapes:
    print(shape.area())
```

Each shape has an `area`.

Each computes it differently.

This is a classic inheritance example.

Later, we will see how abstract base classes can express this more formally.

---

# `NotImplementedError`

Sometimes a parent class defines a method that subclasses are expected to override.

Example:

```python
class Shape:
    def area(self):
        raise NotImplementedError("subclasses must implement area")
```

This means:

```text
Shape defines the expected interface
but does not provide implementation
```

If a subclass forgets:

```python
class Triangle(Shape):
    pass
```

then:

```python
Triangle().area()
```

raises:

```text
NotImplementedError
```

This is a runtime signal.

It is not the same as a formal abstract method.

ABCs will give us stronger tools later.

For now, `NotImplementedError` is a simple way to say:

```text
subclass responsibility
```

---

# Inheritance and Polymorphism Preview

Polymorphism means different objects can be used through the same interface.

Example:

```python
class Dog:
    def speak(self):
        return "woof"


class Cat:
    def speak(self):
        return "meow"
```

No inheritance needed.

But inheritance can also support polymorphism:

```python
class Animal:
    def speak(self):
        raise NotImplementedError


class Dog(Animal):
    def speak(self):
        return "woof"


class Cat(Animal):
    def speak(self):
        return "meow"
```

Use:

```python
animals = [Dog(), Cat()]

for animal in animals:
    print(animal.speak())
```

Same call.

Different behavior.

We will study polymorphism and duck typing in Chapter 49.

---

# Inheritance and Composition Together

Inheritance and composition can work together.

Example:

```python
class Formatter:
    def format(self, data):
        raise NotImplementedError


class TextFormatter(Formatter):
    def format(self, data):
        return str(data)


class Report:
    def __init__(self, formatter):
        self._formatter = formatter

    def render(self, data):
        return self._formatter.format(data)
```

Here:

```text
TextFormatter is a Formatter
Report has a Formatter
```

The formatter hierarchy uses inheritance.

The report uses composition.

Good design often combines both:

* Inheritance for families of related types.
* Composition for assembling objects.

The goal is not to avoid inheritance forever.

The goal is to use it where it expresses the relationship clearly.

---

# Inheritance Depth

Inheritance chains can become deep.

Example:

```text
BaseReport
    ExportableReport
        FormattedReport
            HtmlReport
                InteractiveHtmlReport
```

Deep chains can be hard to understand.

Questions become difficult:

```text
Where is this method defined?
Which class overrides it?
What state does each level require?
Which __init__ methods must run?
```

Inheritance depth is not automatically bad.

Frameworks sometimes use deep hierarchies.

But in application code, prefer shallow hierarchies unless deeper structure is truly justified.

Composition can often replace deep inheritance with clearer collaborator objects.

Rule of thumb:

```text
if understanding a class requires reading five ancestors, the design may be too tangled
```

---

# Inheritance Couples Classes

Inheritance creates coupling between parent and child.

The child depends on:

* Parent method names.
* Parent initialization.
* Parent invariants.
* Parent attribute choices.
* Parent extension points.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Child:

```python
class AdminUser(User):
    def label(self):
        return self.name.upper()
```

If `User` changes internal storage:

```python
self._name = name
```

and no public `name` property exists, `AdminUser` may break.

Inheritance makes internals more tempting to rely on.

This is why encapsulation matters even inside class hierarchies.

Design parent classes with subclasses in mind.

---

# The Fragile Base Class Problem

A base class is fragile when small changes in it unexpectedly break subclasses.

Example:

```python
class Base:
    def process(self):
        self.prepare()
        self.run()

    def prepare(self):
        pass

    def run(self):
        pass
```

Subclasses may override `prepare` or `run`.

If `Base.process` changes the order:

```python
def process(self):
    self.run()
    self.prepare()
```

subclasses may break.

This does not mean inheritance is bad.

It means base classes define contracts.

If subclasses rely on a method being called at a certain time, that behavior is part of the contract.

Inheritance requires careful API design.

Parent classes are not just code-sharing containers.

They are extension points.

---

# Template Method Pattern Preview

Sometimes a parent class defines an algorithm and lets subclasses customize steps.

Example:

```python
class Importer:
    def run(self):
        data = self.load()
        cleaned = self.clean(data)
        self.save(cleaned)

    def load(self):
        raise NotImplementedError

    def clean(self, data):
        return data

    def save(self, data):
        raise NotImplementedError
```

Subclass:

```python
class CsvImporter(Importer):
    def load(self):
        return [" raw "]

    def clean(self, data):
        return [item.strip() for item in data]

    def save(self, data):
        print(data)
```

Use:

```python
CsvImporter().run()
```

This is called the template method pattern.

The base class controls the workflow.

Subclasses override specific steps.

This can be elegant when the workflow is stable.

It can be brittle if the workflow changes often.

---

# Overriding With Compatible Meaning

When overriding a method, keep the meaning compatible.

Parent:

```python
class Repository:
    def get(self, key):
        return None
```

Child should not do something surprising:

```python
class EmailRepository(Repository):
    def get(self, key):
        send_email("someone@example.com")
        return "sent"
```

The method name `get` suggests retrieval.

Sending email is surprising.

Overridden methods should respect the parent method's purpose.

This is part of substitutability:

```text
code expecting the parent should not be shocked by the child
```

We will revisit this when studying polymorphism, SOLID, and design principles.

For now:

```text
override to specialize, not to betray the interface
```

---

# Constructor Compatibility

Subclasses often need compatible construction.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Subclass:

```python
class AdminUser(User):
    def __init__(self, name, permissions):
        super().__init__(name)
        self.permissions = permissions
```

This is fine when callers know they are creating an `AdminUser`.

But if code wants to create objects generically, constructor differences can matter.

Example:

```python
def create_user(user_class, name):
    return user_class(name)
```

This works for `User`.

It fails for `AdminUser` because permissions are required.

Constructor compatibility is a design choice.

Sometimes subclasses require more information.

Sometimes they should provide defaults.

Be aware that construction is part of a class's interface.

---

# Class Hierarchies and APIs

When you create a parent class, you create an API for subclasses.

Example:

```python
class BaseView:
    def render(self):
        context = self.get_context()
        return self.render_template(context)

    def get_context(self):
        return {}

    def render_template(self, context):
        raise NotImplementedError
```

Subclass authors need to know:

* Which methods to override.
* Which methods not to override.
* Which attributes are available.
* Which methods call which hooks.
* What each method must return.

This should be documented or obvious.

Inheritance without clear extension points becomes guesswork.

If you do not want subclasses to depend on internals, do not make internals the only way to extend behavior.

---

# Common Mistake: Inheriting Only for Code Reuse

Bad:

```python
class JsonTools:
    def to_json(self, data):
        return str(data)


class UserService(JsonTools):
    pass
```

This says:

```text
UserService is a JsonTools
```

That is probably not meaningful.

Better:

```python
class UserService:
    def __init__(self, json_tools):
        self._json_tools = json_tools
```

or simply use a function:

```python
to_json(data)
```

Inheritance should model a type relationship.

If you only want a helper method, composition or a module function is often clearer.

---

# Common Mistake: Forgetting `super().__init__`

Parent:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Child:

```python
class AdminUser(User):
    def __init__(self, name, permissions):
        self.permissions = permissions
```

Use:

```python
admin = AdminUser("Ada", ["manage"])
print(admin.name)
```

This raises:

```text
AttributeError
```

because `User.__init__` never ran.

Correct:

```python
class AdminUser(User):
    def __init__(self, name, permissions):
        super().__init__(name)
        self.permissions = permissions
```

When overriding initialization, ask:

```text
does the parent initializer need to run?
```

Often, yes.

---

# Common Mistake: Calling Parent Class Directly

You may see:

```python
class AdminUser(User):
    def __init__(self, name, permissions):
        User.__init__(self, name)
        self.permissions = permissions
```

This can work in simple single inheritance.

But `super()` is generally preferred:

```python
class AdminUser(User):
    def __init__(self, name, permissions):
        super().__init__(name)
        self.permissions = permissions
```

Why?

Because `super()` cooperates with MRO, especially in multiple inheritance.

Direct parent calls hard-code one parent.

They can break cooperative inheritance patterns.

Chapter 48 will explain this deeply.

For now:

```text
prefer super() when extending inherited methods
```

---

# Common Mistake: Breaking the Parent Contract

Parent:

```python
class Storage:
    def save(self, item):
        """Save item and return its id."""
        return 1
```

Child:

```python
class BrokenStorage(Storage):
    def save(self, item):
        print("saved")
```

The child returns `None`.

If callers expect an id:

```python
item_id = storage.save(item)
```

the child breaks the expectation.

When overriding, preserve the parent method's contract unless you are intentionally creating a different interface.

Compatible overriding includes:

* Similar meaning.
* Compatible parameters.
* Compatible return behavior.
* Compatible exceptions.
* Compatible side effects.

This is design discipline.

---

# Common Mistake: Deep Hierarchy Too Early

Bad early design:

```text
BaseThing
    NamedThing
        ActiveNamedThing
            PersistentActiveNamedThing
                User
```

Maybe each layer seemed reusable.

But now understanding `User` requires reading many ancestors.

Prefer flatter designs until real patterns emerge.

Composition can often replace intermediate layers:

```text
User has Persistence
User has Status
User has Profile
```

Inheritance should clarify.

If it creates archaeology, reconsider.

---

# Common Mistake: Confusing Type Specialization With Role

Example:

```python
class User:
    pass


class AdminUser(User):
    pass
```

This may be fine if admin users truly have different behavior.

But if admin is just a role value:

```python
class User:
    def __init__(self, name, role):
        self.name = name
        self.role = role
```

may be simpler.

Ask:

```text
Does AdminUser need different methods or invariants?
Or is admin just data?
```

Do not create subclasses for every category if simple data works.

Inheritance is for behavior and type relationships, not every label.

---

# Common Mistake: Using Inheritance Where Strategy Fits

Suppose reports vary by output format.

Inheritance:

```python
class HtmlReport(Report):
    ...


class PdfReport(Report):
    ...


class CsvReport(Report):
    ...
```

Maybe this is fine.

But if only formatting varies, composition may be better:

```python
report = Report(formatter=HtmlFormatter())
report = Report(formatter=CsvFormatter())
```

This is often called the strategy pattern.

Instead of subclassing the whole report, inject a formatter object.

Question:

```text
Is the whole object a different kind?
Or is one behavior varying?
```

If one behavior varies, composition often wins.

---

# Design Guidance

Before using inheritance, ask:

```text
Is the child truly a kind of the parent?
Will isinstance(child, parent) make conceptual sense?
Does the child preserve the parent's method meanings?
Does the child need to override behavior?
Can composition solve this more clearly?
Is the hierarchy likely to stay shallow?
Are extension points clear?
Does parent initialization need to run?
Will subclasses depend on parent internals?
```

Use inheritance when:

* There is a real is-a relationship.
* Child classes specialize parent behavior.
* Shared behavior belongs naturally in a parent.
* Parent APIs are stable and clear.
* Subclasses can preserve parent contracts.

Prefer composition when:

* You only want to reuse helper behavior.
* The relationship is has-a or uses-a.
* You need swappable behavior.
* You want easier testing with fake collaborators.
* A hierarchy would become deep or awkward.

Inheritance is sharp.

Use it with care.

---

# Exercises

1. Create:

```python
class Animal:
    def speak(self):
        return "sound"


class Dog(Animal):
    pass
```

Create a `Dog` and call `speak`.

Explain where Python finds the method.

---

2. Override `speak` in `Dog` so it returns:

```text
woof
```

Explain how method overriding changes lookup.

---

3. Create a `User` class and an `AdminUser` subclass.

Let `AdminUser` add a `permissions` attribute.

Use `super().__init__`.

Explain why calling the parent initializer matters.

---

4. Create an `Employee` class with `annual_pay`.

Create a `Manager` subclass that adds a bonus and overrides `annual_pay`.

Should `Manager` call `super()`?

Why or why not?

---

5. Use `isinstance()` and `issubclass()` with your classes.

Explain the difference between checking an object and checking a class.

---

6. Create a `Shape` base class whose `area` method raises `NotImplementedError`.

Create `Rectangle` and `Circle` subclasses that override it.

Loop through shapes and call `area`.

---

7. Identify whether each relationship should use inheritance or composition:

```text
Car and Engine
Dog and Animal
Report and Formatter
Order and LineItem
AdminUser and User
Service and Database
```

Explain your choices.

---

8. Rewrite this bad inheritance design using composition:

```python
class EmailSender:
    def send_email(self, message):
        ...


class UserService(EmailSender):
    ...
```

Why is composition better?

---

9. Create a parent class with a method that returns a value.

Override it in a child class but accidentally return `None`.

Explain how this can break callers expecting the parent contract.

---

10. In your own words, explain:

```text
inheritance should model specialization, not convenience
```

Use examples from this chapter.

---

# Summary

In this chapter we learned:

* Inheritance lets one class receive behavior from another class.
* A child class inherits from a parent class.
* A subclass is a specialized kind of superclass.
* Attribute lookup searches child classes and then parent classes.
* Method overriding happens when a child defines a method with the same name as a parent method.
* Overriding can replace or extend parent behavior.
* `super()` is used to extend inherited behavior without hard-coding the parent class.
* If a child does not define `__init__`, it can inherit the parent's initializer.
* If a child overrides `__init__`, it often needs to call `super().__init__`.
* Class attributes can be inherited and overridden.
* Instance attributes are created at runtime, often by inherited initializers.
* `isinstance()` understands inheritance for objects.
* `issubclass()` checks class relationships.
* All normal Python classes ultimately inherit from `object`.
* Inheritance should model is-a relationships.
* Composition should model has-a and uses-a relationships.
* Deep or careless inheritance can create brittle designs.

Core model:

```text
class Child(Parent):
    ...

attribute lookup:
    instance
    Child
    Parent
    object
```

Design model:

```text
is-a relationship -> inheritance
has-a relationship -> composition
override -> child specializes parent behavior
super() -> child extends inherited behavior
```

Inheritance is not just a way to avoid duplicate code.

It is a way to express that one type is a specialized version of another type.

---

# Preview of Chapter 48

Next we study MRO and `super()`.

This chapter used a simple parent-child model.

Real Python inheritance can involve multiple classes, multiple levels, and cooperative method calls.

Chapter 48 explains how Python decides where to look next.

We will study:

* What MRO means.
* How to inspect a class's MRO.
* Why method resolution order matters.
* How `super()` actually works.
* Why `super()` is not simply "call my parent."
* How multiple inheritance changes lookup.
* How cooperative initialization works.
* Why mixins rely on MRO.
* Common MRO and `super()` mistakes.

The transition is direct:

```text
inheritance defines class relationships
MRO defines the lookup path through those relationships
```

Once MRO is clear, multiple inheritance, mixins, ABCs, and descriptors become much easier to reason about.
