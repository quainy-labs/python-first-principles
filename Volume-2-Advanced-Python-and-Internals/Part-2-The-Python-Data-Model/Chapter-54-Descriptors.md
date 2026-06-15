# Chapter 54 — Descriptors

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a descriptor is.
* Why descriptors exist.
* How descriptors customize attribute access.
* How `__get__`, `__set__`, and `__delete__` work.
* Why descriptors must normally live on the class, not the instance.
* What `__set_name__` does.
* How descriptors store per-instance data.
* The difference between data descriptors and non-data descriptors.
* How descriptor lookup interacts with instance dictionaries.
* Why functions become bound methods.
* Why `property`, `staticmethod`, and `classmethod` are descriptor-based.
* How descriptors can implement validation.
* How descriptors can implement lazy computation.
* How descriptors can support ORM-like field behavior.
* When descriptors are better than properties.
* When descriptors are too much machinery.

Chapter 53 showed how operators are syntax over protocol methods.

Descriptors are another protocol family.

But instead of customizing operators, descriptors customize attribute access.

When you write:

```python
obj.name
```

you may think Python simply looks in `obj.__dict__`.

Often it does.

But not always.

Sometimes the attribute found on the class is a descriptor.

When that happens, the object stored in the class can control what attribute access means.

This is one of Python's deepest ideas.

Descriptors explain:

* methods
* bound methods
* properties
* class methods
* static methods
* slots
* many validation patterns
* many ORM field patterns
* many framework internals

Descriptors are not something you write every day.

But understanding them changes how you see Python.

---

# The Simplest Definition

A descriptor is an object that defines at least one of these methods:

```python
__get__
__set__
__delete__
```

These methods form the descriptor protocol.

The most common shape is:

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        ...

    def __set__(self, obj, value):
        ...

    def __delete__(self, obj):
        ...
```

Not every descriptor implements all three.

If an object defines `__get__`, it can customize reading an attribute.

If it defines `__set__`, it can customize assigning an attribute.

If it defines `__delete__`, it can customize deleting an attribute.

Descriptors are usually placed as class attributes:

```python
class Person:
    age = SomeDescriptor()
```

Then:

```python
person.age
```

may call:

```python
Person.__dict__["age"].__get__(person, Person)
```

And:

```python
person.age = 30
```

may call:

```python
Person.__dict__["age"].__set__(person, 30)
```

This is the core idea:

```text
an object stored on the class can control access to an attribute on instances
```

---

# A First Descriptor

Here is the smallest useful demonstration:

```python
class Ten:
    def __get__(self, obj, objtype=None):
        return 10
```

Use it in another class:

```python
class Example:
    value = Ten()
```

Now:

```python
example = Example()
print(example.value)
```

Output:

```python
10
```

`value` is not stored in `example.__dict__`.

It is not an ordinary integer class attribute either.

It is a descriptor object.

When Python sees that the class attribute has `__get__`, it calls the descriptor.

The descriptor returns `10`.

This example is not useful in real code.

If you want a constant, write:

```python
class Example:
    value = 10
```

But the example proves the mechanism.

Attribute access can run code.

---

# The Three Arguments to `__get__`

The descriptor `__get__` method usually has this signature:

```python
def __get__(self, obj, objtype=None):
    ...
```

The arguments are:

```text
self     -> the descriptor object itself
obj      -> the instance being accessed, or None if accessed through the class
objtype  -> the owner class
```

Example:

```python
class DebugDescriptor:
    def __get__(self, obj, objtype=None):
        print(f"self = {self!r}")
        print(f"obj = {obj!r}")
        print(f"objtype = {objtype!r}")
        return "value"
```

Use it:

```python
class Example:
    attr = DebugDescriptor()
```

Instance access:

```python
example = Example()
example.attr
```

Here `obj` is the `example` instance.

Class access:

```python
Example.attr
```

Here `obj` is `None`.

This difference matters.

Many descriptors return themselves when accessed through the class:

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return ...
```

That allows class-level introspection:

```python
Example.attr
```

can expose the descriptor object itself, while:

```python
example.attr
```

returns the managed value.

---

# Descriptors Must Usually Be Class Attributes

Descriptors work when they are found through class attribute lookup.

Example:

```python
class Example:
    attr = Ten()
```

This works:

```python
Example().attr
```

But this does not make the instance attribute act like a descriptor:

```python
example = Example()
example.attr = Ten()
```

Now:

```python
example.attr
```

returns the `Ten` object itself.

Python does not invoke descriptor behavior for descriptors merely stored in the instance dictionary.

The descriptor protocol is part of class-based attribute lookup.

This is why descriptors are normally declared in the class body:

```python
class Person:
    name = ValidatedString()
    age = PositiveNumber()
```

The descriptor objects live on the class.

They manage data for instances.

---

# A Managed Attribute

Let us build a descriptor that logs reads and writes.

```python
class LoggedAttribute:
    def __get__(self, obj, objtype=None):
        print("reading value")
        return obj._value

    def __set__(self, obj, value):
        print(f"setting value to {value!r}")
        obj._value = value
```

Use it:

```python
class Example:
    value = LoggedAttribute()

    def __init__(self, value):
        self.value = value
```

Now:

```python
example = Example(10)
print(example.value)
example.value = 20
```

Output:

```text
setting value to 10
reading value
10
setting value to 20
```

The public attribute is:

```python
value
```

The actual stored data is:

```python
_value
```

This is a common descriptor pattern:

```text
public managed name -> descriptor
private storage name -> instance data
```

---

# The Hardcoded Name Problem

The previous descriptor has a problem.

It always uses:

```python
_value
```

So this class fails conceptually:

```python
class Product:
    price = LoggedAttribute()
    quantity = LoggedAttribute()
```

Both descriptors would store into the same private name:

```python
_value
```

That means `price` and `quantity` would overwrite each other.

We need each descriptor to know the name it was assigned to.

Python provides:

```python
__set_name__
```

---

# `__set_name__`

When a class is created, Python can notify descriptor-like objects about the name they were assigned.

The method is:

```python
def __set_name__(self, owner, name):
    ...
```

Arguments:

```text
self   -> the descriptor object
owner  -> the class being created
name   -> the attribute name used in the class body
```

Example:

```python
class LoggedAttribute:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        print(f"reading {self.public_name}")
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        print(f"setting {self.public_name} to {value!r}")
        setattr(obj, self.private_name, value)
```

Now:

```python
class Product:
    price = LoggedAttribute()
    quantity = LoggedAttribute()

    def __init__(self, price, quantity):
        self.price = price
        self.quantity = quantity
```

Each descriptor learns its own name.

`price` stores in:

```python
_price
```

`quantity` stores in:

```python
_quantity
```

Usage:

```python
product = Product(100, 3)
print(product.price)
print(product.quantity)
```

This is the point where descriptors start to become practical.

One descriptor class can manage many attributes.

---

# Where the Data Lives

In many descriptors, the descriptor object lives on the class and the actual value lives on the instance.

Example:

```python
class Product:
    price = LoggedAttribute()
```

There is one `LoggedAttribute` object attached to `Product`.

But there can be many products:

```python
first = Product(100, 3)
second = Product(200, 5)
```

Each product needs its own `price`.

So the descriptor should not store the price directly on itself:

```python
class BadDescriptor:
    def __set__(self, obj, value):
        self.value = value
```

That would share one value across all instances.

Instead, store per-instance data on the instance:

```python
def __set__(self, obj, value):
    setattr(obj, self.private_name, value)
```

The descriptor is shared.

The instance data is separate.

This distinction is essential.

```text
descriptor object -> class-level manager
managed value -> usually per-instance storage
```

---

# Data Descriptors and Non-Data Descriptors

Descriptors come in two important categories.

A data descriptor defines `__set__` or `__delete__`.

Example:

```python
class DataDescriptor:
    def __get__(self, obj, objtype=None):
        ...

    def __set__(self, obj, value):
        ...
```

A non-data descriptor defines `__get__` but not `__set__` or `__delete__`.

Example:

```python
class NonDataDescriptor:
    def __get__(self, obj, objtype=None):
        ...
```

The distinction matters because of lookup precedence.

Data descriptors generally take priority over instance attributes.

Non-data descriptors can be overridden by instance attributes.

This is not a small detail.

It explains why properties prevent accidental shadowing, while methods can be shadowed on individual instances.

---

# Data Descriptor Precedence

Consider:

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        return "from descriptor"

    def __set__(self, obj, value):
        obj.__dict__["attr"] = value
```

Use it:

```python
class Example:
    attr = Descriptor()
```

Now:

```python
example = Example()
example.__dict__["attr"] = "from instance"

print(example.attr)
```

The descriptor wins because it is a data descriptor.

Output:

```python
from descriptor
```

Even though the instance dictionary has an `attr` key, the data descriptor takes precedence.

That is why a property with a setter is not easily shadowed by assigning the same name into the instance dictionary.

Data descriptors are strong managers.

They control access.

---

# Non-Data Descriptor Precedence

Now consider a descriptor with only `__get__`:

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        return "from descriptor"
```

Use it:

```python
class Example:
    attr = Descriptor()
```

If the instance has no `attr`, descriptor access happens:

```python
example = Example()
print(example.attr)
```

Output:

```python
from descriptor
```

But if the instance dictionary has `attr`:

```python
example.__dict__["attr"] = "from instance"
print(example.attr)
```

Output:

```python
from instance
```

The instance attribute shadows the non-data descriptor.

This behavior is useful.

It allows lazy descriptors and cached properties.

The descriptor can compute a value once and then store the result in the instance dictionary under the same name.

Future lookups find the cached value first.

---

# Attribute Lookup Order

For instance attribute access:

```python
obj.attr
```

the simplified lookup order is:

```text
1. data descriptor on the class or its bases
2. value in obj.__dict__
3. non-data descriptor on the class or its bases
4. plain class attribute
5. __getattr__ fallback, if defined
```

This is simplified, but it is an excellent working model.

Data descriptors win over instance attributes.

Instance attributes win over non-data descriptors.

Non-data descriptors win over plain class attributes only because they are class attributes with `__get__`.

If nothing is found, `__getattr__` may be called.

This ordering explains many Python behaviors that otherwise seem magical.

---

# A Non-Data Descriptor for Lazy Computation

Let us build a lazy attribute.

Goal:

```python
report.total
```

should compute once, store the result, and reuse it.

Descriptor:

```python
class LazyAttribute:
    def __init__(self, function):
        self.function = function
        self.name = function.__name__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.function(obj)
        obj.__dict__[self.name] = value
        return value
```

Use it:

```python
class Report:
    def __init__(self, values):
        self.values = values

    @LazyAttribute
    def total(self):
        print("computing total")
        return sum(self.values)
```

Now:

```python
report = Report([1, 2, 3])
print(report.total)
print(report.total)
```

Output:

```text
computing total
6
6
```

Why did it compute only once?

Because `LazyAttribute` is a non-data descriptor.

After the first lookup, it stores:

```python
report.__dict__["total"] = 6
```

On the next lookup, the instance dictionary wins over the non-data descriptor.

This is the core idea behind many cached-property implementations.

---

# Why Data Descriptors Cannot Be Cached the Same Way

If `LazyAttribute` defined `__set__`, it would become a data descriptor.

Then the lookup order would change.

The descriptor would win over the instance dictionary.

Even if it stored:

```python
obj.__dict__[self.name] = value
```

future lookups would still call `__get__`.

That is why non-data descriptors are useful for lazy caching.

The absence of `__set__` is not a missing feature.

It is part of the design.

Descriptor category affects behavior.

---

# Validation Descriptors

Descriptors are useful when multiple attributes need reusable validation.

Start with a base validator:

```python
class Validator:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        self.validate(value)
        setattr(obj, self.private_name, value)

    def validate(self, value):
        raise NotImplementedError
```

Now create specific validators:

```python
class PositiveNumber(Validator):
    def validate(self, value):
        if not isinstance(value, int | float):
            raise TypeError(f"{self.public_name} must be numeric")
        if value <= 0:
            raise ValueError(f"{self.public_name} must be positive")
```

And:

```python
class NonEmptyString(Validator):
    def validate(self, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.public_name} must be a string")
        if not value:
            raise ValueError(f"{self.public_name} cannot be empty")
```

Use them:

```python
class Product:
    name = NonEmptyString()
    price = PositiveNumber()

    def __init__(self, name, price):
        self.name = name
        self.price = price
```

Now:

```python
Product("Keyboard", 100)
```

works.

But:

```python
Product("", 100)
```

raises an error.

And:

```python
Product("Keyboard", -5)
```

raises an error.

The validation logic is reusable across classes.

That is a strong descriptor use case.

---

# Descriptors Versus Properties

A property manages one attribute inside one class.

Example:

```python
class Product:
    def __init__(self, price):
        self.price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value <= 0:
            raise ValueError("price must be positive")
        self._price = value
```

This is clear for one attribute.

But if you need the same validation in many places:

```python
class Product:
    price = PositiveNumber()
    weight = PositiveNumber()
    rating = PositiveNumber()
```

the descriptor is cleaner.

Properties are usually best when:

* behavior is specific to one class
* logic is simple
* the managed attribute is unique

Descriptors are best when:

* the same attribute behavior is reused
* a framework needs declarative fields
* validation logic should be packaged
* class-level field metadata matters

Chapter 55 studies properties directly.

For now, understand that `property` itself is descriptor-based.

---

# Functions Are Descriptors

One of the most important descriptor facts is this:

```text
functions stored on a class are descriptors
```

Consider:

```python
class Greeter:
    def greet(self):
        return "hello"
```

Inside the class dictionary, `greet` is a function object.

But when you access it through an instance:

```python
greeter = Greeter()
greeter.greet
```

you get a bound method.

The method remembers:

* the original function
* the instance to pass as `self`

That binding behavior happens because function objects implement descriptor behavior.

Conceptually:

```python
Greeter.__dict__["greet"].__get__(greeter, Greeter)
```

returns a bound method.

Then:

```python
greeter.greet()
```

calls the original function with `greeter` as the first argument.

This is why:

```python
Greeter.greet(greeter)
```

and:

```python
greeter.greet()
```

can produce the same result.

Descriptors are not obscure.

They are how methods work.

---

# Class Access Versus Instance Access for Methods

Look at:

```python
class Greeter:
    def greet(self):
        return "hello"
```

Class access:

```python
Greeter.greet
```

returns the function-like object without binding to a particular instance.

Instance access:

```python
Greeter().greet
```

returns a bound method.

That is because `obj` differs in `__get__`.

When accessed through the class, `obj` is `None`.

When accessed through an instance, `obj` is the instance.

The descriptor can choose what to return in each case.

This same pattern appears in custom descriptors:

```python
def __get__(self, obj, objtype=None):
    if obj is None:
        return self
    return ...
```

Class access is often used for introspection.

Instance access is usually used for actual values or bound behavior.

---

# `staticmethod` and `classmethod` Are Descriptor-Based

`staticmethod` changes method binding.

Example:

```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b
```

Access:

```python
Math.add(1, 2)
Math().add(1, 2)
```

Neither call receives an automatic `self`.

`classmethod` binds to the class instead of the instance:

```python
class User:
    @classmethod
    def anonymous(cls):
        return cls("anonymous")

    def __init__(self, name):
        self.name = name
```

Access:

```python
User.anonymous()
```

passes `User` as `cls`.

These binding behaviors are descriptor behaviors.

Chapter 55 covers them directly.

For now, the key idea is:

```text
descriptors control what attribute access returns
```

For normal methods, access returns a bound method.

For static methods, access returns the function without binding.

For class methods, access returns a function bound to the class.

---

# `property` Is a Data Descriptor

A property is also a descriptor.

Example:

```python
class Person:
    def __init__(self, age):
        self.age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("age cannot be negative")
        self._age = value
```

The `property` object is stored on the class:

```python
Person.__dict__["age"]
```

It defines descriptor behavior.

Reading:

```python
person.age
```

calls the property's getter.

Writing:

```python
person.age = 30
```

calls the property's setter.

Deleting:

```python
del person.age
```

can call the property's deleter if one exists.

Properties are the most common descriptor most Python programmers use.

Even if they never write a descriptor class directly, they use descriptor machinery through `property`.

---

# Descriptors and `super()`

Descriptors also interact with `super()`.

When you call:

```python
super().method()
```

Python is not merely fetching a plain function.

It is searching the MRO after the current class and binding the found descriptor appropriately.

This is part of why `super()` works with methods.

The descriptor protocol is invoked with the right object and class context.

You do not need to implement special `super()` behavior for ordinary descriptors most of the time.

But it is useful to know that descriptors participate in:

* instance attribute access
* class attribute access
* `super()` attribute access

This is another reason descriptors sit deep in the object model.

---

# Descriptor Storage Mistake: Shared State

This descriptor is wrong for per-instance data:

```python
class BadPositiveNumber:
    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("must be positive")
        self.value = value

    def __get__(self, obj, objtype=None):
        return self.value
```

Use it:

```python
class Product:
    price = BadPositiveNumber()
```

Now:

```python
first = Product()
second = Product()

first.price = 100
second.price = 200

print(first.price)
```

It may print:

```python
200
```

because the descriptor object is shared by all instances.

The value was stored on the descriptor itself.

Correct approach:

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("must be positive")
        setattr(obj, self.private_name, value)

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)
```

Now each instance stores its own value.

The descriptor stores metadata about how to manage the value.

---

# Descriptor Storage with WeakKeyDictionary

Sometimes you cannot or do not want to store values in the instance dictionary.

One option is `weakref.WeakKeyDictionary`.

Example:

```python
from weakref import WeakKeyDictionary


class ExternalStorage:
    def __init__(self):
        self._values = WeakKeyDictionary()

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return self._values[obj]

    def __set__(self, obj, value):
        self._values[obj] = value
```

Here the descriptor stores values externally, keyed by instance.

Weak keys allow instance entries to disappear when instances are no longer alive.

This avoids keeping instances alive just because the descriptor remembers them.

This technique is useful when:

* instances do not have a `__dict__`
* storage names would conflict
* descriptor-managed data should be external

But it has tradeoffs.

Instances must be weak-referenceable.

Weak dictionaries add complexity.

Instance dictionary storage is simpler when available.

Use external storage only when it solves a real problem.

---

# Descriptors and `__slots__`

Chapter 56 will study `__slots__` deeply.

For now, know this:

Slot attributes are implemented using descriptor machinery.

When a class defines slots:

```python
class Point:
    __slots__ = ("x", "y")
```

Python creates class-level descriptor objects that manage access to slot storage.

This is why slot attributes can work without a normal instance `__dict__`.

The class has descriptors that know how to read and write values from the object's slot layout.

This is another place where descriptors are not optional decoration.

They are part of Python's core object system.

---

# Descriptors and ORMs

Object-relational mappers often use descriptor-like fields.

A model might look like:

```python
class User:
    id = IntegerField(primary_key=True)
    email = StringField(unique=True)
```

Those field objects can do several things:

* remember the field name
* validate assignments
* build database queries
* track changed values
* define schema metadata
* convert Python values to database values
* convert database values back to Python values

A simplified field descriptor:

```python
class Field:
    def __init__(self, column_type):
        self.column_type = column_type

    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, None)

    def __set__(self, obj, value):
        setattr(obj, self.private_name, value)
```

Use it:

```python
class User:
    id = Field("INTEGER")
    email = Field("TEXT")

    def __init__(self, id, email):
        self.id = id
        self.email = email
```

Class access:

```python
User.email
```

can expose the field object for query building.

Instance access:

```python
user.email
```

returns the user's email.

This class-versus-instance behavior is exactly what descriptors are good at.

---

# Descriptors and Declarative APIs

Descriptors support declarative class definitions.

Declarative means the class body describes structure:

```python
class Product:
    name = StringField(max_length=100)
    price = MoneyField(currency="INR")
    inventory = IntegerField(minimum=0)
```

The class body reads like a schema.

Behind the scenes, descriptors can:

* learn their assigned names
* validate values
* store metadata
* participate in query generation
* support introspection

This pattern appears in:

* ORMs
* forms
* serializers
* configuration systems
* validation libraries
* admin-interface frameworks

Descriptors are one reason Python frameworks can provide elegant class-body APIs.

But declarative APIs can become too magical.

Good frameworks balance convenience with debuggability.

---

# Descriptor Lookup Without Triggering

Sometimes you want the descriptor object itself.

But:

```python
Product.price
```

may call `__get__`.

If the descriptor returns itself for class access, that is fine.

But to inspect the raw class dictionary value, use:

```python
vars(Product)["price"]
```

or:

```python
Product.__dict__["price"]
```

This retrieves the descriptor object without invoking descriptor lookup.

Example:

```python
descriptor = vars(Product)["price"]
print(descriptor.private_name)
```

This is useful when debugging descriptor behavior.

It is also useful for framework code that scans class definitions.

---

# Descriptors and `__getattribute__`

Descriptors are invoked by the attribute access machinery.

At the center of normal instance lookup is:

```python
object.__getattribute__
```

When you write:

```python
obj.attr
```

Python calls attribute lookup logic.

That logic checks for descriptors at the right points.

If you override `__getattribute__`, you can accidentally bypass or break descriptor behavior.

Example:

```python
class Broken:
    attr = Ten()

    def __getattribute__(self, name):
        return self.__dict__[name]
```

This is broken in multiple ways.

It can recurse.

It ignores class attributes.

It ignores descriptors.

It fails for missing names.

If you override `__getattribute__`, use:

```python
super().__getattribute__(name)
```

unless you have a very careful reason not to.

Most classes should not override `__getattribute__`.

Descriptors are usually a safer way to customize selected attributes.

---

# Descriptors Versus `__getattr__`

`__getattr__` is called only when normal attribute lookup fails.

Example:

```python
class Dynamic:
    def __getattr__(self, name):
        return f"missing attribute {name}"
```

Then:

```python
Dynamic().anything
```

returns:

```python
missing attribute anything
```

Descriptors are different.

They participate during normal lookup when the class attribute is a descriptor.

Use descriptors when:

* specific attributes need managed behavior
* reusable field logic is needed
* class-level declarations matter

Use `__getattr__` when:

* unknown missing attributes should be handled dynamically
* proxy objects need fallback lookup
* lazy module or object attributes are needed

They solve different problems.

---

# Read-Only Descriptors

To make a descriptor read-only, define `__get__` and a `__set__` that raises.

Example:

```python
class ReadOnly:
    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        raise AttributeError("attribute is read-only")
```

But how do we initialize it?

One option is to write directly to the private name:

```python
class User:
    id = ReadOnly()

    def __init__(self, id):
        self._id = id
```

Now:

```python
user = User(1)
print(user.id)
user.id = 2
```

raises.

This pattern can work.

But for simple read-only attributes, a property may be clearer.

Descriptors shine when the behavior is reusable across multiple attributes or classes.

---

# Deleting Attributes with `__delete__`

Descriptors can control deletion:

```python
class Deletable:
    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        setattr(obj, self.private_name, value)

    def __delete__(self, obj):
        print("deleting managed value")
        delattr(obj, self.private_name)
```

Use it:

```python
class Example:
    value = Deletable()

    def __init__(self, value):
        self.value = value
```

Now:

```python
example = Example(10)
del example.value
```

calls:

```python
Deletable.__delete__
```

Deletion descriptors are less common than get/set descriptors.

Use them when deletion has a meaningful policy.

---

# Descriptor With Defaults

Sometimes a descriptor should provide a default value.

Example:

```python
class Defaulted:
    def __init__(self, default):
        self.default = default

    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, self.default)

    def __set__(self, obj, value):
        setattr(obj, self.private_name, value)
```

Use it:

```python
class Settings:
    theme = Defaulted("light")
```

Now:

```python
settings = Settings()
print(settings.theme)
```

Output:

```python
light
```

After assignment:

```python
settings.theme = "dark"
print(settings.theme)
```

Output:

```python
dark
```

Be careful with mutable defaults.

This has the same problem as other shared mutable defaults:

```python
items = Defaulted([])
```

Every instance could see the same list if you return it directly.

For mutable defaults, use a factory pattern.

---

# Descriptor With Default Factory

Here is a descriptor that creates a default per instance:

```python
class DefaultFactory:
    def __init__(self, factory):
        self.factory = factory

    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if not hasattr(obj, self.private_name):
            setattr(obj, self.private_name, self.factory())
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        setattr(obj, self.private_name, value)
```

Use it:

```python
class Inbox:
    messages = DefaultFactory(list)
```

Now:

```python
first = Inbox()
second = Inbox()

first.messages.append("hello")

print(first.messages)
print(second.messages)
```

Output:

```python
['hello']
[]
```

Each instance receives its own list.

This mirrors the `default_factory` idea from dataclasses.

The object model lesson is the same:

```text
create mutable state per instance, not once at class definition time
```

---

# Descriptors and Type Checking

Descriptors often appear in typed code.

At runtime, a descriptor can validate values.

At static-analysis time, descriptors can be challenging because attribute access returns a managed value, not the descriptor object.

Example:

```python
class Product:
    price = PositiveNumber()
```

At class level:

```python
Product.price
```

may return a descriptor.

At instance level:

```python
Product(100).price
```

returns a number.

Static type checkers need to understand that difference.

Later chapters on type checking will revisit this kind of pattern.

For now, know that descriptors are dynamic.

They are powerful partly because class access and instance access can mean different things.

That power can make type reasoning more complicated.

---

# Descriptors and Dataclasses

Dataclasses and descriptors can interact.

But the combination needs care.

Dataclasses inspect annotated fields and generate initialization code.

Descriptors are class attributes that manage access.

If you place a descriptor as a dataclass field, you must understand whether it should be treated as:

* a field value
* a class-level descriptor
* metadata-like behavior
* a managed attribute

Example:

```python
class PositiveNumber:
    ...


@dataclass
class Product:
    price: float = PositiveNumber()
```

This may not mean what you first expect.

The dataclass sees an annotated class attribute with a default value.

The descriptor sees itself assigned to the class.

The interaction depends on dataclass rules and descriptor behavior.

For ordinary code, keep dataclass fields simple.

Use `__post_init__` for validation unless you specifically need reusable descriptor behavior.

Descriptors are more common in frameworks and hand-designed class APIs than in simple dataclasses.

---

# A Practical Validator System

Let us build a small validation system from scratch.

Base descriptor:

```python
class Validator:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        self.validate(value)
        setattr(obj, self.private_name, value)

    def validate(self, value):
        raise NotImplementedError
```

String validator:

```python
class String(Validator):
    def __init__(self, *, min_length=0, max_length=None):
        self.min_length = min_length
        self.max_length = max_length

    def validate(self, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.public_name} must be a string")
        if len(value) < self.min_length:
            raise ValueError(
                f"{self.public_name} must have at least {self.min_length} characters"
            )
        if self.max_length is not None and len(value) > self.max_length:
            raise ValueError(
                f"{self.public_name} must have at most {self.max_length} characters"
            )
```

Number validator:

```python
class Number(Validator):
    def __init__(self, *, minimum=None, maximum=None):
        self.minimum = minimum
        self.maximum = maximum

    def validate(self, value):
        if not isinstance(value, int | float):
            raise TypeError(f"{self.public_name} must be numeric")
        if self.minimum is not None and value < self.minimum:
            raise ValueError(f"{self.public_name} must be at least {self.minimum}")
        if self.maximum is not None and value > self.maximum:
            raise ValueError(f"{self.public_name} must be at most {self.maximum}")
```

Choice validator:

```python
class OneOf(Validator):
    def __init__(self, *choices):
        self.choices = set(choices)

    def validate(self, value):
        if value not in self.choices:
            raise ValueError(
                f"{self.public_name} must be one of {sorted(self.choices)!r}"
            )
```

Use them:

```python
class Product:
    name = String(min_length=1, max_length=100)
    price = Number(minimum=0)
    status = OneOf("draft", "active", "archived")

    def __init__(self, name, price, status="draft"):
        self.name = name
        self.price = price
        self.status = status
```

Now invalid assignments fail immediately:

```python
product = Product("Keyboard", 100)
product.price = -1
```

raises:

```python
ValueError
```

This is a real descriptor use case.

The validation rules are reusable and declarative.

---

# A Practical Lazy Attribute

Here is a practical lazy attribute descriptor:

```python
class lazy_property:
    def __init__(self, function):
        self.function = function
        self.__name__ = function.__name__
        self.__doc__ = function.__doc__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.function(obj)
        obj.__dict__[self.__name__] = value
        return value
```

Use it:

```python
class DataSet:
    def __init__(self, values):
        self.values = values

    @lazy_property
    def average(self):
        print("computing average")
        return sum(self.values) / len(self.values)
```

Now:

```python
data = DataSet([10, 20, 30])
print(data.average)
print(data.average)
```

Only the first access computes.

The second access reads the cached value from the instance dictionary.

This works because `lazy_property` is a non-data descriptor.

If the data changes:

```python
data.values.append(40)
```

the cached average is stale.

That is a design tradeoff.

Lazy descriptors should be used when:

* computation is expensive
* the value is stable enough
* cache invalidation is clear or unnecessary

Do not cache values that should always reflect changing state unless you also design invalidation.

---

# Invalidating a Lazy Value

One simple invalidation strategy is deletion:

```python
del data.__dict__["average"]
```

Then the next access recomputes.

You can make that cleaner:

```python
class DataSet:
    def __init__(self, values):
        self.values = values

    @lazy_property
    def average(self):
        return sum(self.values) / len(self.values)

    def invalidate_average(self):
        self.__dict__.pop("average", None)
```

This is acceptable for small internal code.

For larger systems, caching deserves deliberate design.

Descriptors make lazy attributes easy.

They do not solve cache invalidation automatically.

---

# Descriptor Error Messages

Good descriptors produce good errors.

Weak error:

```python
raise ValueError("invalid")
```

Better:

```python
raise ValueError(f"{self.public_name} must be positive")
```

Even better:

```python
raise ValueError(
    f"{owner_name}.{self.public_name} must be positive; got {value!r}"
)
```

To include the owner name, store it in `__set_name__`:

```python
def __set_name__(self, owner, name):
    self.owner_name = owner.__name__
    self.public_name = name
    self.private_name = "_" + name
```

Then:

```python
raise ValueError(
    f"{self.owner_name}.{self.public_name} must be positive; got {value!r}"
)
```

Descriptors are often reused.

Clear errors help users understand which managed attribute failed.

---

# Descriptor Reuse and Descriptor Instances

Do not reuse the same descriptor instance for multiple attributes unless it is designed for that.

Bad:

```python
positive = PositiveNumber()


class Product:
    price = positive
    weight = positive
```

The descriptor receives `__set_name__` twice.

The second name may overwrite metadata from the first.

Use separate descriptor instances:

```python
class Product:
    price = PositiveNumber()
    weight = PositiveNumber()
```

Each descriptor object manages one attribute name.

This is usually what you want.

Shared descriptor instances are possible, but they require special design.

Keep the simple rule:

```text
one descriptor instance per managed attribute
```

---

# Inheritance and Descriptors

Descriptors are inherited like other class attributes.

Example:

```python
class Entity:
    id = PositiveNumber()


class User(Entity):
    pass
```

Now:

```python
user = User()
user.id = 1
```

uses the descriptor defined on `Entity`.

This can be useful.

But remember that `__set_name__` was called when `Entity` was created.

The descriptor's owner metadata may refer to `Entity`, not `User`.

That is often fine.

But if the descriptor needs per-subclass registration, inheritance may require more machinery.

Frameworks sometimes use metaclasses or `__init_subclass__` to collect fields per subclass.

That belongs to later chapters.

For ordinary descriptors, inherited behavior is usually enough.

---

# Descriptors and MRO

Attribute lookup searches through the class and its bases according to MRO.

Descriptors found along that path can participate in lookup.

Example:

```python
class Base:
    attr = SomeDescriptor()


class Child(Base):
    pass
```

Access:

```python
Child().attr
```

can invoke the descriptor from `Base`.

If `Child` defines an attribute with the same name:

```python
class Child(Base):
    attr = "plain value"
```

then `Child.attr` shadows the base descriptor.

This is ordinary inheritance plus descriptor lookup.

The MRO decides which class attribute is found.

Then descriptor rules decide how it behaves.

---

# Common Mistake: Forgetting `obj is None`

This descriptor has a problem:

```python
class PositiveNumber:
    def __get__(self, obj, objtype=None):
        return getattr(obj, self.private_name)
```

If accessed through the class:

```python
Product.price
```

then `obj` is `None`.

The descriptor tries:

```python
getattr(None, self.private_name)
```

That fails.

Better:

```python
def __get__(self, obj, objtype=None):
    if obj is None:
        return self
    return getattr(obj, self.private_name)
```

This lets class access return the descriptor object.

It supports introspection and avoids surprising errors.

---

# Common Mistake: Recursive Descriptor Access

This is dangerous:

```python
class BadDescriptor:
    def __get__(self, obj, objtype=None):
        return obj.value
```

If the descriptor manages `value`, then:

```python
obj.value
```

inside `__get__` calls the descriptor again.

Then again.

Then again.

Use private storage:

```python
return getattr(obj, self.private_name)
```

And assignment:

```python
setattr(obj, self.private_name, value)
```

The public name goes through descriptor logic.

The private name stores actual data.

---

# Common Mistake: Too Much Magic

Descriptors can hide a lot.

This can be elegant:

```python
class Product:
    name = String(min_length=1)
    price = Number(minimum=0)
```

But if the descriptor:

* performs network calls
* writes to a database
* changes other attributes
* silently coerces many values
* depends on global state
* has surprising side effects

then simple attribute access becomes hard to trust.

This:

```python
product.price
```

should not unexpectedly perform a large remote operation unless the class is clearly designed around lazy remote fields.

Attribute access should feel reasonably lightweight and unsurprising.

Descriptors are powerful enough to violate that expectation.

Use restraint.

---

# Common Mistake: Choosing Descriptors Too Early

Many validation problems do not need descriptors.

For one class and one attribute, a property is simpler.

For one-time construction validation, `__post_init__` or `__init__` is simpler.

For external input validation, a parser or validation library may be more appropriate.

Do not start with descriptors just because they are elegant.

Start with the simplest clear design.

Use descriptors when repeated managed attribute behavior emerges.

Good descriptor use:

```text
same validation pattern across many attributes/classes
```

Weak descriptor use:

```text
one attribute in one class with simple validation
```

Descriptors are a tool for reuse and protocol integration.

They are not the first answer to every attribute problem.

---

# Common Mistake: Confusing Descriptor Object and Managed Value

In this class:

```python
class Product:
    price = PositiveNumber()
```

`Product.__dict__["price"]` is the descriptor object.

`product.price` is the managed value.

Those are not the same thing.

This distinction matters in debugging.

If you print:

```python
Product.price
```

you may get the descriptor object if `__get__` returns `self` for class access.

If you print:

```python
product.price
```

you get the value stored for that product.

Descriptor code often needs to think at both levels:

```text
class level -> descriptor metadata
instance level -> managed data
```

---

# A Step-by-Step Attribute Lookup Example

Consider:

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("must be positive")
        setattr(obj, self.private_name, value)


class Product:
    price = PositiveNumber()

    def __init__(self, price):
        self.price = price
```

When Python creates `Product`, it calls:

```python
Product.__dict__["price"].__set_name__(Product, "price")
```

When you create:

```python
product = Product(100)
```

inside `__init__`, this line:

```python
self.price = price
```

finds the data descriptor and calls:

```python
Product.__dict__["price"].__set__(product, 100)
```

The descriptor validates and stores:

```python
product._price = 100
```

When you read:

```python
product.price
```

Python finds the data descriptor and calls:

```python
Product.__dict__["price"].__get__(product, Product)
```

The descriptor returns:

```python
product._price
```

That is the whole loop.

The public attribute is managed.

The private attribute stores the value.

---

# Practice: Build a Positive Number Descriptor

Create a descriptor that validates positive numbers.

Requirements:

* learn its name with `__set_name__`
* return itself on class access
* validate on assignment
* store per-instance data in a private attribute

Solution:

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        if not isinstance(value, int | float):
            raise TypeError(f"{self.public_name} must be numeric")
        if value <= 0:
            raise ValueError(f"{self.public_name} must be positive")
        setattr(obj, self.private_name, value)
```

Use it:

```python
class Product:
    price = PositiveNumber()

    def __init__(self, price):
        self.price = price
```

Test:

```python
product = Product(100)
assert product.price == 100
```

Invalid:

```python
Product(-1)
```

should raise `ValueError`.

---

# Practice: Build a Lazy Attribute

Create a descriptor that computes once and caches in the instance dictionary.

Solution:

```python
class lazy_attribute:
    def __init__(self, function):
        self.function = function
        self.name = function.__name__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.function(obj)
        obj.__dict__[self.name] = value
        return value
```

Use it:

```python
class Report:
    def __init__(self, values):
        self.values = values

    @lazy_attribute
    def total(self):
        return sum(self.values)
```

Test:

```python
report = Report([1, 2, 3])
assert report.total == 6
assert report.__dict__["total"] == 6
```

Ask:

```text
Why does the second access not call the descriptor?
```

Because `lazy_attribute` is a non-data descriptor and the instance dictionary now has `total`.

---

# Practice: Explain Method Binding

Given:

```python
class User:
    def greet(self):
        return "hello"
```

Explain the difference:

```python
User.greet
User().greet
```

Answer:

```text
User.greet accesses the function through the class.
User().greet accesses the function through an instance.
Functions are descriptors.
Instance access binds the function to the instance and returns a bound method.
The bound method automatically passes the instance as self when called.
```

This is why:

```python
User.greet(User())
```

and:

```python
User().greet()
```

can both work.

---

# Practice: Data or Non-Data?

Classify each descriptor.

```python
class A:
    def __get__(self, obj, objtype=None):
        ...
```

Answer:

```text
non-data descriptor
```

Because it defines `__get__` only.

```python
class B:
    def __get__(self, obj, objtype=None):
        ...

    def __set__(self, obj, value):
        ...
```

Answer:

```text
data descriptor
```

Because it defines `__set__`.

```python
class C:
    def __delete__(self, obj):
        ...
```

Answer:

```text
data descriptor
```

Because `__delete__` is enough to make it a data descriptor.

---

# Practice: Predict Lookup

Given:

```python
class NonData:
    def __get__(self, obj, objtype=None):
        return "descriptor"


class Example:
    attr = NonData()
```

What does this print?

```python
example = Example()
example.__dict__["attr"] = "instance"
print(example.attr)
```

Answer:

```python
instance
```

The instance dictionary shadows a non-data descriptor.

Now change `NonData`:

```python
class Data:
    def __get__(self, obj, objtype=None):
        return "descriptor"

    def __set__(self, obj, value):
        obj.__dict__["attr"] = value
```

Use:

```python
class Example:
    attr = Data()
```

Now:

```python
example = Example()
example.__dict__["attr"] = "instance"
print(example.attr)
```

prints:

```python
descriptor
```

The data descriptor wins.

---

# Summary

Descriptors are objects that define `__get__`, `__set__`, or `__delete__`.

They customize attribute access.

Descriptors usually live as class attributes.

The descriptor object is shared at the class level, while managed values are usually stored per instance.

`__get__` controls reading.

`__set__` controls assignment.

`__delete__` controls deletion.

`__set_name__` lets a descriptor learn the class and attribute name it was assigned to.

Data descriptors define `__set__` or `__delete__`.

Non-data descriptors define `__get__` without `__set__` or `__delete__`.

Data descriptors take precedence over instance dictionary values.

Instance dictionary values can shadow non-data descriptors.

Functions are descriptors, which is why instance method binding works.

Properties, static methods, class methods, cached properties, slots, and many framework fields are descriptor-based.

Descriptors are excellent for reusable managed attribute behavior.

They are also easy to overuse.

The design principle is:

```text
use descriptors when attribute access itself needs reusable protocol behavior
```

If the behavior belongs to one attribute in one class, a property may be simpler.

If the behavior belongs only at construction time, validation in `__init__` or `__post_init__` may be clearer.

Descriptors are not everyday syntax.

They are infrastructure.

But once you understand them, Python's object model becomes much more transparent.

---

# Preview of Chapter 55

Chapter 54 explained descriptors directly.

Next we study three descriptor-powered tools that Python programmers use constantly:

* `property`
* `staticmethod`
* `classmethod`

We will also study how managed attributes should be designed in ordinary classes.

Chapter 55 will answer questions such as:

* When should an attribute become a property?
* How do getters and setters fit Python style?
* Why is `property` better than Java-style accessor methods in many Python classes?
* When should a method be static?
* When should a method be a class method?
* How do these tools relate to descriptors?
* How can managed attributes preserve backwards compatibility?

The transition is:

```text
descriptors are the mechanism
properties, static methods, and class methods are everyday tools built on that mechanism
```

Now that the machinery is visible, the next chapter can focus on design.

