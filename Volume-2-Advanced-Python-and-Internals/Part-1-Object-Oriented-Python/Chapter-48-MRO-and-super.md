# Chapter 48 — MRO and `super()`

---

# Learning Objectives

By the end of this chapter, you should understand:

* What MRO means.
* Why method resolution order exists.
* How Python chooses which method to call.
* How to inspect a class's MRO.
* Why inheritance lookup follows a deterministic order.
* Why `super()` is not simply "call my parent."
* How `super()` follows the MRO.
* How multiple inheritance changes lookup.
* What the diamond problem is.
* How Python avoids duplicate calls in cooperative inheritance.
* How cooperative `__init__` methods work.
* Why mixins rely on MRO.
* Common mistakes with `super()`.
* How this prepares us for duck typing, mixins, ABCs, and descriptors.

Chapter 47 introduced inheritance and method overriding.

We used a simple mental model:

```text
instance
child class
parent class
object
```

That model is good for single inheritance.

But Python allows more complex class relationships.

A class can inherit from more than one parent.

A class can sit inside a long inheritance chain.

Different parents can define methods with the same name.

When that happens, Python needs a clear rule:

```text
Which class should be searched first?
Which class comes next?
Which method should super() call?
```

That rule is the MRO.

---

# Concept Overview

MRO means method resolution order.

It is the order Python follows when looking for attributes and methods in a class hierarchy.

Example:

```python
class Animal:
    def speak(self):
        return "sound"


class Dog(Animal):
    pass
```

For `Dog`, the MRO is:

```text
Dog
Animal
object
```

When Python evaluates:

```python
dog.speak()
```

it searches:

```text
dog instance
Dog
Animal
object
```

It finds `speak` in `Animal`.

The class MRO is the class-level path:

```text
Dog -> Animal -> object
```

The instance is checked before class lookup begins.

At a practical level:

```text
MRO tells Python where to look next
```

---

# Inspecting MRO

Every class has an MRO.

You can inspect it with:

```python
ClassName.mro()
```

Example:

```python
class Animal:
    pass


class Dog(Animal):
    pass


print(Dog.mro())
```

Output looks like:

```text
[<class '__main__.Dog'>, <class '__main__.Animal'>, <class 'object'>]
```

You can also inspect:

```python
print(Dog.__mro__)
```

Output looks like:

```text
(<class '__main__.Dog'>, <class '__main__.Animal'>, <class 'object'>)
```

Difference:

```text
mro()   -> method returning a list
__mro__ -> attribute storing a tuple
```

Both show the lookup order.

When inheritance feels confusing, print the MRO.

It is one of the best debugging habits in object-oriented Python.

---

# Why MRO Matters

Suppose two classes define the same method name.

Example:

```python
class A:
    def describe(self):
        return "A"


class B(A):
    def describe(self):
        return "B"
```

Use:

```python
obj = B()
print(obj.describe())
```

Output:

```text
B
```

Why?

MRO:

```text
B
A
object
```

Python finds `describe` on `B` first.

The method in `A` still exists.

It is just later in the lookup order.

MRO determines:

* Which method is found.
* Which class attribute is found.
* What `super()` means.
* How multiple inheritance behaves.
* How mixins cooperate.

MRO is not an obscure internal detail.

It is the map for inherited attribute lookup.

---

# Single Inheritance MRO

Single inheritance is straightforward.

Example:

```python
class A:
    pass


class B(A):
    pass


class C(B):
    pass
```

MRO:

```python
print(C.mro())
```

Conceptual output:

```text
C
B
A
object
```

Lookup for:

```python
c.some_method()
```

searches:

```text
c instance
C
B
A
object
```

This matches the beginner model.

If Python only had single inheritance, MRO would be easy.

Multiple inheritance is where MRO becomes essential.

---

# Multiple Inheritance

Python allows a class to inherit from multiple parents.

Example:

```python
class Flyer:
    def move(self):
        return "flying"


class Swimmer:
    def move(self):
        return "swimming"


class Duck(Flyer, Swimmer):
    pass
```

Use:

```python
duck = Duck()
print(duck.move())
```

Output:

```text
flying
```

Why?

MRO:

```python
print(Duck.mro())
```

Conceptually:

```text
Duck
Flyer
Swimmer
object
```

Python finds `move` in `Flyer` before `Swimmer`.

The order in the class definition matters:

```python
class Duck(Flyer, Swimmer):
```

means `Flyer` is considered before `Swimmer`, while respecting Python's MRO rules.

---

# Parent Order Matters

Change parent order:

```python
class Duck(Swimmer, Flyer):
    pass
```

Now MRO is:

```text
Duck
Swimmer
Flyer
object
```

So:

```python
duck = Duck()
print(duck.move())
```

outputs:

```text
swimming
```

This does not mean parent order is arbitrary decoration.

It affects lookup.

When using multiple inheritance, parent order communicates priority.

However, MRO is not simply:

```text
class itself, then parents left to right, then grandparents left to right
```

Python uses a more careful algorithm to preserve consistency.

We will not implement the algorithm in this chapter.

But we will understand the behavior well enough to design with it.

---

# The Diamond Problem

The diamond problem happens when a class inherits from two classes that share a common ancestor.

Example:

```python
class A:
    def process(self):
        print("A")


class B(A):
    def process(self):
        print("B")


class C(A):
    def process(self):
        print("C")


class D(B, C):
    pass
```

Shape:

```text
    A
   / \
  B   C
   \ /
    D
```

What should:

```python
D().process()
```

call?

Python needs a consistent answer.

Inspect:

```python
print(D.mro())
```

Conceptually:

```text
D
B
C
A
object
```

So `D().process()` finds `B.process` first.

But the full MRO is still important for `super()`.

---

# MRO Avoids Duplicate Ancestors

In the diamond:

```text
    A
   / \
  B   C
   \ /
    D
```

Python's MRO includes `A` once:

```text
D
B
C
A
object
```

not:

```text
D
B
A
C
A
object
```

This matters.

If cooperative methods call `super()`, Python can move through the hierarchy without calling the shared ancestor twice.

That is one reason MRO exists.

It gives a single linear path through a potentially branching inheritance graph.

The technical term is linearization.

MRO linearizes the inheritance graph.

In plain words:

```text
MRO turns the class hierarchy into one ordered lookup list
```

---

# Python's MRO Is C3 Linearization

Python uses an algorithm called C3 linearization for new-style classes.

In Python 3, normal classes use this system.

You do not need to manually compute C3 in everyday code.

But you should know what it tries to preserve:

* Child classes come before parent classes.
* Parent order in the class definition is respected where possible.
* Shared ancestors appear only once.
* The order remains consistent across the hierarchy.

If Python cannot create a consistent MRO, class creation fails.

Example:

```python
class A:
    pass


class B(A):
    pass


class C(A):
    pass


class Bad(B, C):
    pass
```

This is fine.

But some conflicting parent order combinations can raise:

```text
TypeError: Cannot create a consistent method resolution order
```

That error means the inheritance graph has contradictory ordering requirements.

---

# `super()` Is About MRO

Many beginners learn:

```text
super() calls the parent method
```

That is a useful first approximation for simple single inheritance.

But it is not the full truth.

More accurate:

```text
super() gives access to the next class in the MRO after the current class
```

Example:

```python
class A:
    def show(self):
        print("A")


class B(A):
    def show(self):
        print("B")
        super().show()
```

Use:

```python
B().show()
```

Output:

```text
B
A
```

In single inheritance:

```text
next class in MRO after B is A
```

So it feels like parent.

In multiple inheritance, the next class may be a sibling branch, not the direct parent you have in mind.

---

# `super()` in Multiple Inheritance

Example:

```python
class A:
    def show(self):
        print("A")


class B(A):
    def show(self):
        print("B")
        super().show()


class C(A):
    def show(self):
        print("C")
        super().show()


class D(B, C):
    def show(self):
        print("D")
        super().show()
```

Use:

```python
D().show()
```

MRO:

```text
D
B
C
A
object
```

Output:

```text
D
B
C
A
```

Notice:

Inside `B.show`, `super().show()` did not call `A.show`.

It called `C.show`.

Why?

Because for a `D` instance, the next class after `B` in the MRO is `C`.

This is the moment `super()` becomes real.

It means:

```text
continue along the MRO
```

not:

```text
call the class written in my parentheses
```

---

# `super()` Depends on the Instance's Class

Consider:

```python
class A:
    def show(self):
        print("A")


class B(A):
    def show(self):
        print("B")
        super().show()


class C(A):
    def show(self):
        print("C")
        super().show()


class D(B, C):
    pass
```

Call:

```python
B().show()
```

MRO for `B`:

```text
B
A
object
```

Output:

```text
B
A
```

Call:

```python
D().show()
```

MRO for `D`:

```text
D
B
C
A
object
```

Output:

```text
B
C
A
```

The same method `B.show` can call different next methods depending on the actual instance's MRO.

This is why `super()` is dynamic and cooperative.

---

# Zero-Argument `super()`

In Python 3, inside a normal instance method, you usually write:

```python
super().method()
```

This is shorthand for:

```python
super(CurrentClass, self).method()
```

Example:

```python
class Child(Parent):
    def method(self):
        super().method()
```

Python knows:

* The current class.
* The current instance.

from the method context.

Use zero-argument `super()` in normal methods.

It is clearer and less error-prone than hard-coding:

```python
Parent.method(self)
```

or:

```python
super(Child, self).method()
```

Modern Python style generally prefers:

```python
super()
```

---

# `super()` Returns a Proxy

`super()` does not immediately call anything by itself.

Example:

```python
s = super()
```

This creates a `super` object.

That object is a proxy.

When you access:

```python
super().show
```

the proxy searches the MRO after the current class for `show`.

Then:

```python
super().show()
```

calls the method it found.

So:

```text
super() creates an MRO-aware proxy
attribute access on that proxy finds the next method
calling that attribute runs the method
```

This matters because `super()` is not a function that calls the parent.

It is a helper for controlled attribute lookup along the MRO.

---

# Cooperative Methods

Cooperative methods are methods designed to work together through `super()`.

Example:

```python
class A:
    def process(self):
        print("A")


class B(A):
    def process(self):
        print("B before")
        super().process()
        print("B after")
```

`B.process` cooperates by calling:

```python
super().process()
```

In multiple inheritance, cooperation means each class calls `super()` so the next class in the MRO gets a chance to run.

Example:

```python
class C(A):
    def process(self):
        print("C before")
        super().process()
        print("C after")
```

Then a subclass combining `B` and `C` can run both behaviors in MRO order.

If one class refuses to call `super()`, the chain may stop.

---

# A Cooperative Chain

Example:

```python
class A:
    def process(self):
        print("A")


class B(A):
    def process(self):
        print("B")
        super().process()


class C(A):
    def process(self):
        print("C")
        super().process()


class D(B, C):
    def process(self):
        print("D")
        super().process()
```

Use:

```python
D().process()
```

Output:

```text
D
B
C
A
```

The chain follows:

```text
D -> B -> C -> A
```

because that is `D`'s MRO.

Each method calls `super().process()`.

Each method gives the next class a chance.

This is cooperative inheritance.

---

# Breaking the Chain

If a class does not call `super()`, the chain stops.

Example:

```python
class A:
    def process(self):
        print("A")


class B(A):
    def process(self):
        print("B")


class C(A):
    def process(self):
        print("C")
        super().process()


class D(B, C):
    def process(self):
        print("D")
        super().process()
```

Use:

```python
D().process()
```

MRO:

```text
D
B
C
A
object
```

Output:

```text
D
B
```

Why?

`D.process` calls `B.process`.

`B.process` does not call `super()`.

So `C.process` and `A.process` never run.

Cooperative inheritance requires cooperation.

One non-cooperative method can stop the chain.

---

# Cooperative `__init__`

Initialization is where `super()` mistakes often hurt.

Simple example:

```python
class Person:
    def __init__(self, name):
        self.name = name


class Employee(Person):
    def __init__(self, name, employee_id):
        super().__init__(name)
        self.employee_id = employee_id
```

This is straightforward single inheritance.

With multiple inheritance, every class in the chain should cooperate.

Example:

```python
class Named:
    def __init__(self, name, **kwargs):
        super().__init__(**kwargs)
        self.name = name


class Identified:
    def __init__(self, identifier, **kwargs):
        super().__init__(**kwargs)
        self.identifier = identifier


class User(Named, Identified):
    def __init__(self, name, identifier):
        super().__init__(name=name, identifier=identifier)
```

This works because each class accepts what it needs and forwards the rest.

This pattern is advanced but important.

---

# Why `**kwargs` Appears in Cooperative Initialization

In multiple inheritance, different classes may need different initialization arguments.

Example:

```python
class Named:
    def __init__(self, name, **kwargs):
        super().__init__(**kwargs)
        self.name = name
```

`Named` takes:

```text
name
```

and forwards the rest:

```python
super().__init__(**kwargs)
```

Another class:

```python
class Timestamped:
    def __init__(self, created_at, **kwargs):
        super().__init__(**kwargs)
        self.created_at = created_at
```

Combined:

```python
class Record(Named, Timestamped):
    def __init__(self, name, created_at):
        super().__init__(name=name, created_at=created_at)
```

MRO controls which initializer receives which arguments.

Each class consumes its own argument and forwards the rest.

This is cooperative initialization.

It is especially useful for mixins.

---

# `object.__init__` Takes No Extra Arguments

At the end of many MRO chains is:

```python
object
```

`object.__init__` does not accept arbitrary extra arguments.

That means cooperative initializers must eventually consume all keyword arguments before reaching `object.__init__`.

Example:

```python
class A:
    def __init__(self, value, **kwargs):
        super().__init__(**kwargs)
        self.value = value
```

If `kwargs` still contains unknown keys when it reaches `object.__init__`, Python raises an error.

This is why cooperative initialization must be carefully designed.

Each class should:

```text
accept its own arguments
forward the rest
not swallow arguments silently unless intended
```

This is not beginner everyday code.

But understanding it helps you read frameworks and mixins.

---

# Mixins Preview

A mixin is a class designed to add a small piece of behavior through multiple inheritance.

Example:

```python
class JsonMixin:
    def to_json(self):
        return str(self.__dict__)
```

Use:

```python
class User(JsonMixin):
    def __init__(self, name):
        self.name = name
```

Now:

```python
user = User("Ada")
print(user.to_json())
```

Mixins often rely on MRO.

More advanced mixins call `super()` so they cooperate with other classes.

Chapter 50 will study mixins properly.

For now:

```text
mixins depend on predictable MRO
```

That is why MRO comes before ABCs and mixins in this book.

---

# MRO and Attribute Lookup Beyond Methods

MRO is called method resolution order, but it affects attribute lookup generally.

Example:

```python
class A:
    value = "A"


class B(A):
    pass


class C(A):
    value = "C"


class D(B, C):
    pass
```

MRO:

```text
D
B
C
A
object
```

Use:

```python
print(D().value)
```

Output:

```text
C
```

Why?

`D` has no `value`.

`B` has no `value`.

`C` has `value`.

So Python stops there.

MRO affects class attributes, methods, properties, descriptors, and more.

It is attribute resolution order.

The traditional name is method resolution order.

---

# MRO and Class Methods Preview

Class methods also follow MRO.

Example:

```python
class A:
    @classmethod
    def name(cls):
        return "A"


class B(A):
    pass
```

Use:

```python
print(B.name())
```

Output:

```text
A
```

The class method is found on `A`.

But it is bound to `B` as `cls`.

We will study class methods more deeply later.

For now, notice:

```text
MRO decides where the attribute is found
binding rules decide how it behaves
```

This same pattern appears in methods, descriptors, properties, and class methods.

---

# MRO and Descriptors Preview

Descriptors can customize attribute access.

Methods, properties, static methods, and class methods all involve descriptor behavior.

MRO determines where Python finds the descriptor.

Then descriptor rules determine what happens when it is accessed.

Example:

```python
class A:
    @property
    def value(self):
        return "A"


class B(A):
    pass
```

Use:

```python
print(B().value)
```

Output:

```text
A
```

Python finds `value` in `A` by MRO.

Then the property descriptor runs.

This is why MRO matters far beyond ordinary methods.

Descriptors will make much more sense after MRO is clear.

---

# Reading MRO Output

Suppose:

```python
class A:
    pass


class B(A):
    pass


class C(A):
    pass


class D(B, C):
    pass
```

Print:

```python
for cls in D.mro():
    print(cls.__name__)
```

Output:

```text
D
B
C
A
object
```

This output is your inheritance map.

When debugging:

```python
print(SomeClass.mro())
```

or:

```python
print([cls.__name__ for cls in SomeClass.mro()])
```

This is often clearer than guessing.

If a method call surprises you, inspect:

```text
the object's class
the class MRO
which classes define that method
```

MRO debugging is disciplined curiosity.

---

# A Complete Example: Logging Mix Behavior

Classes:

```python
class Base:
    def save(self):
        print("Base save")


class AuditMixin(Base):
    def save(self):
        print("Audit before")
        super().save()
        print("Audit after")


class ValidateMixin(Base):
    def save(self):
        print("Validate before")
        super().save()
        print("Validate after")


class Model(AuditMixin, ValidateMixin):
    pass
```

MRO:

```text
Model
AuditMixin
ValidateMixin
Base
object
```

Call:

```python
Model().save()
```

Output:

```text
Audit before
Validate before
Base save
Validate after
Audit after
```

Notice the nesting.

Each class adds behavior around the next class.

This is cooperative method wrapping.

It is powerful when used carefully.

---

# A Complete Example: Cooperative Initialization

Example:

```python
class Base:
    def __init__(self, **kwargs):
        super().__init__()


class Named(Base):
    def __init__(self, name, **kwargs):
        super().__init__(**kwargs)
        self.name = name


class Tagged(Base):
    def __init__(self, tag, **kwargs):
        super().__init__(**kwargs)
        self.tag = tag


class Item(Named, Tagged):
    def __init__(self, name, tag):
        super().__init__(name=name, tag=tag)
```

Use:

```python
item = Item("Book", "inventory")

print(item.name)
print(item.tag)
```

Output:

```text
Book
inventory
```

MRO:

```text
Item
Named
Tagged
Base
object
```

Call chain:

```text
Item.__init__
Named.__init__
Tagged.__init__
Base.__init__
object.__init__
```

Each class takes what it needs.

Each class forwards the rest.

---

# When Not to Use Multiple Inheritance

Multiple inheritance is powerful, but it can be confusing.

Avoid it when:

* Composition is clearer.
* Parent classes were not designed to cooperate.
* Initializers require incompatible arguments.
* Method names collide accidentally.
* You cannot explain the MRO simply.
* You only want to reuse a helper function.

Example:

```python
class UserService(DatabaseClient, EmailSender, Logger):
    ...
```

This may be a design smell.

A service probably has a database client, has an email sender, and has a logger.

Composition may be clearer:

```python
class UserService:
    def __init__(self, database, email_sender, logger):
        self._database = database
        self._email_sender = email_sender
        self._logger = logger
```

Use multiple inheritance when it expresses real type behavior or mixin composition.

Do not use it as a basket for dependencies.

---

# Common Mistake: Thinking `super()` Means Parent

In this hierarchy:

```python
class A:
    def show(self):
        print("A")


class B(A):
    def show(self):
        print("B")
        super().show()


class C(A):
    def show(self):
        print("C")
        super().show()


class D(B, C):
    pass
```

Inside `B.show`, `super().show()` calls `C.show` when the instance is `D`.

It does not call `A.show`.

Why?

MRO for `D`:

```text
D
B
C
A
object
```

The next class after `B` is `C`.

Correct mental model:

```text
super() means next in MRO
```

not:

```text
super() means my parent class
```

---

# Common Mistake: Mixing Direct Parent Calls and `super()`

Bad:

```python
class B(A):
    def process(self):
        A.process(self)
```

In simple single inheritance, this can work.

In multiple inheritance, it can skip classes or call shared ancestors twice.

Prefer:

```python
class B(A):
    def process(self):
        super().process()
```

Direct parent calls hard-code one class.

`super()` follows the MRO.

If every class cooperates with `super()`, Python can move through the hierarchy correctly.

Rule:

```text
In cooperative hierarchies, use super() consistently.
```

Do not mix styles casually.

---

# Common Mistake: One Class Stops the Chain

Example:

```python
class B(A):
    def process(self):
        print("B")
```

If `B.process` should be cooperative but does not call:

```python
super().process()
```

then later classes in the MRO do not run.

This may be intentional.

Sometimes overriding should replace behavior completely.

But in cooperative multiple inheritance, it is often a bug.

Ask:

```text
Should this method replace the chain?
Or participate in the chain?
```

If participate, call `super()`.

---

# Common Mistake: Incompatible `__init__` Signatures

Multiple inheritance becomes painful when initializers do not cooperate.

Example:

```python
class A:
    def __init__(self, name):
        self.name = name


class B:
    def __init__(self, value):
        self.value = value


class C(A, B):
    pass
```

What should:

```python
C(...)
```

receive?

How should arguments flow through `A` and `B`?

If classes are not designed for cooperative initialization, combining them can be awkward.

Cooperative classes often use:

```python
def __init__(self, own_arg, **kwargs):
    super().__init__(**kwargs)
```

But do not force this pattern everywhere.

Use it when multiple inheritance is actually needed.

---

# Common Mistake: Ignoring MRO When Debugging

If a method call surprises you, do not guess.

Inspect the MRO:

```python
print(type(obj).mro())
```

or:

```python
print([cls.__name__ for cls in type(obj).mro()])
```

Then ask:

```text
Which class defines this method?
Which class appears first in the MRO?
Does a subclass override it?
Does a mixin call super()?
Does a class stop the cooperative chain?
```

MRO is visible.

Use it.

Inheritance debugging becomes much easier when you read the lookup path directly.

---

# Common Mistake: Using Multiple Inheritance for Dependencies

Bad:

```python
class ReportService(DatabaseClient, EmailSender):
    ...
```

This suggests:

```text
ReportService is a DatabaseClient
ReportService is an EmailSender
```

Usually false.

Better:

```python
class ReportService:
    def __init__(self, database, email_sender):
        self._database = database
        self._email_sender = email_sender
```

This says:

```text
ReportService has a database
ReportService has an email sender
```

Use inheritance for type relationships and mixin behavior.

Use composition for dependencies.

Chapter 46 matters here.

---

# Design Guidance

When using inheritance and `super()`, ask:

```text
What is this class's MRO?
Do all cooperative methods call super()?
Is this method replacing or extending behavior?
Are initializer signatures compatible?
Are parent classes designed for multiple inheritance?
Would composition be clearer?
Does parent order communicate the intended priority?
Could a method name collision happen accidentally?
Will future subclasses understand the extension points?
```

General guidance:

* Inspect MRO when inheritance is unclear.
* Use `super()` to extend inherited behavior.
* Remember that `super()` means next in MRO.
* Use multiple inheritance sparingly.
* Use mixins for small focused behavior.
* Keep cooperative methods consistent.
* Prefer composition for dependencies.
* Avoid deep and tangled hierarchies.

MRO is not something to fear.

It is the rulebook that makes Python inheritance predictable.

---

# Exercises

1. Create:

```python
class A:
    pass


class B(A):
    pass


class C(B):
    pass
```

Print:

```python
C.mro()
```

Explain the order.

---

2. Create two parent classes that both define `move`.

Create:

```python
class Duck(Flyer, Swimmer):
    pass
```

Then reverse the parent order.

How does output change?

---

3. Build the diamond:

```text
A
B(A)
C(A)
D(B, C)
```

Print `D.mro()`.

Why does `A` appear only once?

---

4. Create a cooperative method chain:

```text
D -> B -> C -> A
```

where each `process` method prints its class name and calls `super().process()`.

What output do you get?

---

5. Remove `super()` from one class in the chain.

What output changes?

Explain why.

---

6. Explain why `super()` inside `B` may call `C` when the instance is `D`.

Use the MRO in your answer.

---

7. Create cooperative `__init__` methods using `**kwargs`.

Have one class consume `name` and another consume `tag`.

Create a class that inherits from both.

Show that both attributes are initialized.

---

8. Create a class hierarchy where parent order causes a different method to be selected.

Explain why parent order matters.

---

9. Rewrite this design using composition:

```python
class Service(DatabaseClient, Logger):
    ...
```

Why is composition clearer?

---

10. In your own words, explain:

```text
MRO turns an inheritance graph into one lookup path
```

Use a diamond diagram in your explanation.

---

# Summary

In this chapter we learned:

* MRO means method resolution order.
* MRO is the order Python follows when looking through classes.
* `Class.mro()` and `Class.__mro__` reveal the lookup order.
* Single inheritance has a simple linear MRO.
* Multiple inheritance requires a consistent lookup path.
* Python uses C3 linearization for normal class MRO.
* The diamond problem happens when multiple paths share an ancestor.
* Python's MRO includes shared ancestors once.
* `super()` does not simply mean "call my parent."
* `super()` means continue lookup at the next class in the MRO.
* The same `super()` call can resolve differently depending on the instance's class.
* Cooperative methods call `super()` so every class in the chain can participate.
* A method that omits `super()` can stop the chain.
* Cooperative `__init__` often uses keyword arguments and `**kwargs`.
* Mixins depend on predictable MRO.
* MRO affects attributes, methods, properties, descriptors, class methods, and more.
* Multiple inheritance is powerful but should be used carefully.

Core model:

```text
MRO:
    one ordered path through a class hierarchy

super():
    proxy that continues attribute lookup after the current class in the MRO
```

Design model:

```text
single inheritance:
    usually simple specialization

multiple inheritance:
    use carefully for cooperative behavior and mixins

composition:
    prefer for dependencies and has-a relationships
```

MRO is the map.

`super()` follows the map.

Once that is clear, advanced inheritance becomes much more understandable.

---

# Preview of Chapter 49

Next we study polymorphism and duck typing.

Inheritance taught us how classes can share and specialize behavior through class relationships.

MRO taught us how Python searches those relationships.

But Python often does not require inheritance at all.

Chapter 49 explains how objects can be used interchangeably because they provide the same behavior.

We will study:

* What polymorphism means.
* How inheritance supports polymorphism.
* What duck typing means.
* Why behavior can matter more than exact class.
* How composition and duck typing work together.
* How protocols emerge naturally from expected methods.
* When `isinstance()` helps and when it makes code too rigid.
* How duck typing prepares us for ABCs, mixins, and static typing.

The transition is important:

```text
inheritance says what an object is
duck typing cares what an object can do
```

This is one of the core ideas of Pythonic design.
