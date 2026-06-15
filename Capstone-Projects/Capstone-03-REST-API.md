# Capstone 03 - REST API

## Project Brief

In this project, you will build a small REST API for managing notes. The API will expose HTTP endpoints for creating, listing, reading, updating, and deleting notes, with request validation, response models, structured errors, persistence, tests, and clear API design.

The project moves from local command-line programs to networked software. It teaches how Python code becomes a service that other programs can call over HTTP, and how application behavior changes when input comes from clients instead of a terminal.

By the end, the reader should have a working API service, a tested application layer, documented routes, and a practical understanding of how REST-style APIs are designed and implemented in Python.

---

# Why This Project Matters

The first two capstones were local programs.

The Todo CLI managed structured local data.

The File Organizer automated local filesystem work.

This capstone crosses an important boundary:

```text
local program -> networked service
```

That boundary changes the engineering problem.

A command-line tool receives arguments from one user.

A REST API receives HTTP requests from many possible clients.

A CLI prints output to a terminal.

An API returns status codes and structured response bodies.

A CLI can assume the user is nearby.

An API must treat every request as external input.

A CLI failure can print an error message.

An API failure must return a useful HTTP response.

This project teaches that APIs are not just functions with URLs.

APIs are contracts.

They define how other software interacts with your system.

---

# What You Will Build

You will build a notes API.

It will support:

```text
create note
list notes
get one note
update note
delete note
search notes
filter by archived status
archive note
restore note
health check
```

The API will expose endpoints such as:

```text
GET     /health
POST    /notes
GET     /notes
GET     /notes/{note_id}
PATCH   /notes/{note_id}
DELETE  /notes/{note_id}
POST    /notes/{note_id}/archive
POST    /notes/{note_id}/restore
```

The API will store notes persistently.

For this capstone, use SQLite.

SQLite is a good fit because:

* it is part of Python's standard library
* it requires no separate database server
* it teaches SQL and persistence
* it is enough for a learning API
* it creates a natural bridge to later projects

The project should use FastAPI for the HTTP layer.

FastAPI is appropriate here because the book has already covered it, and it makes request validation, response models, and OpenAPI documentation visible.

---

# Example API Usage

Create a note:

```bash
curl -X POST http://localhost:8000/notes \
  -H "Content-Type: application/json" \
  -d '{"title": "Read Chapter 83", "body": "Review API design."}'
```

Response:

```json
{
  "id": 1,
  "title": "Read Chapter 83",
  "body": "Review API design.",
  "archived": false,
  "created_at": "2026-06-15T10:00:00+00:00",
  "updated_at": "2026-06-15T10:00:00+00:00"
}
```

List notes:

```bash
curl http://localhost:8000/notes
```

Get one note:

```bash
curl http://localhost:8000/notes/1
```

Update a note:

```bash
curl -X PATCH http://localhost:8000/notes/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Review API chapter"}'
```

Delete a note:

```bash
curl -X DELETE http://localhost:8000/notes/1
```

This API is small, but it contains most of the core ideas in API engineering.

---

# Requirements

The REST API must:

* expose HTTP endpoints
* validate request bodies
* return structured JSON responses
* return useful HTTP status codes
* persist notes in SQLite
* separate HTTP layer from service logic
* have application-specific errors
* have tests for service and API behavior
* support local development
* include OpenAPI documentation through FastAPI

It must support these behaviors:

```text
create a note
list notes
filter notes by archived status
search notes by text
read one note
update title and body
archive a note
restore a note
delete a note
return 404 for missing notes
return 422 for invalid request bodies
```

The API should be deterministic and easy to test.

---

# Non-Requirements

This capstone will not include:

* user accounts
* authentication
* authorization
* multi-tenant data
* PostgreSQL
* Docker
* deployment
* rate limiting
* background jobs
* full-text search engine
* frontend UI

Those are important topics.

They are intentionally excluded.

The goal is to learn clean API design before adding production complexity.

Authentication and authorization will matter in real systems, but adding them now would distract from the core API lifecycle.

---

# Project Structure

A clean structure:

```text
notes-api/
    pyproject.toml
    README.md
    src/
        notes_api/
            __init__.py
            main.py
            api.py
            models.py
            schemas.py
            repository.py
            service.py
            database.py
            errors.py
    tests/
        test_repository.py
        test_service.py
        test_api.py
```

Responsibilities:

`main.py` creates the FastAPI application.

`api.py` defines routes.

`schemas.py` defines request and response models.

`models.py` defines domain objects.

`repository.py` handles database access.

`service.py` implements application operations.

`database.py` manages SQLite connections and schema setup.

`errors.py` defines application errors.

The structure should make this clear:

```text
HTTP routes should not contain all business logic
```

Routes are adapters.

The service layer owns behavior.

The repository owns persistence.

---

# Architecture

The API should have layers:

```text
client
  -> FastAPI route
  -> request schema validation
  -> service function
  -> repository
  -> SQLite
  -> response schema
  -> JSON response
```

Each layer has a job.

The route understands HTTP.

The schema understands request and response shapes.

The service understands application rules.

The repository understands SQL.

The database stores durable data.

Do not let the route do everything.

Weak route:

```python
@app.post("/notes")
def create_note(payload: dict):
    conn = sqlite3.connect("notes.db")
    conn.execute("INSERT INTO notes ...")
    conn.commit()
    return {"ok": True}
```

Better route:

```python
@router.post("/notes", response_model=NoteResponse, status_code=201)
def create_note(payload: CreateNoteRequest, service: NoteService = Depends(get_service)):
    return service.create_note(title=payload.title, body=payload.body)
```

The route is small.

The behavior is testable outside HTTP.

---

# Domain Model

A note needs:

* ID
* title
* body
* archived flag
* created timestamp
* updated timestamp

Domain object:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Note:
    id: int
    title: str
    body: str
    archived: bool
    created_at: str
    updated_at: str
```

Why use a domain object if FastAPI already has Pydantic models?

Because API schemas and domain models have different responsibilities.

The domain model represents the application concept.

The API schema represents the external contract.

They may be similar today.

They may differ later.

Keeping them separate teaches clean boundaries.

---

# Request Schemas

Create request:

```python
from pydantic import BaseModel, Field


class CreateNoteRequest(BaseModel):
    title: str = Field(min_length=1, max_length=120)
    body: str = Field(default="", max_length=5000)
```

Update request:

```python
class UpdateNoteRequest(BaseModel):
    title: str | None = Field(default=None, min_length=1, max_length=120)
    body: str | None = Field(default=None, max_length=5000)
```

The update request uses optional fields because a PATCH request may update only part of the resource.

But it should not allow an empty update.

That rule belongs in the service layer:

```text
at least one field must be provided
```

Validation has two layers:

```text
schema validation -> shape and basic constraints
service validation -> application rules
```

Both are useful.

---

# Response Schema

Response model:

```python
class NoteResponse(BaseModel):
    id: int
    title: str
    body: str
    archived: bool
    created_at: str
    updated_at: str
```

The response should not accidentally expose internal details.

For example, if the database later stores:

```text
deleted_at
internal_version
owner_id
```

those fields should not automatically appear in the API.

Response schemas protect the external contract.

---

# HTTP Status Codes

Use status codes intentionally.

Common choices:

```text
200 OK                  successful read or update
201 Created             resource created
204 No Content          resource deleted successfully
400 Bad Request         invalid application-level request
404 Not Found           note does not exist
422 Unprocessable Entity schema validation failed
500 Internal Server Error unexpected server failure
```

Do not return `200 OK` for everything.

Weak API:

```json
{
  "success": false,
  "error": "not found"
}
```

with status `200`.

Better:

```json
{
  "detail": "note not found"
}
```

with status `404`.

HTTP already has a language.

Use it.

---

# Error Model

Define application errors:

```python
class NotesError(Exception):
    pass


class NoteNotFound(NotesError):
    pass


class InvalidNoteUpdate(NotesError):
    pass
```

The API layer can translate these into HTTP responses.

For example:

```python
from fastapi import HTTPException


def translate_error(error: NotesError) -> HTTPException:
    if isinstance(error, NoteNotFound):
        return HTTPException(status_code=404, detail=str(error))
    if isinstance(error, InvalidNoteUpdate):
        return HTTPException(status_code=400, detail=str(error))
    return HTTPException(status_code=500, detail="internal server error")
```

This keeps domain errors independent of HTTP.

The service layer should not raise `HTTPException`.

Why?

Because `HTTPException` belongs to FastAPI.

The application behavior should remain testable without a web framework.

---

# SQLite Schema

The database table:

```sql
CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    archived INTEGER NOT NULL DEFAULT 0,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL
);
```

SQLite does not have a native boolean type.

Use `0` and `1`.

In Python:

```python
archived = bool(row["archived"])
```

When writing:

```python
int(note.archived)
```

This is a small but important mapping.

Databases and Python objects do not always represent values the same way.

Repository code owns that translation.

---

# Database Connections

SQLite connections should be managed carefully.

For a small FastAPI app, one simple pattern is:

```python
import sqlite3
from pathlib import Path


def connect(path: Path) -> sqlite3.Connection:
    conn = sqlite3.connect(path)
    conn.row_factory = sqlite3.Row
    return conn
```

`row_factory = sqlite3.Row` lets rows behave like mappings:

```python
row["title"]
```

This is clearer than relying on column positions.

Do not scatter `sqlite3.connect()` everywhere.

Centralize connection creation so tests can use temporary databases.

---

# Repository Layer

The repository should contain SQL.

Example methods:

```python
class NoteRepository:
    def create(self, title: str, body: str, now: str) -> Note:
        ...

    def list(self, archived: bool | None = None, search: str | None = None) -> list[Note]:
        ...

    def get(self, note_id: int) -> Note | None:
        ...

    def update(self, note_id: int, title: str | None, body: str | None, now: str) -> Note | None:
        ...

    def set_archived(self, note_id: int, archived: bool, now: str) -> Note | None:
        ...

    def delete(self, note_id: int) -> bool:
        ...
```

The repository returns domain objects.

It should not return raw SQLite rows to the service layer.

That keeps database details contained.

---

# Service Layer

The service layer implements application rules.

Rules:

```text
title cannot be empty
update must include at least one field
missing note raises NoteNotFound
archive sets archived to true
restore sets archived to false
delete missing note raises NoteNotFound
```

Example:

```python
class NoteService:
    def __init__(self, repository: NoteRepository, clock: Clock = utc_now):
        self.repository = repository
        self.clock = clock

    def create_note(self, title: str, body: str) -> Note:
        title = normalize_title(title)
        now = self.clock()
        return self.repository.create(title=title, body=body, now=now)
```

The service layer should be easy to test with a temporary repository.

Or with a fake repository for pure unit tests.

---

# Search and Filtering

The list endpoint should support query parameters:

```text
GET /notes
GET /notes?archived=false
GET /notes?archived=true
GET /notes?q=python
GET /notes?archived=false&q=python
```

Search can be simple:

```sql
WHERE title LIKE ? OR body LIKE ?
```

This is not a full search engine.

That is fine.

The capstone is teaching API shape, not search infrastructure.

Use parameters:

```python
pattern = f"%{query}%"
cursor.execute(
    """
    SELECT * FROM notes
    WHERE title LIKE ? OR body LIKE ?
    """,
    (pattern, pattern),
)
```

Do not build SQL by string concatenation with user input.

This capstone should reinforce injection safety.

---

# API Routes

Routes should be clear:

```python
@router.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

Create:

```python
@router.post("/notes", response_model=NoteResponse, status_code=201)
def create_note(payload: CreateNoteRequest, service: NoteService = Depends(get_service)):
    note = service.create_note(payload.title, payload.body)
    return NoteResponse.from_domain(note)
```

Get:

```python
@router.get("/notes/{note_id}", response_model=NoteResponse)
def get_note(note_id: int, service: NoteService = Depends(get_service)):
    return NoteResponse.from_domain(service.get_note(note_id))
```

The route should read like a translation layer:

```text
HTTP input -> service call -> HTTP output
```

That is the right level of responsibility.

---

# Dependency Injection

FastAPI's `Depends` can provide the service.

Example:

```python
def get_repository() -> NoteRepository:
    conn = connect(settings.database_path)
    return NoteRepository(conn)


def get_service(
    repository: NoteRepository = Depends(get_repository),
) -> NoteService:
    return NoteService(repository)
```

For tests, FastAPI allows dependency overrides.

This is useful.

Test code can use a temporary database instead of the real local database.

The principle is the same as earlier capstones:

```text
do not hard-code production paths into testable logic
```

---

# Application Factory

Use an application factory:

```python
from fastapi import FastAPI


def create_app() -> FastAPI:
    app = FastAPI(title="Notes API")
    app.include_router(router)
    return app


app = create_app()
```

Why?

Because tests can create a fresh app.

Future configuration can be injected.

The app is not a mysterious global object that everything mutates.

Application factories are common in web projects for a reason.

---

# Running the API

During development:

```bash
uv run fastapi dev src/notes_api/main.py
```

or:

```bash
uv run uvicorn notes_api.main:app --reload
```

Then open:

```text
http://localhost:8000/docs
```

FastAPI provides interactive API documentation.

This is one reason it is excellent for learning.

But documentation generated from code is only useful if the code uses clear schemas, status codes, and route definitions.

Good API code produces good API docs.

---

# Testing Strategy

Test three layers:

```text
repository tests
service tests
API tests
```

Repository tests use a temporary SQLite database.

Service tests verify application rules.

API tests use FastAPI's test client.

Do not test only through HTTP.

That makes every test slower and more indirect.

Do not test only service functions either.

That misses request validation, routing, status codes, and JSON responses.

Use both.

---

# Repository Tests

Repository tests should verify persistence behavior:

```text
create returns a note with an ID
get returns created note
list returns notes in expected order
update changes title and body
archive changes archived flag
delete removes note
search filters notes
```

Use a temporary database:

```python
def test_create_and_get_note(tmp_path):
    db_path = tmp_path / "test.db"
    conn = connect(db_path)
    initialize_schema(conn)
    repo = NoteRepository(conn)

    note = repo.create("Title", "Body", now="2026-06-15T10:00:00+00:00")

    assert repo.get(note.id) == note
```

This test is fast and isolated.

It does not require a running server.

---

# Service Tests

Service tests should verify rules:

```text
empty title fails
missing note raises NoteNotFound
empty update raises InvalidNoteUpdate
archive missing note raises NoteNotFound
delete missing note raises NoteNotFound
```

The service layer is where application behavior becomes explicit.

If the API route and repository are the only tested layers, business rules may hide in accidental places.

Service tests keep rules visible.

---

# API Tests

API tests should verify HTTP behavior:

```text
POST /notes returns 201
invalid POST returns 422
GET /notes returns list
GET /notes/{id} returns 200
missing note returns 404
PATCH /notes/{id} returns updated note
DELETE /notes/{id} returns 204
```

Example:

```python
def test_create_note(client):
    response = client.post(
        "/notes",
        json={"title": "Read", "body": "API chapter"},
    )

    assert response.status_code == 201
    assert response.json()["title"] == "Read"
```

API tests are contract tests.

They tell you whether clients can use the system.

---

# Pagination

Real list endpoints often need pagination.

For this capstone, include simple pagination:

```text
GET /notes?limit=20&offset=0
```

Rules:

```text
limit defaults to 20
limit cannot exceed 100
offset defaults to 0
offset cannot be negative
```

Pagination response:

```json
{
  "items": [
    {
      "id": 1,
      "title": "Read",
      "body": "API chapter",
      "archived": false,
      "created_at": "2026-06-15T10:00:00+00:00",
      "updated_at": "2026-06-15T10:00:00+00:00"
    }
  ],
  "limit": 20,
  "offset": 0
}
```

This teaches an important API lesson:

```text
list endpoints should not grow without bounds
```

Even small APIs should learn the habit.

---

# Idempotency

Some API operations should be idempotent.

Archiving a note can be idempotent:

```text
archive an already archived note -> still archived
```

Restoring an open note can be idempotent:

```text
restore an already open note -> still open
```

Delete behavior can be a design choice.

For this project:

```text
DELETE missing note returns 404
```

That is acceptable.

The important thing is to decide and document.

APIs should be predictable.

---

# Validation

Validation should happen at multiple boundaries.

Pydantic validates:

* required fields
* string length
* field types
* query parameter bounds

The service validates:

* title normalization
* empty updates
* note existence
* domain rules

The repository protects SQL boundaries:

* parameterized queries
* explicit column mappings
* database constraints

Validation is layered.

No single layer should carry all responsibility.

---

# Security Notes

This capstone does not include authentication.

But security still matters.

Practice these habits:

* use parameterized SQL
* do not expose tracebacks in normal API responses
* validate request bodies
* limit pagination
* avoid logging sensitive request bodies
* keep database path configurable
* do not run development server as a production deployment

The API is small, but the habits are real.

---

# Packaging

The project should include dependencies:

```toml
[project]
name = "notes-api"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
  "fastapi",
  "uvicorn",
]

[dependency-groups]
dev = [
  "pytest",
  "httpx",
]
```

The exact dependency syntax may vary by tool.

The lesson is:

```text
runtime dependencies and development dependencies are different
```

FastAPI and Uvicorn are runtime dependencies.

Pytest and HTTPX are test/development dependencies.

---

# Milestone 1 - Project Skeleton

Build:

* `src/notes_api`
* `tests`
* `pyproject.toml`
* `main.py`
* health endpoint

Completion criteria:

```text
GET /health returns {"status": "ok"}.
```

This milestone proves the app can run.

---

# Milestone 2 - Schemas and Domain Model

Build:

* `Note`
* `CreateNoteRequest`
* `UpdateNoteRequest`
* `NoteResponse`
* response conversion helpers

Completion criteria:

```text
The API has clear request and response shapes.
```

This milestone teaches contracts.

---

# Milestone 3 - Database and Repository

Build:

* SQLite connection helper
* schema initialization
* repository create/get/list/update/delete methods
* repository tests

Completion criteria:

```text
Notes can be persisted and retrieved without HTTP.
```

This milestone teaches persistence.

---

# Milestone 4 - Service Layer

Build:

* note creation
* note retrieval
* note listing
* note update
* archive and restore
* deletion
* application errors

Completion criteria:

```text
Application behavior is tested independently from FastAPI.
```

This milestone teaches business logic.

---

# Milestone 5 - API Routes

Build:

* create route
* list route
* get route
* patch route
* delete route
* archive route
* restore route
* error translation

Completion criteria:

```text
The HTTP API exposes the service behavior correctly.
```

This milestone teaches route design.

---

# Milestone 6 - API Tests

Build tests for:

* success responses
* validation errors
* not found errors
* pagination
* filtering
* search
* archive and restore

Completion criteria:

```text
The API contract is protected by tests.
```

This milestone teaches regression safety.

---

# Common Mistakes

The first common mistake is putting SQL directly in route functions.

That makes the API hard to test and change.

The second common mistake is returning raw database rows.

That leaks internal representation.

The third common mistake is returning status `200` for errors.

Use HTTP properly.

The fourth common mistake is testing only the happy path.

APIs need tests for invalid input and missing resources.

The fifth common mistake is building SQL with string concatenation.

Use parameters.

The sixth common mistake is allowing unbounded list endpoints.

Use pagination.

The seventh common mistake is letting FastAPI models become the entire domain model.

Schemas are contracts.

Domain models are application concepts.

They may overlap, but they are not the same responsibility.

---

# Extension Ideas

After the core works, add:

* authentication
* user-owned notes
* tags
* full-text search
* soft delete
* database migrations
* Dockerfile
* structured logging
* request IDs
* rate limiting
* OpenAPI examples
* frontend client
* PostgreSQL support

Add extensions only after the core contract is tested.

---

# What This Capstone Teaches

This capstone teaches how Python becomes an HTTP service.

It connects:

```text
HTTP
routes
request validation
response models
status codes
SQLite
repositories
service layers
application errors
dependency injection
testing
API contracts
```

The central lesson is:

```text
an API is a contract between systems
```

Good APIs make that contract clear.

They validate input.

They return meaningful status codes.

They hide internal details.

They are tested as contracts.

---

# Completion Checklist

The capstone is complete when:

```text
The app has a health endpoint.
The app has create/list/get/update/delete note endpoints.
The app has archive and restore endpoints.
Request schemas validate input.
Response schemas control output.
SQLite persistence works.
SQL uses parameters.
Missing notes return 404.
Invalid payloads return 422.
Invalid application updates return 400.
List endpoint supports pagination.
List endpoint supports filtering.
List endpoint supports search.
Routes are thin.
Service logic is tested.
Repository logic is tested.
API behavior is tested.
The app can run locally.
The README explains curl examples.
```

If all of these are true, the reader has built a real REST API foundation.

---

# Exercises

1. Create the project structure.

2. Add a FastAPI app factory.

3. Add `GET /health`.

4. Define the `Note` domain model.

5. Define create, update, and response schemas.

6. Create the SQLite schema.

7. Implement the repository create method.

8. Implement repository get/list/update/delete methods.

9. Write repository tests with a temporary database.

10. Implement the service layer.

11. Add application-specific errors.

12. Write service tests for missing notes and invalid updates.

13. Add API routes.

14. Translate application errors into HTTP responses.

15. Add pagination to the list endpoint.

16. Add archived filtering.

17. Add search.

18. Write API tests using FastAPI's test client.

19. Run the app locally and test with `curl`.

20. Document the API in the README.

---

# Preview of Capstone 04

Capstone 03 built a REST API.

It connected HTTP routes, request validation, response schemas, status codes, SQLite persistence, repository design, service logic, dependency injection, and API tests.

Capstone 04 will build a URL Shortener.

The URL Shortener will deepen API design with redirects, unique code generation, collision handling, analytics counters, validation, persistence, and deployment-oriented thinking.

The transition is:

```text
REST API teaches general resource design
URL Shortener turns one API idea into a product-shaped service
```

The next project will show how a small web service becomes a coherent product with behavior, constraints, and edge cases.
