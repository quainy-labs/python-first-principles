# Capstone 05 - ORM

## Project Brief

In this project, you will build a small educational ORM: a library that maps Python classes to SQLite tables, maps Python attributes to table columns, creates tables, inserts model objects, retrieves rows as objects, updates records, deletes records, and builds simple queries.

The project is not meant to replace SQLAlchemy, Django ORM, SQLModel, or any production ORM. It is meant to teach how ORMs work under the surface by connecting descriptors, class creation, metadata, SQL generation, type mapping, persistence, and query construction.

By the end, the reader should have a working mini ORM and a much clearer understanding of why mature ORMs are powerful, why they are complex, and how Python's object model makes them possible.

---

# Why This Project Matters

The previous projects used persistence directly.

The REST API and URL Shortener used repository classes that wrote SQL by hand.

That is a good pattern.

It is explicit.

It is testable.

It teaches SQL boundaries.

But many real Python applications use an ORM.

ORM stands for object-relational mapper.

An ORM tries to connect two worlds:

```text
Python objects
relational database tables
```

Python thinks in objects:

```python
user.name
user.email
user.save()
```

Relational databases think in tables, rows, columns, and SQL:

```sql
SELECT id, name, email FROM users WHERE id = ?;
```

An ORM creates a bridge.

This capstone builds a small version of that bridge.

The goal is understanding.

Once you build a tiny ORM, mature ORMs feel less magical.

---

# What You Will Build

You will build a package named `miniorm`.

It will let users define models like this:

```python
from miniorm import Model, IntegerField, TextField, BooleanField


class User(Model):
    id = IntegerField(primary_key=True)
    name = TextField(nullable=False)
    email = TextField(nullable=False, unique=True)
    active = BooleanField(default=True)
```

Then:

```python
db.create_tables([User])

user = User(name="Ada", email="ada@example.com")
db.insert(user)

loaded = db.get(User, id=1)

loaded.name = "Ada Lovelace"
db.update(loaded)

users = db.select(User).where(User.active == True).all()
```

The API is intentionally small.

It should support:

```text
field definitions
model metadata
table creation
insert
get by primary key
update
delete
select all
simple where filters
parameterized SQL
SQLite persistence
tests
```

The point is not to cover every ORM feature.

The point is to expose the core mechanics.

---

# Requirements

The mini ORM must:

* define model classes
* define field objects
* collect model metadata at class creation
* map fields to SQL columns
* create SQLite tables
* insert model instances
* load rows into model instances
* update model instances
* delete model instances
* support simple equality filters
* use parameterized SQL
* test descriptors, metadata, SQL generation, and persistence

It should support field types:

```text
IntegerField
TextField
BooleanField
```

It should support field options:

```text
primary_key
nullable
default
unique
```

It should not hide SQL entirely.

Generated SQL should be inspectable in tests.

---

# Non-Requirements

This capstone will not include:

* joins
* relationships
* migrations
* transactions beyond simple commits
* connection pooling
* lazy loading
* eager loading
* many-to-many relationships
* schema diffing
* indexes beyond primary key and unique
* complex query expressions
* database portability beyond SQLite
* async database access

Those are major ORM features.

They are intentionally excluded.

A tiny ORM becomes confusing if it tries to become a full ORM.

The capstone should teach foundations.

---

# Project Structure

A clean structure:

```text
mini-orm/
    pyproject.toml
    README.md
    src/
        miniorm/
            __init__.py
            fields.py
            model.py
            metadata.py
            database.py
            query.py
            expressions.py
            errors.py
    tests/
        test_fields.py
        test_model_metadata.py
        test_sql_generation.py
        test_database.py
        test_query.py
```

Responsibilities:

`fields.py` defines field classes and descriptors.

`model.py` defines the base `Model`.

`metadata.py` stores table and field metadata.

`database.py` creates tables and performs persistence operations.

`query.py` builds and executes select queries.

`expressions.py` defines simple query expressions.

`errors.py` defines application-specific exceptions.

This project is about boundaries.

An ORM mixes Python object behavior with SQL behavior.

Good structure keeps that mixture understandable.

---

# The Core Idea

An ORM model class contains field objects:

```python
class User(Model):
    id = IntegerField(primary_key=True)
    name = TextField(nullable=False)
```

Those field objects do two jobs.

First, they describe database columns.

Second, they manage attribute access on model instances.

That second job uses descriptors.

When Python sees:

```python
user.name
```

it does not simply look in `user.__dict__` if `name` is a descriptor on the class.

The descriptor can control get and set behavior.

This is why descriptors are central to ORMs.

The field object is both:

```text
column metadata
attribute manager
```

That dual role is the heart of the project.

---

# Field Descriptors

A simple field descriptor:

```python
class Field:
    def __init__(self, *, primary_key=False, nullable=True, default=None, unique=False):
        self.name = None
        self.primary_key = primary_key
        self.nullable = nullable
        self.default = default
        self.unique = unique

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance._values.get(self.name, self.default)

    def __set__(self, instance, value):
        instance._values[self.name] = value
```

`__set_name__` is called when the class is created.

It tells the descriptor which attribute name it was assigned to.

This means:

```python
name = TextField()
```

can learn that its name is `"name"`.

This is one of Python's elegant data-model hooks.

---

# Field Types

Field subclasses define SQL type mapping.

Example:

```python
class IntegerField(Field):
    sql_type = "INTEGER"


class TextField(Field):
    sql_type = "TEXT"


class BooleanField(Field):
    sql_type = "INTEGER"

    def to_database(self, value):
        if value is None:
            return None
        return int(bool(value))

    def from_database(self, value):
        if value is None:
            return None
        return bool(value)
```

SQLite does not have a separate boolean storage class.

Store booleans as integers.

Convert on the boundary.

This reinforces the database mapping lessons from earlier capstones.

---

# Model Metadata

The ORM must know which fields belong to a model.

For `User`, metadata should include:

```text
table name: users or user
fields:
    id -> IntegerField(primary_key=True)
    name -> TextField(nullable=False)
    email -> TextField(nullable=False, unique=True)
primary key:
    id
```

Where does this metadata come from?

From the class body.

When the class is created, inspect its attributes and collect fields.

There are two common ways:

* metaclass
* class decorator or `__init_subclass__`

For this capstone, use `__init_subclass__` first.

It is easier than a full metaclass and still teaches class creation.

---

# `__init_subclass__`

`__init_subclass__` runs when a subclass is created.

Example:

```python
class Model:
    def __init_subclass__(cls):
        super().__init_subclass__()
        fields = {}
        for name, value in cls.__dict__.items():
            if isinstance(value, Field):
                fields[name] = value
        cls.__meta__ = ModelMeta(
            table_name=cls.__name__.lower(),
            fields=fields,
        )
```

Now:

```python
class User(Model):
    id = IntegerField(primary_key=True)
    name = TextField()
```

causes `User.__meta__` to be built automatically.

This is a practical use of Python's class creation hooks.

It connects directly to the chapters on metaclasses and the data model.

---

# Table Names

How should table names be chosen?

Simple default:

```text
class User -> user
class BlogPost -> blogpost
```

Better default:

```text
class User -> users
class BlogPost -> blog_posts
```

For this capstone, keep it simple:

```text
lowercase class name
```

Allow override later:

```python
class User(Model):
    __table__ = "users"
```

Then metadata collection uses:

```python
table_name = getattr(cls, "__table__", cls.__name__.lower())
```

This teaches convention with override.

That pattern appears in many frameworks.

---

# Primary Keys

Each model should have one primary key.

For the first version, require exactly one primary key field.

If no primary key exists, raise:

```python
class MissingPrimaryKey(ORMError):
    pass
```

If more than one exists, raise:

```python
class MultiplePrimaryKeys(ORMError):
    pass
```

This keeps the ORM simple.

Composite primary keys are real, but they add complexity.

Simple rules make the project finishable.

---

# Model Initialization

Model instances should accept keyword arguments:

```python
user = User(name="Ada", email="ada@example.com")
```

Base initializer:

```python
class Model:
    def __init__(self, **values):
        self._values = {}
        for name, field in self.__meta__.fields.items():
            if name in values:
                setattr(self, name, values[name])
            elif field.default is not None:
                setattr(self, name, field.default() if callable(field.default) else field.default)
            else:
                setattr(self, name, None)
```

This applies defaults.

It also ensures every field has an entry.

Unknown keyword arguments should raise an error.

Why?

Because typos should not silently create ignored data.

Example:

```python
User(nmae="Ada")
```

should fail.

---

# Representing Objects

Implement a useful `__repr__`.

Example:

```python
def __repr__(self):
    pk_name = self.__meta__.primary_key.name
    pk_value = getattr(self, pk_name)
    return f"<{type(self).__name__} {pk_name}={pk_value!r}>"
```

This helps debugging.

Professional frameworks pay attention to representation because developers inspect objects constantly.

Good `repr` is not decoration.

It is a debugging tool.

---

# SQL Column Generation

Each field should generate column SQL.

Example:

```python
def column_sql(name: str, field: Field) -> str:
    parts = [name, field.sql_type]
    if field.primary_key:
        parts.append("PRIMARY KEY")
    if not field.nullable:
        parts.append("NOT NULL")
    if field.unique:
        parts.append("UNIQUE")
    return " ".join(parts)
```

For:

```python
id = IntegerField(primary_key=True)
email = TextField(nullable=False, unique=True)
```

Generate:

```sql
id INTEGER PRIMARY KEY
email TEXT NOT NULL UNIQUE
```

Table creation:

```sql
CREATE TABLE IF NOT EXISTS user (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);
```

Tests should assert generated SQL.

SQL generation is logic.

Logic needs tests.

---

# Parameterized SQL

Never build value SQL by string interpolation.

Dangerous:

```python
sql = f"INSERT INTO user (name) VALUES ('{user.name}')"
```

Safe:

```python
sql = "INSERT INTO user (name) VALUES (?)"
cursor.execute(sql, [user.name])
```

The ORM may generate SQL structure.

But values must use parameters.

This distinction matters:

```text
column names and table names are generated from trusted model metadata
runtime values come from users and must be parameters
```

This capstone should reinforce SQL injection safety.

---

# Insert

Insert should:

* collect fields
* skip auto primary key if value is `None`
* convert values to database form
* execute parameterized SQL
* assign generated primary key back to the object

Example:

```python
def insert(self, obj: Model) -> Model:
    meta = obj.__meta__
    columns = []
    values = []

    for name, field in meta.fields.items():
        value = getattr(obj, name)
        if field.primary_key and value is None:
            continue
        columns.append(name)
        values.append(field.to_database(value))

    placeholders = ", ".join("?" for _ in columns)
    sql = f"INSERT INTO {meta.table_name} ({', '.join(columns)}) VALUES ({placeholders})"

    cursor = self.conn.execute(sql, values)
    self.conn.commit()

    pk = meta.primary_key
    if getattr(obj, pk.name) is None:
        setattr(obj, pk.name, cursor.lastrowid)

    return obj
```

The table and column names come from trusted model definitions.

Values are parameters.

---

# Loading Objects

When a row comes back from SQLite, convert it to a model instance.

Example:

```python
def object_from_row(model_class: type[Model], row) -> Model:
    values = {}
    for name, field in model_class.__meta__.fields.items():
        values[name] = field.from_database(row[name])
    return model_class(**values)
```

This is the reverse of insertion.

The ORM boundary is:

```text
database row -> Python object
Python object -> database row
```

That mapping is the reason the ORM exists.

---

# Get by Primary Key

API:

```python
user = db.get(User, 1)
```

Implementation:

```python
def get(self, model_class, pk_value):
    meta = model_class.__meta__
    pk = meta.primary_key
    sql = f"SELECT * FROM {meta.table_name} WHERE {pk.name} = ?"
    row = self.conn.execute(sql, [pk_value]).fetchone()
    if row is None:
        return None
    return object_from_row(model_class, row)
```

Returning `None` is acceptable for a small ORM.

A stricter ORM could raise `DoesNotExist`.

The design choice should be documented.

---

# Update

Update should:

* require primary key value
* update non-primary-key fields
* use parameters
* commit

Example SQL:

```sql
UPDATE user
SET name = ?, email = ?, active = ?
WHERE id = ?;
```

If the primary key is missing, raise:

```python
UnsavedObject
```

Why?

Because the ORM does not know which row to update.

This is a common lifecycle distinction:

```text
transient object -> not stored yet
persisted object -> has database identity
```

The project should teach that distinction.

---

# Delete

Delete should also require primary key value.

Example:

```python
def delete(self, obj: Model) -> None:
    meta = obj.__meta__
    pk = meta.primary_key
    value = getattr(obj, pk.name)
    if value is None:
        raise UnsavedObject("cannot delete object without primary key")
    self.conn.execute(
        f"DELETE FROM {meta.table_name} WHERE {pk.name} = ?",
        [value],
    )
    self.conn.commit()
```

Should delete clear the object's primary key?

For this capstone, no.

After deletion, the object is a Python object representing a row that no longer exists.

That awkwardness is real.

Mature ORMs track object state more carefully.

This capstone should mention the limitation.

---

# Query Expressions

A simple query API:

```python
users = db.select(User).where(User.email == "ada@example.com").all()
```

How can `User.email == "ada@example.com"` produce a query expression instead of a boolean?

Because `User.email` on the class returns the field descriptor itself.

The field can implement `__eq__`:

```python
class Field:
    def __eq__(self, other):
        return EqualExpression(field=self, value=other)
```

This is operator overloading.

It connects directly to the data model chapters.

At runtime:

```python
user.email == "ada@example.com"
```

compares a value.

At class level:

```python
User.email == "ada@example.com"
```

builds a query expression.

That difference is powerful.

It is also subtle.

This is why ORMs are advanced Python software.

---

# Expression Objects

Expression:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class EqualExpression:
    field: Field
    value: object

    def to_sql(self):
        return f"{self.field.name} = ?", [self.field.to_database(self.value)]
```

The expression separates SQL fragment from parameter values.

That keeps queries safe:

```text
SQL shape -> generated by ORM
values -> parameter list
```

Do not return a full interpolated SQL string with values embedded.

---

# Query Object

Query object:

```python
class Query:
    def __init__(self, db, model_class):
        self.db = db
        self.model_class = model_class
        self.expression = None

    def where(self, expression):
        self.expression = expression
        return self

    def all(self):
        meta = self.model_class.__meta__
        sql = f"SELECT * FROM {meta.table_name}"
        params = []
        if self.expression is not None:
            where_sql, where_params = self.expression.to_sql()
            sql += f" WHERE {where_sql}"
            params.extend(where_params)
        rows = self.db.conn.execute(sql, params).fetchall()
        return [object_from_row(self.model_class, row) for row in rows]
```

This is intentionally small.

It supports one `where` expression.

That is enough to teach the concept.

Do not implement a full query planner.

---

# Table Creation

The database object should create tables:

```python
db.create_tables([User, Post])
```

For each model:

```python
def create_table_sql(model_class) -> str:
    meta = model_class.__meta__
    columns = [
        column_sql(name, field)
        for name, field in meta.fields.items()
    ]
    return f"CREATE TABLE IF NOT EXISTS {meta.table_name} ({', '.join(columns)})"
```

This should be tested.

Generated SQL can be brittle if formatting is inconsistent.

Tests may normalize whitespace before comparing.

The point is to verify structure.

---

# Defaults

Defaults should be applied when a model instance is created.

Example:

```python
class User(Model):
    id = IntegerField(primary_key=True)
    active = BooleanField(default=True)
```

Then:

```python
user = User()
user.active
```

should be:

```text
True
```

Callable defaults are useful:

```python
created_at = TextField(default=lambda: datetime.now(timezone.utc).isoformat())
```

If default is callable, call it per instance.

Do not evaluate it once at class definition time.

This reinforces the mutable-default lessons from earlier chapters.

---

# Nullability

If a field is `nullable=False`, `None` should not be accepted.

Descriptor setter can validate:

```python
def __set__(self, instance, value):
    if value is None and not self.nullable and not self.primary_key:
        raise ValidationError(f"{self.name} cannot be None")
    instance._values[self.name] = value
```

Why allow primary key to be `None`?

Because an object may not have been inserted yet.

After insertion, the database assigns the ID.

This is another lifecycle rule.

---

# Type Validation

Should `IntegerField` reject strings?

For this capstone, yes, lightly.

Example:

```python
class IntegerField(Field):
    python_type = int
```

Base validation:

```python
if value is not None and self.python_type is not None and not isinstance(value, self.python_type):
    raise ValidationError(...)
```

Be careful with booleans.

In Python, `bool` is a subclass of `int`.

That means:

```python
isinstance(True, int)
```

is `True`.

This capstone can either accept that or special-case it.

The point is to show that type validation has edge cases.

---

# Model Equality

Should two model instances be equal if their fields match?

For this capstone, implement simple value equality:

```python
def __eq__(self, other):
    if type(self) is not type(other):
        return NotImplemented
    return self._values == other._values
```

This makes tests easier.

But mature ORMs often treat identity differently.

Two objects may represent the same database row but be different Python objects.

Object identity and database identity are related but not identical.

This is a deep ORM topic.

Mention it.

Do not solve it fully.

---

# Error Types

Define errors:

```python
class ORMError(Exception):
    pass


class ModelDefinitionError(ORMError):
    pass


class MissingPrimaryKey(ModelDefinitionError):
    pass


class MultiplePrimaryKeys(ModelDefinitionError):
    pass


class UnknownField(ORMError):
    pass


class ValidationError(ORMError):
    pass


class UnsavedObject(ORMError):
    pass
```

Good errors make framework code easier to understand.

Frameworks fail.

They should fail in ways that teach the user what went wrong.

---

# Testing Strategy

Test layers:

```text
field descriptor behavior
metadata collection
SQL generation
database persistence
query expressions
validation errors
```

Field tests:

* descriptor stores value in instance
* class access returns field
* default value works
* non-null validation works
* type validation works

Metadata tests:

* fields are collected
* table name defaults
* table name override works
* primary key is detected
* missing primary key fails
* multiple primary keys fail

Database tests:

* table creation works
* insert assigns primary key
* get returns object
* update changes row
* delete removes row

Query tests:

* equality expression generates SQL and params
* `where` returns matching rows
* no `where` returns all rows

This project is perfect for tests because most behavior is deterministic.

---

# Milestone 1 - Fields

Build:

* base `Field`
* `IntegerField`
* `TextField`
* `BooleanField`
* descriptor get/set behavior
* defaults
* validation

Completion criteria:

```text
Fields can act as descriptors and manage model instance values.
```

This milestone teaches descriptors.

---

# Milestone 2 - Model Metadata

Build:

* base `Model`
* `__init_subclass__`
* field collection
* table name detection
* primary key validation
* model initializer

Completion criteria:

```text
Model classes automatically collect field metadata at class creation.
```

This milestone teaches class creation hooks.

---

# Milestone 3 - SQL Generation

Build:

* column SQL generation
* create table SQL generation
* insert SQL generation
* update SQL generation
* delete SQL generation

Completion criteria:

```text
The ORM can generate parameterized SQL shapes from model metadata.
```

This milestone teaches SQL abstraction.

---

# Milestone 4 - Persistence

Build:

* database connection wrapper
* create tables
* insert
* get
* update
* delete
* row-to-object conversion

Completion criteria:

```text
Objects can be stored in and loaded from SQLite.
```

This milestone teaches object-relational mapping.

---

# Milestone 5 - Query API

Build:

* equality expressions
* field operator overload
* query object
* `select().where().all()`
* `select().all()`

Completion criteria:

```text
Class-level field expressions can build safe SQL queries.
```

This milestone connects descriptors and operator overloading.

---

# Milestone 6 - Documentation

Build:

* README examples
* model definition example
* table creation example
* insert/get/update/delete examples
* query example
* limitations section

Completion criteria:

```text
A reader can understand what the mini ORM supports and what it intentionally does not support.
```

Frameworks need honest documentation.

---

# Common Mistakes

The first common mistake is trying to build a full ORM.

Do not.

The second common mistake is interpolating values into SQL.

Use parameters.

The third common mistake is using a metaclass before understanding the simpler hook.

Use `__init_subclass__` first.

The fourth common mistake is letting field descriptors store values on themselves.

Field objects live on the class and are shared across instances.

Per-instance values must live on the instance.

The fifth common mistake is ignoring object lifecycle.

Unsaved objects do not have database identity yet.

The sixth common mistake is blurring domain objects, SQL rows, and API schemas.

This project focuses on domain objects and database rows only.

The seventh common mistake is hiding too much SQL.

This is a learning ORM.

Generated SQL should remain understandable.

---

# Extension Ideas

After the core works, add:

* `filter()` with multiple conditions
* `order_by`
* `limit`
* `first`
* `count`
* foreign keys
* one-to-many relationships
* migrations
* transactions
* dirty field tracking
* identity map
* SQL logging
* class decorators instead of `__init_subclass__`
* metaclass implementation

Each extension should be small and tested.

ORM complexity grows quickly.

Respect it.

---

# What This Capstone Teaches

This capstone teaches how Python's object model can build frameworks.

It connects:

```text
descriptors
__set_name__
__init_subclass__
dunder methods
operator overloading
dataclasses
metadata
SQL generation
SQLite
validation
persistence
query objects
tests
```

The central lesson is:

```text
ORMs are built from ordinary Python mechanisms composed carefully
```

Descriptors are not trivia.

Metaclasses and class creation hooks are not trivia.

Dunder methods are not trivia.

They are the machinery that lets Python libraries feel declarative.

---

# Completion Checklist

The capstone is complete when:

```text
Model classes can define fields.
Fields know their attribute names.
Fields manage per-instance values.
Defaults work.
Validation works.
Model metadata is collected automatically.
Exactly one primary key is required.
Table creation SQL is generated.
Tables can be created in SQLite.
Objects can be inserted.
Inserted objects receive primary keys.
Objects can be loaded by primary key.
Objects can be updated.
Objects can be deleted.
Boolean values round trip correctly.
Query expressions use operator overloading.
Queries use parameterized SQL.
Tests cover fields, metadata, SQL generation, persistence, and queries.
README documents limitations.
```

If all of these are true, the reader has built a small but real ORM.

---

# Exercises

1. Create the project structure.

2. Implement the base `Field` descriptor.

3. Implement `IntegerField`, `TextField`, and `BooleanField`.

4. Add descriptor tests.

5. Implement default handling.

6. Add nullability validation.

7. Implement the base `Model`.

8. Use `__init_subclass__` to collect fields.

9. Validate primary key rules.

10. Implement model initialization from keyword arguments.

11. Generate column SQL.

12. Generate create table SQL.

13. Implement SQLite table creation.

14. Implement insert and primary key assignment.

15. Implement get by primary key.

16. Implement update.

17. Implement delete.

18. Implement equality expressions with `Field.__eq__`.

19. Implement `select().where().all()`.

20. Document the ORM's limitations.

---

# Preview of Capstone 06

Capstone 05 built an ORM.

It connected descriptors, class creation hooks, metadata collection, field validation, SQL generation, SQLite persistence, query expressions, operator overloading, and framework design.

Capstone 06 will build a Task Queue.

The Task Queue will introduce background work, job states, retries, scheduling, workers, queues, persistence, idempotency, and operational thinking.

The transition is:

```text
ORM maps Python objects to persisted rows
Task Queue uses persisted state to coordinate background work
```

The next project will move from request-time work to asynchronous job processing.
