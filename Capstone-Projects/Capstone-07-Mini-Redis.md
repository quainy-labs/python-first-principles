# Capstone 07 - Mini Redis

## Project Brief

In this project, you will build a small Redis-like server in Python.

Redis is commonly described as an in-memory data store.

That description is correct, but incomplete.

Redis is also a network server, a command protocol, a data structure engine, a cache, a coordination tool, a persistence system, and a practical lesson in careful systems design.

This capstone will build a smaller version.

The final project will support:

```text
a TCP server
multiple client connections
a simple command protocol
PING
SET
GET
DEL
EXISTS
INCR
EXPIRE
TTL
KEYS
basic lists
optional persistence
clear error responses
tests for protocol, store, expiration, and server behavior
```

This project is not a production replacement for Redis.

It does not need clustering, replication, Lua scripting, streams, pub/sub, ACLs, modules, or Redis Cluster behavior.

It is a learning implementation.

The goal is to understand how a networked in-memory data store works from first principles.

By the end, the reader should understand how sockets accept clients, how bytes become commands, how commands manipulate an in-memory store, how expiration works, and why real Redis is both simple in concept and extremely sophisticated in execution.

---

# Why This Project Matters

Most applications need fast shared state.

A web application may need to store:

```text
session data
temporary tokens
rate limit counters
cache entries
background job metadata
leader election locks
recent activity
short-lived feature flags
```

A relational database can store these things.

But not every piece of state needs a relational model.

Sometimes the application needs a fast key-value lookup.

Sometimes it needs a counter.

Sometimes it needs a queue-like list.

Sometimes it needs data that disappears automatically after a short time.

Redis became popular because it made these operations fast and simple.

The user sends a command:

```text
SET name Narendra
```

The server replies:

```text
OK
```

The user sends:

```text
GET name
```

The server replies:

```text
Narendra
```

Behind that simple interaction are several important ideas:

```text
network sockets
byte streams
protocol parsing
command dispatch
shared in-memory state
expiration
concurrency
error handling
server loops
testing network behavior
```

This capstone teaches those ideas through a working system.

---

# The Mental Model

A Mini Redis server has four main layers.

The first layer is the network layer.

It accepts TCP clients.

It reads bytes from sockets.

It writes bytes back.

The second layer is the protocol layer.

It turns bytes into commands.

It turns responses into bytes.

The third layer is the command layer.

It decides what each command means.

The fourth layer is the storage layer.

It holds keys, values, types, and expiration metadata.

The flow looks like this:

```text
client socket
    -> bytes
    -> protocol parser
    -> command object
    -> command dispatcher
    -> in-memory store
    -> response object
    -> encoded bytes
    -> client socket
```

This is a powerful structure.

Many servers follow the same shape.

HTTP servers follow it.

Database servers follow it.

Message brokers follow it.

Language servers follow it.

The details differ, but the pattern is everywhere.

---

# What You Will Build

You will build a Python package named `miniredis`.

Suggested structure:

```text
miniredis/
    __init__.py
    protocol.py
    store.py
    commands.py
    server.py
    persistence.py
    cli.py
tests/
    test_protocol.py
    test_store.py
    test_commands.py
    test_expiration.py
    test_server.py
```

The server should be runnable from the command line:

```bash
python -m miniredis --host 127.0.0.1 --port 6380
```

A client can connect using `nc`:

```bash
nc 127.0.0.1 6380
```

Then type:

```text
PING
SET name Narendra
GET name
INCR visits
EXPIRE name 30
TTL name
```

The server should respond in a predictable protocol.

For this project, you can implement a simple line protocol first.

Then you can optionally add a Redis RESP-like protocol.

---

# Protocol Choice

Real Redis uses RESP.

RESP stands for Redis Serialization Protocol.

It can encode simple strings, errors, integers, bulk strings, arrays, and null values.

For example, a Redis command may be sent as:

```text
*2
$3
GET
$4
name
```

This is excellent for real clients.

But for a learning project, starting with a line protocol is clearer.

The first version can parse:

```text
COMMAND arg1 arg2 arg3
```

Each command ends with a newline.

Examples:

```text
PING
SET name Narendra
GET name
DEL name
```

The response can also be line-based:

```text
+OK
$8 Narendra
:1
-ERR unknown command
_NULL
```

This is not exactly Redis.

It is intentionally simpler.

The important lesson is protocol design:

```text
clients and servers need an agreed byte format
```

Once that is understood, RESP can be added as an extension.

---

# Requirements

The Mini Redis server must:

```text
listen on a TCP socket
accept client connections
read commands ending in newlines
parse commands into name and arguments
dispatch commands by command name
store values in memory
support string keys and string values
support integer counters
support key expiration
return clear success, error, integer, string, and null responses
handle malformed commands without crashing
allow multiple commands on the same connection
allow multiple clients, at least with threads
shut down cleanly when interrupted
have tests for store and command logic
```

The server should be small but real.

A reader should be able to run it, connect from another terminal, issue commands, and see state persist while the server process remains alive.

---

# Non-Requirements

The project does not need to match Redis perfectly.

It does not need binary-safe values in the first version.

It does not need authentication.

It does not need clustering.

It does not need replication.

It does not need eviction policies.

It does not need transactions.

It does not need pub/sub.

It does not need streams.

It does not need Lua scripting.

It does not need sorted sets.

It does not need append-only-file compatibility.

These topics are valuable, but they would turn the project into a full database course.

The capstone should stay focused on the foundational ideas.

---

# Commands To Support

Start with these commands:

```text
PING
SET key value
GET key
DEL key
EXISTS key
INCR key
EXPIRE key seconds
TTL key
KEYS
```

Then add list commands:

```text
LPUSH key value
RPUSH key value
LPOP key
RPOP key
LRANGE key start stop
LLEN key
```

These commands teach two different storage models:

```text
strings
lists
```

That is enough for a serious learning project.

The reader sees that Redis is not only a key-value string store.

It is a key-to-data-structure store.

---

# Response Types

Define explicit response objects.

For example:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class SimpleString:
    value: str

@dataclass(frozen=True)
class Error:
    message: str

@dataclass(frozen=True)
class Integer:
    value: int

@dataclass(frozen=True)
class BulkString:
    value: str | None

@dataclass(frozen=True)
class Array:
    values: list[object]
```

The protocol encoder turns these into bytes.

This is cleaner than returning raw strings everywhere.

Command functions can return meaningful response objects.

The protocol layer decides how to represent them on the wire.

---

# Simple Line Encoding

A simple response encoding can be:

```text
+OK
-ERR message
:123
$5 hello
_NULL
*2
$3 one
$3 two
```

Meaning:

```text
+ means simple string
- means error
: means integer
$ means bulk string
_NULL means no value
* means array, followed by encoded values
```

This is inspired by Redis RESP, but simplified.

The main rule is that the client can tell what kind of response it received.

Avoid ambiguous responses.

For example, if `GET missing` returns:

```text
None
```

is that the string `"None"` or a missing value?

Use a clear null response.

---

# Parsing Commands

The first parser can be simple.

It reads one line.

It strips the newline.

It splits by spaces.

```python
def parse_command(line: str) -> Command:
    parts = line.strip().split()
    if not parts:
        raise ProtocolError("empty command")
    name = parts[0].upper()
    args = parts[1:]
    return Command(name=name, args=args)
```

This parser cannot handle values with spaces.

That is acceptable for the first version.

The chapter should name the limitation clearly.

If a reader wants values with spaces, they can add quoted string parsing or RESP bulk strings later.

The first version should optimize for clarity.

---

# The Command Object

Represent parsed commands with a dataclass.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Command:
    name: str
    args: list[str]
```

This small object gives the rest of the code a stable shape.

The server does not pass raw text into storage.

It passes structured commands into dispatch.

That keeps responsibilities separate.

---

# The Store

The store is the in-memory database.

It can start as two dictionaries:

```python
class Store:
    def __init__(self, clock=monotonic_time):
        self._values = {}
        self._expires_at = {}
        self._clock = clock
```

`_values` maps keys to stored values.

`_expires_at` maps keys to expiration timestamps.

Use monotonic time for expiration.

```python
import time

def monotonic_time() -> float:
    return time.monotonic()
```

Monotonic time moves forward steadily.

It is better for measuring durations than wall-clock time.

Wall-clock time can jump because of system clock changes.

Expiration is based on elapsed seconds, so monotonic time is the right tool.

---

# Stored Values

Redis stores typed values.

A key may hold a string.

A key may hold a list.

A key may hold a set.

In this capstone, support strings and lists.

Represent values explicitly:

```python
@dataclass
class StringValue:
    value: str

@dataclass
class ListValue:
    values: list[str]
```

This is better than storing raw Python objects without type wrappers.

If the code sees `StringValue`, it knows string commands are valid.

If the code sees `ListValue`, it knows list commands are valid.

If a user runs `INCR` on a list, the store can raise a clear wrong-type error.

---

# Expiration

Expiration means a key disappears after a deadline.

For example:

```text
SET token abc123
EXPIRE token 60
```

The key exists for 60 seconds.

After that, `GET token` returns null.

There are two common expiration strategies.

The first is active expiration.

The server periodically scans for expired keys and deletes them.

The second is lazy expiration.

The server checks expiration when a key is accessed.

This project should start with lazy expiration.

When a command touches a key, the store checks:

```python
def _delete_if_expired(self, key):
    expires_at = self._expires_at.get(key)
    if expires_at is not None and expires_at <= self._clock():
        self._values.pop(key, None)
        self._expires_at.pop(key, None)
```

Lazy expiration is simple and effective for a teaching project.

It does mean an expired key may remain in memory until accessed.

That is acceptable here.

An extension can add active cleanup.

---

# TTL

`TTL` means time to live.

It reports how many seconds remain before a key expires.

Common behavior:

```text
TTL missing key      -> -2
TTL key no expiry    -> -1
TTL key with expiry  -> remaining seconds
```

This is useful because the client can distinguish:

```text
missing
existing but permanent
existing and expiring
```

The mini implementation can follow that shape.

When computing remaining seconds, round down or up deliberately.

Use one consistent rule.

For example:

```python
remaining = max(0, int(expires_at - now))
```

Tests should pin the behavior.

---

# String Commands

`SET` stores a string value.

```text
SET name Narendra
```

Response:

```text
+OK
```

`GET` returns a string value or null.

```text
GET name
```

Response:

```text
$8 Narendra
```

`DEL` removes a key.

It returns the number of keys removed.

For this project, support one key:

```text
DEL name
```

Response:

```text
:1
```

If the key did not exist:

```text
:0
```

`EXISTS` returns 1 if the key exists, otherwise 0.

These commands are small, but they establish the full path from protocol to store to response.

---

# INCR

`INCR` is a beautiful command for learning.

It looks simple:

```text
INCR visits
```

But it teaches several rules.

If the key is missing, treat it as 0 and store 1.

If the key contains an integer string, increment it.

If the key contains a non-integer string, return an error.

If the key contains a list, return a wrong-type error.

Examples:

```text
INCR visits      -> :1
INCR visits      -> :2
SET visits ten
INCR visits      -> -ERR value is not an integer
```

`INCR` is useful for counters:

```text
page views
rate limiting
login attempts
usage tracking
```

The implementation should be atomic within the store method.

If multiple client threads call `INCR`, protect the store with a lock.

---

# Lists

Lists are the second data type.

They teach that a Redis key can refer to a structure, not only a scalar value.

`LPUSH` inserts values at the left side.

```text
LPUSH tasks build
LPUSH tasks test
```

The list becomes:

```text
test, build
```

`RPUSH` inserts values at the right side.

```text
RPUSH tasks deploy
```

The list becomes:

```text
test, build, deploy
```

`LPOP` removes and returns the leftmost value.

`RPOP` removes and returns the rightmost value.

`LLEN` returns the list length.

`LRANGE` returns a slice.

These commands are enough to build queue-like behavior.

The reader will see how the earlier task queue could use a Redis list as a broker in a more production-like design.

---

# LRANGE Semantics

`LRANGE key start stop` returns elements from index `start` through index `stop`, inclusive.

That inclusive stop differs from Python slicing.

Python slicing excludes the stop index:

```python
items[start:stop]
```

Redis includes it:

```text
LRANGE numbers 0 2
```

returns indexes 0, 1, and 2.

The implementation can adapt:

```python
values[start:stop + 1]
```

Negative indexes are useful but optional.

If you support them, test them carefully.

The important lesson is that external protocols have their own semantics.

Do not assume they match Python exactly.

---

# Wrong Type Errors

If a key stores a string and the user runs a list command, return an error.

Example:

```text
SET name Narendra
LPUSH name value
```

Response:

```text
-ERR wrong type
```

This is better than silently replacing the string with a list.

Databases should not surprise users by changing data types implicitly.

Clear errors protect data.

---

# Command Dispatch

The command layer maps command names to functions.

```python
class CommandHandler:
    def __init__(self, store):
        self.store = store
        self.handlers = {
            "PING": self.ping,
            "SET": self.set,
            "GET": self.get,
            "DEL": self.delete,
            "EXISTS": self.exists,
            "INCR": self.incr,
            "EXPIRE": self.expire,
            "TTL": self.ttl,
            "KEYS": self.keys,
        }

    def handle(self, command):
        handler = self.handlers.get(command.name)
        if handler is None:
            return Error(f"unknown command: {command.name}")
        return handler(command.args)
```

Each handler validates argument counts.

For example:

```python
def get(self, args):
    if len(args) != 1:
        return Error("GET requires 1 argument")
    value = self.store.get(args[0])
    return BulkString(value)
```

Argument validation belongs near the command layer.

The store should receive clean method calls.

---

# Store Errors

Use explicit exceptions inside the store.

```python
class StoreError(Exception):
    pass

class WrongTypeError(StoreError):
    pass

class InvalidIntegerError(StoreError):
    pass
```

The command handler catches these and returns protocol errors.

```python
try:
    value = self.store.incr(key)
except InvalidIntegerError:
    return Error("value is not an integer")
except WrongTypeError:
    return Error("wrong type")
```

This separation is useful.

The store speaks Python exceptions.

The protocol speaks response objects.

The server speaks bytes.

Each layer uses the language that makes sense for it.

---

# Thread Safety

If the server handles each client in a separate thread, the store is shared between threads.

That means the store must be protected.

Use `threading.RLock` or `threading.Lock`.

```python
import threading

class Store:
    def __init__(self, clock=monotonic_time):
        self._values = {}
        self._expires_at = {}
        self._clock = clock
        self._lock = threading.RLock()
```

Each public store method should lock:

```python
def set(self, key, value):
    with self._lock:
        self._delete_if_expired(key)
        self._values[key] = StringValue(value)
        self._expires_at.pop(key, None)
```

This prevents two client threads from corrupting shared state.

The Global Interpreter Lock does not remove the need for logical locks.

The GIL does not make multi-step operations atomic at the application level.

This is an important lesson.

---

# TCP Server

Use Python's `socket` module.

The basic server shape is:

```python
import socket

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((host, port))
    server.listen()
    while True:
        client, address = server.accept()
        handle_client(client, address)
```

This single-client version is a good first milestone.

Then add threads:

```python
thread = threading.Thread(
    target=handle_client,
    args=(client, address),
    daemon=True,
)
thread.start()
```

Each client gets its own handler.

The store remains shared.

That gives the project real multi-client behavior without introducing `asyncio` yet.

The upcoming Mini Event Loop capstone will revisit this area from a different angle.

---

# Client Handler

A client handler should:

```text
wrap the socket in a file-like object
read one line at a time
parse command
dispatch command
encode response
write response
continue until client disconnects
```

Example shape:

```python
def handle_client(sock, address, handler):
    with sock:
        reader = sock.makefile("r", encoding="utf-8", newline="\n")
        writer = sock.makefile("w", encoding="utf-8", newline="\n")
        for line in reader:
            response = handle_line(line, handler)
            writer.write(encode_response(response))
            writer.flush()
```

This is easier than repeatedly calling `recv` and manually splitting buffers.

The chapter should still explain that TCP is a byte stream.

Lines are not built into TCP.

The file wrapper is doing buffering and line splitting for us.

---

# TCP Is A Stream

This is one of the most important lessons in the project.

TCP does not send messages.

TCP sends a stream of bytes.

If a client writes:

```text
SET a 1\nGET a\n
```

The server might receive:

```text
SET a 1\nGET a\n
```

Or:

```text
SET a
```

then later:

```text
 1\nGET a\n
```

Or any other chunking.

The protocol must define how messages end.

In this capstone, messages end with a newline.

That is why the server reads lines.

This one idea unlocks a lot of network programming.

---

# Clean Shutdown

The server should stop cleanly on `KeyboardInterrupt`.

A simple implementation can catch it in the CLI:

```python
try:
    server.serve_forever()
except KeyboardInterrupt:
    print("Server stopped")
```

For a more testable server, store a flag:

```python
class Server:
    def __init__(...):
        self._stopping = threading.Event()

    def stop(self):
        self._stopping.set()
```

The accept loop can use a timeout:

```python
server.settimeout(0.5)
```

Then it periodically checks whether it should stop.

This avoids a test hanging forever.

Network servers should be designed so tests can start and stop them.

---

# Persistence Options

Redis is usually in-memory, but it can persist data.

For this capstone, persistence is optional.

If you add it, use a simple JSON snapshot file.

Commands:

```text
SAVE
LOAD
```

Or save on shutdown and load on startup.

The snapshot might store:

```json
{
  "values": {
    "name": {"type": "string", "value": "Narendra"},
    "tasks": {"type": "list", "values": ["build", "test"]}
  },
  "expires_at": {
    "name": 12345.67
  }
}
```

Be careful with expiration timestamps.

If expiration uses monotonic time, snapshots cannot safely persist monotonic deadlines across process restarts.

Monotonic time has meaning only inside the current process.

For persistence, use wall-clock UTC expiration times or store remaining TTL at save time.

This is a wonderful design discussion.

The main implementation can skip persistence.

If it includes persistence, the chapter should explain this time difference clearly.

---

# Expiration And Persistence

Suppose a key expires in 60 seconds.

The server saves a snapshot and exits immediately.

The server restarts 10 minutes later.

Should the key exist?

Probably not.

This means a persisted expiration should represent an absolute wall-clock deadline, not only a process-local monotonic timestamp.

One practical design is:

```text
use monotonic time internally for live TTL checks
when saving, convert remaining TTL into a UTC expiration deadline
when loading, convert the UTC deadline back into an internal expiration deadline
```

That is more complex than the first version needs.

This is why persistence is optional.

The reader should know the tradeoff before adding it.

---

# KEYS

`KEYS` returns the current keys.

In real Redis, `KEYS *` can be dangerous on large production databases because it scans the keyspace.

In this capstone, the store is small.

`KEYS` is useful for learning and debugging.

Before returning keys, delete expired keys.

```python
def keys(self):
    with self._lock:
        self._delete_expired_keys()
        return sorted(self._values.keys())
```

Sorting is not required, but it makes tests deterministic.

Deterministic tests are easier to read.

---

# Active Expiration

Lazy expiration deletes expired keys only when touched.

Active expiration deletes expired keys in the background.

For this capstone, active expiration can be an extension.

It might run in a thread:

```python
while not stopping:
    store.delete_expired_keys()
    time.sleep(1)
```

This teaches background maintenance.

But it also introduces synchronization and shutdown behavior.

Keep it optional unless the reader is ready.

---

# Testing The Protocol

Protocol tests should not need sockets.

Test parser functions directly.

Examples:

```text
PING\n -> Command("PING", [])
set name Narendra\n -> Command("SET", ["name", "Narendra"])
empty line -> protocol error
```

Test encoder functions directly.

Examples:

```text
SimpleString("OK") -> "+OK\n"
Integer(3) -> ":3\n"
BulkString(None) -> "_NULL\n"
Error("bad") -> "-ERR bad\n"
```

This gives fast feedback.

The protocol layer is pure logic.

Pure logic should have pure tests.

---

# Testing The Store

Store tests should focus on behavior.

Examples:

```text
set then get returns value
get missing returns None
del existing returns 1
del missing returns 0
exists respects expiration
incr missing creates counter
incr integer string increments
incr non-integer returns error
expire missing returns 0
expire existing returns 1
ttl missing returns -2
ttl no expiry returns -1
ttl expiring key returns remaining seconds
lpush creates list
rpush appends
lpop removes left
rpop removes right
lrange returns inclusive range
wrong type raises clear error
```

Use a fake clock.

```python
class FakeClock:
    def __init__(self):
        self.now = 0.0

    def __call__(self):
        return self.now

    def advance(self, seconds):
        self.now += seconds
```

This avoids slow tests.

No test should need to actually wait 60 seconds.

---

# Testing Commands

Command tests should check argument validation.

Examples:

```text
GET with no args returns error
GET with two args returns error
SET with missing value returns error
INCR with one arg works
EXPIRE with non-integer seconds returns error
unknown command returns error
```

These tests protect the boundary between protocol and store.

They also make the server friendlier.

A client mistake should not crash the process.

It should produce a clear error response.

---

# Testing The Server

Server tests are slower and more fragile than pure tests, so keep them focused.

Start the server on port `0`.

Port `0` asks the operating system to choose a free port.

Then read the actual port from the socket.

Use a client socket to send:

```text
PING\n
```

Assert the response is:

```text
+PONG
```

Then send:

```text
SET name Narendra\n
GET name\n
```

Assert the server responds correctly.

Make sure the test stops the server.

Avoid tests that leave background threads running.

---

# Milestone 1 - Protocol

Create `protocol.py`.

Implement:

```text
Command dataclass
response dataclasses
parse_command
encode_response
ProtocolError
```

Add tests for parsing and encoding.

At this stage there is no store and no socket server.

That is good.

The wire format is already becoming concrete.

---

# Milestone 2 - Store Strings

Create `store.py`.

Implement:

```text
set
get
delete
exists
keys
```

Use typed values internally.

Add lazy expiration hooks even if `EXPIRE` is not implemented yet.

Write tests for normal string behavior.

The store should not know about command text.

It should expose Python methods.

---

# Milestone 3 - Expiration

Add:

```text
expire
ttl
delete expired key helper
delete all expired keys helper
```

Use an injectable clock.

Test:

```text
key exists before expiration
key disappears after expiration
ttl reports missing
ttl reports no expiry
ttl reports remaining seconds
keys omits expired keys
```

Expiration is a small feature with large design implications.

Take it seriously.

---

# Milestone 4 - Counters

Add `incr`.

Rules:

```text
missing key -> create "1"
integer string -> increment
non-integer string -> error
list value -> wrong type error
```

Test each rule.

This milestone teaches atomic read-modify-write behavior.

Make sure the method locks around the whole operation.

---

# Milestone 5 - Lists

Add list commands to the store:

```text
lpush
rpush
lpop
rpop
llen
lrange
```

Test creation, mutation, popping from empty lists, and wrong-type behavior.

Decide what should happen when popping the last element.

It is reasonable to leave the empty list stored.

It is also reasonable to delete the key.

Choose one behavior and document it.

Consistency matters more than matching every Redis detail.

---

# Milestone 6 - Command Handler

Create `commands.py`.

Implement `CommandHandler`.

It receives a `Command` object and returns a response object.

Add handlers for all supported commands.

Every handler should validate argument count.

Every store exception should become an error response.

Unknown commands should become error responses.

At the end of this milestone, you should be able to test the system without sockets:

```python
store = Store()
handler = CommandHandler(store)

response = handler.handle(Command("SET", ["name", "Narendra"]))
assert response == SimpleString("OK")
```

This is a clean design.

---

# Milestone 7 - Single-Client Server

Create `server.py`.

Implement a TCP server that handles one client at a time.

This first version can be blocking.

It should:

```text
bind host and port
listen
accept one client
read lines
parse commands
dispatch commands
write responses
```

Use this to learn the socket flow.

Do not add threads until the simple version works.

---

# Milestone 8 - Multi-Client Server

Add one thread per client.

The accept loop continues accepting clients.

Each client thread handles commands for one socket.

The shared store must be locked.

Test manually:

```text
terminal 1: start server
terminal 2: connect and SET name Narendra
terminal 3: connect and GET name
```

The third terminal should see the value set by the second terminal.

That proves shared process state.

---

# Milestone 9 - CLI

Create `cli.py`.

Support:

```bash
python -m miniredis --host 127.0.0.1 --port 6380
```

Options:

```text
--host
--port
--snapshot
```

Snapshot can be optional.

The CLI should print:

```text
Mini Redis listening on 127.0.0.1:6380
```

This is helpful during manual testing.

---

# Milestone 10 - Optional Snapshot Persistence

Add persistence only after the in-memory server works.

Implement:

```text
store.to_snapshot()
Store.from_snapshot(...)
save_snapshot(path)
load_snapshot(path)
```

Decide how expiration should persist.

Document the decision.

Test string and list round trips.

If expiration is too complex, skip expiring keys during snapshot save or save remaining TTL.

The key is to explain the tradeoff.

---

# Common Mistake 1 - Treating TCP Like Messages

Do not assume one `recv` equals one command.

This is wrong:

```python
data = sock.recv(1024)
command = parse_command(data.decode())
```

It may work during a tiny demo.

It will break eventually.

TCP is a stream.

Use line-based reading or implement a real buffer parser.

---

# Common Mistake 2 - No Type System In The Store

If the store uses raw Python values casually, wrong-type behavior becomes confusing.

For example:

```python
self._values[key] = "hello"
self._values[key] = ["a", "b"]
```

This can work, but explicit wrappers make errors clearer.

Use `StringValue` and `ListValue`.

Then every command can check the value type deliberately.

---

# Common Mistake 3 - Forgetting Expiration Checks

If `GET` checks expiration but `EXISTS` does not, the database becomes inconsistent.

The key appears missing in one command and present in another.

Every command that observes a key must respect expiration.

Centralize expiration checks inside store helpers.

Do not duplicate expiration logic in every command handler.

---

# Common Mistake 4 - Trusting The GIL

The GIL does not make your store logically safe.

This operation is not one indivisible database operation:

```python
value = self._values[key]
value += 1
self._values[key] = value
```

Another thread may interleave at the wrong moment.

Use a lock around multi-step operations.

The lock documents intent.

It also prepares the reader for concurrency outside CPython.

---

# Common Mistake 5 - Letting Client Errors Kill The Server

A malformed command should not crash the process.

This input:

```text
GET
```

should return an error.

This input:

```text
EXPIRE name abc
```

should return an error.

The server should keep running.

Network servers live in a world of imperfect clients.

Robust error handling is part of the contract.

---

# Common Mistake 6 - Testing Only Through Sockets

Socket tests are useful, but they should not be the only tests.

Most behavior is easier to test directly:

```text
parser tests
encoder tests
store tests
command handler tests
```

Then use a small number of server tests to prove integration.

This keeps the suite fast and reliable.

---

# Extension Ideas

Add RESP parsing.

Add `MGET` and `MSET`.

Add `APPEND`.

Add `SET key value EX seconds`.

Add `PERSIST` to remove expiration.

Add sets:

```text
SADD
SREM
SMEMBERS
SISMEMBER
```

Add hashes:

```text
HSET
HGET
HGETALL
```

Add pub/sub.

Add snapshot persistence.

Add append-only persistence.

Add active expiration.

Add max memory and eviction policy.

Add a small Python client library.

Add benchmark scripts.

Add asyncio server mode.

Add command latency metrics.

Add a compatibility mode for a tiny subset of Redis RESP.

Each extension opens a real systems topic.

Do not add all of them at once.

Build the small server first.

Then grow it deliberately.

---

# What This Project Teaches

This project teaches network programming.

It teaches that sockets move bytes, not Python objects.

It teaches protocol parsing.

It teaches response encoding.

It teaches command dispatch.

It teaches in-memory data modeling.

It teaches expiration and TTL.

It teaches thread safety.

It teaches why shared state needs locks.

It teaches how tests can be layered from pure functions to integration behavior.

It teaches why real data stores have carefully defined command semantics.

It also connects several earlier parts of the book:

```text
dictionaries become key-value storage
lists become server-side data structures
exceptions become protocol errors
threads become client concurrency
time handling becomes expiration
serialization becomes persistence
testing becomes confidence in systems behavior
```

The reader should leave with a strong new mental model:

```text
A database server is a loop that turns bytes into safe state transitions.
```

That sentence is simple.

It is also the heart of the project.

---

# Completion Checklist

The project is complete when:

```text
The server starts from the CLI.
The server listens on a TCP port.
Clients can connect using nc or a socket client.
PING returns PONG.
SET stores string values.
GET returns stored values.
GET missing returns null.
DEL removes keys.
EXISTS respects expiration.
INCR handles counters correctly.
EXPIRE sets key deadlines.
TTL distinguishes missing, permanent, and expiring keys.
KEYS returns current non-expired keys.
LPUSH and RPUSH create and update lists.
LPOP and RPOP remove list values.
LRANGE returns list ranges.
Wrong-type commands return clear errors.
Malformed commands return clear errors.
Multiple commands can run on one connection.
Multiple clients can use the server.
The shared store is protected by a lock.
Protocol tests cover parsing and encoding.
Store tests cover strings, counters, lists, and expiration.
Command tests cover validation and errors.
At least one server integration test proves socket behavior.
The README explains protocol limitations.
```

When all of these are true, the reader has built a real miniature in-memory network data store.

---

# Exercises

1. Create the `miniredis` package structure.

2. Implement the `Command` dataclass.

3. Implement response dataclasses.

4. Implement `parse_command`.

5. Implement `encode_response`.

6. Add protocol tests.

7. Implement typed stored values.

8. Implement `Store.set`.

9. Implement `Store.get`.

10. Implement `Store.delete`.

11. Implement `Store.exists`.

12. Implement `Store.keys`.

13. Add a fake clock for tests.

14. Implement `expire`.

15. Implement `ttl`.

16. Test lazy expiration.

17. Implement `incr`.

18. Test invalid integer errors.

19. Implement list storage.

20. Implement `lpush` and `rpush`.

21. Implement `lpop` and `rpop`.

22. Implement `llen`.

23. Implement `lrange`.

24. Test wrong-type errors.

25. Implement `CommandHandler`.

26. Add command validation tests.

27. Implement a single-client TCP server.

28. Add threaded client handling.

29. Add CLI startup.

30. Add one socket integration test.

31. Document protocol limitations.

32. Add optional snapshot persistence.

---

# Preview of Capstone 08

Capstone 07 built a Mini Redis.

It introduced sockets, command protocols, response encoding, in-memory storage, typed values, expiration, TTL, thread-safe shared state, and network server testing.

Capstone 08 will build a Mini Web Framework.

The Mini Web Framework project will move from a data server to an HTTP application framework.

It will introduce routing, request objects, response objects, middleware, templates, static files, error handling, WSGI or ASGI concepts, and framework ergonomics.

The transition is:

```text
Mini Redis speaks a custom command protocol over TCP
Mini Web Framework speaks HTTP and organizes application code
```

The next project will show how Python turns web requests into structured application behavior.
