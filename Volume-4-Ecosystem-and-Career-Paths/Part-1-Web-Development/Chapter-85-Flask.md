# Chapter 85 — Flask

Flask is a small Python web framework.

It is often called a microframework.

That word can be misunderstood.

Micro does not mean toy.

Micro does not mean only for tiny applications.

Micro means Flask keeps the core small.

It gives you the essential pieces for building web applications:

* routing
* request handling
* response handling
* templates
* configuration
* sessions
* error handling
* testing support
* extension hooks

It does not force a full application architecture on you.

It does not include an ORM by default.

It does not require one project layout.

It does not decide your database layer, authentication system, admin interface, or background job tool.

That flexibility is Flask's strength.

It is also Flask's risk.

Flask makes it easy to start.

It does not automatically keep a growing application organized.

That responsibility belongs to the engineer.

---

# Why Flask Comes First in Volume IV

Volume IV moves from engineering foundations into Python's ecosystem.

Flask is a good first web framework because it makes the web request-response model visible.

When you write Flask code, you can see:

```text
URL -> route -> function -> response
```

That is the heart of web programming.

Frameworks differ in how much machinery they provide around that flow.

Flask keeps the flow close to the surface.

That makes it excellent for learning:

* routing
* HTTP methods
* request data
* response data
* templates
* cookies and sessions
* application structure
* testing web endpoints
* deployment boundaries

After Flask, Django and FastAPI will make more sense.

Django adds a large integrated framework.

FastAPI emphasizes APIs, type hints, validation, and async support.

Flask shows the web foundation with less ceremony.

---

# The Request-Response Model

A web application receives requests and returns responses.

The browser or client sends:

```text
request
```

The server returns:

```text
response
```

An HTTP request includes:

* method
* path
* headers
* query parameters
* body
* cookies

An HTTP response includes:

* status code
* headers
* body
* cookies

Flask sits between Python code and the web server interface.

It lets you write Python functions that respond to URLs.

Example:

```python
from flask import Flask


app = Flask(__name__)


@app.get("/")
def index():
    return "Hello from Flask"
```

When a client requests `/`, Flask calls `index`.

The return value becomes the response.

---

# A Minimal Flask App

A minimal Flask application:

```python
from flask import Flask


app = Flask(__name__)


@app.route("/")
def hello():
    return "<p>Hello, World!</p>"
```

There are three important pieces.

First:

```python
app = Flask(__name__)
```

This creates the Flask application object.

Second:

```python
@app.route("/")
```

This registers a route.

Third:

```python
def hello():
    return "<p>Hello, World!</p>"
```

This view function returns the response body.

Run it during development:

```bash
flask --app hello run
```

If the file is named `app.py` or `wsgi.py`, Flask can often discover it without `--app`, but explicit is clearer while learning.

Do not name your file `flask.py`.

That can shadow the real Flask package and break imports.

---

# The Development Server

Flask includes a development server.

Run:

```bash
flask --app hello run
```

Debug mode:

```bash
flask --app hello run --debug
```

Debug mode is useful during development because it can reload code and show an interactive debugger.

Do not use the development server or interactive debugger in production.

The debugger can execute Python code from the browser when accessed.

That is powerful locally.

It is dangerous publicly.

Production deployment uses a proper WSGI server and deployment setup.

Flask's development server is for development.

Treat it that way.

---

# Routing

Routing maps URLs to view functions.

Example:

```python
@app.get("/")
def index():
    return "Index"


@app.get("/about")
def about():
    return "About"
```

When the path is `/`, Flask calls `index`.

When the path is `/about`, Flask calls `about`.

Routes are the public web API of a Flask app.

They should be named and organized intentionally.

URLs are not internal details.

Users, clients, search engines, scripts, and tests may depend on them.

---

# Dynamic Routes

Routes can contain variable parts.

Example:

```python
@app.get("/users/<username>")
def show_user(username: str):
    return f"User {username}"
```

Request:

```text
/users/narendra
```

calls:

```python
show_user("narendra")
```

Converters can specify shape:

```python
@app.get("/posts/<int:post_id>")
def show_post(post_id: int):
    return f"Post {post_id}"
```

Now `post_id` is converted to an integer.

Common converters include:

* `string`
* `int`
* `float`
* `path`
* `uuid`

Route variables are external input.

Validate them as needed.

Do not assume a URL value is safe just because Flask matched it.

---

# HTTP Methods

Routes can respond to specific HTTP methods.

GET:

```python
@app.get("/users")
def list_users():
    ...
```

POST:

```python
@app.post("/users")
def create_user():
    ...
```

Or:

```python
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        ...
    return render_template("login.html")
```

Use method semantics honestly.

GET should retrieve.

POST can create or trigger processing.

PUT and PATCH update.

DELETE deletes.

This connects directly to Chapter 83.

HTTP method choice is API design.

---

# View Functions

A view function handles a request and returns a response.

Example:

```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

Flask can convert dictionaries to JSON responses.

A view may return:

* string
* dictionary
* list
* response object
* tuple with body and status
* tuple with body, status, and headers

Example:

```python
@app.post("/users")
def create_user():
    user = register_user(request.json)
    return {"id": user.id}, 201
```

View functions should stay focused.

They should translate HTTP into application calls.

They should not become giant business logic containers.

---

# The request Object

Flask provides the current request through `request`.

Example:

```python
from flask import request


@app.get("/search")
def search():
    query = request.args.get("q", "")
    return {"query": query}
```

`request.args` contains query parameters.

For form data:

```python
request.form["username"]
```

For JSON:

```python
data = request.get_json()
```

The request object is context-local.

It looks global, but it refers to the active request context.

That is why code using `request` outside a request fails unless you create a test request context.

This is Flask magic worth understanding.

---

# Context Locals

Flask's `request`, `session`, `g`, and `current_app` are context-local proxies.

They are not ordinary global variables.

They point to objects associated with the current application or request context.

This lets code write:

```python
from flask import request


request.path
```

without passing the request object into every function.

This is convenient.

It can also hide dependencies.

If ordinary domain logic imports `request`, it becomes tied to Flask.

Prefer using `request` in the API layer.

Translate request data into application objects.

Pass those objects into core logic.

---

# URL Building

Use `url_for` to build URLs.

Example:

```python
from flask import url_for


url_for("show_user", username="narendra")
```

If the route is:

```python
@app.get("/users/<username>")
def show_user(username):
    ...
```

then `url_for` can generate:

```text
/users/narendra
```

Why not hard-code URLs?

Because route paths may change.

`url_for` uses endpoint names, so changes are more centralized.

In templates, use:

```html
<a href="{{ url_for('show_user', username=user.username) }}">Profile</a>
```

URL building is API maintenance.

---

# Templates

Flask uses Jinja for templates.

Templates generate text, usually HTML.

View:

```python
from flask import render_template


@app.get("/hello/<name>")
def hello(name: str):
    return render_template("hello.html", name=name)
```

Template:

```html
<!doctype html>
<title>Hello</title>
<h1>Hello {{ name }}</h1>
```

Templates usually live in a `templates` folder.

If your app is a package:

```text
myapp/
    __init__.py
    templates/
        hello.html
```

Templates keep HTML out of Python strings.

They also provide autoescaping for common HTML templates, which helps prevent injection bugs.

---

# HTML Escaping

Untrusted user input must be escaped before appearing in HTML.

Bad:

```python
@app.get("/hello")
def hello():
    name = request.args.get("name", "")
    return f"<h1>Hello {name}</h1>"
```

If `name` contains HTML or JavaScript, the browser may interpret it.

Better:

```python
from markupsafe import escape


@app.get("/hello")
def hello():
    name = request.args.get("name", "")
    return f"<h1>Hello {escape(name)}</h1>"
```

Better still for real HTML pages:

```python
return render_template("hello.html", name=name)
```

Jinja escapes variables in HTML templates by default for common HTML template types.

Security begins at output boundaries too.

---

# Static Files

Web applications often need static files:

* CSS
* JavaScript
* images
* fonts

Flask serves static files during development from a `static` folder.

Example:

```text
myapp/
    static/
        style.css
```

Generate URL:

```python
url_for("static", filename="style.css")
```

In production, a web server or CDN often serves static files more efficiently.

Flask can help during development, but production static delivery is deployment design.

---

# Sessions

Flask provides a `session` object for storing data across requests.

Example:

```python
from flask import session


@app.post("/login")
def login():
    session["user_id"] = user.id
    return redirect(url_for("dashboard"))
```

Flask's built-in sessions are signed cookies by default.

This means the client stores the cookie, but Flask signs it so tampering can be detected.

You need a secret key:

```python
app.config["SECRET_KEY"] = "change-this"
```

Do not commit real secret keys.

Use a long random value.

Sessions are security-sensitive.

Store minimal data.

Do not store secrets or large objects in client-side sessions.

---

# Configuration

Flask configuration lives in `app.config`.

Example:

```python
app.config.update(
    TESTING=True,
    SECRET_KEY="dev",
)
```

Common configuration includes:

* secret key
* database URL
* testing mode
* upload limits
* session cookie settings
* trusted hosts
* extension settings

Configuration should be loaded at startup.

Avoid reading environment variables deep inside route handlers.

Better:

```python
def create_app(config: dict | None = None) -> Flask:
    app = Flask(__name__)
    app.config.from_mapping(
        SECRET_KEY="dev",
        DATABASE_URL="sqlite:///app.db",
    )
    if config:
        app.config.update(config)
    return app
```

This connects to Chapter 82's composition root.

---

# Application Factory

An application factory is a function that creates and configures a Flask app.

Example:

```python
from flask import Flask


def create_app(config: dict | None = None) -> Flask:
    app = Flask(__name__)
    app.config.from_mapping(
        SECRET_KEY="dev",
    )

    if config is not None:
        app.config.update(config)

    @app.get("/health")
    def health():
        return {"status": "ok"}

    return app
```

Why use a factory?

Because it makes it easier to:

* create apps with different configuration
* test with isolated app instances
* register blueprints cleanly
* initialize extensions after configuration
* avoid global setup problems

For serious Flask applications, application factories are a strong default.

---

# Blueprints

Blueprints organize related routes.

Example:

```python
from flask import Blueprint


users_bp = Blueprint("users", __name__, url_prefix="/users")


@users_bp.get("/")
def list_users():
    return {"users": []}


@users_bp.get("/<user_id>")
def get_user(user_id: str):
    return {"id": user_id}
```

Register:

```python
def create_app():
    app = Flask(__name__)
    app.register_blueprint(users_bp)
    return app
```

Blueprints help keep routes modular.

They are useful for:

* users
* orders
* admin
* auth
* API versions

Blueprints are not architecture by themselves.

They are a Flask tool for organizing route registration.

---

# A Practical Flask Layout

A small but structured Flask app might look like:

```text
myapp/
    pyproject.toml
    src/
        myapp/
            __init__.py
            config.py
            web/
                __init__.py
                users.py
                orders.py
            application/
                register_user.py
                checkout.py
            domain/
                users.py
                orders.py
            infrastructure/
                database.py
                email.py
            templates/
            static/
    tests/
```

`src/myapp/__init__.py`:

```python
from flask import Flask


def create_app(config=None):
    app = Flask(__name__)
    app.config.from_mapping(SECRET_KEY="dev")
    if config:
        app.config.update(config)

    from .web.users import users_bp

    app.register_blueprint(users_bp)
    return app
```

This keeps Flask route code near the web boundary.

Application and domain logic can remain framework-independent.

---

# Keeping Views Thin

Thin views translate HTTP into application calls.

Bad:

```python
@app.post("/orders")
def create_order():
    data = request.get_json()
    user = db.session.query(User).filter_by(id=data["user_id"]).one()
    total = Decimal("0")
    for item in data["items"]:
        product = db.session.query(Product).filter_by(id=item["product_id"]).one()
        total += product.price * item["quantity"]
    order = Order(user_id=user.id, total=total)
    db.session.add(order)
    db.session.commit()
    send_email(user.email, "Order created")
    return {"id": order.id}, 201
```

This route handles HTTP, database, pricing, transactions, and email.

Better:

```python
@orders_bp.post("/orders")
def create_order():
    payload = request.get_json()
    command = CreateOrder.from_payload(payload)
    result = current_app.config["CREATE_ORDER"].execute(command)
    return {"id": str(result.order_id)}, 201
```

The view is now an adapter.

The application service owns the workflow.

---

# JSON APIs With Flask

Flask can build JSON APIs cleanly.

Example:

```python
@app.get("/api/users/<user_id>")
def get_user(user_id: str):
    user = users.find(UserId(user_id))
    if user is None:
        return {"error": {"code": "USER_NOT_FOUND"}}, 404
    return {
        "id": str(user.id),
        "email": user.email,
    }
```

For larger APIs, design:

* request validation
* response schemas
* error format
* pagination
* authentication
* versioning
* tests

Flask does not impose these decisions.

That is flexibility.

It is also responsibility.

If your API is schema-heavy, FastAPI may provide more out of the box.

If you want full-stack batteries, Django may be more natural.

---

# Error Handling

Flask lets you register error handlers.

Example:

```python
from werkzeug.exceptions import NotFound


@app.errorhandler(NotFound)
def not_found(error):
    return {"error": {"code": "NOT_FOUND"}}, 404
```

Custom domain error:

```python
class UserNotFound(Exception):
    pass
```

Handler:

```python
@app.errorhandler(UserNotFound)
def user_not_found(error):
    return {"error": {"code": "USER_NOT_FOUND"}}, 404
```

Error handlers translate internal failures into HTTP responses.

They belong near the web boundary.

Do not expose raw tracebacks to clients.

Log internal details.

Return safe, stable error shapes.

---

# before_request and after_request

Flask supports request hooks.

`before_request` runs before a request.

`after_request` runs after a response is created.

Example:

```python
from flask import g, request


@app.before_request
def load_request_id():
    g.request_id = request.headers.get("X-Request-ID", generate_request_id())
```

Example after hook:

```python
@app.after_request
def add_request_id(response):
    response.headers["X-Request-ID"] = g.request_id
    return response
```

Hooks are useful for cross-cutting web concerns:

* request IDs
* authentication loading
* timing
* logging
* security headers

Do not hide complex business logic in hooks.

Hooks should support the request lifecycle.

---

# g

`g` is a request-local namespace.

It can store data during one request.

Example:

```python
from flask import g


@app.before_request
def load_current_user():
    g.current_user = find_user_from_request()
```

Later:

```python
user = g.current_user
```

Use `g` for request-scoped data.

Do not use it as a general global variable.

If domain logic needs the current user, pass the user explicitly.

The request context should not leak into core business code.

---

# Extensions

Flask has many extensions.

Extensions can provide:

* database integration
* migrations
* login management
* forms
* admin pages
* caching
* rate limiting
* CORS
* API tooling

Common pattern with factories:

```python
db = SQLAlchemy()


def create_app():
    app = Flask(__name__)
    db.init_app(app)
    return app
```

The extension object is created without an app.

It is initialized inside the factory.

This supports multiple app instances and cleaner tests.

Extensions are useful.

But every extension adds conventions and dependencies.

Choose them deliberately.

---

# Database Access

Flask does not include an ORM by default.

You can choose:

* SQLAlchemy
* raw SQL
* another ORM
* external service APIs
* document databases
* no database

This is different from Django, which includes an ORM as a central part of the framework.

In Flask, database design is your decision.

For maintainable applications, avoid spreading database access everywhere.

Use repositories or application services when helpful.

Keep transactions clear.

Do not let route functions become database scripts.

---

# Testing Flask Apps

Flask includes test support.

With an application factory:

```python
import pytest
from myapp import create_app


@pytest.fixture
def app():
    app = create_app({"TESTING": True})
    return app
```

Test client:

```python
@pytest.fixture
def client(app):
    return app.test_client()
```

Test:

```python
def test_health(client):
    response = client.get("/health")

    assert response.status_code == 200
    assert response.json == {"status": "ok"}
```

The test client lets you exercise routes without running a real server.

This makes route tests fast and local.

---

# Testing Request Contexts

Sometimes you need a request context without making a full test client request.

Example:

```python
def test_url_for(app):
    with app.test_request_context():
        assert url_for("health") == "/health"
```

Request context is useful when testing code that depends on `request`, `url_for`, `session`, or `g`.

Use it carefully.

If too much non-web code needs Flask context, your architecture may be leaking framework concerns into core logic.

---

# Security Basics

Flask gives tools.

It does not make your application secure automatically.

Important concerns:

* secret key management
* session cookie settings
* CSRF protection for forms
* authentication
* authorization
* input validation
* output escaping
* file upload safety
* rate limiting
* secure headers
* HTTPS deployment

For file uploads, never blindly trust filenames.

For user input, validate at the boundary.

For HTML, rely on escaping and templates.

For sessions, protect secret keys.

Security is architecture and discipline, not one Flask setting.

---

# File Uploads

Flask can handle file uploads through `request.files`.

Example:

```python
@app.post("/upload")
def upload():
    uploaded = request.files["file"]
    uploaded.save(upload_path)
    return {"status": "uploaded"}
```

Real upload handling needs more:

* size limits
* allowed extensions
* content validation
* safe filenames
* storage location
* virus scanning in some domains
* authorization
* cleanup

Use configuration such as maximum content length where appropriate.

Uploads are untrusted input.

Treat them as a security boundary.

---

# Deployment

Flask applications run through WSGI in many production deployments.

The development server is not for production.

Production setups often include:

* WSGI server
* reverse proxy
* process manager
* environment configuration
* logging collection
* metrics
* health checks
* static file strategy
* HTTPS termination

Common WSGI servers include tools such as Gunicorn or uWSGI.

The exact deployment depends on platform.

The Flask app object is the WSGI application.

In factory style, deployment points to the factory:

```bash
flask --app 'myapp:create_app()' run
```

Production servers have their own configuration for loading the app.

---

# Flask and WSGI

WSGI is the traditional Python web server interface.

It defines how a web server calls a Python web application.

Flask is a WSGI application framework.

This means Flask apps can be served by WSGI servers.

Understanding WSGI helps explain the shape:

```text
server receives HTTP
server builds WSGI environ
Flask handles request
Flask returns response
server sends HTTP response
```

You do not need to implement WSGI to use Flask.

But knowing that Flask sits behind a server interface helps separate development server, application object, and production server.

---

# Flask vs Django

Flask is small and flexible.

Django is larger and integrated.

Flask gives you:

* routing
* request handling
* templates
* extensions
* freedom to choose architecture

Django gives you:

* ORM
* admin
* authentication
* forms
* migrations
* project conventions
* integrated batteries

Flask can be better when:

* you want a small service
* you want custom architecture
* you want minimal framework assumptions
* you are building APIs or focused web apps

Django can be better when:

* you need a full-stack application
* admin and ORM matter
* conventions help the team
* the app fits Django's model well

Neither is universally better.

They optimize for different tradeoffs.

---

# Flask vs FastAPI

Flask and FastAPI can both build APIs.

Flask is WSGI-first and minimal.

FastAPI is ASGI-first, type-hint-driven, and built around automatic validation and OpenAPI generation.

Flask can be excellent for:

* simple web apps
* small services
* HTML and JSON routes
* teams that want flexible choices
* apps with existing Flask ecosystem knowledge

FastAPI can be excellent for:

* typed JSON APIs
* automatic request validation
* OpenAPI-first workflows
* async endpoints
* data-model-heavy APIs

Flask can use extensions for validation and OpenAPI.

FastAPI includes many of those ideas centrally.

Again, choose based on requirements and team fit.

---

# Common Flask Mistakes

Common mistakes include:

* putting all routes in one huge file
* putting business logic inside route functions
* using the development server in production
* enabling debug mode in production
* committing secret keys
* skipping request validation
* returning inconsistent error shapes
* letting Flask context leak into domain code
* spreading database access through routes
* not using application factories for larger apps
* overusing global state
* forgetting tests for routes
* not planning static files and deployment

These mistakes are common because Flask is easy to start.

The cure is not making Flask heavy.

The cure is adding structure when the app grows.

---

# A Flask Checklist

When building a Flask app, ask:

* Is this route thin?
* Is request data validated?
* Is business logic outside the web layer?
* Are errors translated consistently?
* Is configuration loaded at startup?
* Are secrets kept out of code?
* Is debug mode off in production?
* Are blueprints used where routes grow?
* Is there an application factory?
* Are tests using the test client?
* Are sessions and cookies configured safely?
* Is static file handling planned?
* Is deployment using a production server?
* Are logs and request IDs present?

This checklist turns Flask from a quick script into maintainable web software.

---

# Chapter Summary

Flask is a small, flexible Python web framework.

It exposes the request-response model clearly.

A Flask application maps URLs and HTTP methods to view functions.

The `Flask` application object is the WSGI application.

Routes are registered with decorators such as `@app.route`, `@app.get`, and `@app.post`.

The `request` object provides access to request data through context-local behavior.

Templates are rendered with Jinja and help keep HTML out of Python strings.

User input must be escaped when rendered into HTML.

Static files live under `static` during development and often need a production serving strategy.

Sessions require a secret key and should be treated as security-sensitive.

Configuration belongs at startup, not scattered through route code.

Application factories make testing, configuration, and modular setup easier.

Blueprints organize groups of routes.

Thin views translate HTTP into application use cases.

Flask can build HTML apps, JSON APIs, small services, internal tools, and focused web systems.

Flask does not impose an ORM, validation layer, or full architecture.

That flexibility is powerful, but it means engineers must design boundaries deliberately.

Testing Flask apps with the test client is fast and practical.

Production Flask should use a production deployment stack, not the development server.

The central lesson is:

```text
Flask gives you the web foundation without deciding the whole building
```

That makes it excellent for learning and powerful for teams that can manage structure intentionally.

---

# Exercises

1. Build a minimal Flask app with a `/health` route returning JSON.

2. Add a dynamic route `/users/<username>` and escape the username when rendering HTML.

3. Create separate GET and POST routes for `/login`.

4. Render a Jinja template with a variable.

5. Add a blueprint for user routes and register it in an application factory.

6. Write a pytest fixture that creates a Flask app with `TESTING=True`.

7. Use the Flask test client to test a JSON endpoint.

8. Add a custom error handler that returns a consistent JSON error shape.

9. Refactor a route with business logic into an application service.

10. Write a short deployment checklist for a Flask app moving from development to production.

---

# Preview of Chapter 86

Chapter 85 introduced Flask as a small, flexible web framework that makes the request-response model visible.

We studied routes, view functions, HTTP methods, request data, templates, escaping, sessions, configuration, application factories, blueprints, testing, deployment, and Flask's place among Python web frameworks.

Next we study Django.

Django takes a different approach.

It is a full-stack web framework with many batteries included:

* project structure
* URL routing
* ORM
* migrations
* admin interface
* authentication
* forms
* templates
* settings
* security features
* testing tools

The transition is:

```text
Flask shows the web core with minimal assumptions
Django shows the power of integrated conventions
```

Chapter 86 will explain Django as a framework for building larger database-backed web applications with strong conventions and a rich built-in ecosystem.
