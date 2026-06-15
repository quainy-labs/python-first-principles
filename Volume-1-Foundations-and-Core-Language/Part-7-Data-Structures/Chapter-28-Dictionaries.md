# Chapter 28 — Dictionaries

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a dictionary is.
* Why dictionaries exist.
* How key-value mapping differs from sequence indexing.
* How dictionary literals work.
* How to read, add, update, and delete entries.
* Why dictionary keys must be hashable.
* Why immutable keys matter.
* How dictionary lookup works conceptually.
* How insertion order behaves in modern Python.
* How to use `.get()`, `.keys()`, `.values()`, `.items()`, `.update()`, `.pop()`, and `.setdefault()`.
* How to iterate over dictionaries.
* How nested dictionaries model structured data.
* How dictionaries are used for counting, grouping, dispatch, configuration, and records.
* How dictionary aliasing and copying work.
* Basic complexity intuition for dictionary operations.
* Common mistakes with dictionaries.

Dictionaries are one of Python's most important data structures.

They organize values by meaning, not by position.

---

# Concept Overview

A dictionary maps keys to values.

Example:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
    "active": True,
}
```

Conceptually:

```text
dictionary
    key "name"   -> value "Ada"
    key "email"  -> value "ada@example.com"
    key "active" -> value True
```

A list uses indexes:

```python
user = ["Ada", "ada@example.com", True]

print(user[0])
```

A dictionary uses keys:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
    "active": True,
}

print(user["name"])
```

Output:

```text
Ada
```

The dictionary version is more meaningful.

`user["name"]` explains what is being accessed.

`user[0]` requires the reader to remember what position `0` means.

---

# Why Dictionaries Exist

Programs constantly need relationships.

Examples:

```text
username -> user object
email -> account
setting name -> setting value
product id -> product
word -> frequency count
route path -> handler function
environment name -> configuration
country code -> country name
```

A dictionary represents these relationships directly:

```text
key -> value
```

This is called a mapping.

Without dictionaries, many programs would rely on parallel lists:

```python
names = ["Ada", "Linus", "Grace"]
emails = ["ada@example.com", "linus@example.com", "grace@example.com"]
```

To find Ada's email, you would search one list and use the same index in another.

That is fragile.

A dictionary expresses the relationship directly:

```python
emails = {
    "Ada": "ada@example.com",
    "Linus": "linus@example.com",
    "Grace": "grace@example.com",
}
```

Now lookup is meaningful:

```python
print(emails["Ada"])
```

---

# Dictionary Literals

A dictionary literal uses curly braces with key-value pairs.

Syntax:

```python
{
    key: value,
    key: value,
}
```

Example:

```python
settings = {
    "debug": True,
    "theme": "dark",
    "retries": 3,
}
```

Keys and values are expressions.

Example:

```python
key = "language"
value = "Python"

data = {
    key: value,
    "length": len(value),
}

print(data)
```

Output:

```text
{'language': 'Python', 'length': 6}
```

Python evaluates the key expressions and value expressions, then creates the dictionary.

---

# Empty Dictionaries

An empty dictionary is written:

```python
data = {}
```

This is a dictionary, not a set.

Check:

```python
print(type({}))
```

Output:

```text
<class 'dict'>
```

An empty set uses:

```python
set()
```

Sets are covered in the next chapter.

Empty dictionaries are common when building mappings:

```python
counts = {}
```

Then entries are added as data is processed.

---

# Creating Dictionaries With `dict()`

You can create dictionaries with `dict()`.

Example:

```python
user = dict(name="Ada", active=True)

print(user)
```

Output:

```text
{'name': 'Ada', 'active': True}
```

This form requires keyword-style keys, so the keys must be valid identifier-like names.

You can also create a dictionary from pairs:

```python
pairs = [("name", "Ada"), ("active", True)]
user = dict(pairs)

print(user)
```

Output:

```text
{'name': 'Ada', 'active': True}
```

This works because each pair provides:

```text
key, value
```

---

# Accessing Values By Key

Use square brackets to get a value by key.

Example:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
}

print(user["name"])
```

Output:

```text
Ada
```

The key must exist.

If it does not:

```python
print(user["age"])
```

Python raises:

```text
KeyError
```

This is different from list indexing, which raises `IndexError` for invalid positions.

Dictionary lookup raises `KeyError` for missing keys.

---

# Adding Entries

Assigning to a new key adds an entry.

Example:

```python
user = {
    "name": "Ada",
}

user["email"] = "ada@example.com"

print(user)
```

Output:

```text
{'name': 'Ada', 'email': 'ada@example.com'}
```

The dictionary is mutable.

Adding an entry changes the dictionary object in place.

Conceptually:

```text
before:
"name" -> "Ada"

after:
"name"  -> "Ada"
"email" -> "ada@example.com"
```

---

# Updating Entries

Assigning to an existing key replaces its value.

Example:

```python
settings = {
    "theme": "light",
}

settings["theme"] = "dark"

print(settings)
```

Output:

```text
{'theme': 'dark'}
```

The key is the same.

The value reference is replaced.

This is similar to list index assignment:

```python
items[0] = new_value
```

but dictionaries use keys instead of positions:

```python
mapping[key] = new_value
```

---

# Deleting Entries

Use `del` to remove an entry by key.

Example:

```python
settings = {
    "debug": True,
    "theme": "dark",
}

del settings["debug"]

print(settings)
```

Output:

```text
{'theme': 'dark'}
```

If the key does not exist, `del` raises `KeyError`.

Safe pattern:

```python
if "debug" in settings:
    del settings["debug"]
```

Another option is `.pop()`, covered later.

---

# Keys Must Be Unique

A dictionary cannot have duplicate keys.

Example:

```python
data = {
    "name": "Ada",
    "name": "Grace",
}

print(data)
```

Output:

```text
{'name': 'Grace'}
```

The later value replaced the earlier value.

There is only one `"name"` key.

This is true whether the duplicate occurs in a literal or through assignment:

```python
data = {"name": "Ada"}
data["name"] = "Grace"
```

The key maps to one current value.

---

# Keys Can Have Different Types

Dictionary keys can be different hashable types.

Example:

```python
data = {
    "name": "Ada",
    1: "one",
    (10, 20): "point",
}

print(data["name"])
print(data[1])
print(data[(10, 20)])
```

Output:

```text
Ada
one
point
```

This is legal.

But mixed key types can make dictionaries harder to understand.

Most dictionaries use consistent key types:

```text
str -> configuration value
int -> object
tuple -> compound key
```

Consistency makes code easier to reason about.

---

# Values Can Be Any Object

Dictionary values can be any objects.

Example:

```python
data = {
    "name": "Ada",
    "scores": [90, 95, 100],
    "metadata": {"active": True},
    "callback": print,
}
```

Values do not need to be hashable.

Only keys need to be hashable.

This is because values are retrieved by keys.

The dictionary must be able to locate keys efficiently.

It does not need to locate values efficiently.

---

# Hashability

Dictionary keys must be hashable.

Hashability means an object can produce a stable hash value for use in hash-based collections.

Examples of commonly hashable objects:

```python
str
int
float
bool
None
tuple containing only hashable elements
```

Examples of unhashable objects:

```python
list
dict
set
```

This works:

```python
data = {
    ("x", "y"): 10,
}
```

This fails:

```python
data = {
    ["x", "y"]: 10,
}
```

Python raises:

```text
TypeError
```

A list is mutable.

If a list could be a key and then changed, the dictionary might not be able to find it again.

---

# Why Immutable Keys Matter

Dictionary lookup depends on the key staying stable.

Imagine this were allowed:

```python
key = ["user", 1]
data = {key: "Ada"}
key.append("changed")
```

Now the key's contents changed after insertion.

How should the dictionary find the entry?

Hash-based lookup assumes the key's hash does not change while it is in the dictionary.

That is why mutable built-in containers are unhashable.

Tuples can be keys only when all contained values are hashable:

```python
valid = ("user", 1)
invalid = ("user", [])
```

The first can be a key.

The second cannot.

---

# Conceptual Lookup Model

At a high level, dictionary lookup works like this:

```text
key
 |
 v
hash(key)
 |
 v
candidate location
 |
 v
check equality with stored key
 |
 v
return associated value
```

This is a simplified model.

The real implementation has more details.

For now, the important ideas are:

* Hashing helps find where a key should be.
* Equality confirms the matching key.
* Lookup usually does not scan every entry.

This is why dictionary lookup is usually much faster than searching a list for a matching key.

---

# Membership Checks

The `in` operator checks keys.

Example:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
}

print("name" in user)
print("Ada" in user)
```

Output:

```text
True
False
```

`"Ada"` is a value, not a key.

If you want to check values:

```python
print("Ada" in user.values())
```

But value membership scans values.

Key membership is the normal fast dictionary operation.

---

# `.get()`

`.get()` retrieves a value without raising `KeyError` for missing keys.

Example:

```python
user = {
    "name": "Ada",
}

print(user.get("name"))
print(user.get("email"))
```

Output:

```text
Ada
None
```

You can provide a default:

```python
print(user.get("email", "unknown"))
```

Output:

```text
unknown
```

Use square brackets when a missing key is an error.

Use `.get()` when a missing key is expected or acceptable.

---

# `.keys()`, `.values()`, And `.items()`

Dictionaries provide views.

Example:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
}

print(user.keys())
print(user.values())
print(user.items())
```

Output resembles:

```text
dict_keys(['name', 'email'])
dict_values(['Ada', 'ada@example.com'])
dict_items([('name', 'Ada'), ('email', 'ada@example.com')])
```

These are view objects, not lists.

They reflect the dictionary's current contents.

Example:

```python
keys = user.keys()
user["active"] = True

print(keys)
```

The view shows the new key.

If you need a list snapshot:

```python
key_list = list(user.keys())
```

---

# Iterating Over Dictionaries

Iterating over a dictionary gives keys.

Example:

```python
settings = {
    "debug": True,
    "theme": "dark",
}

for key in settings:
    print(key)
```

Output:

```text
debug
theme
```

To iterate over values:

```python
for value in settings.values():
    print(value)
```

To iterate over key-value pairs:

```python
for key, value in settings.items():
    print(key, value)
```

This is one of the most common dictionary patterns.

---

# Insertion Order

Modern Python dictionaries preserve insertion order.

Example:

```python
data = {}
data["first"] = 1
data["second"] = 2
data["third"] = 3

print(list(data))
```

Output:

```text
['first', 'second', 'third']
```

This means iteration follows insertion order.

Updating an existing key does not move it:

```python
data["second"] = 99
print(list(data))
```

Output:

```text
['first', 'second', 'third']
```

Do not confuse insertion order with sorted order.

If you need sorted keys:

```python
for key in sorted(data):
    print(key, data[key])
```

---

# `.update()`

`.update()` adds or replaces entries from another mapping or iterable of pairs.

Example:

```python
settings = {
    "debug": False,
    "theme": "light",
}

settings.update({
    "debug": True,
    "retries": 3,
})

print(settings)
```

Output:

```text
{'debug': True, 'theme': 'light', 'retries': 3}
```

Existing keys are updated.

New keys are added.

`.update()` mutates the dictionary and returns `None`.

---

# Dictionary Merge Operators

Python supports dictionary merge operators.

Example:

```python
defaults = {
    "debug": False,
    "retries": 3,
}

overrides = {
    "debug": True,
}

settings = defaults | overrides

print(settings)
```

Output:

```text
{'debug': True, 'retries': 3}
```

`|` creates a new dictionary.

`|=` updates in place:

```python
settings = defaults.copy()
settings |= overrides
```

When keys overlap, the right side wins.

This is useful for configuration layering.

---

# `.pop()`

`.pop()` removes a key and returns its value.

Example:

```python
settings = {
    "debug": True,
    "theme": "dark",
}

debug = settings.pop("debug")

print(debug)
print(settings)
```

Output:

```text
True
{'theme': 'dark'}
```

If the key is missing, `.pop()` raises `KeyError`.

You can provide a default:

```python
value = settings.pop("missing", None)
```

This avoids an exception.

Use `.pop()` when removal and retrieval belong together.

---

# `.setdefault()`

`.setdefault()` gets a value if the key exists, otherwise inserts a default value and returns it.

Example:

```python
groups = {}

groups.setdefault("admin", []).append("Ada")
groups.setdefault("admin", []).append("Grace")

print(groups)
```

Output:

```text
{'admin': ['Ada', 'Grace']}
```

This works, but it can be dense for beginners.

A clearer version:

```python
groups = {}

if "admin" not in groups:
    groups["admin"] = []

groups["admin"].append("Ada")
```

For grouping, `collections.defaultdict` is often better.

That comes later.

---

# Counting With Dictionaries

Dictionaries are useful for counting.

Example:

```python
text = "banana"
counts = {}

for character in text:
    if character not in counts:
        counts[character] = 0

    counts[character] += 1

print(counts)
```

Output:

```text
{'b': 1, 'a': 3, 'n': 2}
```

Using `.get()`:

```python
counts = {}

for character in text:
    counts[character] = counts.get(character, 0) + 1
```

This pattern appears constantly.

Later, `collections.Counter` provides a specialized tool for counting.

---

# Grouping With Dictionaries

Dictionaries are useful for grouping records.

Example:

```python
users = [
    {"name": "Ada", "role": "admin"},
    {"name": "Linus", "role": "user"},
    {"name": "Grace", "role": "admin"},
]

groups = {}

for user in users:
    role = user["role"]

    if role not in groups:
        groups[role] = []

    groups[role].append(user)

print(groups)
```

Result conceptually:

```text
admin -> [Ada, Grace]
user  -> [Linus]
```

Grouping is one of the most common real-world dictionary uses.

---

# Dictionary As Record

A dictionary can represent a record.

Example:

```python
user = {
    "id": 1,
    "name": "Ada",
    "email": "ada@example.com",
    "active": True,
}
```

This is more readable than a positional tuple when fields have names.

Compare:

```python
user = (1, "Ada", "ada@example.com", True)
```

Which is clearer?

```python
user["email"]
```

or:

```python
user[2]
```

Dictionaries are excellent for dynamic or JSON-like records.

For fixed program-domain records, dataclasses or classes may be better later.

---

# Nested Dictionaries

Dictionaries can contain dictionaries.

Example:

```python
user = {
    "name": "Ada",
    "profile": {
        "timezone": "UTC",
        "language": "Python",
    },
}

print(user["profile"]["language"])
```

Output:

```text
Python
```

Nested dictionaries are common in:

* JSON data.
* API responses.
* configuration files.
* application state.
* structured logs.

But deep nesting can become hard to manage.

Use clear intermediate names:

```python
profile = user["profile"]
language = profile["language"]
```

This is often easier to debug.

---

# Safe Nested Access

Direct nested access can raise `KeyError`.

Example:

```python
language = user["profile"]["language"]
```

This assumes:

* `profile` exists.
* `profile` is a dictionary-like object.
* `language` exists inside it.

Safer:

```python
profile = user.get("profile", {})
language = profile.get("language", "unknown")
```

This is common for optional data.

But avoid hiding data-quality problems.

If a key should always exist, direct access may be better because it fails loudly.

---

# Dictionaries And Functions

Passing a dictionary to a function passes a reference to the dictionary object.

Example:

```python
def activate(user):
    user["active"] = True

account = {"name": "Ada", "active": False}
activate(account)

print(account)
```

Output:

```text
{'name': 'Ada', 'active': True}
```

The function mutates the dictionary.

If you want to avoid mutation, return a new dictionary:

```python
def activated(user):
    updated = user.copy()
    updated["active"] = True
    return updated
```

This is the same design decision seen with lists:

```text
mutate input
or return new object
```

Make the choice explicit.

---

# Copying Dictionaries

`.copy()` creates a shallow copy.

Example:

```python
original = {
    "name": "Ada",
    "tags": ["python"],
}

copy = original.copy()

copy["name"] = "Grace"
copy["tags"].append("math")

print(original)
print(copy)
```

Output:

```text
{'name': 'Ada', 'tags': ['python', 'math']}
{'name': 'Grace', 'tags': ['python', 'math']}
```

The outer dictionary was copied.

The nested list was shared.

This is shallow-copy behavior.

If nested objects need to be copied too, use a deeper copying strategy.

Deep copying is covered later with object lifecycle and memory topics.

---

# Aliasing Dictionaries

Assignment does not copy a dictionary.

Example:

```python
a = {"count": 1}
b = a

b["count"] = 2

print(a)
print(b)
```

Output:

```text
{'count': 2}
{'count': 2}
```

There is one dictionary object.

Both names refer to it:

```text
a ─────┐
       v
b ─▶ dictionary object
```

This is the same aliasing behavior you saw with lists.

---

# Dictionary Comprehensions Preview

Dictionary comprehensions are covered later with comprehension patterns.

Preview:

```python
squares = {number: number * number for number in range(5)}

print(squares)
```

Output:

```text
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

This creates a dictionary from an expression and a loop.

For now, understand the shape:

```python
{key_expression: value_expression for item in iterable}
```

Detailed comprehension rules come later.

---

# Dispatch Dictionaries

A dispatch dictionary maps commands to functions.

Example:

```python
def start():
    print("starting")

def stop():
    print("stopping")

def restart():
    print("restarting")

commands = {
    "start": start,
    "stop": stop,
    "restart": restart,
}

command = "start"
handler = commands[command]
handler()
```

Output:

```text
starting
```

This works because functions are objects.

The dictionary maps strings to function objects.

Dispatch dictionaries are useful when:

* There are many commands.
* Commands are data-driven.
* Behavior should be configurable.

For a few simple branches, `if` / `elif` may be clearer.

---

# Dictionaries And Namespaces

Python uses dictionaries heavily internally.

At a conceptual level, namespaces are mappings:

```text
name -> object
```

Module globals are stored in a namespace.

Object attributes are often stored in dictionaries.

Class namespaces involve dictionaries.

This is not accidental.

Dictionaries are Python's general-purpose mapping structure.

Later chapters on modules, classes, descriptors, and internals will build on this idea.

---

# Complexity Intuition

Common dictionary operation intuition:

| Operation | Intuition |
|---|---|
| `data[key]` | Usually fast key lookup |
| `data[key] = value` | Usually fast insert/update |
| `key in data` | Usually fast key membership |
| `del data[key]` | Usually fast deletion |
| `for key in data` | Visits every key |
| `value in data.values()` | Scans values |
| `data.copy()` | Copies outer dictionary |

Dictionaries are optimized for key lookup.

They are not optimized for finding a value without a key.

If your program frequently asks:

```text
which key has this value?
```

you may need a second dictionary in the reverse direction.

---

# Choosing Keys

Good dictionary keys are:

* Stable.
* Hashable.
* Meaningful.
* Consistent in type.

Good:

```python
users_by_id = {
    1: user1,
    2: user2,
}
```

Good:

```python
cache = {
    ("GET", "/users"): response,
}
```

Risky:

```python
data = {
    True: "yes",
    1: "one",
}
```

Why risky?

In Python:

```python
True == 1
```

and they have related hashing behavior.

Keys that compare equal are treated as the same key.

Avoid clever mixed-key designs.

---

# Common Mistakes

## Misconception 1

### Dictionaries are indexed by position.

Dictionaries are accessed by key.

Modern dictionaries preserve insertion order, but that does not make them positional sequences.

## Misconception 2

### `in` checks dictionary values.

`in` checks keys.

Use `.values()` if you specifically need value membership.

## Misconception 3

### Missing keys return `None`.

Square-bracket lookup raises `KeyError`.

`.get()` returns `None` by default for missing keys.

## Misconception 4

### Lists can be dictionary keys.

Lists are mutable and unhashable.

Use tuples containing hashable elements for fixed compound keys.

## Misconception 5

### `.copy()` deeply copies nested objects.

Dictionary `.copy()` is shallow.

Nested mutable objects are shared.

## Misconception 6

### Updating a key creates a duplicate key.

Dictionaries cannot contain duplicate keys.

Updating a key replaces its value.

## Misconception 7

### Dictionary order is sorted order.

Dictionaries preserve insertion order.

They do not automatically sort keys.

## Misconception 8

### `.setdefault()` is always the clearest grouping tool.

It can be useful, but it can also be dense.

Clear `if key not in dict` logic or `defaultdict` may be more readable.

---

# Real-world Usage

## Configuration

```python
settings = {
    "debug": False,
    "database_url": "sqlite:///app.db",
    "retries": 3,
}
```

## API Responses

```python
response = {
    "status": "ok",
    "data": {
        "id": 1,
        "name": "Ada",
    },
}
```

## Counting

```python
counts = {}

for word in words:
    counts[word] = counts.get(word, 0) + 1
```

## Grouping

```python
groups = {}

for user in users:
    groups.setdefault(user.role, []).append(user)
```

## Lookup Tables

```python
status_messages = {
    200: "OK",
    404: "Not Found",
    500: "Server Error",
}
```

## Dispatch

```python
handlers = {
    "create": create_item,
    "delete": delete_item,
}
```

## Caching

```python
cache = {}

def get_user(user_id):
    if user_id not in cache:
        cache[user_id] = load_user(user_id)

    return cache[user_id]
```

---

# Internal Mechanics

At a high level, dictionaries are hash tables.

Conceptual model:

```text
key -> hash -> lookup location -> stored key/value pair
```

The dictionary stores key-value pairs.

When you look up:

```python
data[key]
```

Python uses the key's hash to find a likely location.

Then it checks equality to confirm the key.

This is why keys need stable hash behavior.

It is also why dictionary key lookup is usually efficient.

The exact CPython implementation is more advanced than this simplified model.

Later internals chapters can discuss the details.

For now:

```text
dict = mutable mapping from hashable keys to object references
```

---

# Concept Connections

Dictionaries connect to earlier chapters:

* Objects: dictionaries are objects.
* Names and references: dictionaries store references to keys and values.
* Mutability: dictionaries can be changed in place.
* Identity and equality: key equality affects lookup.
* Strings: string keys are common for records and JSON-like data.
* Booleans: membership checks and `.get()` results are used in conditions.
* Operators: `in`, `|`, and `|=` interact with dictionaries.
* Tuples: tuples can be compound dictionary keys when hashable.
* Lists: dictionaries often contain lists for grouping.
* Functions: dictionaries are passed by reference and can store functions.
* Functional programming: dispatch dictionaries map data to behavior.

Dictionaries prepare you for:

* Sets.
* Hashability.
* JSON-style data.
* Modules and namespaces.
* Object attributes.
* Classes.
* Caching.
* Indexes and lookup tables.

---

# Active Recall

## Easy Recall Questions

1. What is a dictionary?
2. What is a key?
3. What is a value?
4. What happens when you access a missing key with square brackets?
5. What does `.get()` do?
6. What does `in` check for a dictionary?
7. Can a list be a dictionary key?
8. What does `.items()` provide?
9. Does dictionary order mean sorted order?
10. What does `.update()` do?

## Deep Understanding Questions

1. Why must dictionary keys be hashable?
2. Why are mutable built-in containers unhashable?
3. Why does `.copy()` not protect nested mutable values?
4. Why is `user["email"]` often clearer than `user[2]`?
5. Why is dictionary lookup usually faster than scanning a list?
6. Why can dispatch dictionaries store functions?
7. When should missing keys raise errors instead of being hidden with `.get()`?

## Predict-the-Output Questions

### Question 1

```python
data = {"name": "Ada"}
data["name"] = "Grace"
print(data)
```

### Question 2

```python
data = {"name": "Ada"}
print("name" in data)
print("Ada" in data)
```

### Question 3

```python
data = {"count": 1}
alias = data
alias["count"] = 2

print(data)
```

### Question 4

```python
original = {"items": []}
copy = original.copy()
copy["items"].append("x")

print(original)
print(copy)
```

### Question 5

```python
data = {
    "a": 1,
    "a": 2,
}

print(data)
```

### Question 6

```python
data = {}
data["first"] = 1
data["second"] = 2
data["first"] = 99

print(list(data))
print(data["first"])
```

---

# Practical Exercises

## Exercise 1

Create a dictionary representing a user with:

* `id`
* `name`
* `email`
* `active`

Access each value by key.

## Exercise 2

Write a function that receives a user dictionary and returns the email.

Then decide whether missing email should raise `KeyError` or return a default.

Explain your choice.

## Exercise 3

Count character frequencies in a string using a dictionary.

Then rewrite it using `.get()`.

## Exercise 4

Group a list of users by role.

Use a dictionary where each key is a role and each value is a list of users.

## Exercise 5

Create a dictionary with tuple keys representing grid coordinates.

Example:

```python
grid[(0, 0)] = "start"
```

## Exercise 6

Create a dispatch dictionary with at least four commands.

Each command should map to a function.

## Exercise 7

Create a nested dictionary representing an API response.

Access a deeply nested value using intermediate names.

## Exercise 8

Demonstrate shallow copying with a dictionary containing a list.

Show which mutation affects both dictionaries.

## Exercise 9

Use `.update()` or `|` to combine default settings with environment-specific overrides.

Explain which values win when keys overlap.

## Exercise 10

Choose whether each should be represented by a list, tuple, or dictionary:

* A user profile with named fields.
* A 2D point.
* A queue of tasks.
* A mapping from product id to product details.
* A sequence of log lines.
* A count of words in a document.

Explain your choices.

---

# Summary

In this chapter we learned:

* Dictionaries map keys to values.
* Dictionaries organize data by meaning, not by position.
* Dictionary literals use key-value pairs.
* Square-bracket lookup raises `KeyError` for missing keys.
* `.get()` allows missing-key defaults.
* Assigning to a new key adds an entry.
* Assigning to an existing key updates the value.
* `del` removes an entry by key.
* Keys must be unique.
* Keys must be hashable.
* Values can be any objects.
* `in` checks keys.
* `.keys()`, `.values()`, and `.items()` return views.
* Iterating over a dictionary gives keys.
* Modern dictionaries preserve insertion order.
* `.update()` mutates a dictionary with new or replacement entries.
* `|` creates a merged dictionary.
* `.pop()` removes and returns a value.
* `.setdefault()` can help with grouping.
* Dictionaries are useful for records, counting, grouping, configuration, dispatch, caching, and nested data.
* Dictionary copying is shallow.
* Dictionary lookup is usually efficient because dictionaries are hash-table-based.

Core model:

```text
dictionary object
    |
    ├── key ─▶ value
    ├── key ─▶ value
    └── key ─▶ value
```

Dictionaries are mutable mappings from hashable keys to object references.

---

# Preview of Chapter 29

Next we study sets.

Dictionaries map keys to values.

Sets store unique hashable values.

The connection is direct:

```text
dict -> hashable key maps to value
set  -> hashable value exists or does not exist
```

We will study:

* What sets are.
* Why uniqueness matters.
* Set literals.
* Adding and removing values.
* Membership checks.
* Hashability.
* Set operations such as union, intersection, and difference.
* How sets compare to lists and dictionaries.
* Real-world use cases for deduplication, membership, and relationship logic.

Sets are the next step because they use the same hashability ideas as dictionaries, but focus on membership and uniqueness instead of key-value mapping.
