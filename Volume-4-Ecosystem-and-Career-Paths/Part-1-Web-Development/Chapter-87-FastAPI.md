# Chapter 87 — FastAPI

FastAPI is a modern Python framework for building web APIs.

It is not trying to be Django.

It is not trying to be Flask with a different router.

FastAPI is designed around a different idea:

```text
Python type hints can describe the contract of an API
```

That one idea changes the feel of the framework.

In Flask, a route function receives request data, and you usually decide how to parse, validate, convert, and document that data.

In Django, a view often sits inside a larger integrated system of models, forms, templates, middleware, authentication, and admin tools.

In FastAPI, the route function signature itself becomes important.

The parameters, annotations, defaults, request body models, return types, and dependency declarations all describe the API.

FastAPI reads that description and uses it for:

* request parsing
* type conversion
* validation
* response serialization
* error responses
* dependency injection
* OpenAPI generation
* interactive API documentation
* editor support
* testing clarity

This makes FastAPI feel very Pythonic when you already understand functions, annotations, objects, decorators, exceptions, context managers, async functions, and testing.

It also means FastAPI rewards precision.

When your function signature is clear, your API becomes clearer.

When your data models are clear, your documentation becomes clearer.

When your dependencies are clear, your application composition becomes clearer.

FastAPI is especially common in:

* JSON APIs
* microservices
* backend services for frontend applications
* machine learning model serving
* internal platform services
* automation backends
* systems that need generated API documentation
* systems that benefit from type-driven validation

This chapter is not just about writing a route that returns JSON.

You already know that a web framework can do that.

This chapter is about understanding why FastAPI feels different.

---

# Why FastAPI Matters

FastAPI matters because modern software is often API-shaped.

A web product may have:

* a browser frontend
* a mobile app
* a backend API
* internal admin tools
* worker services
* third-party integrations
* machine learning services
* monitoring services
* automation scripts

These pieces talk to each other through interfaces.

Very often, those interfaces are HTTP APIs returning JSON.

The challenge is not merely sending JSON.

The challenge is keeping the contract clear.

For every endpoint, the team needs to know:

* what path exists
* what method it accepts
* which path parameters are required
* which query parameters are optional
* what request body shape is valid
* what response body shape is returned
* which status codes can occur
* which authentication rules apply
* which errors are possible
* how clients should call it
* how tests should verify it

In a weakly described API, this knowledge lives in scattered places:

* route code
* serializer code
* validation code
* docs written by hand
* frontend assumptions
* test fixtures
* tribal memory

That becomes fragile.

FastAPI tries to bring much of this into the Python code itself.

The endpoint signature and Pydantic models become the source of truth.

This is valuable because Python is dynamically typed at runtime, but Python's type annotations can still express intent.

FastAPI uses that intent.

It turns type hints into API behavior.

This is the main reason FastAPI belongs after Flask and Django in this book.

Flask shows you the minimal shape of a web application.

Django shows you the integrated full-stack shape.

FastAPI shows you the type-driven API shape.

---

# The Mental Model

A FastAPI application has a simple visible structure:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello"}
```

This looks small.

But several things are happening.

`FastAPI()` creates an ASGI application object.

`@app.get("/")` registers a path operation.

The function `read_root` becomes the handler for `GET /`.

The returned dictionary is serialized to JSON.

FastAPI also adds generated API documentation.

By default, a developer can usually open:

```text
/docs
/redoc
/openapi.json
```

Those are not decorative extras.

They are a major part of the framework's value.

FastAPI is designed so that the code and the API schema stay close together.

The most useful mental model is:

```text
FastAPI reads function signatures and model definitions
then builds request handling, validation, serialization, and docs from them
```

This is why signatures matter more in FastAPI than they do in many older Python web frameworks.

---

# ASGI, Starlette, and Pydantic

FastAPI stands on two important foundations.

The first is Starlette.

Starlette provides much of the web machinery:

* routing
* ASGI integration
* requests
* responses
* middleware
* background tasks
* WebSockets
* test client support

The second is Pydantic.

Pydantic provides data validation and serialization based on type annotations.

FastAPI combines these worlds:

```text
Starlette handles web mechanics
Pydantic handles data shape and validation
FastAPI connects them through Python type hints
```

This means FastAPI is not magic in the mystical sense.

It is composition.

It uses Python introspection to inspect your functions and models.

It uses Starlette to receive and send HTTP messages.

It uses Pydantic to validate and serialize structured data.

It uses OpenAPI to describe the resulting API.

Understanding that stack helps you debug real applications.

If routing behaves strangely, you think about FastAPI and Starlette.

If validation behaves strangely, you think about Pydantic.

If async behavior behaves strangely, you think about ASGI, event loops, and blocking I/O.

If generated documentation looks wrong, you think about annotations, response models, metadata, and OpenAPI.

---

# Installing and Running FastAPI

In a real project, install FastAPI inside a virtual environment.

A common install is:

```bash
pip install "fastapi[standard]"
```

The quotes matter in many shells because square brackets have special meaning in shell globbing.

A minimal app can live in `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello, FastAPI"}
```

You can run it with:

```bash
fastapi dev main.py
```

or with an ASGI server directly:

```bash
uvicorn main:app --reload
```

The expression `main:app` means:

```text
import the module named main
find the object named app inside it
serve that ASGI application
```

The `--reload` flag is useful during development because it restarts the server when files change.

It is not a production scaling strategy.

Development servers optimize feedback.

Production servers optimize reliability, process management, logging, startup behavior, and operational control.

---

# Path Operations

FastAPI uses the term path operation for the combination of:

* HTTP method
* URL path
* Python function

For example:

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

This defines:

```text
GET /items/{item_id}
```

The path contains a dynamic segment:

```text
{item_id}
```

The function has a parameter with the same name:

```python
item_id: int
```

FastAPI connects them.

If the client calls:

```text
GET /items/42
```

FastAPI receives `"42"` from the URL.

It sees that `item_id` is annotated as `int`.

It converts the string to an integer.

The function receives:

```python
item_id == 42
```

If the client calls:

```text
GET /items/not-a-number
```

FastAPI returns a validation error instead of calling the function with invalid data.

This is a major difference from frameworks where path values arrive as strings unless you manually convert them.

FastAPI lets your function signature describe what valid input means.

---

# HTTP Methods

FastAPI supports common HTTP method decorators:

```python
@app.get("/items")
def list_items():
    ...


@app.post("/items")
def create_item():
    ...


@app.put("/items/{item_id}")
def replace_item(item_id: int):
    ...


@app.patch("/items/{item_id}")
def update_item(item_id: int):
    ...


@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    ...
```

The method should match the meaning of the operation.

`GET` reads data.

`POST` usually creates something or triggers an action.

`PUT` usually replaces a resource.

`PATCH` usually partially updates a resource.

`DELETE` removes a resource.

FastAPI will not force perfect REST design.

It gives you tools.

You still need judgment.

A professional API should make method choices predictable.

If an endpoint changes server state, do not hide it behind `GET`.

If an endpoint creates a resource, return an appropriate status code and response body.

If an endpoint partially updates a resource, distinguish absent fields from fields intentionally set to `null`.

These design details matter because clients build assumptions around them.

---

# Query Parameters

If a function parameter is not part of the path, FastAPI treats it as a query parameter by default.

```python
@app.get("/items")
def list_items(limit: int = 20, offset: int = 0):
    return {"limit": limit, "offset": offset}
```

A client can call:

```text
GET /items?limit=10&offset=30
```

FastAPI converts both values to integers.

The defaults make the parameters optional.

If a parameter has no default, it is required:

```python
@app.get("/search")
def search(q: str):
    return {"query": q}
```

This requires:

```text
GET /search?q=python
```

The key idea is:

```text
required or optional is expressed through the function signature
```

No default means required.

A default value means optional.

This mirrors ordinary Python.

FastAPI turns ordinary Python meaning into API meaning.

---

# Validating Query Parameters

Simple annotations are useful.

But real APIs often need stronger constraints.

For example:

* `limit` should not be negative
* `limit` should not exceed a maximum
* `q` should have a minimum length
* an enum-like value should accept only known options

FastAPI commonly uses `Annotated` with helper functions such as `Query`.

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items")
def list_items(
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
    q: Annotated[str | None, Query(min_length=2, max_length=50)] = None,
):
    return {"limit": limit, "q": q}
```

The annotation tells Python and tools the type.

The `Query` metadata tells FastAPI validation and documentation details.

The meaning is:

```text
limit must be an integer from 1 through 100
q may be omitted, but if supplied must be between 2 and 50 characters
```

This is clearer than burying validation inside the function body.

The route body should focus on business behavior.

The signature should describe the request contract.

---

# Request Bodies

Most useful APIs accept structured request bodies.

FastAPI uses Pydantic models for this.

```python
from pydantic import BaseModel


class ProductCreate(BaseModel):
    name: str
    price: float
    in_stock: bool = True
```

Now use the model as a function parameter:

```python
@app.post("/products")
def create_product(product: ProductCreate):
    return product
```

If a client sends:

```json
{
  "name": "Keyboard",
  "price": 49.99
}
```

FastAPI parses the JSON body.

Pydantic validates it.

The route receives a `ProductCreate` object.

If `name` is missing, validation fails.

If `price` cannot be parsed as a number, validation fails.

If `in_stock` is omitted, it defaults to `True`.

This is one of FastAPI's central benefits:

```text
request validation happens before your endpoint logic runs
```

That lets your endpoint logic assume it is receiving data in the expected shape.

---

# Pydantic Models as Contracts

A Pydantic model is more than a convenient class.

In FastAPI, it is an API contract.

Consider:

```python
from pydantic import BaseModel, Field


class UserCreate(BaseModel):
    email: str
    full_name: str = Field(min_length=1, max_length=120)
    age: int | None = Field(default=None, ge=13)
```

This model says:

* `email` is required
* `full_name` is required
* `full_name` must not be empty
* `full_name` must not be longer than 120 characters
* `age` is optional
* if `age` is present, it must be at least 13

FastAPI can use this for:

* request validation
* generated JSON Schema
* generated OpenAPI documentation
* editor hints
* test expectations

This is much stronger than a docstring saying "expects user data."

The model is executable documentation.

But models should be designed carefully.

Do not use one model for every direction of data.

A common professional pattern is:

```python
class UserCreate(BaseModel):
    email: str
    password: str
    full_name: str


class UserPublic(BaseModel):
    id: int
    email: str
    full_name: str


class UserUpdate(BaseModel):
    email: str | None = None
    full_name: str | None = None
```

These models represent different contracts.

`UserCreate` accepts a password.

`UserPublic` must not expose a password.

`UserUpdate` allows partial input.

This separation prevents accidental data leaks and ambiguous update behavior.

---

# Response Models

FastAPI can also validate and shape responses.

```python
@app.post("/users", response_model=UserPublic, status_code=201)
def create_user(user: UserCreate):
    saved_user = {
        "id": 1,
        "email": user.email,
        "full_name": user.full_name,
        "hashed_password": "not-for-clients",
    }
    return saved_user
```

The function returns a dictionary containing `hashed_password`.

But the response model is `UserPublic`.

FastAPI serializes the response according to that model.

The client receives only the public fields.

This is not a substitute for careful security design.

But it is a useful guardrail.

Response models help with:

* avoiding accidental extra fields
* documenting output shape
* catching mismatches during development
* giving frontend clients a predictable contract

The route's input model answers:

```text
what may the client send?
```

The response model answers:

```text
what will the API return?
```

Professional APIs need both.

---

# Status Codes

HTTP status codes are part of the API contract.

FastAPI lets you declare them at the route level:

```python
from fastapi import status


@app.post("/products", status_code=status.HTTP_201_CREATED)
def create_product(product: ProductCreate):
    return {"id": 1, **product.model_dump()}
```

Using constants from `fastapi.status` is clearer than writing numbers everywhere.

Some common status codes are:

* `200 OK`
* `201 Created`
* `204 No Content`
* `400 Bad Request`
* `401 Unauthorized`
* `403 Forbidden`
* `404 Not Found`
* `409 Conflict`
* `422 Unprocessable Content`
* `500 Internal Server Error`

FastAPI commonly returns validation errors with status code `422`.

Do not treat status codes as decoration.

Clients use them for control flow.

Monitoring systems use them for alerts.

Retries may depend on them.

API documentation includes them.

Tests should assert them.

If a create operation returns `200` sometimes and `201` other times without reason, clients become harder to write.

Consistency is part of professionalism.

---

# Raising HTTP Errors

When an endpoint cannot complete normally, raise `HTTPException`.

```python
from fastapi import HTTPException


@app.get("/products/{product_id}")
def read_product(product_id: int):
    product = find_product(product_id)

    if product is None:
        raise HTTPException(status_code=404, detail="Product not found")

    return product
```

This is better than returning ad hoc dictionaries such as:

```python
return {"error": "Product not found"}
```

The status code and error body should communicate the failure clearly.

A good error response helps clients decide what to do next.

For example:

* `404` means the resource does not exist
* `401` means authentication is missing or invalid
* `403` means authentication exists but permission is denied
* `409` often means a conflict with current state
* `422` often means input validation failed

Use errors deliberately.

An API is not only successful responses.

It is also a stable failure contract.

---

# Sync and Async Endpoints

FastAPI supports both normal functions and async functions.

```python
@app.get("/sync")
def sync_endpoint():
    return {"mode": "sync"}


@app.get("/async")
async def async_endpoint():
    return {"mode": "async"}
```

This flexibility is useful.

But it also creates confusion.

The rule of thumb is:

```text
use async def when the endpoint awaits non-blocking async I/O
use def when the endpoint calls ordinary blocking code
```

Async is not magic speed.

Async helps when work is I/O-bound and the libraries involved are async-compatible.

Examples:

* async database clients
* async HTTP clients
* async message clients
* async file or stream operations when supported

Bad use of async can make an application worse.

For example:

```python
@app.get("/bad")
async def bad_endpoint():
    result = slow_blocking_database_call()
    return result
```

This looks async, but the blocking database call can still block the event loop.

A blocked event loop cannot efficiently serve other requests.

If you use `async def`, be disciplined about what runs inside it.

Do not call slow blocking libraries directly inside async endpoints unless you understand how FastAPI and the server will execute them.

---

# The Event Loop View

An ASGI server such as Uvicorn runs an event loop.

The event loop can manage many concurrent tasks when those tasks spend time waiting on non-blocking I/O.

Imagine three requests:

```text
Request A waits for a database response
Request B waits for an external HTTP API
Request C waits for a cache response
```

If the waits are non-blocking, the event loop can switch between them.

That is the promise of async web servers.

But CPU-heavy work is different.

If an endpoint performs a large computation, it occupies CPU time.

The event loop cannot turn one CPU core into many CPU cores.

For CPU-heavy work, consider:

* moving work to a task queue
* using worker processes
* using optimized libraries
* using a separate service
* caching results
* designing the API as an asynchronous job flow

FastAPI is often used for machine learning inference APIs.

That can work well.

But a model inference call may be CPU-bound, GPU-bound, memory-bound, or I/O-bound depending on the model and serving setup.

Do not assume `async def` solves model serving performance.

Measure the real bottleneck.

---

# Dependency Injection

FastAPI's dependency system is one of its most important features.

A dependency is a callable that provides something an endpoint needs.

```python
from typing import Annotated

from fastapi import Depends


def get_current_user():
    return {"id": 1, "email": "user@example.com"}


@app.get("/me")
def read_me(user: Annotated[dict, Depends(get_current_user)]):
    return user
```

The endpoint declares that it needs `user`.

FastAPI calls `get_current_user`.

The returned value is passed into the endpoint.

This keeps endpoint code focused.

Instead of manually doing authentication setup in every route, you declare the dependency.

Dependencies can provide:

* current user
* database sessions
* configuration
* service objects
* authorization checks
* pagination options
* request-scoped resources
* shared validation logic
* feature flags

The important idea is:

```text
dependencies make route requirements explicit
```

When you read the signature, you can see what the endpoint needs.

---

# Dependencies with Yield

Some dependencies need setup and cleanup.

Database sessions are a common example.

```python
from collections.abc import Generator


def get_session() -> Generator[Session, None, None]:
    session = Session(engine)
    try:
        yield session
    finally:
        session.close()
```

The dependency creates a session.

It yields the session to the endpoint.

After the request is finished, the cleanup block runs.

This pattern should remind you of context managers.

The shape is similar:

```text
setup
yield resource
cleanup
```

This is a clean way to manage request-scoped resources.

But be careful.

Do not hide too much business behavior inside dependencies.

Dependencies are excellent for composition and cross-cutting concerns.

They become confusing when they perform surprising side effects.

Good dependency:

```text
get a database session
```

Good dependency:

```text
load and verify the current user
```

Risky dependency:

```text
silently create an order, charge a card, and send an email
```

Dependencies should make the endpoint easier to reason about, not harder.

---

# Sub-Dependencies

Dependencies can depend on other dependencies.

```python
def get_token_header(x_token: Annotated[str, Header()]):
    if x_token != "expected-token":
        raise HTTPException(status_code=400, detail="Invalid X-Token")
    return x_token


def get_current_user(token: Annotated[str, Depends(get_token_header)]):
    return load_user_from_token(token)
```

This creates a dependency graph.

FastAPI resolves the graph for the request.

This is powerful because common behavior can be assembled from smaller pieces.

But dependency graphs should stay readable.

If a route's behavior depends on five nested dependency layers, debugging becomes harder.

Prefer names that explain purpose:

```python
CurrentUser = Annotated[User, Depends(get_current_user)]
AdminUser = Annotated[User, Depends(require_admin_user)]
SessionDep = Annotated[Session, Depends(get_session)]
```

Aliases like these make endpoint signatures easier to scan.

```python
@app.get("/admin/reports")
def list_reports(user: AdminUser, session: SessionDep):
    ...
```

This reads like a contract:

```text
this route requires an admin user and a database session
```

---

# Routers and Larger Applications

A single-file FastAPI app is fine for learning.

Real applications should be split into modules.

FastAPI uses `APIRouter` for grouping routes.

Example structure:

```text
app/
    main.py
    api/
        users.py
        products.py
        orders.py
    core/
        config.py
        security.py
    db/
        session.py
        models.py
    services/
        users.py
        orders.py
```

A router might look like:

```python
from fastapi import APIRouter

router = APIRouter(prefix="/users", tags=["users"])


@router.get("/{user_id}")
def read_user(user_id: int):
    return {"user_id": user_id}
```

Then `main.py` includes it:

```python
from fastapi import FastAPI

from app.api import users

app = FastAPI()

app.include_router(users.router)
```

Routers help with:

* grouping related endpoints
* applying shared prefixes
* applying shared tags
* applying shared dependencies
* keeping modules small
* making OpenAPI docs easier to navigate

Good application structure is not about making many folders.

It is about keeping responsibility clear.

If every endpoint, model, database function, and service lives in `main.py`, the app becomes hard to maintain.

If every tiny function gets its own file, the app becomes annoying to navigate.

Structure should follow the size of the system.

---

# Tags, Summaries, and API Documentation

FastAPI generates documentation from your routes.

You can improve that documentation with metadata.

```python
@app.get(
    "/products/{product_id}",
    response_model=ProductPublic,
    summary="Read one product",
    tags=["products"],
)
def read_product(product_id: int):
    ...
```

The generated docs are only as useful as the code contract.

Good docs come from:

* clear route paths
* clear method choices
* clear model names
* clear field names
* response models
* accurate status codes
* route summaries where helpful
* consistent tags

Do not treat generated docs as a license to ignore documentation quality.

Generated docs expose your design.

If the design is messy, the docs will reveal the mess.

That is useful.

It gives you feedback.

When the OpenAPI page is hard to understand, the API probably needs better names, grouping, models, or status behavior.

---

# OpenAPI

OpenAPI is a standard format for describing HTTP APIs.

FastAPI automatically generates an OpenAPI schema.

This schema can be used for:

* interactive documentation
* client generation
* SDK generation
* API gateways
* contract testing
* documentation publishing
* frontend-backend collaboration

The schema is available as JSON.

That matters because it is machine-readable.

Humans can read `/docs`.

Tools can read `/openapi.json`.

This is one of the reasons FastAPI is useful in teams.

Instead of manually maintaining a separate API document, the API contract can be generated from the application.

But there is a warning:

```text
generated does not automatically mean correct
```

If your response model is wrong, the schema is wrong.

If your status codes are incomplete, the schema is incomplete.

If you return dynamic shapes without declaring them, clients cannot rely on the generated contract.

FastAPI makes good API contracts easier.

It does not make API design automatic.

---

# Data Flow Through a FastAPI Endpoint

Consider this endpoint:

```python
@app.post("/products", response_model=ProductPublic, status_code=201)
def create_product(product: ProductCreate, session: SessionDep):
    saved = save_product(session, product)
    return saved
```

The request flow is:

```text
HTTP request arrives
path and method are matched
query/path/header/body data is extracted
dependencies are resolved
request body is validated into ProductCreate
endpoint function is called
return value is validated/serialized as ProductPublic
HTTP response is sent
cleanup for yield dependencies runs
```

This flow explains where different errors occur.

If the path does not match, the endpoint is not called.

If validation fails, the endpoint is not called.

If dependency resolution fails, the endpoint is not called successfully.

If the endpoint raises an HTTP exception, FastAPI converts it into an error response.

If response serialization fails, the bug is usually in your code, not the client's request.

Knowing the flow helps debugging.

You can ask:

```text
did the request reach the endpoint?
did dependency resolution run?
did request validation fail?
did business logic fail?
did response serialization fail?
```

Those questions quickly narrow the problem.

---

# Headers and Cookies

APIs often need headers.

FastAPI can declare them explicitly:

```python
from fastapi import Header


@app.get("/request-info")
def request_info(user_agent: Annotated[str | None, Header()] = None):
    return {"user_agent": user_agent}
```

Headers are commonly used for:

* authentication
* content negotiation
* idempotency keys
* request tracing
* client versioning
* correlation IDs

Cookies can also be declared:

```python
from fastapi import Cookie


@app.get("/session")
def read_session(session_id: Annotated[str | None, Cookie()] = None):
    return {"session_id": session_id}
```

For browser-heavy applications, cookies may matter.

For service APIs, bearer tokens in headers are more common.

FastAPI supports both.

The choice should come from the client model and security requirements.

---

# Forms and File Uploads

Not every request body is JSON.

Some APIs receive form data or files.

FastAPI supports those too.

```python
from fastapi import File, UploadFile


@app.post("/upload")
async def upload_file(file: UploadFile = File()):
    return {"filename": file.filename, "content_type": file.content_type}
```

`UploadFile` is usually better than reading an entire file into memory immediately.

It gives you file metadata and a file-like interface.

File uploads require careful design:

* limit accepted file size
* validate content type
* avoid trusting filenames
* store files outside the application code directory
* scan untrusted files if needed
* use object storage for larger systems
* avoid loading huge files fully into memory

Framework support is only the starting point.

Production file handling is a security and operations problem too.

---

# Background Tasks

FastAPI supports background tasks that run after the response is sent.

```python
from fastapi import BackgroundTasks


def send_welcome_email(email: str):
    ...


@app.post("/signup")
def signup(user: UserCreate, background_tasks: BackgroundTasks):
    created = create_user(user)
    background_tasks.add_task(send_welcome_email, created.email)
    return {"id": created.id}
```

This is useful for small follow-up work:

* sending a notification
* writing an audit log
* updating a lightweight cache
* triggering a small side effect

But background tasks are not a replacement for a durable task queue.

If the process crashes, in-process background work may be lost.

For important work, use a real job system:

* Celery
* RQ
* Dramatiq
* cloud task queues
* message brokers

The professional rule is:

```text
use FastAPI background tasks for convenient non-critical work
use durable queues for important asynchronous work
```

---

# Middleware

Middleware wraps request handling.

It can run before and after endpoint logic.

Middleware is useful for cross-cutting concerns:

* request logging
* response headers
* CORS
* timing
* tracing
* metrics
* authentication in some architectures

Example:

```python
import time

from fastapi import Request


@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    elapsed = time.perf_counter() - start
    response.headers["X-Process-Time"] = str(elapsed)
    return response
```

Middleware should be used carefully.

It affects many routes.

A bug in middleware can break the whole application.

Endpoint-specific behavior often belongs in dependencies instead.

Use middleware when the behavior is truly around the request-response cycle.

Use dependencies when the behavior is part of a route's declared requirements.

---

# CORS

CORS stands for Cross-Origin Resource Sharing.

It controls whether browsers allow frontend JavaScript from one origin to call an API at another origin.

For example:

```text
frontend: https://app.example.com
api:      https://api.example.com
```

These are different origins.

Browser security rules apply.

FastAPI can configure CORS middleware:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Avoid using wildcard origins casually in production.

Development settings are often loose.

Production settings should be deliberate.

CORS is not authentication.

CORS does not protect your API from non-browser clients.

It tells browsers which cross-origin calls are allowed.

You still need authentication, authorization, input validation, rate limiting, and monitoring.

---

# Security

FastAPI includes security utilities for common API authentication patterns.

For example, OAuth2 bearer token flows can be represented in code and documentation.

A simplified token dependency might look like:

```python
from typing import Annotated

from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = decode_and_load_user(token)
    if user is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return user
```

Then routes can require the user:

```python
@app.get("/me")
def read_me(user: Annotated[User, Depends(get_current_user)]):
    return user
```

Security has layers:

* authentication proves who the caller is
* authorization decides what the caller may do
* validation checks the shape of input
* rate limiting reduces abuse
* logging supports investigation
* secret management protects credentials
* transport security protects data in transit

FastAPI gives you useful building blocks.

It does not design your security model for you.

Avoid common mistakes:

* storing plain-text passwords
* returning password hashes
* trusting user IDs from request bodies
* confusing authentication with authorization
* using weak token validation
* leaking internal error details
* hard-coding secrets in source code

Security should be designed early.

Adding it later is much harder.

---

# Database Access

FastAPI does not force one database layer.

This is different from Django, where the ORM is built into the framework's normal way of working.

With FastAPI, you may use:

* SQLModel
* SQLAlchemy
* async SQLAlchemy
* encode/databases
* Tortoise ORM
* raw database drivers
* MongoDB drivers
* Redis clients
* external service clients

The framework does not care as long as your endpoint code and dependencies are designed correctly.

The common pattern is:

```text
create a database/session dependency
inject it into routes or services
perform data access in a controlled layer
commit or rollback deliberately
close resources after request
```

A simple route should not become a dumping ground for SQL, validation, authorization, and business rules.

For small apps, direct database calls in routes may be acceptable.

For growing apps, split responsibilities:

```text
route layer: HTTP concerns
schema layer: request/response models
service layer: business use cases
repository/data layer: persistence details
```

Do not over-engineer too early.

But do not let every route become a script.

The larger the API becomes, the more valuable boundaries become.

---

# SQLModel Example Shape

FastAPI's official docs often show SQLModel for relational examples.

SQLModel combines ideas from SQLAlchemy and Pydantic.

A simplified model might look like:

```python
from sqlmodel import Field, SQLModel


class Product(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    price: float
```

You can create a session dependency:

```python
from typing import Annotated

from fastapi import Depends
from sqlmodel import Session


def get_session():
    with Session(engine) as session:
        yield session


SessionDep = Annotated[Session, Depends(get_session)]
```

Then use it:

```python
@app.post("/products", response_model=ProductPublic)
def create_product(product: ProductCreate, session: SessionDep):
    db_product = Product(**product.model_dump())
    session.add(db_product)
    session.commit()
    session.refresh(db_product)
    return db_product
```

The exact database tool may differ in your project.

The architectural lesson remains:

```text
database access should have a clear lifecycle and a clear boundary
```

FastAPI's dependency system makes request-scoped database resources natural.

---

# Partial Updates

Partial updates are subtle.

Suppose a client sends:

```json
{
  "full_name": "New Name"
}
```

That is different from:

```json
{
  "full_name": null
}
```

And both are different from:

```json
{}
```

A common update model uses optional fields:

```python
class UserUpdate(BaseModel):
    email: str | None = None
    full_name: str | None = None
```

Then:

```python
update_data = user_update.model_dump(exclude_unset=True)
```

`exclude_unset=True` means:

```text
include only fields the client actually sent
```

This matters.

Without it, you may accidentally overwrite fields with `None`.

Professional API design requires thinking about:

* omitted field
* explicit null
* empty string
* unchanged value
* invalid value

They are not always the same thing.

FastAPI and Pydantic give you tools to express the difference.

You must still choose the behavior.

---

# Testing FastAPI Applications

FastAPI applications are straightforward to test.

The basic pattern uses `TestClient`.

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_read_root():
    response = client.get("/")

    assert response.status_code == 200
    assert response.json() == {"message": "Hello"}
```

Tests should cover:

* successful requests
* validation failures
* authentication failures
* authorization failures
* missing resources
* duplicate resources
* status codes
* response shapes
* important headers

Because FastAPI validates input before route logic runs, tests should verify validation behavior explicitly.

For example:

```python
def test_invalid_product_id():
    response = client.get("/products/not-an-int")

    assert response.status_code == 422
```

This confirms the API contract.

The point is not to test FastAPI itself.

The point is to test the contract your app exposes.

---

# Overriding Dependencies in Tests

Dependency injection becomes especially useful in tests.

Suppose your endpoint depends on `get_current_user`.

In a test, you can override that dependency.

```python
def fake_current_user():
    return User(id=1, email="test@example.com")


app.dependency_overrides[get_current_user] = fake_current_user
```

Now requests in that test can behave as if a user is authenticated.

After the test, clear overrides:

```python
app.dependency_overrides.clear()
```

This makes tests cleaner.

Instead of building real tokens for every route test, you can test route behavior with controlled dependencies.

But do not overuse overrides.

Some tests should exercise real authentication behavior too.

A useful test mix is:

* unit tests for pure business logic
* API tests with dependency overrides
* integration tests with real database behavior
* a few end-to-end tests through the real authentication path

Testing strategy should match risk.

---

# Lifespan Events

Applications often need startup and shutdown behavior.

Examples:

* opening connection pools
* loading configuration
* warming caches
* initializing clients
* closing resources
* flushing telemetry

FastAPI supports lifespan handling.

A common modern pattern uses an async context manager:

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI


@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.started = True
    yield
    app.state.started = False


app = FastAPI(lifespan=lifespan)
```

The code before `yield` runs during startup.

The code after `yield` runs during shutdown.

Use lifespan for application-level resources.

Use dependencies for request-level resources.

That distinction is important:

```text
lifespan: one setup for the application process
dependency: setup for a request or declared operation
```

Mixing these can lead to leaks, duplicate resources, or slow requests.

---

# Configuration

FastAPI does not impose a configuration system.

Common options include:

* environment variables
* Pydantic settings models
* `.env` files for local development
* secrets managers in production
* platform-specific configuration

A settings object might describe:

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False
    allowed_origins: list[str] = []


settings = Settings()
```

Configuration should be explicit and environment-aware.

Do not hard-code production secrets.

Do not make development defaults accidentally safe-looking in production.

Do not scatter environment reads throughout the codebase.

Prefer a central settings object that is imported where needed or provided through dependencies.

Good configuration design makes deployment easier.

Bad configuration design causes strange failures across machines.

---

# Responses Beyond JSON

FastAPI is often used for JSON APIs.

But it can return other response types:

* plain text
* HTML
* redirects
* files
* streaming responses
* custom response classes

Example:

```python
from fastapi.responses import PlainTextResponse


@app.get("/health", response_class=PlainTextResponse)
def health():
    return "ok"
```

For most APIs, JSON remains the default.

But knowing response classes matters when building:

* file download endpoints
* streaming endpoints
* health checks
* webhook receivers
* HTML previews
* internal tools

Use the response type that matches the job.

Do not force everything into JSON when the protocol or client needs something else.

---

# WebSockets

FastAPI supports WebSockets through Starlette.

WebSockets are useful for long-lived bidirectional communication.

Examples:

* chat
* live dashboards
* collaborative tools
* streaming status updates
* real-time notifications

A small example:

```python
from fastapi import WebSocket


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        message = await websocket.receive_text()
        await websocket.send_text(f"Echo: {message}")
```

WebSockets are more operationally complex than ordinary request-response APIs.

You must think about:

* connection lifecycle
* authentication
* reconnect behavior
* backpressure
* message format
* horizontal scaling
* load balancers
* timeouts

Use WebSockets when you need real-time bidirectional communication.

Do not use them just because they feel modern.

For many use cases, polling, server-sent events, or background job status endpoints are simpler.

---

# FastAPI and Machine Learning Services

FastAPI is popular for machine learning APIs.

The reason is easy to understand.

ML services often need:

* JSON input
* validation
* model loading at startup
* predictable response shapes
* OpenAPI documentation
* async calls to storage or other services
* deployment as an independent service

Example shape:

```python
class PredictionRequest(BaseModel):
    text: str


class PredictionResponse(BaseModel):
    label: str
    score: float


@app.post("/predict", response_model=PredictionResponse)
def predict(payload: PredictionRequest):
    result = model.predict(payload.text)
    return PredictionResponse(label=result.label, score=result.score)
```

This is clean.

But production model serving has extra concerns:

* model load time
* memory usage
* CPU or GPU contention
* request batching
* latency targets
* timeout behavior
* versioned models
* rollback strategy
* observability
* input size limits
* adversarial inputs

FastAPI can expose the service.

It does not solve model serving architecture by itself.

The framework is the front door.

The system behind the door still needs engineering.

---

# FastAPI Versus Flask

FastAPI and Flask can both build APIs.

The difference is in defaults and emphasis.

Flask emphasizes:

* small core
* simple routing
* extensions
* flexibility
* explicit choices

FastAPI emphasizes:

* type hints
* validation
* generated docs
* dependency injection
* async support
* API contracts

In Flask, you often add libraries for request validation, serialization, API docs, and dependency patterns.

In FastAPI, those concerns are central to the framework experience.

Choose Flask when:

* you want minimalism
* you need a small web surface
* you prefer choosing every supporting tool
* you are building simple server-rendered pages or small services

Choose FastAPI when:

* your system is API-first
* request and response schemas matter
* generated OpenAPI is useful
* type hints are already part of the team culture
* dependency injection fits your design
* async I/O is relevant

Neither framework makes the other obsolete.

They optimize for different developer experiences.

---

# FastAPI Versus Django

Django is a full-stack web framework.

FastAPI is an API framework.

Django gives you:

* ORM
* migrations
* admin
* templates
* forms
* auth
* sessions
* middleware
* project conventions

FastAPI gives you:

* API routing
* validation
* serialization
* dependency injection
* OpenAPI docs
* ASGI-first design
* flexible database choices

Django is excellent when you need an integrated database-backed web application.

FastAPI is excellent when you need a clear HTTP API service.

They can even coexist.

For example:

* Django powers the main admin and business application
* FastAPI powers a high-throughput API service
* FastAPI powers an ML inference service
* Django and FastAPI share a database carefully through well-defined ownership rules

The main question is not:

```text
which framework is best?
```

The better question is:

```text
what kind of system am I building?
```

Full-stack application?

Admin-heavy business system?

API-first backend?

Microservice?

Model serving API?

Internal automation service?

The answer changes the framework choice.

---

# FastAPI and Type Hints

FastAPI depends heavily on type hints.

This makes earlier chapters in this book important.

When you write:

```python
def read_item(item_id: int, q: str | None = None):
    ...
```

FastAPI sees:

* `item_id` should be an integer
* `q` may be a string or `None`
* `q` defaults to `None`

When you write:

```python
class Item(BaseModel):
    name: str
    tags: list[str] = []
```

Pydantic sees:

* `name` is required
* `tags` should be a list of strings
* `tags` has a default

This is powerful because the same syntax helps:

* humans read code
* editors provide completion
* type checkers catch mistakes
* FastAPI validate requests
* Pydantic serialize data
* OpenAPI describe the API

This is one of the most satisfying examples of Python's gradual typing story.

Type hints are optional in the language.

But frameworks can use them to create stronger tools.

FastAPI is a strong example of that design.

---

# Common Mistakes

The first common mistake is putting all logic in route functions.

Routes should handle HTTP concerns.

Business rules should often live in services or domain functions.

The second common mistake is using one Pydantic model for everything.

Create, update, internal, and public response models often have different shapes.

The third common mistake is misusing async.

An async endpoint that calls blocking code is not truly non-blocking.

The fourth common mistake is trusting generated docs without checking them.

Generated docs reflect your declarations.

Wrong declarations create wrong docs.

The fifth common mistake is returning raw database objects without controlling response shape.

Response models should protect API boundaries.

The sixth common mistake is treating dependencies as a place to hide business side effects.

Dependencies should make requirements explicit.

The seventh common mistake is ignoring status codes.

Clients need consistent status behavior.

The eighth common mistake is weak configuration.

Hard-coded secrets and scattered environment reads cause production pain.

The ninth common mistake is skipping tests for validation failures.

Invalid input is part of the API contract.

The tenth common mistake is treating FastAPI as a complete architecture.

FastAPI is a framework.

Architecture still requires decisions about boundaries, data ownership, deployment, observability, and failure handling.

---

# Professional FastAPI Checklist

Before calling a FastAPI API production-ready, check:

* Are routes grouped with routers?
* Are path names consistent?
* Are HTTP methods appropriate?
* Are request models separate from response models?
* Are partial update semantics clear?
* Are status codes deliberate?
* Are errors predictable?
* Are dependencies named clearly?
* Are database sessions request-scoped?
* Are blocking calls avoided inside async endpoints?
* Are secrets loaded from safe configuration?
* Is CORS configured deliberately?
* Is authentication implemented correctly?
* Is authorization checked explicitly?
* Are response models preventing data leaks?
* Are validation failures tested?
* Are dependency overrides used cleanly in tests?
* Are important endpoints covered by integration tests?
* Are logs useful for debugging?
* Are metrics and health checks present?
* Is deployment using a proper ASGI server setup?
* Are timeouts and request size limits considered?
* Is generated OpenAPI documentation reviewed?

This checklist is not glamorous.

It is how APIs become dependable.

---

# A Small Complete Example

Here is a compact but realistic shape.

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from pydantic import BaseModel, Field

app = FastAPI(title="Products API")


class ProductCreate(BaseModel):
    name: str = Field(min_length=1, max_length=120)
    price: float = Field(gt=0)


class ProductPublic(BaseModel):
    id: int
    name: str
    price: float


class ProductUpdate(BaseModel):
    name: str | None = Field(default=None, min_length=1, max_length=120)
    price: float | None = Field(default=None, gt=0)


products: dict[int, ProductPublic] = {}
next_id = 1


def get_store() -> dict[int, ProductPublic]:
    return products


StoreDep = Annotated[dict[int, ProductPublic], Depends(get_store)]


@app.post("/products", response_model=ProductPublic, status_code=status.HTTP_201_CREATED)
def create_product(payload: ProductCreate, store: StoreDep):
    global next_id

    product = ProductPublic(id=next_id, **payload.model_dump())
    store[next_id] = product
    next_id += 1

    return product


@app.get("/products/{product_id}", response_model=ProductPublic)
def read_product(product_id: int, store: StoreDep):
    product = store.get(product_id)

    if product is None:
        raise HTTPException(status_code=404, detail="Product not found")

    return product


@app.patch("/products/{product_id}", response_model=ProductPublic)
def update_product(product_id: int, payload: ProductUpdate, store: StoreDep):
    product = store.get(product_id)

    if product is None:
        raise HTTPException(status_code=404, detail="Product not found")

    update_data = payload.model_dump(exclude_unset=True)
    updated = product.model_copy(update=update_data)
    store[product_id] = updated

    return updated
```

This example is not production-ready.

The in-memory dictionary is only for demonstration.

But the API shape is useful:

* create model
* public response model
* update model
* dependency
* status code
* not-found error
* partial update behavior
* response model boundary

A real version would replace the dictionary with a database layer.

The route contracts would remain similar.

---

# Summary

FastAPI is a type-driven framework for building APIs.

Its core idea is that Python function signatures and type annotations can describe HTTP API contracts.

FastAPI uses Starlette for web mechanics and Pydantic for data validation and serialization.

Routes are declared with path operation decorators.

Path parameters, query parameters, headers, cookies, and request bodies are extracted and validated from function signatures and models.

Response models shape output and help prevent accidental data leaks.

Dependencies declare what endpoints need and provide a powerful composition mechanism.

Routers organize larger applications.

OpenAPI generation makes the API contract visible to humans and tools.

Async support is valuable when used with non-blocking I/O, but it does not automatically solve CPU-heavy or blocking work.

Testing is straightforward, and dependency overrides make many route tests clean.

FastAPI is especially strong for API-first systems, service backends, internal platforms, and machine learning serving APIs.

Its danger is that the elegant surface can hide architectural responsibilities.

You still need good boundaries, clear models, security, configuration, tests, observability, and deployment discipline.

The central lesson is:

```text
FastAPI turns Python type declarations into executable API contracts
```

Use that power to make APIs clearer, not merely shorter.

---

# Exercises

1. Create a FastAPI app with a `GET /health` endpoint that returns `{"status": "ok"}`.

2. Add a path parameter endpoint `GET /items/{item_id}` where `item_id` must be an integer.

3. Add query parameters `limit` and `offset` with defaults and validation constraints.

4. Create separate Pydantic models for creating, updating, and returning a product.

5. Add a `POST /products` endpoint that returns status code `201`.

6. Add a `GET /products/{product_id}` endpoint that raises `404` when the product does not exist.

7. Add a `PATCH /products/{product_id}` endpoint that uses `exclude_unset=True`.

8. Create a dependency that provides a fake current user.

9. Protect one route with that dependency.

10. Split product routes into an `APIRouter` and include it in `main.py`.

11. Write tests for successful creation, invalid input, missing product, and partial update.

12. Override a dependency in a test.

13. Open the generated API docs and inspect whether model names and route tags are clear.

14. Identify one blocking operation in a hypothetical async endpoint and explain why it is dangerous.

15. Design a production checklist for a FastAPI service your team would deploy.

---

# Part I Closing

Part I of Volume IV covered three major Python web frameworks.

Flask taught the minimal web application shape.

Django taught the integrated full-stack application shape.

FastAPI taught the type-driven API service shape.

Together, they show that "web development in Python" is not one thing.

It can mean:

* a small service
* a full-stack product
* an admin-heavy business system
* a public API
* a backend for a frontend
* a machine learning serving layer
* an internal automation platform

The professional skill is not memorizing one framework and using it everywhere.

The professional skill is understanding the shape of the problem.

Then choose the framework whose strengths fit that shape.

---

# Preview of Chapter 88

Chapter 87 studied FastAPI.

We saw how Python type hints, Pydantic models, dependency injection, ASGI, OpenAPI, routers, testing, async support, and deployment concerns come together to create modern API services.

Next we begin Part II of Volume IV: Data and Scientific Computing.

The first chapter in that part is NumPy.

NumPy is one of the most important libraries in the Python ecosystem.

It gives Python efficient multidimensional arrays and vectorized numerical operations.

The transition is:

```text
web frameworks help Python communicate with users and systems
NumPy helps Python compute efficiently over structured numerical data
```

Chapter 88 will show why NumPy became the foundation for scientific computing, data analysis, and machine learning in Python.
