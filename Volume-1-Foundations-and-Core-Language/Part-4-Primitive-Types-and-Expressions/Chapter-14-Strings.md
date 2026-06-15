# Chapter 14 — Strings

---

# Learning Objectives

By the end of this chapter, you should understand:

* What strings are in Python.
* Why strings are objects.
* Why strings are immutable.
* How string literals work.
* How quotes, escape sequences, and triple-quoted strings work.
* How indexing accesses individual characters.
* How slicing creates new strings.
* How common string methods behave.
* Why string methods return new strings instead of modifying the original.
* How string concatenation and repetition work.
* How string formatting works with f-strings.
* What Unicode is at a practical level.
* Why text and bytes are different.
* How encoding and decoding connect text to bytes.
* Common mistakes with strings, identity, mutation, and formatting.

This chapter continues Part IV of Volume I by studying text as Python objects.

The core model remains:

```text
expression -> object -> operations depend on type
```

For strings:

```text
string expression -> str object -> text behavior
```

---

# Concept Overview

A string is a sequence of Unicode characters.

Examples:

```python
"hello"
"Python"
"नमस्ते"
"こんにちは"
"🙂"
```

In Python, strings have type `str`.

Example:

```python
message = "hello"

print(type(message))
```

Output:

```text
<class 'str'>
```

Strings are objects.

They have:

```text
identity
type
value
behavior
```

Conceptually:

```text
str object
├── identity: specific runtime object
├── type: str
└── value: sequence of Unicode characters
```

Strings are used for:

* Names
* Messages
* User input
* File paths
* URLs
* JSON keys
* Log messages
* SQL queries
* Configuration
* Protocol data
* Human-readable output

Text is everywhere.

String correctness matters.

---

# Strings Are Objects

Strings support methods because they are objects.

Example:

```python
text = "python"

print(text.upper())
print(text.capitalize())
print(text.replace("p", "P"))
```

Output:

```text
PYTHON
Python
Python
```

These methods exist because `text` refers to a `str` object.

You can inspect the type:

```python
print(type(text))
```

Output:

```text
<class 'str'>
```

The object model from earlier chapters applies directly.

---

# Why Strings Exist

Programs need to represent text.

Text is not just decorative output.

It is data.

Examples:

```python
username = "ada"
email = "ada@example.com"
path = "/users/ada/report.txt"
status = "approved"
query = "python strings"
```

Computers ultimately store bytes.

Humans read text.

Strings are Python's high-level text representation.

They let programmers work with characters and text operations without manually managing raw byte encodings most of the time.

---

# String Literals

A string literal is text written directly in source code.

Examples:

```python
"hello"
'hello'
```

Both create string objects with the same value:

```python
print("hello" == 'hello')
```

Output:

```text
True
```

Python allows single quotes and double quotes so you can choose the form that avoids escaping.

Example:

```python
message = "It's fine"
```

This is easier than:

```python
message = 'It\'s fine'
```

Example:

```python
quote = 'She said "hello"'
```

This is easier than:

```python
quote = "She said \"hello\""
```

Choose the quote style that makes the string readable.

---

# Triple-Quoted Strings

Triple-quoted strings can span multiple lines.

Example:

```python
text = """Line one
Line two
Line three"""

print(text)
```

Output:

```text
Line one
Line two
Line three
```

Triple quotes can use double quotes:

```python
"""text"""
```

or single quotes:

```python
'''text'''
```

Triple-quoted strings are useful for:

* Multi-line text
* Docstrings
* SQL snippets
* Templates
* Long messages

Be aware that indentation and newlines become part of the string unless handled deliberately.

---

# Escape Sequences

Escape sequences represent characters that are hard to type directly.

Common escape sequences:

| Escape | Meaning |
| --- | --- |
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Backslash |
| `\"` | Double quote |
| `\'` | Single quote |

Example:

```python
text = "Hello\nWorld"
print(text)
```

Output:

```text
Hello
World
```

The source code contains two visible characters:

```text
\n
```

The string value contains one newline character.

---

# Raw Strings

Raw strings reduce escape processing.

Example:

```python
path = r"C:\Users\Ada\Documents"
print(path)
```

Output:

```text
C:\Users\Ada\Documents
```

Without `r`, sequences such as `\U` may be interpreted as escapes.

Raw strings are useful for:

* Windows paths
* Regular expressions
* Text containing many backslashes

Important limitation:

A raw string cannot end with a single trailing backslash.

This is invalid:

```python
r"C:\Users\"
```

The final backslash escapes the closing quote in the source syntax.

---

# Strings Are Sequences

A string is a sequence of characters.

Example:

```python
text = "Python"
```

Conceptually:

```text
index:   0   1   2   3   4   5
char:    P   y   t   h   o   n
```

You can get the length:

```python
print(len(text))
```

Output:

```text
6
```

`len()` returns the number of characters in the string, in Python's text model.

For Unicode text, this is not always the same as user-perceived visual characters.

We will return to that later.

---

# Indexing

Indexing accesses one character by position.

Example:

```python
text = "Python"

print(text[0])
print(text[1])
print(text[5])
```

Output:

```text
P
y
n
```

Indexes start at zero.

This is common in programming because positions are offsets from the beginning.

Mental model:

```text
text[0] -> first character
text[1] -> second character
text[5] -> sixth character
```

---

# Negative Indexing

Negative indexes count from the end.

Example:

```python
text = "Python"

print(text[-1])
print(text[-2])
```

Output:

```text
n
o
```

Mental model:

```text
index:   -6  -5  -4  -3  -2  -1
char:     P   y   t   h   o   n
index:    0   1   2   3   4   5
```

Negative indexing is useful when you care about the end of a sequence.

---

# IndexError

If an index is out of range, Python raises `IndexError`.

Example:

```python
text = "abc"
print(text[3])
```

Valid indexes are:

```text
0, 1, 2
```

Index `3` is past the end.

Python raises:

```text
IndexError
```

This is a runtime error.

The syntax is valid.

The requested position does not exist.

---

# Slicing

Slicing extracts a substring.

Example:

```python
text = "Python"

print(text[0:2])
print(text[2:6])
```

Output:

```text
Py
thon
```

Slice syntax:

```python
text[start:stop]
```

Important rule:

```text
start is included
stop is excluded
```

So:

```python
text[0:2]
```

means:

```text
characters at indexes 0 and 1
```

not index 2.

---

# Why Stop Is Excluded

Exclusive stop indexes make lengths easy.

Example:

```python
text[0:2]
```

Length:

```text
2 - 0 = 2
```

Example:

```python
text[2:6]
```

Length:

```text
6 - 2 = 4
```

Exclusive stop also makes adjacent slices fit cleanly:

```python
text[0:2] + text[2:6]
```

reconstructs:

```python
text[0:6]
```

---

# Omitted Slice Bounds

You can omit start or stop.

Example:

```python
text = "Python"

print(text[:2])
print(text[2:])
print(text[:])
```

Output:

```text
Py
thon
Python
```

Meaning:

```text
text[:2]  -> from beginning to index 2, exclusive
text[2:]  -> from index 2 to end
text[:]   -> entire string
```

Because strings are immutable, `text[:]` creates an equal string value but mutation concerns do not apply like they do for lists.

---

# Slice Step

Slices can include a step.

Syntax:

```python
text[start:stop:step]
```

Example:

```python
text = "abcdef"

print(text[::2])
print(text[1::2])
```

Output:

```text
ace
bdf
```

A negative step can reverse:

```python
print(text[::-1])
```

Output:

```text
fedcba
```

This is a common Python idiom for reversing a string.

---

# Slicing Creates a New String

Strings are immutable.

Slicing does not modify the original string.

Example:

```python
text = "Python"
part = text[:2]

print(text)
print(part)
```

Output:

```text
Python
Py
```

Mental model:

```text
text ─────▶ str object "Python"
part ─────▶ str object "Py"
```

The slice result is a string object.

---

# Strings Are Immutable

You cannot change a string in place.

Example:

```python
text = "Python"
text[0] = "J"
```

Python raises:

```text
TypeError
```

Why?

`str` objects are immutable.

To produce changed text, create a new string.

Example:

```python
text = "Python"
text = "J" + text[1:]

print(text)
```

Output:

```text
Jython
```

This is rebinding.

The old string object was not modified.

---

# String Methods Return New Strings

Because strings are immutable, methods such as `upper`, `lower`, `strip`, and `replace` return new strings.

Example:

```python
text = " python "
cleaned = text.strip()

print(text)
print(cleaned)
```

Output:

```text
 python 
python
```

The original string still contains spaces.

The cleaned string is a new string object.

Common mistake:

```python
text = " python "
text.strip()
print(text)
```

Output:

```text
 python 
```

The result was ignored.

Correct:

```python
text = text.strip()
```

---

# Common String Methods

Useful methods:

| Method | Purpose |
| --- | --- |
| `lower()` | Return lowercase version |
| `upper()` | Return uppercase version |
| `strip()` | Remove leading/trailing whitespace |
| `replace(old, new)` | Return string with replacements |
| `split(sep)` | Split into a list |
| `join(parts)` | Join strings |
| `startswith(prefix)` | Check prefix |
| `endswith(suffix)` | Check suffix |
| `find(sub)` | Find substring index or `-1` |
| `count(sub)` | Count occurrences |

Methods are type-defined behavior.

They exist because `str` objects provide them.

---

# Case Conversion

Examples:

```python
text = "Python"

print(text.lower())
print(text.upper())
print(text.swapcase())
```

Output:

```text
python
PYTHON
pYTHON
```

These methods return new strings.

Case conversion can be more complex with Unicode text.

For case-insensitive comparisons, `casefold()` is often stronger than `lower()`.

Example:

```python
print("Straße".casefold())
```

Output:

```text
strasse
```

Use `casefold()` for robust caseless matching.

---

# Whitespace Stripping

`strip()` removes leading and trailing whitespace.

Example:

```python
text = "  hello\n"
print(text.strip())
```

Output:

```text
hello
```

Related methods:

```python
text.lstrip()
text.rstrip()
```

`strip()` does not remove whitespace in the middle.

Example:

```python
text = "hello   world"
print(text.strip())
```

Output:

```text
hello   world
```

---

# Replacing Text

`replace()` returns a new string.

Example:

```python
text = "red blue red"
new_text = text.replace("red", "green")

print(new_text)
```

Output:

```text
green blue green
```

You can limit replacements:

```python
print(text.replace("red", "green", 1))
```

Output:

```text
green blue red
```

Again, the original string is unchanged.

---

# Splitting Strings

`split()` turns a string into a list of strings.

Example:

```python
line = "red,green,blue"
parts = line.split(",")

print(parts)
```

Output:

```text
['red', 'green', 'blue']
```

Without an argument, `split()` splits on runs of whitespace:

```python
text = "a   b\nc"
print(text.split())
```

Output:

```text
['a', 'b', 'c']
```

The result is a list object.

Strings and lists work together often.

---

# Joining Strings

`join()` combines strings.

Example:

```python
parts = ["red", "green", "blue"]
text = ",".join(parts)

print(text)
```

Output:

```text
red,green,blue
```

The separator is the string on which `join()` is called.

This design can look odd at first:

```python
",".join(parts)
```

Read it as:

```text
use "," as the separator to join parts
```

All items must be strings.

Example:

```python
"-".join([1, 2, 3])
```

raises:

```text
TypeError
```

Convert numbers first:

```python
"-".join(str(number) for number in [1, 2, 3])
```

---

# Searching Strings

Use `in` to check containment:

```python
text = "python programming"

print("python" in text)
print("java" in text)
```

Output:

```text
True
False
```

Use `find()` to get an index:

```python
print(text.find("program"))
print(text.find("java"))
```

Output:

```text
7
-1
```

Use `index()` when absence should be an error:

```python
text.index("java")
```

raises:

```text
ValueError
```

---

# Prefixes and Suffixes

Use `startswith()` and `endswith()`.

Example:

```python
filename = "report.pdf"

print(filename.endswith(".pdf"))
print(filename.startswith("report"))
```

Output:

```text
True
True
```

These methods are clearer than manual slicing for many cases.

They also support tuples:

```python
filename.endswith((".pdf", ".txt", ".md"))
```

This checks multiple possible suffixes.

---

# String Concatenation

The `+` operator concatenates strings.

Example:

```python
first = "Py"
second = "thon"

print(first + second)
```

Output:

```text
Python
```

This creates a new string.

It does not mutate either original string.

For many pieces, prefer `join()`:

```python
parts = ["a", "b", "c"]
text = "".join(parts)
```

Repeated string concatenation in loops can be inefficient because each result is a new string.

---

# String Repetition

The `*` operator repeats strings.

Example:

```python
print("ha" * 3)
```

Output:

```text
hahaha
```

The count must be an integer.

Example:

```python
"ha" * 2.5
```

raises:

```text
TypeError
```

String repetition creates a new string object.

---

# Comparing Strings

Strings compare by value.

Example:

```python
print("abc" == "abc")
print("abc" == "ABC")
```

Output:

```text
True
False
```

String comparison is case-sensitive.

For case-insensitive comparison:

```python
a = "Python"
b = "python"

print(a.casefold() == b.casefold())
```

Output:

```text
True
```

Do not use `is` for string value comparison.

Use `==`.

---

# Lexicographic Ordering

Strings support ordering:

```python
print("apple" < "banana")
```

Output:

```text
True
```

This is lexicographic ordering based on Unicode code points.

It is not the same as locale-aware human dictionary sorting for every language.

For simple English-like text, it often matches intuition.

For internationalized applications, sorting text correctly can require specialized locale-aware tools.

---

# f-Strings

f-strings are the preferred formatting style for many Python cases.

Example:

```python
name = "Ada"
age = 36

message = f"{name} is {age} years old"
print(message)
```

Output:

```text
Ada is 36 years old
```

Inside `{}` you can put expressions:

```python
x = 10
y = 20

print(f"Total: {x + y}")
```

Output:

```text
Total: 30
```

An f-string is evaluated at runtime.

It produces a string object.

---

# Format Specifiers

f-strings support format specifiers.

Example:

```python
price = 19.99
print(f"Price: ${price:.2f}")
```

Output:

```text
Price: $19.99
```

`.2f` means:

```text
format as fixed-point with 2 digits after decimal
```

Alignment:

```python
name = "Ada"
print(f"|{name:<10}|")
print(f"|{name:>10}|")
print(f"|{name:^10}|")
```

Meaning:

```text
left align
right align
center align
```

Formatting controls representation.

It does not change the underlying object.

---

# repr() and str()

Python has two common text representations:

```python
str(obj)
repr(obj)
```

`str()` is for human-readable output.

`repr()` is for developer-oriented representation.

Example:

```python
text = "hello\nworld"

print(str(text))
print(repr(text))
```

Output:

```text
hello
world
'hello\nworld'
```

In f-strings:

```python
print(f"{text}")
print(f"{text!r}")
```

`!r` uses `repr()`.

This is useful for debugging invisible characters.

---

# Unicode

Python strings are Unicode text.

Unicode is a standard for representing text from many writing systems.

Examples:

```python
print("hello")
print("नमस्ते")
print("こんにちは")
print("🙂")
```

Python can store these in `str` objects.

This is a major improvement over older systems that assumed text was only ASCII.

Practical rule:

```text
str is text
bytes is raw bytes
```

Do not confuse them.

---

# Code Points

Unicode assigns code points to characters.

Example:

```python
print(ord("A"))
print(chr(65))
```

Output:

```text
65
A
```

`ord()` gives the Unicode code point integer for a one-character string.

`chr()` creates a string from a code point.

Example:

```python
print(ord("🙂"))
```

This prints that character's Unicode code point.

You do not need to memorize code points.

Understand that strings represent Unicode text.

---

# Visual Characters Can Be Complicated

Some user-perceived characters may be made of multiple Unicode code points.

Example:

```python
text = "é"
```

There can be more than one Unicode representation for visually similar text.

Emoji and combining marks can also complicate length and slicing.

For most beginner examples:

```python
len("hello") == 5
```

But for international text, "number of characters" can be more subtle than it appears.

Practical rule:

> Python handles Unicode text well, but human language text is complex.

---

# Text vs Bytes

Strings are text.

Bytes are raw byte sequences.

Example:

```python
text = "hello"
data = b"hello"

print(type(text))
print(type(data))
```

Output:

```text
<class 'str'>
<class 'bytes'>
```

They are different types.

This fails:

```python
"hello" + b"world"
```

because Python will not silently mix text and bytes.

You must encode or decode explicitly.

---

# Encoding

Encoding converts text to bytes.

Example:

```python
text = "hello"
data = text.encode("utf-8")

print(data)
print(type(data))
```

Output:

```text
b'hello'
<class 'bytes'>
```

UTF-8 is the most common encoding for modern text systems.

For non-ASCII:

```python
text = "नमस्ते"
data = text.encode("utf-8")

print(data)
```

The bytes may not look human-readable.

That is expected.

Bytes are storage/transmission representation.

---

# Decoding

Decoding converts bytes to text.

Example:

```python
data = b"hello"
text = data.decode("utf-8")

print(text)
print(type(text))
```

Output:

```text
hello
<class 'str'>
```

Encoding and decoding must agree.

If bytes were encoded as UTF-8, decode them as UTF-8.

Mismatched encodings produce errors or incorrect text.

This matters for files, network data, APIs, and databases.

---

# Reading and Writing Text Files

When working with text files, specify encoding.

Example:

```python
with open("notes.txt", "w", encoding="utf-8") as file:
    file.write("hello")
```

Read:

```python
with open("notes.txt", "r", encoding="utf-8") as file:
    text = file.read()
```

This avoids platform-dependent encoding surprises.

We will study files later.

For now, remember:

```text
text file boundary -> encoding matters
```

---

# Strings and Truthiness

Strings have truth values.

Example:

```python
print(bool(""))
print(bool("hello"))
```

Output:

```text
False
True
```

An empty string is falsy.

A non-empty string is truthy.

This will be studied more in the booleans and control flow chapters.

For now:

```python
if name:
    print("name was provided")
```

means:

```text
if name is not an empty string
```

assuming `name` is a string.

---

# Strings and Identity

Do not compare strings with `is`.

Example:

```python
a = "python"
b = "python"

print(a is b)
```

This may print:

```text
True
```

because Python may intern some strings.

But this is an implementation detail.

Use:

```python
a == b
```

for string value comparison.

Correct:

```python
if command == "quit":
    ...
```

Incorrect:

```python
if command is "quit":
    ...
```

---

# Strings and Membership

Use `in` to check substrings.

Example:

```python
email = "ada@example.com"

print("@" in email)
print(email.endswith(".com"))
```

Output:

```text
True
True
```

Membership checks are common in validation and parsing.

Be careful with simplistic validation.

Example:

```python
"@" in email
```

does not fully validate an email address.

It only checks whether the character appears.

---

# Strings Are Iterable

You can loop over a string.

Example:

```python
for char in "abc":
    print(char)
```

Output:

```text
a
b
c
```

The loop receives one-character strings.

This prepares later chapters on iteration and loops.

For now, understand:

```text
str is a sequence
sequences can be iterated
```

---

# Common Mistakes

## Misconception 1

### Strings are mutable because they have methods.

String methods return new strings.

They do not modify the original string.

---

## Misconception 2

### `text.strip()` changes `text`.

It returns a new string.

Use:

```python
text = text.strip()
```

if you want to keep the stripped result.

---

## Misconception 3

### Strings should be compared with `is`.

Use `==` for string values.

`is` checks identity and can be affected by interning.

---

## Misconception 4

### `len()` always matches visible user-perceived characters.

For simple ASCII text, it often does.

For Unicode text with combining characters or emoji sequences, visible characters can be more complex.

---

## Misconception 5

### Text and bytes are interchangeable.

They are different types.

Use encoding and decoding explicitly.

---

## Misconception 6

### `join()` is called on the list.

`join()` is called on the separator string:

```python
",".join(parts)
```

---

## Misconception 7

### String formatting changes the original values.

Formatting produces a string representation.

It does not change the underlying objects.

---

# Real-world Usage

## User Input

User input is text.

Example:

```python
name = input("Name: ").strip()
```

Input often needs normalization:

```python
email = email.strip().casefold()
```

Decide which normalization is appropriate for your domain.

---

## Logging

Log messages are strings.

Example:

```python
print(f"Processed {count} records")
```

In production, logging frameworks are preferred over `print`, but the formatted message is still text.

---

## File Paths

Paths are often represented as strings, but Python also provides path objects through `pathlib`.

Example:

```python
from pathlib import Path

path = Path("data") / "input.txt"
```

Path objects are usually safer than manual string concatenation for file paths.

---

## Protocols and APIs

HTTP methods, headers, JSON keys, and database fields often involve strings.

Example:

```python
method = "GET"
content_type = "application/json"
```

Precise string values matter in protocols.

Small spelling or casing errors can break behavior.

---

## Security

Do not build SQL queries or shell commands by unsafe string concatenation.

Risky:

```python
query = "SELECT * FROM users WHERE name = '" + name + "'"
```

Use parameterized queries with database libraries.

String handling is not just formatting.

It can affect security.

---

# Concept Connections

This chapter builds on:

```text
Chapter 09:
Strings are objects with identity, type, and value.

Chapter 10:
Names refer to string objects.

Chapter 11:
Strings are immutable; methods return new strings.

Chapter 12:
Use == for string equality, not is.

Chapter 13:
Like numbers, strings are core immutable objects.
```

It prepares:

```text
Booleans:
    string comparisons and membership checks produce bool objects

Operators:
    +, *, in, ==, and indexing all involve operator behavior

Expressions:
    string operations evaluate to objects

Control flow:
    string truthiness and comparisons control branches

Files and APIs:
    text boundaries require encoding and decoding
```

Core model:

```text
string expression
    |
    v
str object
    |
    v
string operation
    |
    v
new object or boolean result
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| `str` | Python text type |
| String literal | Text written directly in source code |
| Escape sequence | Source notation for special characters |
| Raw string | String literal with reduced escape processing |
| Index | Position in a sequence |
| Slice | Subsequence extraction |
| Unicode | Standard for representing text |
| Encoding | Text to bytes |
| Decoding | Bytes to text |
| `bytes` | Raw byte sequence type |
| f-string | Runtime string formatting syntax |

Core rules:

```text
Strings are objects.
Strings are immutable.
String methods return new objects.
Use == for string equality.
Use encoding/decoding at text-byte boundaries.
Use f-strings for clear formatting.
Use join for combining many strings.
```

---

# Active Recall

## Easy Recall Questions

1. What is the type of `"hello"`?
2. Are strings mutable or immutable?
3. What does `len("Python")` return?
4. What does `text[0]` access?
5. Is the stop index in a slice included or excluded?
6. What does `strip()` do?
7. What does `split()` return?
8. What does `join()` do?
9. What does encoding convert?
10. Should strings be compared with `is` or `==`?

---

## Deep Understanding Questions

1. Why do string methods return new strings?
2. Why is `text.strip()` alone often a bug?
3. Why does slicing not modify the original string?
4. Why is `",".join(parts)` designed around the separator string?
5. Why are text and bytes separate types?
6. Why should file encodings often be specified explicitly?
7. Why can Unicode text make length and indexing more subtle?
8. Why is `casefold()` better than `lower()` for robust caseless matching?
9. Why is unsafe string concatenation dangerous for SQL?
10. Why does string comparison use `==`, not `is`?

---

## Explain In Your Own Words

1. Explain what a string object represents.
2. Explain string immutability.
3. Explain indexing and slicing.
4. Explain why `text = text.upper()` is rebinding.
5. Explain the difference between text and bytes.
6. Explain encoding and decoding.
7. Explain why f-strings produce new string objects.

---

## Predict-the-Output Questions

### Question 1

```python
text = "Python"
print(text[0])
print(text[-1])
```

Answer:

```text
P
n
```

Reason:

Index `0` is the first character. Index `-1` is the last.

---

### Question 2

```python
text = "Python"
print(text[1:4])
```

Answer:

```text
yth
```

Reason:

Start index `1` is included. Stop index `4` is excluded.

---

### Question 3

```python
text = " hello "
text.strip()
print(text)
```

Answer:

```text
 hello 
```

Reason:

`strip()` returns a new string. The result was ignored.

---

### Question 4

```python
text = "hello"
text[0] = "H"
```

Answer:

Python raises `TypeError`.

Reason:

Strings are immutable.

---

### Question 5

```python
parts = ["a", "b", "c"]
print("-".join(parts))
```

Answer:

```text
a-b-c
```

Reason:

The separator string `"-"` is used between the parts.

---

### Question 6

```python
text = "abc"
print(text[::-1])
```

Answer:

```text
cba
```

Reason:

A slice step of `-1` traverses the string backward.

---

# Mental Model Questions

1. Draw `text = "hello"` as a name pointing to a `str` object.
2. Draw `text = text.upper()` as rebinding.
3. Draw `part = text[:2]` as a new string object.
4. Draw text encoding from `str` to `bytes`.
5. Draw bytes decoding back to `str`.
6. Draw why `"a" + "b"` creates a new string.

---

# Practical Exercises

## Exercise 1

Inspect strings:

```python
values = ["hello", "", "नमस्ते", "🙂"]

for value in values:
    print(value, type(value), len(value), bool(value))
```

Explain the type, length, and truth value of each.

---

## Exercise 2

Practice slicing:

```python
text = "programming"

print(text[:3])
print(text[3:])
print(text[::2])
print(text[::-1])
```

Explain each result.

---

## Exercise 3

Normalize input:

```python
email = "  Ada@Example.COM\n"
normalized = email.strip().casefold()

print(normalized)
```

Explain each method call.

---

## Exercise 4

Split and join:

```python
line = "red,green,blue"
parts = line.split(",")
result = " | ".join(parts)

print(parts)
print(result)
```

Explain the object types at each step.

---

## Exercise 5

Encoding and decoding:

```python
text = "नमस्ते"
data = text.encode("utf-8")
restored = data.decode("utf-8")

print(data)
print(restored)
print(text == restored)
```

Explain why bytes and text display differently.

---

## Exercise 6

Debug invisible characters:

```python
text = "hello\n"

print(text)
print(repr(text))
```

Explain why `repr()` is useful.

---

## Exercise 7

Build a message using an f-string:

```python
name = "Ada"
score = 97.5

message = f"{name} scored {score:.1f}%"
print(message)
```

Explain the formatting result.

---

# Summary

In this chapter we learned:

* Strings are Python `str` objects.
* Strings represent Unicode text.
* Strings are immutable.
* String literals can use single quotes, double quotes, or triple quotes.
* Escape sequences represent special characters.
* Raw strings are useful for backslash-heavy text.
* Strings are sequences and support indexing, slicing, and iteration.
* Slicing creates new strings.
* String methods return new strings rather than mutating the original.
* `split()` creates a list of strings.
* `join()` combines strings using a separator.
* f-strings format values into new string objects.
* `str()` and `repr()` serve different representation goals.
* Python strings are Unicode text.
* Bytes are not strings.
* Encoding converts text to bytes.
* Decoding converts bytes to text.
* Use `==`, not `is`, for string comparison.
* Empty strings are falsy; non-empty strings are truthy.

The core model is:

```text
name ─────▶ str object
             ├── identity
             ├── type: str
             └── value: Unicode text
```

String operations either return new objects or boolean results.

---

# Preview of Chapter 15

Next we study booleans.

Booleans represent truth values:

```python
True
False
```

They appear in comparisons, conditions, loops, membership tests, and validation.

We will study:

* `bool`
* Truthiness
* Comparison results
* Logical operators
* Short-circuiting
* Common boolean mistakes

This will prepare us for operators, expressions, and control flow.
