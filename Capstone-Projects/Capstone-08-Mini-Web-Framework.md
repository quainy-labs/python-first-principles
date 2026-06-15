# Capstone 08 - Mini Web Framework

## Project Brief

In this project, you will build a small Python web framework.

A web framework is the layer that turns incoming HTTP requests into structured application behavior.

It lets a developer define routes, receive request objects, return response objects, render templates, serve static files, handle errors, and compose reusable middleware.

This project will build a framework that feels familiar after using Flask, Django, FastAPI, Starlette, Bottle, or Sanic, but is small enough to understand from first principles.

The final project will support:

```text
route registration
path parameters
HTTP methods
request objects
response objects
JSON responses
HTML responses
redirects
middleware
error handlers
static files
simple templates
a development server
tests for routing, requests, responses, middleware, and errors
```

This project is not a production replacement for Flask, Django, FastAPI, Starlette, or any mature framework.

It is a learning implementation.

The goal is to understand the moving pieces hidden behind a simple decorator like:

```python
@app.get("/users/{user_id}")
def get_user(request, user_id):
    return {"id": user_id}
```

By the end, the reader should understand how HTTP requests become Python objects, how routes match paths, how handlers produce responses, how middleware wraps application behavior, and why framework design is mostly about clean boundaries.

---

# Why This Project Matters

Many Python developers first meet web development through a framework.

They write:

```python
from flask import Flask

app = Flask(__name__)

@app.get("/")
def home():
    return "Hello"
```

Then they run a server and open a browser.

That is a wonderful experience.

But it can also hide the important parts.

What is a route?

What is a request?

What does a handler return?

Who decides status codes?

Where do headers come from?

How does middleware run before and after a route?

How does the framework turn a dictionary into JSON?

How does it return a 404?

This capstone answers those questions by making the framework visible.

The reader will build the machinery rather than only use it.

That makes real frameworks easier to understand and debug.

---

# The Mental Model

A web framework receives an HTTP request.

It finds the matching route.

It calls the route handler.

It converts the result into an HTTP response.

The central flow is:

```text
HTTP request
    -> server adapter
    -> Request object
    -> middleware chain
    -> router
    -> handler function
    -> Response object
    -> server adapter
    -> HTTP response
```

The framework itself is not necessarily the web server.

That distinction matters.

A web server speaks sockets and HTTP.

A framework organizes application logic.

In this capstone, you can use Python's standard library HTTP server for the low-level server.

That lets the project focus on the framework layer.

The earlier Mini Redis project already taught raw TCP.

This project should teach HTTP application design.

---

# What You Will Build

You will build a package named `miniweb`.

Suggested structure:

```text
miniweb/
    __init__.py
    app.py
    routing.py
    request.py
    response.py
    middleware.py
    templates.py
    static.py
    server.py
    exceptions.py
tests/
    test_routing.py
    test_request.py
    test_response.py
    test_app.py
    test_middleware.py
    test_errors.py
```

The user-facing API should be pleasant:

```python
from miniweb import App, JSONResponse

app = App()

@app.get("/")
def home(request):
    return "Hello from Mini Web"

@app.get("/users/{user_id}")
def get_user(request, user_id):
    return {"id": user_id}

@app.post("/users")
def create_user(request):
    data = request.json()
    return JSONResponse(data, status=201)

app.run(host="127.0.0.1", port=8000)
```

This API looks simple.

The project will show everything required to make it work.

---

# Requirements

The mini framework must:

```text
register routes with decorators
support GET and POST at minimum
match exact paths
match parameterized paths like /users/{user_id}
extract path parameters
represent incoming requests with a Request object
represent outgoing responses with a Response object
convert strings to HTML/text responses
convert dictionaries and lists to JSON responses
support explicit Response subclasses
return 404 for unmatched routes
return 405 for unsupported methods on matched paths
support middleware
support custom error handlers
serve static files from a configured directory
render simple templates
run using a development server
have tests for routing, response conversion, middleware, and errors
```

The framework should be synchronous.

Async support belongs later, after the Mini Event Loop capstone.

The framework should avoid hidden magic.

Every major behavior should be readable in the code.

---

# Non-Requirements

The framework does not need production-grade security.

It does not need HTTP/2.

It does not need WebSockets.

It does not need ASGI concurrency.

It does not need database integration.

It does not need authentication.

It does not need sessions.

It does not need CSRF protection.

It does not need form validation.

It does not need streaming responses.

It does not need a full template language.

It does not need automatic OpenAPI generation.

These features are important in real frameworks.

They are intentionally outside the first implementation.

A small framework is useful because the reader can understand all of it.

---

# HTTP Refresher

HTTP is a request-response protocol.

A client sends a request:

```text
GET /users/42 HTTP/1.1
Host: example.com
Accept: application/json
```

The server sends a response:

```text
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 42}
```

A request has:

```text
method
path
query string
headers
body
client address
```

A response has:

```text
status code
headers
body
```

The framework's job is to make those pieces convenient in Python.

It should not erase the HTTP model.

Good frameworks make HTTP easier without pretending it is not there.

---

# The App Object

The `App` object is the center of the framework.

It should own:

```text
router
middleware list
error handlers
static file configuration
template renderer
development server entry point
```

The simplest shape is:

```python
class App:
    def __init__(self):
        self.router = Router()
        self.middleware = []
        self.error_handlers = {}

    def route(self, path, methods):
        def decorator(func):
            self.router.add_route(path, methods, func)
            return func
        return decorator

    def get(self, path):
        return self.route(path, ["GET"])

    def post(self, path):
        return self.route(path, ["POST"])
```

The route decorator should return the original function.

That preserves normal Python behavior.

The function can still be called directly in tests.

---

# Route Registration

Route registration records a rule.

For example:

```python
@app.get("/users/{user_id}")
def get_user(request, user_id):
    ...
```

The framework stores:

```text
path pattern: /users/{user_id}
methods: GET
handler: get_user
```

It does not call the handler during registration.

Registration happens at import time.

Execution happens later when a request arrives.

This distinction is important.

A decorator is often used to record metadata.

Here the metadata is:

```text
which function handles which HTTP method and path
```

---

# Router

The router finds the right handler for a request.

It receives:

```text
method
path
```

It returns:

```text
matched handler
path parameters
allowed methods
```

There are two kinds of route matching.

The first is exact matching:

```text
/about
/health
/login
```

The second is parameterized matching:

```text
/users/{user_id}
/posts/{post_id}/comments/{comment_id}
```

Parameterized routes turn path segments into variables.

For example:

```text
/users/42
```

matches:

```text
/users/{user_id}
```

with:

```text
user_id = "42"
```

Keep path parameters as strings at first.

Type conversion can be an extension.

---

# Compiling Route Patterns

A route pattern can be compiled into a regular expression.

Pattern:

```text
/users/{user_id}
```

Regex:

```text
^/users/(?P<user_id>[^/]+)$
```

Python code:

```python
import re

def compile_path(pattern):
    parts = pattern.strip("/").split("/")
    regex_parts = []
    param_names = []

    for part in parts:
        if part.startswith("{") and part.endswith("}"):
            name = part[1:-1]
            param_names.append(name)
            regex_parts.append(f"(?P<{name}>[^/]+)")
        else:
            regex_parts.append(re.escape(part))

    regex = "^/" + "/".join(regex_parts) + "$"
    return re.compile(regex), param_names
```

This is not a complete routing engine.

It is enough to teach the principle.

The regex uses named groups.

Named groups turn matched path values into a dictionary.

```python
match.groupdict()
```

That dictionary can be passed to the handler.

---

# Route Conflicts

The framework should decide what happens when two routes conflict.

For example:

```python
@app.get("/users/{id}")
def by_id(...):
    ...

@app.get("/users/me")
def me(...):
    ...
```

If parameterized routes are checked first, `/users/me` may match `{id}`.

A simple policy is:

```text
routes are checked in registration order
```

That is easy to explain.

The developer can register more specific routes first.

A more sophisticated framework may sort routes by specificity.

For this capstone, registration order is fine.

Document it.

---

# 404 And 405

The router should distinguish two cases.

Case one:

```text
no route path matches
```

Return 404 Not Found.

Case two:

```text
path matches, but method does not
```

Return 405 Method Not Allowed.

This distinction matters.

If the app has:

```python
@app.get("/users")
def list_users(request):
    ...
```

Then:

```text
POST /users
```

should not be 404.

The path exists.

The method is not allowed.

The response should include an `Allow` header:

```text
Allow: GET
```

This teaches that routing is not only path matching.

HTTP method matters.

---

# Request Object

The `Request` object wraps incoming HTTP data.

It should expose:

```text
method
path
query_params
headers
body
client
```

A simple dataclass works:

```python
from dataclasses import dataclass

@dataclass
class Request:
    method: str
    path: str
    query_params: dict[str, list[str]]
    headers: dict[str, str]
    body: bytes
    client: tuple[str, int] | None = None
```

Add helpers:

```python
def text(self, encoding="utf-8") -> str:
    return self.body.decode(encoding)

def json(self):
    return json.loads(self.text())
```

Do not parse JSON automatically for every request.

Only parse it when the handler asks.

That keeps the request object predictable.

---

# Query Parameters

The URL:

```text
/search?q=python&page=2&tag=web&tag=framework
```

has:

```text
path: /search
query string: q=python&page=2&tag=web&tag=framework
```

Use `urllib.parse`.

```python
from urllib.parse import urlsplit, parse_qs

parts = urlsplit(raw_path)
path = parts.path
query_params = parse_qs(parts.query)
```

`parse_qs` returns lists because a parameter can appear multiple times.

```python
{"tag": ["web", "framework"]}
```

That is more accurate than keeping only one value.

The framework can later add convenience helpers like:

```python
request.query("page", default="1")
```

But the underlying data should preserve the real HTTP shape.

---

# Headers

HTTP headers are case-insensitive.

These are the same header:

```text
Content-Type
content-type
CONTENT-TYPE
```

The mini framework can normalize header names to lowercase.

```python
headers = {name.lower(): value for name, value in incoming_headers}
```

Then handlers can write:

```python
request.headers.get("content-type")
```

This avoids many small bugs.

The framework should document the normalization behavior.

---

# Response Object

The `Response` object represents outgoing HTTP data.

It should contain:

```text
status
headers
body bytes
```

Example:

```python
class Response:
    def __init__(self, body=b"", status=200, headers=None, content_type=None):
        self.status = status
        self.headers = headers or {}
        self.body = body if isinstance(body, bytes) else str(body).encode("utf-8")
        if content_type is not None:
            self.headers["Content-Type"] = content_type
```

The framework should also set `Content-Length`.

```python
self.headers["Content-Length"] = str(len(self.body))
```

A correct response should tell the client how many bytes are in the body.

The standard library server may handle some details, but the framework should still teach the concept.

---

# Response Subclasses

Add convenience response types.

```python
class HTMLResponse(Response):
    def __init__(self, html, status=200, headers=None):
        super().__init__(
            html,
            status=status,
            headers=headers,
            content_type="text/html; charset=utf-8",
        )
```

```python
class JSONResponse(Response):
    def __init__(self, data, status=200, headers=None):
        body = json.dumps(data).encode("utf-8")
        super().__init__(
            body,
            status=status,
            headers=headers,
            content_type="application/json",
        )
```

```python
class RedirectResponse(Response):
    def __init__(self, location, status=302):
        super().__init__(b"", status=status, headers={"Location": location})
```

These classes are small.

They make application code pleasant.

They also show that framework ergonomics often come from a few thoughtful wrappers.

---

# Return Value Conversion

Handlers should be allowed to return a few simple types.

```text
Response object -> use directly
dict or list -> JSONResponse
str -> HTMLResponse or text response
bytes -> Response
None -> empty 204 response
```

The app can implement:

```python
def make_response(value):
    if isinstance(value, Response):
        return value
    if isinstance(value, (dict, list)):
        return JSONResponse(value)
    if isinstance(value, str):
        return HTMLResponse(value)
    if isinstance(value, bytes):
        return Response(value)
    if value is None:
        return Response(b"", status=204)
    raise TypeError(f"Unsupported response type: {type(value).__name__}")
```

This is one of the places where frameworks feel magical.

The magic is only a conversion function.

Once the reader sees it, it becomes understandable.

---

# Handler Calling

When a route matches, the framework calls the handler.

For a route without path parameters:

```python
handler(request)
```

For a route with path parameters:

```python
handler(request, **params)
```

Example:

```python
@app.get("/users/{user_id}")
def get_user(request, user_id):
    return {"id": user_id}
```

The router extracts:

```python
{"user_id": "42"}
```

The app calls:

```python
get_user(request, user_id="42")
```

This is ordinary Python.

The framework only connects the pieces.

---

# Middleware

Middleware wraps request handling.

It can run before and after route handlers.

Common middleware examples:

```text
logging
timing
authentication
sessions
CORS
compression
error tracking
request ids
```

The simplest middleware signature is:

```python
def middleware(request, next_handler):
    response = next_handler(request)
    return response
```

Example:

```python
def add_powered_by(request, next_handler):
    response = next_handler(request)
    response.headers["X-Powered-By"] = "MiniWeb"
    return response
```

The app can register middleware:

```python
app.add_middleware(add_powered_by)
```

Middleware is powerful because it composes cross-cutting behavior without putting it into every route.

---

# Building The Middleware Chain

Suppose the app has three middleware functions:

```text
logger
auth
timer
```

And one final handler:

```text
route_handler
```

The call chain should be:

```text
logger -> auth -> timer -> route_handler -> timer -> auth -> logger
```

This means middleware wraps inward.

One implementation:

```python
def build_chain(self, endpoint):
    handler = endpoint
    for middleware in reversed(self.middleware):
        next_handler = handler
        def wrapped(request, middleware=middleware, next_handler=next_handler):
            return middleware(request, next_handler)
        handler = wrapped
    return handler
```

The default arguments are important.

Without them, Python's late binding closure behavior can make every wrapper use the last middleware.

This is a perfect place to connect back to closures from Volume I.

---

# Error Handling

Applications raise exceptions.

Frameworks should turn expected exceptions into responses.

Define HTTP exceptions:

```python
class HTTPException(Exception):
    status = 500

    def __init__(self, message=None):
        self.message = message or self.__class__.__name__
```

```python
class NotFound(HTTPException):
    status = 404
```

```python
class MethodNotAllowed(HTTPException):
    status = 405
```

The app can catch these:

```python
try:
    response = dispatch(request)
except HTTPException as exc:
    response = handle_http_exception(exc)
except Exception as exc:
    response = handle_unexpected_exception(exc)
```

Do not expose full tracebacks in production responses.

For a development framework, a debug mode can show more details.

But the default should be safe.

---

# Custom Error Handlers

Allow users to register handlers by status code or exception type.

```python
@app.error_handler(404)
def not_found(request, exc):
    return HTMLResponse("<h1>Not Found</h1>", status=404)
```

Or:

```python
@app.error_handler(ValueError)
def value_error(request, exc):
    return JSONResponse({"error": str(exc)}, status=400)
```

The simple version can support status codes first.

Exception-type handlers can be an extension.

The point is that framework users need a way to control error output without wrapping every route in try/except.

---

# Static Files

Static files are files served directly from disk.

Examples:

```text
CSS
JavaScript
images
fonts
```

The mini framework can support:

```python
app.static("/static", directory="static")
```

Then:

```text
GET /static/site.css
```

serves:

```text
static/site.css
```

Static file serving must prevent path traversal.

This request must not escape the static directory:

```text
GET /static/../../secrets.txt
```

Use `pathlib.Path.resolve`.

Check that the resolved requested path is inside the resolved static root.

```python
root = Path(directory).resolve()
target = (root / relative_path).resolve()
if root not in target.parents and target != root:
    raise NotFound()
```

This is an important security lesson.

Even a learning framework should not teach unsafe file serving.

---

# Content Types

Static files need `Content-Type`.

Use Python's `mimetypes` module.

```python
import mimetypes

content_type, _ = mimetypes.guess_type(path.name)
content_type = content_type or "application/octet-stream"
```

Examples:

```text
.html -> text/html
.css  -> text/css
.js   -> text/javascript or application/javascript
.png  -> image/png
```

Content types help browsers interpret files correctly.

Without them, a browser may display content incorrectly or block resources.

---

# Templates

A full template engine is a large project.

This capstone should implement a tiny renderer.

For example:

Template file:

```html
<h1>Hello, {{ name }}</h1>
```

Render call:

```python
render_template("hello.html", name="Narendra")
```

Output:

```html
<h1>Hello, Narendra</h1>
```

The first version can replace `{{ variable }}` tokens using a regular expression.

It should HTML-escape values by default.

```python
import html

escaped = html.escape(str(value))
```

Escaping matters.

Without escaping, templates can create cross-site scripting vulnerabilities.

This chapter should introduce the idea even if it does not become a full security course.

---

# Template Limitations

The tiny template engine does not need:

```text
loops
conditionals
inheritance
filters
macros
includes
auto-reload
bytecode caching
```

It only needs variable substitution.

That is enough to teach:

```text
load template file
replace placeholders
escape user values
return HTML response
```

The reader can later compare this with Jinja2 and understand why mature template engines are sophisticated.

---

# WSGI And ASGI

Python web frameworks usually sit behind a standard server interface.

The older standard is WSGI.

WSGI is synchronous.

It lets servers and frameworks communicate through a callable interface.

The newer common standard is ASGI.

ASGI supports async, WebSockets, lifespan events, and more modern concurrency patterns.

This capstone does not need to fully implement WSGI or ASGI.

But it should explain the idea:

```text
servers should not need to know every framework
frameworks should not need to implement every server
standard interfaces connect them
```

The development server in this project is enough for learning.

Production deployments would use battle-tested servers and mature framework adapters.

---

# Development Server

Use `http.server` from the standard library.

Create a custom handler that adapts `BaseHTTPRequestHandler` to the framework.

The handler receives HTTP data from the standard library.

It builds a `Request`.

It calls `app.handle_request(request)`.

It sends the `Response` back.

The shape:

```python
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer

class MiniWebHandler(BaseHTTPRequestHandler):
    app = None

    def do_GET(self):
        self.handle_method("GET")

    def do_POST(self):
        self.handle_method("POST")

    def handle_method(self, method):
        request = build_request(self, method)
        response = self.app.handle_request(request)
        send_response(self, response)
```

Then:

```python
def run(app, host, port):
    handler_cls = type("AppHandler", (MiniWebHandler,), {"app": app})
    server = ThreadingHTTPServer((host, port), handler_cls)
    server.serve_forever()
```

This keeps the framework focused.

The standard library handles low-level HTTP parsing.

The framework handles application dispatch.

---

# Reading Request Bodies

POST requests may have a body.

The standard library handler exposes headers and a stream.

Read the body using `Content-Length`.

```python
length = int(self.headers.get("Content-Length", "0"))
body = self.rfile.read(length) if length else b""
```

Do not call `read()` without a length.

That can block while waiting for the client to close the connection.

This is another valuable server lesson.

Network streams need clear boundaries.

HTTP request bodies use `Content-Length` or transfer encoding to define those boundaries.

The mini framework can support `Content-Length` only.

---

# Sending Responses

The server adapter sends:

```python
self.send_response(response.status)
for name, value in response.headers.items():
    self.send_header(name, value)
self.end_headers()
self.wfile.write(response.body)
```

Make sure `Content-Length` is present.

Make sure header values are strings.

Make sure body is bytes.

This adapter is the boundary between framework objects and HTTP bytes.

The rest of the framework should not need to know about `BaseHTTPRequestHandler`.

---

# Testing Without A Server

Most tests should not start a server.

Test `app.handle_request` directly.

Example:

```python
def test_get_route():
    app = App()

    @app.get("/hello")
    def hello(request):
        return "Hello"

    request = Request(method="GET", path="/hello", query_params={}, headers={}, body=b"")
    response = app.handle_request(request)

    assert response.status == 200
    assert response.body == b"Hello"
```

This is fast and stable.

Server tests should be fewer.

They prove the adapter works.

They should not be the only way to test routing.

---

# Testing Routing

Routing tests should cover:

```text
exact route match
missing route returns 404
wrong method returns 405
path parameter extraction
multiple path parameters
registration order behavior
allowed methods header
```

Examples:

```python
@app.get("/users/{user_id}")
def get_user(request, user_id):
    return {"user_id": user_id}
```

Request:

```text
GET /users/42
```

Expected JSON:

```json
{"user_id": "42"}
```

Keep the path parameter as a string unless type conversion is implemented.

---

# Testing Responses

Response tests should cover:

```text
string return conversion
dict return conversion
list return conversion
explicit Response return
bytes return
None return
content type
content length
redirect headers
unsupported return type
```

These tests protect framework ergonomics.

A small change in response conversion can affect every application route.

---

# Testing Middleware

Middleware tests should prove order.

Example:

```python
events = []

def first(request, next_handler):
    events.append("first before")
    response = next_handler(request)
    events.append("first after")
    return response

def second(request, next_handler):
    events.append("second before")
    response = next_handler(request)
    events.append("second after")
    return response
```

Expected:

```text
first before
second before
handler
second after
first after
```

This test also catches closure late-binding bugs.

---

# Testing Static Files

Static file tests should use `tmp_path`.

Create:

```text
tmp_path/static/site.css
```

Configure:

```python
app.static("/static", tmp_path / "static")
```

Request:

```text
GET /static/site.css
```

Assert:

```text
status 200
body matches file
content type looks like CSS
```

Also test path traversal:

```text
GET /static/../secret.txt
```

It should return 404 or 403.

Do not skip this test.

It turns a theoretical security warning into concrete behavior.

---

# Testing Templates

Template tests should cover:

```text
variable substitution
missing variable behavior
HTML escaping
template not found
```

Example:

```html
Hello, {{ name }}
```

Context:

```python
{"name": "<Narendra>"}
```

Output should include escaped HTML:

```html
&lt;Narendra&gt;
```

This teaches that HTML generation is not just string formatting.

It is output in a security-sensitive context.

---

# Milestone 1 - Response Objects

Start with responses.

Implement:

```text
Response
HTMLResponse
JSONResponse
RedirectResponse
make_response
```

Add tests for:

```text
body bytes
status code
headers
content type
content length
return value conversion
```

This gives the project a stable output model before routing begins.

---

# Milestone 2 - Request Object

Implement `Request`.

Add:

```text
text()
json()
query parameter handling
case-normalized headers
```

Test JSON parsing.

Test body decoding.

Test query strings with repeated keys.

The request object should be simple but dependable.

---

# Milestone 3 - Router

Implement:

```text
Route dataclass
compile_path
Router.add_route
Router.match
```

The match result should include:

```text
handler
path params
allowed methods when method is wrong
```

Add tests for exact and parameterized routes.

Do not connect to HTTP yet.

Routing is pure logic.

Pure logic should be tested directly.

---

# Milestone 4 - App Dispatch

Implement `App.handle_request`.

It should:

```text
ask router for a match
call handler with request and params
convert return value to Response
handle 404
handle 405
handle HTTP exceptions
handle unexpected exceptions safely
```

At this point, a test can create a fake request and receive a response.

The framework is alive before any server exists.

---

# Milestone 5 - Decorators

Add:

```text
app.route
app.get
app.post
app.put
app.delete
```

The route decorators should register handlers and return the original function.

Test that:

```text
decorated route works
function identity is preserved
multiple methods can be registered
```

This gives the framework the familiar feel users expect.

---

# Milestone 6 - Middleware

Implement:

```text
app.add_middleware
middleware chain building
```

Middleware should wrap route dispatch.

Test order.

Test that middleware can modify response headers.

Test that middleware can return early without calling `next_handler`.

Returning early is useful for authentication:

```python
def require_token(request, next_handler):
    if request.headers.get("authorization") != "secret":
        return JSONResponse({"error": "unauthorized"}, status=401)
    return next_handler(request)
```

---

# Milestone 7 - Error Handlers

Add:

```text
app.error_handler(status_or_exception)
```

Start with status code handlers.

Test:

```text
custom 404 page
custom 500 response
handler return conversion
```

Error handlers should use the same return conversion as normal route handlers.

This consistency keeps the framework easy to learn.

---

# Milestone 8 - Static Files

Implement static file registration.

Use a special route or a static handler inside app dispatch.

Support:

```python
app.static("/static", directory="static")
```

Test:

```text
existing file
missing file
path traversal rejection
content type
```

This milestone adds the first real filesystem behavior to the framework.

---

# Milestone 9 - Templates

Implement:

```text
TemplateRenderer
render_template
HTML escaping
```

Configure:

```python
app = App(template_dir="templates")
```

Usage:

```python
@app.get("/")
def home(request):
    return app.render_template("home.html", name="Narendra")
```

The return value can be an `HTMLResponse`.

Test escaping.

Keep the language small.

---

# Milestone 10 - Development Server

Implement the standard library server adapter.

Support:

```python
app.run(host="127.0.0.1", port=8000)
```

Manual test:

```bash
python example.py
curl http://127.0.0.1:8000/
curl http://127.0.0.1:8000/users/42
```

Add one integration test if practical.

Most behavior should already be covered without the server.

---

# Common Mistake 1 - Mixing Server And Framework

Do not put all logic inside `BaseHTTPRequestHandler`.

That creates a framework that is hard to test.

The handler should adapt HTTP to framework objects.

The framework should handle routing and responses.

The distinction is:

```text
server adapter: knows about BaseHTTPRequestHandler
framework app: knows about Request and Response
```

This separation keeps the design clean.

---

# Common Mistake 2 - Returning Raw Values Everywhere

If every part of the framework returns a different shape, the code becomes fragile.

Use `Response` as the internal standard.

Handlers may return friendly values.

But after `make_response`, the framework should deal with one response shape.

That makes middleware, error handlers, and server adapters simpler.

---

# Common Mistake 3 - Forgetting 405

Many tiny routers return 404 for everything.

That loses useful HTTP information.

If the path exists but the method is wrong, return 405.

Include the allowed methods.

This teaches the reader that HTTP semantics matter.

---

# Common Mistake 4 - Unsafe Static Files

Never join user paths to a directory and serve the result blindly.

This is dangerous:

```python
path = static_root / requested_path
return path.read_bytes()
```

The request may contain:

```text
../
```

Always resolve and check that the final path stays inside the static root.

Security begins with path handling.

---

# Common Mistake 5 - Late Binding In Middleware

When building middleware wrappers in a loop, be careful with closures.

This can be wrong:

```python
for middleware in reversed(self.middleware):
    def wrapped(request):
        return middleware(request, handler)
```

Every wrapper may use the last value of `middleware`.

Use default arguments or a helper function.

This is an excellent real-world closure lesson.

---

# Common Mistake 6 - Treating Templates As Plain Formatting

HTML templates must escape untrusted values.

This is unsafe:

```python
template.replace("{{ name }}", value)
```

If `value` contains HTML or JavaScript, it may be executed by the browser.

Use `html.escape`.

Even in a tiny framework, teach the safe habit.

---

# Common Mistake 7 - Testing Only Through Curl

Manual curl testing is useful.

It is not enough.

Most framework behavior should be tested with direct `Request` objects.

That makes tests fast.

Fast tests encourage frequent testing.

Frequent testing makes framework design safer.

---

# Extension Ideas

Add route parameter converters:

```text
/users/{user_id:int}
/files/{path:path}
```

Add blueprints or routers.

Add before-request and after-request hooks.

Add sessions.

Add signed cookies.

Add form parsing.

Add multipart file uploads.

Add streaming responses.

Add template inheritance.

Add Jinja2 integration.

Add WSGI adapter.

Add ASGI adapter after the async chapters.

Add dependency injection.

Add request validation.

Add OpenAPI generation.

Add CORS middleware.

Add compression middleware.

Add a test client:

```python
client = app.test_client()
response = client.get("/users/42")
```

Add command-line project scaffolding.

Each extension teaches a real framework feature.

Do not add them all at once.

First make the core request-response cycle beautiful and clear.

---

# What This Project Teaches

This project teaches framework design.

It teaches route registration.

It teaches path matching.

It teaches request parsing.

It teaches response construction.

It teaches HTTP status codes.

It teaches middleware composition.

It teaches error handling.

It teaches safe static file serving.

It teaches basic template rendering.

It teaches server adapters.

It teaches testable architecture.

It connects many earlier ideas:

```text
decorators become route registration
regular expressions become path matching
dictionaries become headers and query params
functions become handlers
closures become middleware chains
exceptions become HTTP responses
files become static assets and templates
JSON becomes API output
tests become framework confidence
```

The reader should leave with this mental model:

```text
A web framework is a disciplined pipeline from HTTP input to HTTP output.
```

Once that pipeline is clear, mature frameworks feel less mysterious.

---

# Completion Checklist

The project is complete when:

```text
App can register routes.
GET routes work.
POST routes work.
Exact paths match.
Parameterized paths match.
Path parameters are passed to handlers.
Missing paths return 404.
Wrong methods return 405.
405 responses include allowed methods.
Request exposes method, path, query params, headers, body, text, and json.
Response stores status, headers, and body bytes.
String returns become HTML/text responses.
Dict and list returns become JSON responses.
Explicit Response objects pass through unchanged.
RedirectResponse sets Location.
Middleware wraps route handling in the correct order.
Middleware can modify responses.
Middleware can return early.
Custom error handlers work.
Static files can be served.
Path traversal is rejected.
Templates render variables.
Template values are HTML-escaped.
Development server runs the app.
Tests cover routing, request, response, middleware, errors, static files, and templates.
Documentation explains framework limitations.
```

When all of these are true, the reader has built a real miniature web framework.

---

# Exercises

1. Create the `miniweb` package structure.

2. Implement the base `Response`.

3. Implement `HTMLResponse`.

4. Implement `JSONResponse`.

5. Implement `RedirectResponse`.

6. Implement `make_response`.

7. Add response conversion tests.

8. Implement the `Request` object.

9. Add `text()` and `json()` helpers.

10. Parse query strings with repeated keys.

11. Normalize headers.

12. Implement route pattern compilation.

13. Implement exact route matching.

14. Implement path parameter matching.

15. Implement 404 handling.

16. Implement 405 handling.

17. Add route decorators.

18. Implement `App.handle_request`.

19. Test handler return conversion.

20. Implement middleware registration.

21. Build the middleware chain.

22. Test middleware order.

23. Implement custom error handlers.

24. Implement static file serving.

25. Test path traversal rejection.

26. Implement the template renderer.

27. Test HTML escaping.

28. Implement the development server adapter.

29. Add a small example application.

30. Document limitations and extension ideas.

---

# Preview of Capstone 09

Capstone 08 built a Mini Web Framework.

It introduced routing, request objects, response objects, middleware, error handlers, static files, templates, and HTTP server adapters.

Capstone 09 will build a Mini Event Loop.

The Mini Event Loop project will move below frameworks and into asynchronous execution itself.

It will introduce cooperative multitasking, callbacks, futures, tasks, non-blocking sockets, timers, selectors, and the foundation of async frameworks.

The transition is:

```text
Mini Web Framework organizes synchronous HTTP application flow
Mini Event Loop explains how asynchronous systems schedule work
```

The next project will show how Python can juggle many waiting operations without one thread per task.
