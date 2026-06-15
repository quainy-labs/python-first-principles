# Chapter 63 — Files and Serialization

Programs become serious when they stop living only in memory.

Until now, most examples have created objects, transformed values, called functions, raised exceptions, and returned results while the program was running. That is enough to understand Python's language model, but it is not enough to build useful systems.

Useful systems must cross boundaries.

They read configuration from disk.

They load input files.

They write logs.

They export reports.

They cache expensive results.

They save user data.

They exchange messages with other programs.

They recover after the process exits.

They preserve information across time.

That is the job of files and serialization.

A file is persistent data managed by the operating system.

Serialization is the process of turning in-memory objects into a representation that can be stored or transmitted.

Deserialization is the reverse process: reading stored or transmitted data and turning it back into in-memory objects.

These ideas sound simple, but they are one of the places where professional Python code differs most sharply from beginner code.

Beginner code often assumes:

```python
file = open("data.txt")
text = file.read()
```

Professional code asks:

```text
Where is the file?
Does it exist?
Who controls its contents?
What encoding is it in?
How large is it?
What happens if the read fails halfway through?
Who closes the file?
What format is the data in?
Can older versions still read it?
Can attackers provide this input?
Is the write safe if the program crashes?
```

Chapter 62 gave us exception handling.

Chapter 63 makes those exceptions concrete.

File systems fail in ordinary ways:

* the path does not exist
* the program lacks permission
* the disk is full
* another process is writing the same file
* the bytes are not valid text
* the JSON is malformed
* the CSV has unexpected columns
* the pickle came from an untrusted source
* the write partially succeeds before an error occurs

Robust file handling is not about pretending those failures cannot happen.

It is about writing code that is honest about boundaries.

---

# The Big Picture

Files and serialization sit between Python objects and external reality.

Inside Python, you work with objects:

```python
user = {
    "id": 42,
    "name": "Asha",
    "active": True,
}
```

On disk, that object cannot exist directly as a Python dictionary.

It needs a representation.

For example, JSON text:

```json
{
  "id": 42,
  "name": "Asha",
  "active": true
}
```

Or CSV text:

```csv
id,name,active
42,Asha,true
```

Or a Python-specific binary representation such as pickle.

The relationship is:

```text
Python object -> serialize -> bytes or text -> file
file -> bytes or text -> deserialize -> Python object
```

Every arrow matters.

When writing data, you must decide:

* what format to use
* how to encode text into bytes
* where to write
* how to handle failure
* whether the result should be readable by humans
* whether other languages need to read it
* whether future versions of your program need compatibility

When reading data, you must decide:

* whether to trust the source
* how to validate the shape
* how to handle missing fields
* how to report errors
* how much data to load at once
* whether malformed input is normal, suspicious, or fatal

Files are not just storage.

Files are interfaces.

Serialization formats are contracts.

---

# Files Are Managed by the Operating System

Python does not store files by itself.

The operating system owns the file system.

When Python opens a file, it asks the operating system for access.

The operating system checks:

* whether the path exists
* whether it is a file or directory
* whether the process has permission
* whether the requested mode is allowed
* whether the file system can satisfy the operation

Python receives a file object, which acts as a handle to that operating system resource.

For example:

```python
f = open("notes.txt", "r", encoding="utf-8")
```

The variable `f` is not the file itself.

It is a Python object that represents an open connection to the file.

That object has methods such as:

```python
f.read()
f.readline()
f.write("hello")
f.close()
```

The file object manages buffering, encoding, decoding, and communication with the operating system.

This is why closing files matters.

An open file consumes a resource outside normal Python memory.

If a program opens many files and never closes them, it can exhaust operating system limits.

Even worse, writes may remain buffered and not reach the disk immediately.

That is why file code should almost always use a context manager:

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

At the end of the `with` block, Python closes the file automatically.

This happens whether the block exits normally or because of an exception.

That is the connection between Chapter 60 and this chapter:

```text
context managers make resource cleanup reliable
```

---

# Paths Are Not Just Strings

A path identifies a location in the file system.

Examples:

```text
data/users.json
/var/log/app.log
C:\Users\Asha\Documents\notes.txt
```

At first, paths look like ordinary strings.

You can pass a string path to `open()`:

```python
with open("data/users.json", "r", encoding="utf-8") as f:
    users = f.read()
```

But paths are richer than strings.

They have structure:

```text
parent directories
file name
suffix
absolute or relative location
platform-specific separators
```

Python's `pathlib` module gives you an object-oriented path API:

```python
from pathlib import Path

path = Path("data") / "users.json"

print(path.parent)
print(path.name)
print(path.suffix)
```

Output:

```text
data
users.json
.json
```

The `/` operator builds paths without hardcoding separators:

```python
path = Path("reports") / "2026" / "january.csv"
```

This is better than:

```python
path = "reports/2026/january.csv"
```

The string version assumes the separator style.

The `Path` version expresses intent.

Many modern Python APIs accept either strings or path objects.

For new code, prefer `pathlib.Path` when you are doing path manipulation.

Use strings when you are only passing a path through unchanged or when a library expects a string.

---

# Relative and Absolute Paths

A relative path is interpreted from the current working directory.

Example:

```python
Path("data/users.json")
```

An absolute path starts from the root of the file system.

Example on Unix-like systems:

```python
Path("/Users/asha/project/data/users.json")
```

Example on Windows:

```python
Path("C:/Users/Asha/project/data/users.json")
```

Relative paths are convenient but can surprise you.

Suppose this project layout exists:

```text
project/
    app.py
    data/
        users.json
```

Inside `app.py`, you might write:

```python
from pathlib import Path

path = Path("data/users.json")
```

This works if the program is launched from the project directory:

```text
project/
```

But it may fail if the program is launched from somewhere else.

The relative path depends on the process's current working directory, not on the file containing the code.

For scripts, a common pattern is to anchor paths relative to the script file:

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent
DATA_DIR = BASE_DIR / "data"
USERS_FILE = DATA_DIR / "users.json"
```

Now the location is based on the source file's location.

This makes the program less sensitive to where it is launched from.

For applications, path strategy often depends on the type of data:

```text
source-controlled examples -> relative to the project
user configuration -> user config directory
logs -> configured log directory
temporary files -> system temporary directory
uploaded files -> controlled storage directory
```

Avoid scattering string paths throughout a program.

Centralize important paths.

Named path constants make file behavior easier to audit:

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent
CONFIG_PATH = BASE_DIR / "config.json"
EXPORT_DIR = BASE_DIR / "exports"
```

---

# Opening Files

The built-in `open()` function opens a file and returns a file object.

The common form is:

```python
open(file, mode="r", encoding=None)
```

For text files, you should usually provide an explicit encoding:

```python
with open("message.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

The main mode characters are:

```text
r  read
w  write, truncating existing contents
a  append
x  create, failing if the file already exists
b  binary mode
t  text mode
+  read and write
```

Some common modes:

```text
r   read text
w   write text, replacing existing contents
a   append text
x   create new text file
rb  read binary
wb  write binary
ab  append binary
r+  read and write without truncating immediately
w+  read and write, truncating first
```

The default mode is `"r"`, meaning text read mode.

This:

```python
open("notes.txt")
```

is equivalent to:

```python
open("notes.txt", "r")
```

But explicit code is usually clearer:

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

The mode is not a decoration.

It is a contract with the operating system.

If you open in read mode, the file must exist.

If you open in write mode, existing contents are removed.

If you open in exclusive creation mode, Python refuses to overwrite an existing file.

That difference matters.

---

# The Danger of Write Mode

Write mode truncates the file.

This means the file is emptied as soon as it is opened successfully.

Example:

```python
with open("settings.json", "w", encoding="utf-8") as f:
    f.write("{}")
```

If `settings.json` already had content, that content is gone before `write()` is called.

This can surprise beginners.

This code is risky:

```python
with open("important.txt", "w", encoding="utf-8") as f:
    maybe_fail()
    f.write("new contents")
```

If `maybe_fail()` raises an exception, the file may already have been truncated.

A safer structure is:

```python
new_contents = build_contents()

with open("important.txt", "w", encoding="utf-8") as f:
    f.write(new_contents)
```

This computes the new data before opening the file for writing.

For important files, even that may not be enough.

Later in this chapter, we will use a temporary file plus replacement.

The principle is:

```text
do risky computation before destructive file operations
```

---

# Text Files and Binary Files

Files store bytes.

That is always true.

Text files are bytes interpreted as characters using an encoding.

Binary files are bytes treated directly.

Text mode:

```python
with open("poem.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

Binary mode:

```python
with open("image.png", "rb") as f:
    data = f.read()
```

In text mode, Python decodes bytes into `str` objects when reading.

In text mode, Python encodes `str` objects into bytes when writing.

In binary mode, Python reads and writes `bytes`.

Example:

```python
with open("data.bin", "wb") as f:
    f.write(b"\x00\x01\x02")
```

In binary mode, there is no `encoding` argument.

This is invalid:

```python
open("image.png", "rb", encoding="utf-8")
```

Binary files include:

* images
* audio files
* video files
* compressed archives
* database files
* encrypted data
* Python pickle files

Text files include:

* `.txt`
* `.py`
* `.json`
* `.csv`
* `.md`
* `.html`
* `.xml`

The extension is a convention.

The actual distinction is how the bytes are interpreted.

---

# Encodings

An encoding defines how characters are represented as bytes.

Python strings contain Unicode text.

Files contain bytes.

Encoding bridges the two.

For example:

```python
text = "cafe"
data = text.encode("utf-8")
```

`data` is bytes:

```python
b'cafe'
```

Decoding reverses the operation:

```python
text = data.decode("utf-8")
```

Most modern text files should use UTF-8.

That is why you will frequently see:

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

If you omit `encoding`, Python uses a platform-dependent default.

That can make code work on one machine and fail on another.

For portable code, specify the encoding.

This is especially important for:

* project files
* configuration
* tests
* data imports
* exported text
* command-line tools

---

# Encoding Errors

If bytes cannot be decoded using the requested encoding, Python raises `UnicodeDecodeError`.

Example:

```python
with open("legacy.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

If `legacy.txt` contains bytes that are not valid UTF-8, reading may fail.

That failure is useful.

It tells you that your assumption about the file's encoding was wrong.

You can pass an `errors` strategy:

```python
with open("legacy.txt", "r", encoding="utf-8", errors="replace") as f:
    text = f.read()
```

Common error strategies include:

```text
strict   raise an exception
ignore   skip invalid data
replace  insert a replacement character
```

The default is usually strict behavior.

For data processing, strict is often best because it prevents silent corruption.

For best-effort display, replacement may be acceptable.

Avoid `errors="ignore"` unless data loss is genuinely acceptable.

Silently dropping characters can turn a data problem into a hidden correctness problem.

---

# Newlines

Text files use newline characters to separate lines.

Common newline styles:

```text
\n    Unix and modern macOS
\r\n  Windows
\r    older classic Mac systems
```

Python text mode performs universal newline handling by default.

When reading text, different platform newline styles can be translated into `\n`.

Example:

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(repr(line))
```

Lines usually include the trailing newline:

```text
'first line\n'
'second line\n'
```

You can remove it:

```python
line = line.rstrip("\n")
```

Use `rstrip("\n")` when you specifically want to remove a newline.

Do not use plain `rstrip()` unless you also want to remove other trailing whitespace.

This matters for data where spaces are meaningful.

---

# Reading an Entire File

The simplest read is `read()`:

```python
from pathlib import Path

path = Path("message.txt")

with path.open("r", encoding="utf-8") as f:
    text = f.read()
```

`Path` also provides convenience methods:

```python
text = Path("message.txt").read_text(encoding="utf-8")
```

This is excellent for small text files.

Examples:

* configuration snippets
* templates
* test fixtures
* small JSON documents
* Markdown files

But `read()` loads the entire file into memory.

That can be a problem for large files.

If a file is 8 GB, `read()` attempts to create an 8 GB object.

That may crash the program or force the operating system to start swapping memory.

For large files, stream the file instead.

---

# Reading Line by Line

File objects are iterable.

This is the standard way to process text files line by line:

```python
with open("events.log", "r", encoding="utf-8") as f:
    for line in f:
        line = line.rstrip("\n")
        handle_event(line)
```

This does not load the entire file at once.

Python reads chunks internally and yields one line at a time.

This pattern works well for:

* logs
* CSV-like text
* large data exports
* command output captured to files
* line-oriented protocols

Avoid this beginner pattern for large files:

```python
for line in f.readlines():
    handle_event(line)
```

`readlines()` builds a list of all lines.

Iteration over the file object streams naturally.

Use:

```python
for line in f:
    ...
```

not:

```python
for line in f.readlines():
    ...
```

unless you truly need all lines in a list.

---

# Reading in Chunks

Some files are not line-oriented.

Binary data is often processed in chunks.

Example:

```python
chunk_size = 1024 * 1024

with open("archive.zip", "rb") as source:
    while chunk := source.read(chunk_size):
        process(chunk)
```

This reads up to one megabyte at a time.

The walrus operator binds the chunk and tests whether it is empty.

Without the walrus operator:

```python
chunk_size = 1024 * 1024

with open("archive.zip", "rb") as source:
    while True:
        chunk = source.read(chunk_size)
        if not chunk:
            break
        process(chunk)
```

Chunked reading is useful for:

* copying files
* hashing files
* uploading large files
* processing binary formats
* limiting memory usage

The design question is:

```text
Does the program need the whole file at once?
```

If not, stream it.

---

# Writing Text Files

To write text, open the file in write mode:

```python
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("hello\n")
    f.write("world\n")
```

`write()` returns the number of characters written in text mode.

Usually you ignore that return value.

`Path.write_text()` is convenient for small files:

```python
from pathlib import Path

Path("output.txt").write_text("hello\nworld\n", encoding="utf-8")
```

This opens, writes, and closes the file for you.

It is concise and readable for small outputs.

For larger or incremental outputs, use `with open(...)`.

Example:

```python
with open("report.txt", "w", encoding="utf-8") as f:
    for row in rows:
        f.write(format_row(row))
        f.write("\n")
```

This avoids building the entire report string in memory.

---

# Writing Binary Files

Binary writes use bytes:

```python
data = b"\x89PNG\r\n\x1a\n"

with open("image-header.bin", "wb") as f:
    f.write(data)
```

`Path.write_bytes()` is the binary counterpart to `write_text()`:

```python
from pathlib import Path

Path("data.bin").write_bytes(b"\x00\x01\x02")
```

Binary mode is required when data is already bytes.

This includes:

* downloaded content
* compressed data
* encrypted data
* images
* pickled objects
* hashes and signatures

Do not convert arbitrary binary data to strings just to write it.

That creates encoding problems where none are needed.

---

# Appending

Append mode writes at the end of the file.

```python
with open("app.log", "a", encoding="utf-8") as f:
    f.write("started\n")
```

If the file does not exist, append mode creates it.

Append mode is useful for:

* logs
* audit records
* simple journals
* append-only event streams

But append mode is not automatically a database.

Problems still exist:

* multiple processes may write at the same time
* records may be partially written if the process crashes
* old data can grow without limit
* parsing may become expensive

For serious logging, use Python's `logging` module.

For serious durable storage, use a database or a carefully designed file format.

---

# Exclusive Creation

Mode `"x"` creates a file and fails if it already exists.

```python
with open("new-report.txt", "x", encoding="utf-8") as f:
    f.write("report contents\n")
```

If the file already exists, Python raises `FileExistsError`.

This is useful when overwriting would be a bug.

Example:

```python
from pathlib import Path

def create_user_file(user_id, contents):
    path = Path("users") / f"{user_id}.json"
    with path.open("x", encoding="utf-8") as f:
        f.write(contents)
```

This expresses:

```text
creating a new file is expected
overwriting an existing one is not
```

That is better than manually checking first:

```python
if not path.exists():
    path.write_text(contents, encoding="utf-8")
```

The manual check has a race condition.

Another process could create the file after `exists()` returns but before `write_text()` writes.

Mode `"x"` asks the operating system to do the creation atomically.

---

# Checking Whether Paths Exist

`Path.exists()` checks whether a path exists:

```python
from pathlib import Path

path = Path("config.json")

if path.exists():
    print("found config")
```

`Path.is_file()` checks whether the path is an existing regular file:

```python
if path.is_file():
    print("regular file")
```

`Path.is_dir()` checks whether it is a directory:

```python
if path.is_dir():
    print("directory")
```

These methods are useful for user-facing checks and diagnostics.

But do not overuse them as a substitute for exception handling.

This pattern has a race:

```python
if path.exists():
    text = path.read_text(encoding="utf-8")
```

The file could be deleted between the check and the read.

Often this is better:

```python
try:
    text = path.read_text(encoding="utf-8")
except FileNotFoundError:
    text = "{}"
```

This is EAFP in file form:

```text
try the operation and handle the failure that matters
```

Checks are fine when they improve messages.

Operations still need exception handling when failure matters.

---

# Creating Directories

Use `Path.mkdir()` to create directories.

```python
from pathlib import Path

output_dir = Path("exports")
output_dir.mkdir()
```

If the directory already exists, this raises `FileExistsError`.

To accept an existing directory:

```python
output_dir.mkdir(exist_ok=True)
```

To create parent directories too:

```python
output_dir = Path("exports") / "2026" / "january"
output_dir.mkdir(parents=True, exist_ok=True)
```

This creates:

```text
exports/
exports/2026/
exports/2026/january/
```

Use `parents=True` when intermediate directories may not exist.

Use `exist_ok=True` only when an existing directory is acceptable.

Do not casually use `exist_ok=True` when existence might indicate a bug.

---

# File Metadata

Files have metadata.

`Path.stat()` returns information from the operating system:

```python
from pathlib import Path

path = Path("data.json")
info = path.stat()

print(info.st_size)
print(info.st_mtime)
```

Common metadata includes:

```text
size
modification time
permissions
file type information
```

Metadata can be useful for:

* rejecting unexpectedly large files
* displaying file information
* deciding whether a cache is stale
* audit logs
* synchronization tools

But metadata can become stale.

Another process can modify the file after you inspect it.

For security-sensitive code, avoid assuming a path remains unchanged between checks and operations.

---

# Basic File Exceptions

Common file exceptions include:

```text
FileNotFoundError
FileExistsError
PermissionError
IsADirectoryError
NotADirectoryError
UnicodeDecodeError
UnicodeEncodeError
OSError
```

These exceptions are part of normal file handling.

Example:

```python
from pathlib import Path

def load_config(path):
    try:
        return path.read_text(encoding="utf-8")
    except FileNotFoundError:
        return "{}"
    except PermissionError as error:
        raise RuntimeError(f"cannot read config file: {path}") from error
```

Notice the distinction:

* missing config has a fallback
* permission failure becomes a higher-level application error

Do not handle all file errors the same way.

Some failures are expected alternatives.

Some are operational problems.

Some indicate programmer mistakes.

Some should stop the program.

---

# Keep Try Blocks Small

When handling file exceptions, keep `try` blocks focused.

Less clear:

```python
try:
    text = path.read_text(encoding="utf-8")
    data = json.loads(text)
    user = build_user(data)
    activate_user(user)
except FileNotFoundError:
    ...
except json.JSONDecodeError:
    ...
```

This mixes several responsibilities.

Better:

```python
try:
    text = path.read_text(encoding="utf-8")
except FileNotFoundError:
    text = "{}"

try:
    data = json.loads(text)
except json.JSONDecodeError as error:
    raise ValueError(f"invalid JSON in {path}") from error

user = build_user(data)
activate_user(user)
```

Small `try` blocks make it clear which operation is expected to fail.

They also reduce accidental catches.

If `activate_user()` raises `FileNotFoundError` internally, the first version might catch it incorrectly.

The second version avoids that ambiguity.

---

# File-Like Objects

Many Python APIs accept file-like objects.

A file-like object behaves like a file, even if it is not backed by a real file on disk.

For example, `io.StringIO` provides an in-memory text file:

```python
from io import StringIO

buffer = StringIO()
buffer.write("name,score\n")
buffer.write("Asha,98\n")

contents = buffer.getvalue()
```

`io.BytesIO` provides an in-memory binary file:

```python
from io import BytesIO

buffer = BytesIO()
buffer.write(b"\x00\x01\x02")

data = buffer.getvalue()
```

File-like objects are useful for:

* tests
* generated data
* network streams
* compressed streams
* in-memory transformations
* APIs that should not care where data comes from

Instead of writing a function that only accepts a path:

```python
def count_lines(path):
    with open(path, "r", encoding="utf-8") as f:
        return sum(1 for _ in f)
```

You can write a function that accepts an already-open text stream:

```python
def count_lines(stream):
    return sum(1 for _ in stream)
```

Then callers decide where the stream comes from:

```python
with open("events.log", "r", encoding="utf-8") as f:
    total = count_lines(f)
```

And tests can avoid touching disk:

```python
from io import StringIO

fake_file = StringIO("a\nb\nc\n")
assert count_lines(fake_file) == 3
```

This is duck typing applied to I/O.

The function does not need a real file.

It needs an object that can be iterated line by line.

---

# Serialization

Serialization turns objects into a storable representation.

Deserialization turns that representation back into objects.

A simple example:

```python
import json

user = {"id": 1, "name": "Asha"}
text = json.dumps(user)
restored = json.loads(text)
```

`json.dumps()` serializes to a string.

`json.loads()` deserializes from a string.

The file-based versions are:

```python
import json

with open("user.json", "w", encoding="utf-8") as f:
    json.dump(user, f)

with open("user.json", "r", encoding="utf-8") as f:
    restored = json.load(f)
```

The final `s` in `dumps` and `loads` stands for string.

Remember:

```text
dump   object -> file
dumps  object -> string
load   file -> object
loads  string -> object
```

That naming pattern appears in several serialization APIs.

---

# Choosing a Serialization Format

The format should match the boundary.

Common choices:

```text
JSON     human-readable, language-neutral, common for APIs and config
CSV      tabular text, common for spreadsheets and exports
pickle   Python-specific binary object serialization
TOML     human-readable configuration
YAML     human-readable, flexible, but requires third-party libraries in Python
SQLite   relational storage in a single file
Parquet  columnar analytics format, usually third-party
```

This chapter focuses on formats in the standard library:

* JSON
* CSV
* pickle

The choice is not only about convenience.

It affects:

* security
* compatibility
* readability
* speed
* file size
* schema evolution
* support by other languages

As a rough guide:

```text
configuration -> JSON or TOML
API exchange -> JSON
spreadsheet export -> CSV
trusted Python cache -> pickle
long-term structured storage -> database or documented format
analytics data -> specialized formats outside the standard library
```

Do not choose pickle just because it is easy.

Pickle is powerful, but it is not safe for untrusted input and not portable across languages.

---

# JSON

JSON stands for JavaScript Object Notation.

Despite the name, it is language-neutral and widely used.

JSON supports a small set of data types:

```text
object
array
string
number
true
false
null
```

These map naturally to Python:

```text
JSON object  -> dict
JSON array   -> list
JSON string  -> str
JSON number  -> int or float
JSON true    -> True
JSON false   -> False
JSON null    -> None
```

Example:

```python
import json

profile = {
    "id": 42,
    "name": "Asha",
    "active": True,
    "tags": ["admin", "billing"],
    "manager": None,
}

text = json.dumps(profile)
print(text)
```

Output:

```json
{"id": 42, "name": "Asha", "active": true, "tags": ["admin", "billing"], "manager": null}
```

JSON is a good default format when:

* humans may inspect the data
* other languages may consume it
* the data is tree-shaped
* values are simple
* interoperability matters

JSON is not ideal for:

* arbitrary Python objects
* circular references
* preserving tuple identity
* exact Decimal values without custom handling
* comments in config files
* very large tabular data

---

# Pretty JSON

JSON can be formatted for readability:

```python
import json

text = json.dumps(profile, indent=2, sort_keys=True)
print(text)
```

Output:

```json
{
  "active": true,
  "id": 42,
  "manager": null,
  "name": "Asha",
  "tags": [
    "admin",
    "billing"
  ]
}
```

`indent=2` makes nested structures readable.

`sort_keys=True` gives stable key ordering.

Stable ordering helps when:

* reviewing diffs
* comparing files
* writing tests
* generating deterministic output

For compact output:

```python
text = json.dumps(profile, separators=(",", ":"))
```

Compact JSON saves space but is harder to read.

Use readable output for configuration and files people inspect.

Use compact output for network or storage efficiency when needed.

---

# Writing JSON Files

Use `json.dump()` to write directly to a file object:

```python
import json
from pathlib import Path

path = Path("profile.json")

profile = {
    "id": 42,
    "name": "Asha",
    "active": True,
}

with path.open("w", encoding="utf-8") as f:
    json.dump(profile, f, indent=2)
    f.write("\n")
```

The trailing newline is optional but common for text files.

Many command-line tools and version-control diffs behave more nicely when text files end with a newline.

You can also use `dumps()` plus `write_text()`:

```python
text = json.dumps(profile, indent=2) + "\n"
path.write_text(text, encoding="utf-8")
```

Both styles are valid.

Use `dump()` when you already have a file object or want to stream to one.

Use `dumps()` when you want the serialized string first.

---

# Reading JSON Files

Use `json.load()` to read from a file object:

```python
import json
from pathlib import Path

path = Path("profile.json")

with path.open("r", encoding="utf-8") as f:
    profile = json.load(f)
```

Or use `read_text()` plus `loads()`:

```python
text = path.read_text(encoding="utf-8")
profile = json.loads(text)
```

Malformed JSON raises `json.JSONDecodeError`.

Example:

```python
import json

try:
    profile = json.loads("{bad json")
except json.JSONDecodeError as error:
    print(error)
```

When loading JSON from a file, distinguish file errors from format errors:

```python
import json
from pathlib import Path

def load_profile(path):
    try:
        text = path.read_text(encoding="utf-8")
    except FileNotFoundError as error:
        raise ValueError(f"profile file does not exist: {path}") from error

    try:
        return json.loads(text)
    except json.JSONDecodeError as error:
        raise ValueError(f"profile file is not valid JSON: {path}") from error
```

This gives callers a clearer error at the application boundary.

---

# JSON Is Not Validation

JSON parsing tells you whether the text is valid JSON.

It does not tell you whether the data is valid for your program.

This is valid JSON:

```json
{
  "id": "not an integer",
  "name": null,
  "active": "sometimes"
}
```

But it may be invalid for your application.

You still need validation:

```python
def parse_profile(data):
    if not isinstance(data, dict):
        raise TypeError("profile must be a JSON object")

    if not isinstance(data.get("id"), int):
        raise TypeError("profile id must be an integer")

    if not isinstance(data.get("name"), str):
        raise TypeError("profile name must be a string")

    if not isinstance(data.get("active"), bool):
        raise TypeError("profile active flag must be a boolean")

    return data
```

Then:

```python
raw = json.loads(text)
profile = parse_profile(raw)
```

Parsing answers:

```text
Is this syntactically JSON?
```

Validation answers:

```text
Is this meaningful for this program?
```

Do not confuse the two.

---

# JSON and Non-JSON Types

Not every Python object can be serialized to JSON.

This fails:

```python
import json
from datetime import datetime

json.dumps({"created_at": datetime.now()})
```

Python raises `TypeError` because `datetime` is not directly JSON serializable.

A common solution is to convert values into JSON-compatible shapes:

```python
from datetime import datetime, timezone

event = {
    "created_at": datetime.now(timezone.utc).isoformat(),
}
```

Now the datetime is represented as a string.

For `Decimal`, you must choose a representation:

```python
from decimal import Decimal

price = Decimal("19.99")

data = {
    "price": str(price),
}
```

Using a string preserves exact decimal value.

Using a float may introduce floating-point rounding behavior.

Serialization is full of choices like this.

The format does not decide your domain semantics.

You do.

---

# Custom JSON Encoding

The `json` module lets you provide a `default` function for unsupported objects.

Example:

```python
import json
from datetime import datetime

def encode_default(value):
    if isinstance(value, datetime):
        return value.isoformat()
    raise TypeError(f"cannot serialize {type(value).__name__}")

text = json.dumps(
    {"created_at": datetime(2026, 1, 1, 9, 30)},
    default=encode_default,
)
```

The `default` function should return a JSON-compatible value.

If it does not know how to handle the object, it should raise `TypeError`.

Do not quietly convert unknown objects to strings:

```python
json.dumps(obj, default=str)
```

That may look convenient, but it can hide data loss.

For example, converting an object to its display string may not preserve enough information to reconstruct it later.

Serialization should be deliberate.

---

# JSON Lines

Plain JSON represents one JSON value.

For large streams of records, a common pattern is JSON Lines.

Each line contains one complete JSON value:

```json
{"id": 1, "event": "login"}
{"id": 2, "event": "logout"}
{"id": 3, "event": "purchase"}
```

This is not one JSON array.

It is line-delimited JSON.

Writing JSON Lines:

```python
import json

events = [
    {"id": 1, "event": "login"},
    {"id": 2, "event": "logout"},
]

with open("events.jsonl", "w", encoding="utf-8") as f:
    for event in events:
        f.write(json.dumps(event))
        f.write("\n")
```

Reading JSON Lines:

```python
import json

with open("events.jsonl", "r", encoding="utf-8") as f:
    for line_number, line in enumerate(f, start=1):
        line = line.strip()
        if not line:
            continue
        try:
            event = json.loads(line)
        except json.JSONDecodeError as error:
            raise ValueError(f"invalid JSON on line {line_number}") from error
        handle_event(event)
```

JSON Lines is useful because it streams naturally.

A huge file does not need to be loaded as one giant list.

---

# CSV

CSV stands for comma-separated values.

It is a text format for tabular data.

Example:

```csv
name,score,active
Asha,98,true
Ben,87,false
```

CSV is common because spreadsheets understand it.

But CSV is deceptively tricky.

Fields can contain commas:

```csv
name,address
Asha,"Mumbai, India"
```

Fields can contain quotes:

```csv
name,note
Asha,"She said ""hello"""
```

Fields can contain newlines.

Different systems use different delimiters.

That is why you should use Python's `csv` module instead of splitting strings by comma.

Do not parse CSV like this:

```python
line = "Asha,98,true"
fields = line.split(",")
```

It breaks as soon as a quoted field contains a comma.

Use `csv.reader`.

---

# Reading CSV

Basic CSV reading:

```python
import csv

with open("scores.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

Output rows are lists of strings:

```python
["name", "score", "active"]
["Asha", "98", "true"]
["Ben", "87", "false"]
```

Notice `newline=""`.

The `csv` module expects to manage newlines itself.

When opening files for `csv`, use `newline=""`.

CSV does not know your types.

Everything comes in as text.

You must convert values:

```python
name = row[0]
score = int(row[1])
active = row[2].lower() == "true"
```

Again:

```text
parsing is not validation
```

CSV parsing gives you rows and fields.

Your program decides what those fields mean.

---

# DictReader

`csv.DictReader` treats the first row as headers.

```python
import csv

with open("scores.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["score"])
```

Each row behaves like a dictionary:

```python
{"name": "Asha", "score": "98", "active": "true"}
```

This is often clearer than indexing by position.

Compare:

```python
score = int(row[1])
```

with:

```python
score = int(row["score"])
```

Named fields make code easier to read and safer to change.

But they introduce new validation questions:

* Are all required columns present?
* Are there unexpected columns?
* Are column names spelled correctly?
* Are empty fields allowed?
* Are duplicate headers possible?

CSV is simple at the syntax level.

It is not automatically simple at the data-contract level.

---

# Writing CSV

Use `csv.writer` to write rows:

```python
import csv

rows = [
    ["name", "score", "active"],
    ["Asha", 98, True],
    ["Ben", 87, False],
]

with open("scores.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(rows)
```

The writer handles quoting when needed.

For dictionaries, use `csv.DictWriter`:

```python
import csv

rows = [
    {"name": "Asha", "score": 98, "active": True},
    {"name": "Ben", "score": 87, "active": False},
]

fieldnames = ["name", "score", "active"]

with open("scores.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(rows)
```

`fieldnames` controls the output column order.

This is important.

Normal dictionaries preserve insertion order in modern Python, but explicit fieldnames make the file contract visible.

---

# CSV Dialects

Not all CSV files use commas.

Some use tabs.

Some use semicolons.

Some use different quote rules.

The `csv` module supports dialects and formatting options.

Example with semicolon delimiter:

```python
import csv

with open("data.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f, delimiter=";")
    for row in reader:
        print(row)
```

Tab-separated values:

```python
with open("data.tsv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f, delimiter="\t")
```

When building import tools, make the dialect explicit.

Do not assume every CSV-like file follows the same convention.

---

# CSV Error Handling

CSV data often comes from humans, spreadsheets, exports, or other systems.

Expect surprises.

Example:

```python
import csv

def read_scores(path):
    scores = []

    with path.open("r", encoding="utf-8", newline="") as f:
        reader = csv.DictReader(f)

        required = {"name", "score"}
        missing = required - set(reader.fieldnames or [])
        if missing:
            raise ValueError(f"missing columns: {sorted(missing)}")

        for line_number, row in enumerate(reader, start=2):
            try:
                score = int(row["score"])
            except ValueError as error:
                raise ValueError(
                    f"invalid score on line {line_number}: {row['score']!r}"
                ) from error

            scores.append((row["name"], score))

    return scores
```

This function checks:

* headers
* line numbers
* numeric conversion
* meaningful error messages

Line numbers matter.

Bad data is easier to fix when the error points to a specific row.

---

# Pickle

The `pickle` module serializes Python objects into a Python-specific binary format.

Example:

```python
import pickle

data = {
    "name": "Asha",
    "scores": [98, 95, 91],
}

with open("data.pickle", "wb") as f:
    pickle.dump(data, f)

with open("data.pickle", "rb") as f:
    restored = pickle.load(f)
```

Pickle can handle many Python objects that JSON cannot.

That includes:

* tuples
* sets
* custom class instances
* shared references
* more complex object graphs

But pickle has a serious rule:

```text
never unpickle data from an untrusted source
```

Loading a pickle can execute code.

This is not a minor caveat.

It is central to the design.

Pickle is suitable for trusted Python-to-Python persistence.

It is not suitable for accepting uploads, API input, email attachments, or data from unknown users.

---

# When Pickle Is Reasonable

Pickle can be reasonable when:

* the data is produced and consumed by your own Python code
* the storage location is trusted
* long-term compatibility is not critical
* human readability is not needed
* performance and convenience matter
* the objects are Python-specific

Examples:

* local development caches
* internal machine learning artifacts from trusted pipelines
* short-lived intermediate files
* controlled batch-processing checkpoints

Even then, think carefully.

A pickle file is tied to Python object definitions.

If you rename classes, move modules, or change object structure, old pickle files may stop loading or may load into objects that no longer make sense.

For long-term storage, prefer a more explicit format.

---

# Pickle and Versioning

Pickle stores enough information to reconstruct Python objects.

That can include module and class names.

Suppose you pickle an instance:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Later, you rename `User` or move it to another module.

Old pickle files may fail to load.

This is why pickle is not a stable public data format.

If compatibility matters, design a schema:

```json
{
  "version": 1,
  "users": [
    {"name": "Asha"}
  ]
}
```

Then your program can decide how to migrate old versions.

With pickle, that boundary is less explicit.

The principle:

```text
pickle preserves Python objects, not long-term public contracts
```

---

# Serialization Boundaries

A serialization boundary is where objects leave one representation and enter another.

Examples:

```text
Python object -> JSON file
CSV file -> Python rows
Python object -> pickle bytes
HTTP response -> JSON object
database row -> Python model
```

At each boundary, ask:

```text
What is allowed?
What is required?
What is optional?
What types are expected?
What defaults are acceptable?
What errors should callers see?
What is trusted?
What must remain compatible over time?
```

A good serialization boundary does not leak random internal objects.

It deliberately converts between:

```text
internal model
external representation
```

Example:

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    active: bool

def user_to_json_dict(user):
    return {
        "id": user.id,
        "name": user.name,
        "active": user.active,
    }

def user_from_json_dict(data):
    return User(
        id=int(data["id"]),
        name=str(data["name"]),
        active=bool(data["active"]),
    )
```

This may look like extra work.

But it creates a clear contract.

Your internal class can change without automatically changing the file format.

---

# Avoid Serializing Everything

One common mistake is trying to serialize an entire live object graph.

Example:

```python
class Session:
    def __init__(self, user, database_connection, cache, open_file):
        self.user = user
        self.database_connection = database_connection
        self.cache = cache
        self.open_file = open_file
```

This object contains things that do not belong in a serialized file:

* database connections
* caches
* open file handles
* locks
* temporary runtime state

The data you persist should usually be smaller and more explicit:

```python
def session_snapshot(session):
    return {
        "user_id": session.user.id,
        "started_at": session.started_at.isoformat(),
    }
```

Persist state, not machinery.

The question is:

```text
What information is needed to reconstruct the useful state later?
```

Not:

```text
How do I dump this whole object?
```

---

# Safe-ish File Writes

Writing files safely is harder than reading files.

A simple write can leave a broken file if the program crashes halfway through.

Example:

```python
path.write_text(new_text, encoding="utf-8")
```

This may:

* truncate the old file
* write part of the new file
* fail before finishing

For important files, a safer pattern is:

```text
write new contents to a temporary file in the same directory
flush and close it
replace the target file with the temporary file
```

Example:

```python
from pathlib import Path
from tempfile import NamedTemporaryFile
import os

def write_text_safely(path, text):
    path = Path(path)
    path.parent.mkdir(parents=True, exist_ok=True)

    with NamedTemporaryFile(
        "w",
        encoding="utf-8",
        dir=path.parent,
        delete=False,
    ) as tmp:
        tmp.write(text)
        tmp_name = tmp.name

    os.replace(tmp_name, path)
```

`os.replace()` replaces the destination if it exists.

On many file systems, replacing a file within the same directory is atomic from the point of view of other processes.

This pattern is not magic.

There are still platform and file-system details.

But it is much safer than truncating the target first.

For serious durability requirements, you also need to understand flushing, `fsync`, directories, operating system behavior, and file-system guarantees.

For many application config and cache files, temporary write plus replace is a practical improvement.

---

# Cleaning Up Temporary Files

The previous safe-write example has a weakness.

If an exception occurs after creating the temporary file but before replacement, the temporary file may remain.

A more careful version cleans up:

```python
from pathlib import Path
from tempfile import NamedTemporaryFile
import os

def write_text_safely(path, text):
    path = Path(path)
    path.parent.mkdir(parents=True, exist_ok=True)
    tmp_name = None

    try:
        with NamedTemporaryFile(
            "w",
            encoding="utf-8",
            dir=path.parent,
            delete=False,
        ) as tmp:
            tmp.write(text)
            tmp_name = tmp.name

        os.replace(tmp_name, path)
        tmp_name = None
    finally:
        if tmp_name is not None:
            try:
                os.unlink(tmp_name)
            except FileNotFoundError:
                pass
```

This code is more verbose because the boundary is more serious.

It demonstrates several ideas from earlier chapters:

* context managers close the temporary file
* `finally` handles cleanup
* specific exceptions are ignored only when harmless
* the path is normalized with `Path`
* replacement happens after a full write

This is professional file code: not glamorous, but careful.

---

# Flushing and Buffering

File writes are usually buffered.

When you call `write()`, Python may store data in an internal buffer before passing it to the operating system.

The operating system may also buffer data before writing it to physical storage.

Closing the file flushes Python's buffer.

You can flush explicitly:

```python
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("hello\n")
    f.flush()
```

For stronger durability, `os.fsync()` asks the operating system to flush file data to storage:

```python
import os

with open("output.txt", "w", encoding="utf-8") as f:
    f.write("hello\n")
    f.flush()
    os.fsync(f.fileno())
```

This is slower.

Most everyday file writing does not need manual `fsync`.

But for databases, financial systems, transactional logs, and critical state files, buffering and durability become design topics.

The key idea is:

```text
write() returning does not always mean the data is physically durable forever
```

For ordinary scripts, closing files with `with` is enough.

For critical persistence, learn the durability guarantees of your platform and file system.

---

# Path Traversal

If users provide file names, be careful.

Suppose a web app lets users download files from a safe directory:

```python
base_dir = Path("uploads")
requested = input_filename
path = base_dir / requested
```

If `requested` is:

```text
../../secrets.txt
```

then `path` may point outside `uploads`.

This is called path traversal.

A safer approach resolves the path and checks that it stays under the allowed directory:

```python
from pathlib import Path

def resolve_under(base_dir, user_path):
    base_dir = Path(base_dir).resolve()
    candidate = (base_dir / user_path).resolve()

    if base_dir not in candidate.parents and candidate != base_dir:
        raise ValueError("path escapes base directory")

    return candidate
```

Security-sensitive path handling has many edge cases:

* symbolic links
* case-insensitive file systems
* Windows drive paths
* race conditions
* permissions

The main lesson here is simple:

```text
never blindly join trusted directories with untrusted path input
```

Validate paths at the boundary.

---

# File Names Are Data Too

File names can contain surprising characters.

They may include spaces, Unicode, punctuation, or names that are awkward on some platforms.

If your program creates files from user input, sanitize deliberately.

Risky:

```python
path = export_dir / f"{title}.txt"
```

If `title` comes from a user, it might contain slashes or other problematic characters.

Safer:

```python
import re

def slugify_filename(value):
    value = value.strip().lower()
    value = re.sub(r"[^a-z0-9._-]+", "-", value)
    value = value.strip(".-")
    return value or "untitled"
```

Then:

```python
filename = slugify_filename(title) + ".txt"
path = export_dir / filename
```

This is not a universal filename policy.

It is an example of having a policy.

Professional code does not let arbitrary strings become file paths without thought.

---

# Configuration Files

Configuration files are a common use of serialization.

Example `config.json`:

```json
{
  "host": "localhost",
  "port": 8000,
  "debug": false
}
```

Loader:

```python
import json
from pathlib import Path

DEFAULT_CONFIG = {
    "host": "localhost",
    "port": 8000,
    "debug": False,
}

def load_config(path):
    path = Path(path)

    try:
        data = json.loads(path.read_text(encoding="utf-8"))
    except FileNotFoundError:
        return DEFAULT_CONFIG.copy()
    except json.JSONDecodeError as error:
        raise ValueError(f"invalid configuration file: {path}") from error

    config = DEFAULT_CONFIG | data
    validate_config(config)
    return config

def validate_config(config):
    if not isinstance(config["host"], str):
        raise TypeError("config host must be a string")
    if not isinstance(config["port"], int):
        raise TypeError("config port must be an integer")
    if not isinstance(config["debug"], bool):
        raise TypeError("config debug must be a boolean")
```

This separates:

* reading
* parsing
* defaults
* validation
* error reporting

Do not cram all of that into one expression.

File code is clearer when each boundary has a name.

---

# Caches

A cache stores data that can be recreated if missing.

That changes error handling.

For a cache, missing or invalid data may be harmless.

Example:

```python
import json
from pathlib import Path

def load_cache(path):
    path = Path(path)

    try:
        return json.loads(path.read_text(encoding="utf-8"))
    except (FileNotFoundError, json.JSONDecodeError):
        return {}
```

This might be acceptable because the cache is optional.

But use this pattern carefully.

Do not silently treat important user data as disposable.

Ask:

```text
Can this data be safely rebuilt?
```

If yes, tolerant loading may be fine.

If no, report the failure clearly.

---

# Logs Are Not Data Models

Logs are written for observation and diagnosis.

They are not usually the primary source of truth.

It is tempting to use append-only text logs as a simple database:

```text
2026-01-01 created user 1
2026-01-02 renamed user 1
```

This can work for tiny tools.

But log parsing becomes fragile:

* message text changes
* time formats vary
* partial lines occur
* concurrent writes interleave
* old logs rotate away

If data must be queried, updated, constrained, or recovered reliably, use a structured storage mechanism.

Logs are for telling humans and operators what happened.

Data stores are for preserving application state.

---

# Serialization and Compatibility

Once a file format exists, changing it becomes a compatibility problem.

Suppose version 1 stores:

```json
{
  "name": "Asha"
}
```

Version 2 stores:

```json
{
  "first_name": "Asha",
  "last_name": "Nair"
}
```

Can version 2 still read version 1 files?

Can version 1 read version 2 files?

Should migration happen automatically?

One common pattern is to include a version:

```json
{
  "version": 2,
  "user": {
    "first_name": "Asha",
    "last_name": "Nair"
  }
}
```

Then:

```python
def parse_document(data):
    version = data.get("version", 1)

    if version == 1:
        return parse_v1(data)
    if version == 2:
        return parse_v2(data)

    raise ValueError(f"unsupported document version: {version}")
```

Versioning is not only for public APIs.

It matters for files your own program writes if those files may outlive one release.

---

# Separating Domain Objects from File Formats

Avoid making your domain model depend too heavily on one file format.

Less flexible:

```python
@dataclass
class User:
    id: int
    name: str
    active: bool

    def to_json(self):
        return {"id": self.id, "name": self.name, "active": self.active}
```

This can be fine for small programs.

But in larger systems, it can be cleaner to keep serialization functions near the boundary:

```python
@dataclass
class User:
    id: int
    name: str
    active: bool

def encode_user(user):
    return {"id": user.id, "name": user.name, "active": user.active}

def decode_user(data):
    return User(
        id=int(data["id"]),
        name=str(data["name"]),
        active=bool(data["active"]),
    )
```

This keeps the core object focused on domain behavior.

The serializer knows about the external representation.

For small scripts, direct methods are acceptable.

For larger applications, boundaries deserve names.

---

# Streaming Serialization

Not all serialization should build the entire output first.

For a large export, avoid:

```python
rows = build_all_rows()
path.write_text(json.dumps(rows), encoding="utf-8")
```

This stores all rows in memory.

For CSV, stream rows:

```python
import csv

with open("export.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["id", "name"])

    for user in iter_users():
        writer.writerow([user.id, user.name])
```

For JSON Lines, stream records:

```python
import json

with open("events.jsonl", "w", encoding="utf-8") as f:
    for event in iter_events():
        f.write(json.dumps(event))
        f.write("\n")
```

Plain JSON arrays are less naturally streamable because commas must appear between elements.

You can still stream carefully:

```python
import json

with open("users.json", "w", encoding="utf-8") as f:
    f.write("[\n")
    first = True
    for user in iter_users():
        if not first:
            f.write(",\n")
        first = False
        f.write(json.dumps(encode_user(user)))
    f.write("\n]\n")
```

This is more error-prone than JSON Lines.

Choose a format that matches your access pattern.

---

# Working with Compressed Files

The standard library includes modules for compressed files, such as `gzip`.

Example:

```python
import gzip

with gzip.open("events.jsonl.gz", "rt", encoding="utf-8") as f:
    for line in f:
        process(line)
```

The mode `"rt"` means read text through the gzip layer.

Writing:

```python
import gzip
import json

with gzip.open("events.jsonl.gz", "wt", encoding="utf-8") as f:
    for event in events:
        f.write(json.dumps(event))
        f.write("\n")
```

The same file ideas still apply:

* use context managers
* specify encodings for text
* stream large data
* handle errors at boundaries
* validate deserialized values

Compression changes the storage layer.

It does not remove the need for good file design.

---

# Temporary Files and Directories

The `tempfile` module creates temporary files and directories safely.

Temporary files are useful for:

* intermediate processing
* tests
* safe write patterns
* external tools that require file paths
* staging generated output

Temporary directory example:

```python
from tempfile import TemporaryDirectory
from pathlib import Path

with TemporaryDirectory() as directory:
    path = Path(directory) / "data.txt"
    path.write_text("temporary data\n", encoding="utf-8")
    process(path)
```

When the `with` block exits, the temporary directory and its contents are removed.

Temporary file example:

```python
from tempfile import NamedTemporaryFile

with NamedTemporaryFile("w+", encoding="utf-8") as f:
    f.write("hello\n")
    f.seek(0)
    print(f.read())
```

Temporary files are better than manually inventing names like:

```text
temp.txt
tmp-data-final-2.txt
```

The standard library handles uniqueness and placement more safely.

---

# Seeking

File objects have a current position.

Reading or writing moves that position.

You can inspect it with `tell()`:

```python
with open("data.txt", "r", encoding="utf-8") as f:
    print(f.tell())
    text = f.read(5)
    print(f.tell())
```

You can move it with `seek()`:

```python
with open("data.txt", "r", encoding="utf-8") as f:
    first = f.read(5)
    f.seek(0)
    again = f.read(5)
```

Seeking is common in binary formats, indexes, and temporary files.

For normal text processing, line iteration is usually clearer.

Be careful mixing iteration and manual seeking unless you understand the file object's buffering behavior.

---

# Updating Files In Place

Updating text files in place is often harder than beginners expect.

Suppose you want to replace one line in a file.

Text lines may have different lengths.

Changing:

```text
name=Asha
```

to:

```text
name=Ashalata
```

requires more bytes.

That can shift everything after it.

For many text formats, the simple and safe approach is:

```text
read the file
construct the new contents
write the file safely
```

Example:

```python
def replace_setting(path, key, value):
    lines = path.read_text(encoding="utf-8").splitlines()
    prefix = key + "="
    new_lines = []

    for line in lines:
        if line.startswith(prefix):
            new_lines.append(f"{key}={value}")
        else:
            new_lines.append(line)

    path.write_text("\n".join(new_lines) + "\n", encoding="utf-8")
```

For large files, databases, indexed binary formats, or append-only logs may be better designs.

Plain text is excellent for readability.

It is not always excellent for random in-place modification.

---

# Permissions

File operations depend on permissions.

Trying to read a forbidden file may raise `PermissionError`:

```python
try:
    text = path.read_text(encoding="utf-8")
except PermissionError as error:
    raise RuntimeError(f"permission denied while reading {path}") from error
```

Trying to write in a protected directory may also raise `PermissionError`.

Permissions are environment-dependent.

Code may work:

* on your laptop
* inside a development container
* under one user account

and fail:

* in production
* under a service account
* in a read-only file system
* in a packaged application

Good programs make important paths configurable.

They also report permission failures clearly.

`Permission denied` without the path is frustrating.

`cannot write export file /var/app/reports/january.csv: permission denied` is actionable.

---

# Concurrency and Files

Files can be shared by multiple processes.

That creates coordination problems.

Examples:

* two programs write the same file
* one program reads while another writes
* one process deletes a file while another expects it
* log lines from multiple writers interleave
* a reader sees a partially written file

The temporary-file-and-replace pattern helps readers avoid partial target files.

Append-only patterns can help for some logs.

Databases handle many coordination problems better than ad hoc files.

File locking exists, but it is platform-dependent and easy to misuse.

The main design lesson:

```text
if multiple writers need coordination, a plain file may not be the right abstraction
```

For small tools, simple file coordination may be enough.

For production systems, choose storage that matches the concurrency requirements.

---

# A Practical Loader Example

Here is a more complete example that loads users from a JSON file.

It separates the work into small pieces:

```python
import json
from dataclasses import dataclass
from pathlib import Path

@dataclass(frozen=True)
class User:
    id: int
    name: str
    active: bool

class UserFileError(Exception):
    pass

def decode_user(data):
    if not isinstance(data, dict):
        raise TypeError("user must be an object")

    try:
        user_id = data["id"]
        name = data["name"]
        active = data["active"]
    except KeyError as error:
        raise ValueError(f"missing user field: {error.args[0]}") from error

    if not isinstance(user_id, int):
        raise TypeError("user id must be an integer")
    if not isinstance(name, str):
        raise TypeError("user name must be a string")
    if not isinstance(active, bool):
        raise TypeError("user active must be a boolean")

    return User(id=user_id, name=name, active=active)

def load_users(path):
    path = Path(path)

    try:
        text = path.read_text(encoding="utf-8")
    except OSError as error:
        raise UserFileError(f"cannot read user file: {path}") from error

    try:
        raw = json.loads(text)
    except json.JSONDecodeError as error:
        raise UserFileError(f"invalid JSON in user file: {path}") from error

    if not isinstance(raw, list):
        raise UserFileError("user file must contain a list")

    users = []
    for index, item in enumerate(raw):
        try:
            users.append(decode_user(item))
        except (TypeError, ValueError) as error:
            raise UserFileError(f"invalid user at index {index}") from error

    return users
```

This code is longer than a one-liner.

But it gives the program excellent failure behavior.

It can distinguish:

* file read failure
* malformed JSON
* wrong top-level shape
* missing fields
* invalid field types
* the index of the bad record

That is the difference between:

```text
something broke
```

and:

```text
the user file is invalid at index 3 because active must be a boolean
```

Good error messages are part of good file handling.

---

# A Practical Writer Example

Now write users back to JSON.

```python
import json
import os
from pathlib import Path
from tempfile import NamedTemporaryFile

def encode_user(user):
    return {
        "id": user.id,
        "name": user.name,
        "active": user.active,
    }

def write_users(path, users):
    path = Path(path)
    payload = [encode_user(user) for user in users]
    text = json.dumps(payload, indent=2, sort_keys=True) + "\n"

    path.parent.mkdir(parents=True, exist_ok=True)
    tmp_name = None

    try:
        with NamedTemporaryFile(
            "w",
            encoding="utf-8",
            dir=path.parent,
            delete=False,
        ) as tmp:
            tmp.write(text)
            tmp_name = tmp.name

        os.replace(tmp_name, path)
        tmp_name = None
    finally:
        if tmp_name is not None:
            try:
                os.unlink(tmp_name)
            except FileNotFoundError:
                pass
```

Important details:

* the JSON is built before opening the temporary file
* parent directories are created intentionally
* output is deterministic with sorted keys
* the file ends with a newline
* replacement happens after the write finishes
* abandoned temporary files are cleaned up

This is the kind of detail that makes file code boring in production.

Boring is good.

---

# Common Mistakes

Do not leave files open:

```python
f = open("data.txt")
text = f.read()
```

Use:

```python
with open("data.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

Do not rely on platform default encodings for project data:

```python
text = open("data.txt").read()
```

Use:

```python
text = Path("data.txt").read_text(encoding="utf-8")
```

Do not parse CSV by splitting commas:

```python
fields = line.split(",")
```

Use:

```python
reader = csv.reader(f)
```

Do not unpickle untrusted input:

```python
data = pickle.loads(user_supplied_bytes)
```

Use a safer format such as JSON for untrusted data.

Do not catch all file errors blindly:

```python
try:
    ...
except Exception:
    return {}
```

Handle the failures you understand.

Do not let user input become a path without validation:

```python
path = upload_dir / user_filename
```

Sanitize and constrain paths deliberately.

Do not overwrite important files casually:

```python
path.write_text(new_data, encoding="utf-8")
```

For important state, use a safer write pattern.

---

# Mental Checklist

When reading a file, ask:

* Where does the path come from?
* Is the path trusted?
* Is the file expected to exist?
* What encoding should text use?
* How large can the file be?
* Should the file be read all at once or streamed?
* What exceptions are expected?
* What message should the caller see?
* Does parsed data still need validation?

When writing a file, ask:

* Is overwriting allowed?
* Should the parent directory be created?
* Should the write be temporary-then-replace?
* What happens if serialization fails?
* What happens if the disk is full?
* Should the output be deterministic?
* Should the file be human-readable?
* Does another process read or write the same file?

When choosing a serialization format, ask:

* Is the data trusted?
* Does another language need to read it?
* Does a human need to inspect it?
* Does the format need versioning?
* Are values simple or Python-specific?
* Is the data record-oriented, tabular, or tree-shaped?
* How long must the file remain compatible?

These questions are not bureaucracy.

They are how you avoid hidden assumptions at boundaries.

---

# Exercises

1. Write a function `count_non_empty_lines(path)` that reads a text file line by line and returns the number of non-empty lines.

2. Write a function `load_json_file(path)` that distinguishes `FileNotFoundError` from `json.JSONDecodeError` and raises clear `ValueError` messages.

3. Write a function `save_json_file(path, data)` that writes pretty JSON with a trailing newline.

4. Modify `save_json_file` to write to a temporary file first and then replace the target file.

5. Write a CSV importer that requires `name` and `email` columns and reports the line number for missing or invalid fields.

6. Write a JSON Lines reader that skips blank lines but reports malformed JSON with a line number.

7. Create a dataclass `Product` and write `encode_product` and `decode_product` functions for JSON serialization.

8. Try serializing a `datetime` object with `json.dumps()`. Then write a custom `default` function that converts datetimes to ISO strings.

9. Create a pickle file from a trusted local object, then change the class name and observe what happens when loading it.

10. Write a function that accepts a text stream rather than a path. Test it with both a real file and `StringIO`.

---

# Summary

Files are persistent data managed by the operating system.

Python file objects are handles to external resources, so they should be closed reliably.

Use `with` statements for file handling.

Use `pathlib.Path` for path manipulation.

Prefer explicit encodings, especially UTF-8, for text files.

Distinguish text mode from binary mode.

Do not load huge files into memory unless the program truly needs the entire file.

Stream large files line by line or chunk by chunk.

Write mode truncates existing files, so use it carefully.

Exclusive creation mode helps avoid accidental overwrites.

File checks such as `exists()` are useful, but operations can still fail.

Handle file exceptions where you can add meaning or recover safely.

JSON is a language-neutral format for simple structured data.

CSV is for tabular text data, and it should be parsed with the `csv` module.

Pickle is powerful but unsafe for untrusted input and unsuitable as a public long-term format.

Serialization is not validation.

Parsing tells you whether data has the right syntax.

Validation tells you whether data is meaningful for your program.

For important files, consider temporary-write-and-replace patterns.

For user-supplied paths, guard against path traversal and unsafe file names.

The deep lesson is:

```text
files are boundaries, and boundaries deserve explicit contracts
```

If your file code clearly states paths, encodings, formats, failure behavior, and validation rules, it becomes much easier to trust.

---

# Preview of Chapter 64

Chapter 63 focused on files and serialization.

We saw how Python crosses the boundary between memory and persistent external data.

Next we move into a standard library deep dive.

The standard library is one of Python's greatest strengths.

It gives you production-ready tools for common problems without reaching immediately for third-party packages.

Chapter 64 will study important standard-library modules and patterns, including:

* `collections`
* `itertools`
* `functools`
* `math`
* `statistics`
* `datetime`
* `pathlib`
* `tempfile`
* `shutil`
* `subprocess`
* `argparse`
* `logging`

The point will not be to memorize every function.

The point will be to understand how to recognize standard-library tools and compose them with the language features we have already built.

The transition is:

```text
files teach boundary discipline
the standard library teaches practical leverage
```

Professional Python is not only knowing syntax.

It is knowing when Python already gives you the right tool.
