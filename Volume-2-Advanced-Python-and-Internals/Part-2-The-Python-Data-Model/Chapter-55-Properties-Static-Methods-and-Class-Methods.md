# Chapter 55 — Properties, Static Methods, and Class Methods

---

# Learning Objectives

By the end of this chapter, you should understand:

* What `property` does.
* Why properties are descriptor-based managed attributes.
* How getters, setters, and deleters work.
* When an ordinary public attribute is enough.
* When a property improves a class.
* Why Python often prefers properties over Java-style getter and setter methods.
* How properties preserve backwards compatibility.
* How to validate assignment with a property.
* How to expose computed attributes with a property.
* How to design read-only properties.
* When cached values are better than computed properties.
* What `staticmethod` does.
* When a static method is useful.
* When a module-level function is better than a static method.
* What `classmethod` does.
* Why class methods receive `cls` instead of `self`.
* How class methods support alternate constructors.
* How class methods support inheritance-friendly factories.
* How these tools relate to descriptors.
* Which common design mistakes to avoid.

Chapter 54 explained descriptors.

That chapter gave us the machinery.

This chapter focuses on three everyday tools built on that machinery:

```python
property
staticmethod
classmethod
```

You do not need to write custom descriptors every day.

But you will regularly read and write properties.

You will often see class methods.

You will sometimes see static methods.

These tools answer practical design questions:

```text
Should this be an attribute or a method?
Should this value be computed or stored?
Should this constructor have a name?
Should this helper belong to the class?
Should this method receive the instance, the class, or neither?
```

Those questions matter because they shape how users experience your objects.

---

# The Three Method Binding Modes

Before studying each tool, compare the three binding modes.

An instance method receives the instance:

```python
class User:
    def describe(self):
        return f"user: {self.name}"
```

Call:

```python
user.describe()
```

Python passes `user` as `self`.

A class method receives the class:

```python
class User:
    @classmethod
    def anonymous(cls):
        return cls("anonymous")
```

Call:

```python
User.anonymous()
```

Python passes `User` as `cls`.

A static method receives neither automatically:

```python
class User:
    @staticmethod
    def normalize_email(email):
        return email.strip().lower()
```

Call:

```python
User.normalize_email(" MAYA@EXAMPLE.COM ")
```

Python passes only the explicit argument.

The difference:

```text
instance method -> gets self
class method    -> gets cls
static method   -> gets neither
```

This chapter is largely about choosing the right binding mode.

---

# Start with Plain Attributes

Python does not require getters and setters for every field.

This is good Python:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Usage:

```python
user = User("Maya")
print(user.name)
user.name = "Asha"
```

This is simple.

It is readable.

It does not hide unnecessary machinery.

In some languages, programmers create methods for every field:

```python
user.get_name()
user.set_name("Asha")
```

That style is usually not Pythonic when the access is simple.

Python's default preference is:

```text
use public attributes for simple data
add properties later when access needs behavior
```

This is possible because properties let you turn a plain attribute into a managed attribute without changing the external API.

That is one of their greatest strengths.

---

# The Problem Properties Solve

Suppose version one of a class is simple:

```python
class Product:
    def __init__(self, price):
        self.price = price
```

Users write:

```python
product = Product(100)
product.price = 120
print(product.price)
```

Later, you need validation.

The price cannot be negative.

Without properties, you might create methods:

```python
class Product:
    def __init__(self, price):
        self.set_price(price)

    def get_price(self):
        return self._price

    def set_price(self, price):
        if price < 0:
            raise ValueError("price cannot be negative")
        self._price = price
```

But now users must change code:

```python
product.set_price(120)
print(product.get_price())
```

Properties avoid this.

You can keep the public attribute syntax:

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

Usage stays:

```python
product.price = 120
print(product.price)
```

But access is now managed.

This is the Pythonic attribute story:

```text
begin with simple public attributes
upgrade to properties when behavior is needed
keep the external interface stable
```

---

# What a Property Is

A property is an object that implements the descriptor protocol.

When you write:

```python
class Product:
    @property
    def price(self):
        return self._price
```

Python creates a `property` object and stores it on the class under the name `price`.

Conceptually:

```python
Product.__dict__["price"]
```

is a property object.

When you write:

```python
product.price
```

the property object's `__get__` behavior calls the getter function.

When you write:

```python
product.price = 100
```

the property object's `__set__` behavior calls the setter function if one exists.

When you write:

```python
del product.price
```

the property object's delete behavior calls the deleter function if one exists.

You do not usually need to think about `__get__` and `__set__` while using properties.

But after Chapter 54, you can see what is happening:

```text
property is a descriptor designed for managed attributes
```

---

# A Read-Only Property

The simplest property has only a getter.

Example:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2
```

Usage:

```python
circle = Circle(10)
print(circle.area)
```

`area` looks like an attribute.

But it is computed when accessed.

This is appropriate because area is conceptually a property of the circle.

Users should not need to call:

```python
circle.area()
```

unless area computation is expensive, side-effectful, or parameterized.

Trying to assign to `area` fails:

```python
circle.area = 100
```

because the property has no setter.

This is a read-only managed attribute.

---

# Computed Attribute or Method?

Should this be a property?

```python
circle.area
```

or a method?

```python
circle.area()
```

Use a property when:

* the value conceptually belongs to the object
* access is cheap or reasonably expected
* no arguments are needed
* access has no visible side effects
* the result reads like data

Use a method when:

* the operation is expensive
* arguments are needed
* the operation has side effects
* the name is a command
* the result may surprise users as attribute access

Good property:

```python
rectangle.area
```

Good method:

```python
report.generate_pdf()
```

Questionable property:

```python
user.profile_from_remote_server
```

if accessing it performs a network call.

Attribute access should feel lightweight.

Properties can run code, but they should not make simple-looking access dangerously expensive or surprising.

---

# A Property with Validation

Properties often validate assignment.

Example:

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("temperature cannot be below absolute zero")
        self._celsius = value
```

Now:

```python
temperature = Temperature(25)
temperature.celsius = 30
```

works.

But:

```python
temperature.celsius = -500
```

raises:

```python
ValueError
```

The setter protects the object's invariant.

The invariant is:

```text
celsius is not below absolute zero
```

This is a strong use of a property.

The class allows normal attribute syntax while preventing invalid state.

---

# Use the Public Name in `__init__`

When a property validates assignment, initialize through the public property:

```python
class Product:
    def __init__(self, price):
        self.price = price
```

Do not bypass validation accidentally:

```python
class Product:
    def __init__(self, price):
        self._price = price
```

If `__init__` writes directly to `_price`, invalid objects can be created:

```python
Product(-100)
```

The setter exists to protect the invariant.

Use it:

```python
self.price = price
```

This calls:

```python
Product.price.__set__(self, price)
```

through the descriptor protocol.

The constructor should not sneak around the class's own rules unless there is a deliberate reason.

---

# The Backing Attribute

A property usually stores data in a backing attribute.

Common convention:

```python
public name: price
backing name: _price
```

Example:

```python
class Product:
    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        self._price = value
```

The leading underscore communicates:

```text
internal implementation detail
```

It does not make the attribute truly private.

Python trusts programmers.

Users can still write:

```python
product._price = -100
```

But they should understand they are bypassing the public interface.

The property provides the intended path.

---

# Avoid Recursive Properties

This property is wrong:

```python
class Product:
    @property
    def price(self):
        return self.price
```

Accessing `self.price` inside the getter calls the getter again.

Then again.

Then again.

Eventually Python raises a recursion error.

This setter is also wrong:

```python
@price.setter
def price(self, value):
    self.price = value
```

It calls itself.

Use the backing attribute:

```python
@property
def price(self):
    return self._price

@price.setter
def price(self, value):
    self._price = value
```

The public property manages access.

The backing attribute stores the value.

---

# Deleters

Properties can define deleters.

Example:

```python
class Session:
    def __init__(self, token):
        self.token = token

    @property
    def token(self):
        return self._token

    @token.setter
    def token(self, value):
        if not value:
            raise ValueError("token cannot be empty")
        self._token = value

    @token.deleter
    def token(self):
        self._token = None
```

Usage:

```python
session = Session("abc")
del session.token
```

Deleters are less common than getters and setters.

Use them when deletion has a meaningful policy.

If deleting an attribute would leave the object in a strange state, do not support deletion.

Just because properties can define deleters does not mean they should.

---

# Properties and Documentation

Properties can have docstrings.

Example:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        """The area of the circle."""
        return 3.14159 * self.radius ** 2
```

Tools such as `help()` can show this documentation.

Because properties appear as attributes, their documentation should describe the value:

Good:

```python
"""The area of the circle."""
```

Weak:

```python
"""Gets the area."""
```

The user sees an attribute-like API.

Document it as an attribute-like concept.

---

# Properties Preserve API Compatibility

This is one of the most important reasons properties exist.

Version one:

```python
class User:
    def __init__(self, email):
        self.email = email
```

Users write:

```python
user.email
user.email = "maya@example.com"
```

Version two adds normalization:

```python
class User:
    def __init__(self, email):
        self.email = email

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, value):
        self._email = value.strip().lower()
```

User code still writes:

```python
user.email
user.email = "MAYA@EXAMPLE.COM"
```

The external interface stayed stable.

The implementation changed.

This is why Python does not require defensive getters and setters from the beginning.

You can start simple and evolve.

---

# Properties in Dataclasses

Dataclasses and properties can work together, but use care.

Simple computed property:

```python
from dataclasses import dataclass


@dataclass
class Rectangle:
    width: float
    height: float

    @property
    def area(self):
        return self.width * self.height
```

This is excellent.

`width` and `height` are stored fields.

`area` is computed.

Avoid storing stale derived values:

```python
@dataclass
class Rectangle:
    width: float
    height: float
    area: float
```

unless `area` is truly independent input.

For validation, dataclasses often use `__post_init__`:

```python
@dataclass
class Product:
    price: float

    def __post_init__(self):
        if self.price < 0:
            raise ValueError("price cannot be negative")
```

Use properties with dataclasses when ongoing assignment must be managed.

Use `__post_init__` when validation is only needed after construction.

---

# Properties and Mutability

A read-only property does not make the returned object immutable.

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    @property
    def members(self):
        return self._members
```

This prevents:

```python
team.members = []
```

because there is no setter.

But it does not prevent:

```python
team.members.append("new member")
```

The returned list is still mutable.

If you want to protect internal state, return an immutable view or copy:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    @property
    def members(self):
        return tuple(self._members)
```

Now callers cannot mutate the internal list through the property.

This connects to the object model from Volume I:

```text
read-only attribute access is not the same as deep immutability
```

---

# Cached Properties

A normal property computes every time:

```python
class Report:
    @property
    def total(self):
        print("computing")
        return sum(self.values)
```

If computation is expensive and the result is stable, a cached property may help.

Python's standard library provides `functools.cached_property`.

Conceptually, it computes once and stores the result.

Example:

```python
from functools import cached_property


class Report:
    def __init__(self, values):
        self.values = values

    @cached_property
    def total(self):
        print("computing")
        return sum(self.values)
```

Usage:

```python
report = Report([1, 2, 3])
print(report.total)
print(report.total)
```

The computation happens once.

But caching has a cost.

If `values` changes, `total` may be stale.

Use cached properties when:

* the value is expensive to compute
* the underlying data is stable
* stale values are acceptable or invalidation is designed

Do not use caching just because it is available.

---

# Property or Cached Property?

Use a normal property when:

```text
the value should always reflect current state
```

Example:

```python
@property
def area(self):
    return self.width * self.height
```

If `width` changes, area should change too.

Use a cached property when:

```text
the value is expensive and effectively stable
```

Example:

```python
@cached_property
def parsed_schema(self):
    return parse_large_schema_file(self.path)
```

If the file changes, you need an invalidation policy.

Caching is a design decision.

It is not just a performance button.

---

# Static Methods

A static method is a function stored on a class that does not receive `self` or `cls`.

Example:

```python
class Email:
    @staticmethod
    def normalize(value):
        return value.strip().lower()
```

Call it on the class:

```python
Email.normalize(" MAYA@EXAMPLE.COM ")
```

Call it on an instance:

```python
email = Email()
email.normalize(" MAYA@EXAMPLE.COM ")
```

In both cases, Python does not pass the instance or class automatically.

The method behaves like a plain function namespaced inside the class.

That is the key:

```text
staticmethod = function in class namespace, no automatic binding
```

---

# When Static Methods Are Useful

Use a static method when:

* the function belongs conceptually with the class
* it does not need instance state
* it does not need class state
* keeping it in the class namespace improves readability

Example:

```python
class Slug:
    @staticmethod
    def normalize(text):
        return text.strip().lower().replace(" ", "-")
```

This can be okay if `Slug` is the only context where normalization makes sense.

Another example:

```python
class PasswordPolicy:
    @staticmethod
    def has_digit(value):
        return any(character.isdigit() for character in value)
```

But be careful.

Many static methods are better as module-level functions.

---

# Static Method or Module Function?

This is often a better design:

```python
def normalize_email(value):
    return value.strip().lower()
```

than:

```python
class User:
    @staticmethod
    def normalize_email(value):
        return value.strip().lower()
```

Why?

If the function does not need the class, putting it inside the class may create unnecessary coupling.

Module-level functions are first-class Python design tools.

They are not second-class citizens.

Use a static method when class namespacing genuinely helps.

Use a module function when the behavior is general.

Ask:

```text
Would this function still make sense if the class disappeared?
```

If yes, a module-level function may be better.

---

# Static Method Versus Instance Method

This is suspicious:

```python
class Product:
    @staticmethod
    def discounted_price(price, discount):
        return price * (1 - discount)
```

If the class already has price and discount state, use an instance method:

```python
class Product:
    def __init__(self, price, discount):
        self.price = price
        self.discount = discount

    def discounted_price(self):
        return self.price * (1 - self.discount)
```

The method uses object state.

It belongs to the instance.

Static methods should not be used to avoid learning instance methods.

If behavior uses `self`, make it an instance method.

If behavior uses `cls`, make it a class method.

If behavior uses neither, consider a static method or module function.

---

# Class Methods

A class method receives the class as its first argument.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name

    @classmethod
    def anonymous(cls):
        return cls("anonymous")
```

Call:

```python
user = User.anonymous()
```

Python passes `User` as `cls`.

The method returns:

```python
User("anonymous")
```

The convention is to name the first parameter:

```python
cls
```

not:

```python
self
```

because it is the class, not an instance.

---

# Class Methods as Alternate Constructors

The most common use of `classmethod` is an alternate constructor.

Example:

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, text):
        year_text, month_text, day_text = text.split("-")
        return cls(
            int(year_text),
            int(month_text),
            int(day_text),
        )
```

Usage:

```python
date = Date.from_string("2026-06-12")
```

The constructor:

```python
Date(...)
```

creates from already parsed parts.

The class method:

```python
Date.from_string(...)
```

creates from a string representation.

The name tells the construction story.

This is much clearer than making `__init__` accept every possible input shape.

---

# Why Use `cls` Instead of the Class Name?

Inside a class method, prefer:

```python
return cls(...)
```

over:

```python
return Date(...)
```

Why?

Inheritance.

Suppose:

```python
class Date:
    @classmethod
    def from_string(cls, text):
        year, month, day = map(int, text.split("-"))
        return cls(year, month, day)


class BusinessDate(Date):
    pass
```

Now:

```python
BusinessDate.from_string("2026-06-12")
```

passes `BusinessDate` as `cls`.

The method returns a `BusinessDate`.

If the method hardcoded `Date(...)`, subclass construction would be broken.

Class methods are inheritance-friendly factory methods when they use `cls`.

---

# Class Methods and Dataclasses

Class methods pair nicely with dataclasses.

Example:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    @classmethod
    def rupees(cls, amount):
        return cls(amount * 100, "INR")

    @classmethod
    def paise(cls, amount):
        return cls(amount, "INR")
```

Usage:

```python
price = Money.rupees(100)
fee = Money.paise(50)
```

The generated dataclass initializer stays simple:

```python
Money(amount, currency)
```

Named constructors provide domain-specific alternatives:

```python
Money.rupees(100)
Money.paise(50)
```

This is often cleaner than overloading `__init__` with flags.

---

# Class Method Versus Static Method

Use `classmethod` when the method needs the class.

Use `staticmethod` when it does not.

Example:

```python
class User:
    def __init__(self, email):
        self.email = email

    @staticmethod
    def normalize_email(email):
        return email.strip().lower()

    @classmethod
    def from_raw_email(cls, email):
        return cls(cls.normalize_email(email))
```

`normalize_email` does not need class or instance state.

`from_raw_email` needs the class so it can construct `cls`.

If a subclass calls:

```python
AdminUser.from_raw_email(...)
```

it can produce an `AdminUser` if the subclass constructor is compatible.

That is the class method advantage.

---

# Class Method Versus Instance Method

An instance method operates on an existing object.

A class method operates on the class.

Example:

```python
class User:
    def __init__(self, email):
        self.email = email

    def domain(self):
        return self.email.split("@")[-1]

    @classmethod
    def from_parts(cls, name, domain):
        return cls(f"{name}@{domain}")
```

`domain` uses an existing user's email.

It needs `self`.

`from_parts` creates a new user.

It needs `cls`.

Do not use class methods for behavior that belongs to a specific instance.

Do not use instance methods for alternate constructors.

The first parameter tells the reader what kind of method it is.

---

# Factory Methods

A factory method is a method that creates objects.

Class methods are often factory methods.

Example:

```python
class Config:
    def __init__(self, host, port, debug=False):
        self.host = host
        self.port = port
        self.debug = debug

    @classmethod
    def from_env(cls, env):
        return cls(
            host=env.get("HOST", "127.0.0.1"),
            port=int(env.get("PORT", "8000")),
            debug=env.get("DEBUG", "false").lower() == "true",
        )
```

Usage:

```python
config = Config.from_env(os.environ)
```

The class constructor remains focused.

The factory method handles parsing.

This separation is healthy:

```text
__init__ receives clean values
factory methods adapt external forms
```

Factory methods are also easy to name:

```python
from_env
from_json
from_file
from_row
from_dict
anonymous
default
empty
```

Good names make construction intent visible.

---

# Avoid Overloaded Constructors

This is tempting:

```python
class User:
    def __init__(self, value, kind="email"):
        if kind == "email":
            ...
        elif kind == "id":
            ...
        elif kind == "dict":
            ...
```

This makes `__init__` harder to understand.

Named class methods are clearer:

```python
class User:
    def __init__(self, id, email):
        self.id = id
        self.email = email

    @classmethod
    def from_id(cls, id, repository):
        data = repository.get_user(id)
        return cls(data["id"], data["email"])

    @classmethod
    def from_dict(cls, data):
        return cls(data["id"], data["email"])
```

Now call sites explain themselves:

```python
User.from_id(10, repository)
User.from_dict(data)
```

This is a major class method use case.

---

# Properties and Class Methods Together

Consider a `Version` class:

```python
class Version:
    def __init__(self, major, minor, patch=0):
        self.major = major
        self.minor = minor
        self.patch = patch

    @property
    def parts(self):
        return (self.major, self.minor, self.patch)

    @classmethod
    def from_string(cls, text):
        parts = [int(part) for part in text.split(".")]
        return cls(*parts)
```

Usage:

```python
version = Version.from_string("1.2.3")
print(version.parts)
```

The property exposes derived object state.

The class method provides alternate construction.

These tools serve different roles.

Good class design uses each where it fits.

---

# `property` as a Decorator

The common style is decorator syntax:

```python
class Product:
    @property
    def price(self):
        return self._price
```

This is equivalent to creating a property object from a function.

Then setters are attached with:

```python
@price.setter
def price(self, value):
    ...
```

The setter function must use the same public name:

```python
price
```

This keeps the property object assigned to the same class attribute.

Bad:

```python
@price.setter
def set_price(self, value):
    ...
```

That creates confusing class namespace behavior.

Use the same name:

```python
@price.setter
def price(self, value):
    ...
```

The repeated name looks strange at first.

It is the normal property pattern.

---

# `property()` Without Decorators

You can also create properties manually.

Example:

```python
class Product:
    def get_price(self):
        return self._price

    def set_price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")
        self._price = value

    price = property(get_price, set_price)
```

This works.

Decorator syntax is usually clearer:

```python
class Product:
    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")
        self._price = value
```

The manual form is useful to understand what decorators are doing.

But most production Python uses decorator syntax.

---

# Descriptor View of the Three Tools

After Chapter 54, we can place these tools in descriptor terms.

`property` is a descriptor that manages attribute access.

It can define behavior for:

```text
get
set
delete
```

`staticmethod` is a descriptor that returns the underlying function without binding `self` or `cls`.

`classmethod` is a descriptor that binds the function to the class.

Normal functions in class bodies are descriptors too.

They bind to instances and become methods.

The binding table:

```text
plain function  -> instance access returns method bound to self
staticmethod    -> access returns function without self or cls
classmethod     -> access returns method bound to cls
property        -> access calls getter/setter/deleter
```

This is why descriptors matter.

They explain everyday Python behavior.

---

# Inheritance with Static Methods

Static methods can be inherited:

```python
class Base:
    @staticmethod
    def normalize(value):
        return value.strip().lower()


class Child(Base):
    pass
```

Now:

```python
Child.normalize(" X ")
```

works.

But static methods do not receive `cls`.

So if the method needs subclass-specific behavior, static method may be wrong.

Example:

```python
class Base:
    suffix = "base"

    @staticmethod
    def label(name):
        return f"{name}:base"
```

If subclasses change `suffix`, `label` will not automatically use it unless it explicitly names the class somehow.

A class method is better:

```python
class Base:
    suffix = "base"

    @classmethod
    def label(cls, name):
        return f"{name}:{cls.suffix}"
```

Now subclasses can override `suffix`.

Use class methods when inheritance matters.

---

# Inheritance with Class Methods

Class methods shine in inherited factories.

Example:

```python
class Shape:
    @classmethod
    def unit(cls):
        return cls(1)


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
```

Call:

```python
circle = Circle.unit()
```

Inside `unit`, `cls` is `Circle`.

So the method returns:

```python
Circle(1)
```

This is inheritance-friendly.

If `unit` used `Shape(1)`, subclass behavior would be lost.

This is why alternate constructors should usually be class methods, not static methods.

---

# Class Attributes and Class Methods

Class methods often work with class attributes.

Example:

```python
class User:
    default_domain = "example.com"

    def __init__(self, email):
        self.email = email

    @classmethod
    def from_username(cls, username):
        return cls(f"{username}@{cls.default_domain}")
```

Subclass:

```python
class InternalUser(User):
    default_domain = "company.local"
```

Now:

```python
InternalUser.from_username("maya")
```

returns an `InternalUser` with:

```python
maya@company.local
```

Because the method used:

```python
cls.default_domain
```

not:

```python
User.default_domain
```

Class methods are often about respecting subclass customization.

---

# Static Methods and Testability

Static methods can be easy to call.

But they can also hide dependency design.

Example:

```python
class Report:
    @staticmethod
    def read_file(path):
        with open(path) as file:
            return file.read()
```

This may be less testable than passing dependencies explicitly.

Often better:

```python
def read_file(path):
    with open(path) as file:
        return file.read()
```

or:

```python
class ReportLoader:
    def __init__(self, filesystem):
        self.filesystem = filesystem

    def load(self, path):
        return self.filesystem.read_text(path)
```

Do not use static methods as a place to hide global behavior.

They are a namespacing tool, not a design cure.

---

# Property Error Messages

Good property setters produce useful errors.

Weak:

```python
raise ValueError("invalid")
```

Better:

```python
raise ValueError("price cannot be negative")
```

Better still when appropriate:

```python
raise ValueError(f"price must be non-negative; got {value!r}")
```

An error message should tell the caller:

* which value failed
* why it failed
* what rule was expected

Example:

```python
@price.setter
def price(self, value):
    if not isinstance(value, int | float):
        raise TypeError(f"price must be numeric; got {type(value).__name__}")
    if value < 0:
        raise ValueError(f"price must be non-negative; got {value!r}")
    self._price = value
```

Properties are often where invalid state is blocked.

Good errors make the class much easier to use.

---

# Property Setters Should Avoid Surprising Coercion

This setter coerces aggressively:

```python
@price.setter
def price(self, value):
    self._price = float(value)
```

Now:

```python
product.price = "10.5"
```

works.

Maybe that is good.

Maybe it hides a bug.

Coercion is a design decision.

Validation says:

```text
I accept values that already have the right shape.
```

Coercion says:

```text
I will try to convert values into the right shape.
```

Both can be valid.

But silent coercion can surprise users.

For internal domain objects, strict validation is often safer.

For boundary parsing, coercion may be appropriate.

Do not mix those responsibilities casually.

---

# Read-Only Does Not Mean Constructor-Only

A read-only property has no public setter.

Example:

```python
class User:
    def __init__(self, id):
        self._id = id

    @property
    def id(self):
        return self._id
```

Users cannot write:

```python
user.id = 2
```

But the class can still change `_id` internally:

```python
self._id = 2
```

If you need real immutability, design for it more deeply:

* avoid internal mutation
* use immutable referenced objects
* consider frozen dataclasses
* avoid exposing mutable internals

Properties define public access behavior.

They are not a complete immutability system.

---

# Common Mistake: Getter and Setter Everything

This is unnecessary:

```python
class User:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = value
```

If there is no validation, computation, compatibility need, or access policy, use a plain attribute:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Properties are useful.

But a property that only returns and assigns a backing attribute adds noise.

Start simple.

Add a property when it earns its place.

---

# Common Mistake: Expensive Properties

This can be surprising:

```python
class User:
    @property
    def orders(self):
        return database.fetch_orders_for_user(self.id)
```

Now:

```python
user.orders
```

looks like simple attribute access.

But it performs a database query.

That may be acceptable in some ORM-style systems if users know the convention.

In ordinary application classes, it is often better as a method:

```python
user.fetch_orders()
```

or:

```python
order_repository.for_user(user)
```

Properties should not hide expensive or unreliable work unless the abstraction clearly promises it.

---

# Common Mistake: Static Method by Habit

This is often a sign of misplaced behavior:

```python
class User:
    @staticmethod
    def validate_email(email):
        ...
```

Ask:

```text
Does this need to be inside User?
```

If the answer is no:

```python
def validate_email(email):
    ...
```

may be better.

Static methods are useful, but they are less common in idiomatic Python than many beginners expect.

Python modules are already namespaces.

You do not need a class just to group functions.

---

# Common Mistake: Static Method Instead of Class Method

This is less flexible:

```python
class User:
    @staticmethod
    def from_email(email):
        return User(email)
```

Subclass:

```python
class AdminUser(User):
    pass
```

Now:

```python
AdminUser.from_email("admin@example.com")
```

returns a `User`, not necessarily an `AdminUser`.

Better:

```python
class User:
    @classmethod
    def from_email(cls, email):
        return cls(email)
```

Now subclasses get a chance to construct themselves.

Factories that construct the class should usually be class methods.

---

# Common Mistake: Class Method Instead of Instance Method

This is awkward:

```python
class User:
    @classmethod
    def domain(cls, user):
        return user.email.split("@")[-1]
```

The method operates on a user instance.

Make it an instance method:

```python
class User:
    def domain(self):
        return self.email.split("@")[-1]
```

Call:

```python
user.domain()
```

The method type should match the data it uses.

If it needs an existing object, use an instance method.

If it needs the class, use a class method.

If it needs neither, use a static method or function.

---

# Common Mistake: Overusing Class Methods as Global Registries

Sometimes class methods mutate class-level registries:

```python
class Plugin:
    registry = {}

    @classmethod
    def register(cls, name, plugin):
        cls.registry[name] = plugin
```

This can be useful.

But class-level mutable state can become hard to test and reason about.

Questions:

* Is the registry global?
* Does it need reset behavior in tests?
* Do subclasses share it or override it?
* Is registration thread-safe?
* Should registration belong in a separate object?

Class methods can manage class state.

That does not mean class state is always a good idea.

Use it deliberately.

---

# A Full Example: Email Address

Let us design a small class.

Requirements:

* store a normalized email
* expose the domain as a property
* provide an alternate constructor from user/domain parts
* provide a helper for normalization

Implementation:

```python
class EmailAddress:
    def __init__(self, value):
        self.value = value

    @staticmethod
    def normalize(value):
        return value.strip().lower()

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, raw):
        normalized = self.normalize(raw)
        if "@" not in normalized:
            raise ValueError("email address must contain @")
        self._value = normalized

    @property
    def domain(self):
        return self.value.split("@", 1)[1]

    @classmethod
    def from_parts(cls, username, domain):
        return cls(f"{username}@{domain}")
```

Usage:

```python
email = EmailAddress.from_parts("Maya", "Example.COM")

print(email.value)
print(email.domain)
```

Output:

```text
maya@example.com
example.com
```

Design notes:

* `value` is managed because assignment needs validation and normalization.
* `domain` is a property because it is derived from the value.
* `from_parts` is a class method because it constructs the class.
* `normalize` is a static method because it does not need `self` or `cls`.

You could also make `normalize` a module-level function.

That would be reasonable too.

---

# A Full Example: Money

Now a money class:

```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency

    @property
    def amount(self):
        return self._amount

    @amount.setter
    def amount(self, value):
        if not isinstance(value, int):
            raise TypeError("amount must be stored as integer minor units")
        self._amount = value

    @property
    def currency(self):
        return self._currency

    @currency.setter
    def currency(self, value):
        normalized = value.strip().upper()
        if len(normalized) != 3:
            raise ValueError("currency must be a 3-letter code")
        self._currency = normalized

    @property
    def major_units(self):
        return self.amount / 100

    @classmethod
    def rupees(cls, amount):
        return cls(amount * 100, "INR")

    @classmethod
    def paise(cls, amount):
        return cls(amount, "INR")
```

Usage:

```python
price = Money.rupees(100)
print(price.amount)
print(price.major_units)
print(price.currency)
```

This example shows:

* properties for validation
* a computed property
* class methods for named constructors

But ask whether a frozen dataclass might be better.

If money objects should be immutable, this class still allows:

```python
price.amount = 200
```

For immutable money, use a frozen dataclass or stronger immutability design.

Properties manage access.

They do not automatically define the entire value-object policy.

---

# A Full Example: Configuration

Configuration often benefits from named constructors.

```python
class ServerConfig:
    def __init__(self, host, port, debug=False):
        self.host = host
        self.port = port
        self.debug = debug

    @property
    def port(self):
        return self._port

    @port.setter
    def port(self, value):
        value = int(value)
        if not 1 <= value <= 65535:
            raise ValueError("port must be between 1 and 65535")
        self._port = value

    @classmethod
    def from_dict(cls, data):
        return cls(
            host=data["host"],
            port=data["port"],
            debug=data.get("debug", False),
        )

    @classmethod
    def local(cls):
        return cls("127.0.0.1", 8000, debug=True)
```

Usage:

```python
config = ServerConfig.from_dict({
    "host": "0.0.0.0",
    "port": "8080",
})
```

The `port` setter validates and converts.

The class methods provide meaningful construction paths.

Depending on the project, you might prefer parsing outside the class and keeping the class stricter.

That is a design choice.

The important part is that each method has a clear responsibility.

---

# A Full Example: Shape Hierarchy

Class methods can support polymorphic construction.

```python
class Shape:
    @classmethod
    def unit(cls):
        return cls(1)
```

Subclasses:

```python
class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2


class Square(Shape):
    def __init__(self, side):
        self.side = side

    @property
    def area(self):
        return self.side ** 2
```

Usage:

```python
circle = Circle.unit()
square = Square.unit()
```

The inherited class method constructs the subclass because it uses `cls`.

The `area` property exposes computed state.

This is a compact example of object-oriented Python:

* inheritance
* class methods
* properties
* polymorphic construction
* computed attributes

---

# Design Checklist for Properties

Before writing a property, ask:

```text
Is a plain public attribute enough?
```

If yes, use the plain attribute.

Ask:

```text
Does access need validation, computation, normalization, compatibility, or protection?
```

If yes, a property may help.

Ask:

```text
Is the property cheap and side-effect free?
```

If no, consider a method.

Ask:

```text
Does the property return mutable internal state?
```

If yes, consider returning a copy or immutable view.

Ask:

```text
Will assignment preserve object invariants?
```

If no, do not provide a setter.

Ask:

```text
Is the logic repeated across many attributes or classes?
```

If yes, consider a descriptor.

---

# Design Checklist for Static Methods

Before writing a static method, ask:

```text
Does this function need self?
```

If yes, use an instance method.

Ask:

```text
Does this function need cls or subclass behavior?
```

If yes, use a class method.

Ask:

```text
Does this function truly belong in the class namespace?
```

If no, use a module-level function.

Ask:

```text
Would subclasses need to customize this?
```

If yes, static method may be too rigid.

Static methods are best when they are small, closely related helpers that benefit from class namespacing.

---

# Design Checklist for Class Methods

Before writing a class method, ask:

```text
Does this method operate on the class rather than an instance?
```

If yes, a class method may fit.

Ask:

```text
Is this an alternate constructor?
```

If yes, a class method is often ideal.

Ask:

```text
Should subclasses receive themselves as cls?
```

If yes, use a class method and return `cls(...)`.

Ask:

```text
Am I using class-level mutable state?
```

If yes, think carefully about testing, inheritance, and lifecycle.

Class methods are powerful because they preserve polymorphism at the class level.

Use them when that power matters.

---

# Practice: Add a Validated Property

Create a `Product` class with a validated `price` property.

Requirements:

* `price` must be numeric
* `price` cannot be negative
* initialization should use the property

Solution:

```python
class Product:
    def __init__(self, price):
        self.price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if not isinstance(value, int | float):
            raise TypeError("price must be numeric")
        if value < 0:
            raise ValueError("price cannot be negative")
        self._price = value
```

Test:

```python
product = Product(100)
assert product.price == 100

product.price = 200
assert product.price == 200
```

Invalid:

```python
Product(-1)
```

should raise `ValueError`.

---

# Practice: Computed Property

Create a `Rectangle` class with:

* `width`
* `height`
* read-only `area`

Solution:

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height
```

Test:

```python
rectangle = Rectangle(10, 20)
assert rectangle.area == 200

rectangle.width = 5
assert rectangle.area == 100
```

This should be a normal property, not a cached property, because area should reflect current width and height.

---

# Practice: Alternate Constructor

Create a `User` class with:

* `email`
* class method `from_username`
* class attribute `default_domain`

Solution:

```python
class User:
    default_domain = "example.com"

    def __init__(self, email):
        self.email = email

    @classmethod
    def from_username(cls, username):
        return cls(f"{username}@{cls.default_domain}")
```

Subclass:

```python
class InternalUser(User):
    default_domain = "company.local"
```

Test:

```python
assert User.from_username("maya").email == "maya@example.com"
assert InternalUser.from_username("maya").email == "maya@company.local"
```

This works because the method uses `cls.default_domain`.

---

# Practice: Static Method or Function?

Decide whether each should be a static method or module-level function:

```text
normalize_email(value)
parse_date(value)
Vector.zero()
Money.rupees(amount)
is_valid_slug(value)
```

Possible answers:

```text
normalize_email(value) -> module function unless tightly tied to EmailAddress
parse_date(value) -> module function or Date class method if constructing Date
Vector.zero() -> class method if it constructs a Vector
Money.rupees(amount) -> class method
is_valid_slug(value) -> module function unless tightly tied to Slug
```

The deeper lesson:

```text
constructing the class -> class method
general helper -> module function
class-specific helper needing no class state -> possible static method
```

---

# Practice: Fix the Wrong Method Type

Original:

```python
class Report:
    @staticmethod
    def from_file(path):
        text = Path(path).read_text()
        return Report(text)
```

Problem:

The method constructs `Report` directly.

Subclasses calling `from_file` will still get `Report`.

Better:

```python
class Report:
    def __init__(self, text):
        self.text = text

    @classmethod
    def from_file(cls, path):
        text = Path(path).read_text()
        return cls(text)
```

Now subclass construction can work:

```python
class MarkdownReport(Report):
    pass


report = MarkdownReport.from_file("README.md")
```

The factory respects the subclass.

---

# Summary

Properties, static methods, and class methods are everyday tools built on Python's descriptor machinery.

A property is a managed attribute.

Use properties for validation, computed attributes, normalization, read-only public access, and backwards-compatible evolution from plain attributes.

Do not use properties for every field.

Start with plain public attributes when access is simple.

Properties should usually be cheap, side-effect free, and conceptually attribute-like.

A static method is a function stored in a class namespace without automatic `self` or `cls` binding.

Use static methods sparingly, when class namespacing genuinely helps.

Many static methods are better as module-level functions.

A class method receives the class as `cls`.

Use class methods for alternate constructors, inheritance-friendly factories, and behavior that operates on the class rather than an instance.

Use `cls(...)` inside class methods when constructing, so subclasses are respected.

The binding rule is:

```text
instance method -> self
class method    -> cls
static method   -> neither
property        -> attribute access through descriptor behavior
```

The design principle is:

```text
choose the form that matches the data the behavior needs
```

If behavior needs instance state, use an instance method.

If behavior needs class state or constructs the class, use a class method.

If behavior needs neither, consider a module function before a static method.

If access should look like an attribute but needs controlled behavior, use a property.

---

# Preview of Chapter 56

Chapter 55 focused on managed attributes and method binding tools that Python programmers use every day.

Next we study `__slots__`.

Slots change how instance attributes are stored.

They connect directly to:

* instance dictionaries
* memory usage
* attribute lookup
* descriptors
* weak references
* inheritance
* dataclasses with `slots=True`

Chapter 56 will answer:

* What does `__slots__` do?
* Why can it reduce memory usage?
* Why can it prevent arbitrary new attributes?
* How does it interact with descriptors?
* Why do slotted objects sometimes lack `__dict__`?
* How do slots interact with inheritance?
* How do slots interact with weak references?
* When should you use slots?
* When should you avoid them?

The transition is:

```text
properties manage attribute access
slots change attribute storage
```

That distinction matters.

Python's object model is not just about what names exist.

It is also about where those names are stored and how lookup finds them.

