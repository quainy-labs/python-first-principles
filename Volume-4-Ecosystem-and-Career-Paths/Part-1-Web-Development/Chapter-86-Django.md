# Chapter 86 — Django

Django is a full-stack Python web framework.

Flask gives you a small core and asks you to choose much of the surrounding structure.

Django gives you a larger integrated system.

It includes tools for:

* URL routing
* views
* templates
* models
* ORM queries
* migrations
* forms
* validation
* authentication
* authorization
* sessions
* messages
* admin interface
* static files
* testing
* security features
* deployment settings

Django's philosophy is often described as "batteries included."

That means common web application needs are built into the framework or strongly supported by it.

This is powerful.

It is also opinionated.

Django works best when you understand its conventions and let them help you.

If Flask is a small toolkit, Django is a workshop with many tools already mounted on the wall.

You can build quickly.

But you should know what each tool is for.

---

# Why Django Matters

Django matters because many real web applications need the same things:

* users
* authentication
* database models
* forms
* admin screens
* migrations
* templates
* security protections
* tests
* deployment settings

Writing all of that from scratch for every project is wasteful and risky.

Django provides mature defaults and conventions.

This makes it excellent for:

* database-backed web applications
* internal tools
* admin-heavy systems
* content management
* SaaS dashboards
* business applications
* teams that benefit from shared conventions

Django's strength is not minimalism.

Django's strength is integration.

It gives the application a shape.

That shape can make teams faster.

---

# Django's Architectural Style

Django is often described as using MTV:

```text
Model
Template
View
```

This is related to MVC, but the names differ.

In Django:

The model represents data and database structure.

The template renders presentation.

The view handles a request and returns a response.

URL configuration maps paths to views.

The flow is:

```text
request
    -> URLconf
    -> view
    -> model/query/service
    -> template or response
```

The view is not the visual template.

The view is the request-handling function or class.

This naming can confuse people coming from other frameworks.

Learn Django's words on Django's terms.

---

# Projects and Apps

Django separates projects and apps.

A project is the whole website or service.

An app is a Python package inside the project that provides a focused piece of functionality.

Example:

```text
mysite/
    manage.py
    mysite/
        settings.py
        urls.py
        asgi.py
        wsgi.py
    polls/
        models.py
        views.py
        urls.py
        admin.py
        apps.py
        tests.py
        migrations/
```

The outer `mysite` directory is the project container.

The inner `mysite` package contains project configuration.

`polls` is an app.

Apps are reusable units.

A project can contain many apps:

* users
* orders
* billing
* inventory
* reports

Good Django architecture often starts with good app boundaries.

---

# manage.py

`manage.py` is Django's command-line utility for a project.

Common commands:

```bash
python manage.py runserver
python manage.py startapp polls
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py test
python manage.py shell
```

`manage.py` sets up the Django settings module and delegates to Django's command system.

It is not where business logic belongs.

It is the project's command entry point.

Custom management commands can be added for project-specific operations.

Examples:

* import users
* send scheduled reports
* rebuild search index
* clean old sessions

Management commands are part of Django's operational surface.

---

# Settings

Django settings configure the project.

Settings include:

* installed apps
* database configuration
* middleware
* templates
* static files
* secret key
* allowed hosts
* time zone
* authentication settings
* logging
* security settings

Example:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "polls",
]
```

Settings are loaded when Django starts.

Do not casually read environment variables throughout application code.

Centralize configuration.

Production settings must not expose secrets or enable unsafe development behavior.

---

# URLconf

URLconf maps URLs to views.

Project `urls.py`:

```python
from django.urls import include, path


urlpatterns = [
    path("admin/", admin.site.urls),
    path("polls/", include("polls.urls")),
]
```

App `polls/urls.py`:

```python
from django.urls import path

from . import views


urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
]
```

URL names matter:

```python
name="detail"
```

They allow reverse URL building.

Avoid hard-coding URLs in templates and views when named routes can be used.

URL design is API design.

---

# Views

A Django view receives a request and returns a response.

Function view:

```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello from Django")
```

Template view:

```python
from django.shortcuts import render


def index(request):
    return render(request, "polls/index.html", {"title": "Polls"})
```

Views should translate HTTP into application behavior.

They should not become enormous business logic containers.

If a view handles validation, database queries, business rules, email sending, and response formatting all at once, it is doing too much.

Django makes it easy to put logic in views.

Good design still asks where logic belongs.

---

# Request and Response Objects

Django passes an `HttpRequest` object to each view.

It contains:

* method
* path
* GET parameters
* POST data
* files
* cookies
* session
* user
* headers

Example:

```python
def search(request):
    query = request.GET.get("q", "")
    ...
```

A view returns an `HttpResponse` or a subclass.

Common response helpers include:

```python
from django.http import JsonResponse
from django.shortcuts import redirect, render
```

Example:

```python
return JsonResponse({"status": "ok"})
```

Django's request and response objects are the web boundary.

Do not let them leak into deep domain code unless there is a strong reason.

---

# Templates

Django templates render presentation.

Example:

```html
<h1>{{ question.question_text }}</h1>

{% for choice in question.choice_set.all %}
  <p>{{ choice.choice_text }}</p>
{% endfor %}
```

Django templates are intentionally not full Python.

They provide a designer-friendly language for presentation logic.

Templates should not contain complex business decisions.

They should display data prepared by views or context builders.

Django templates escape output by default in HTML contexts.

This helps protect against cross-site scripting.

Do not mark content safe unless you truly trust and understand it.

---

# Models

Django models define data structure and database mapping.

Example:

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Each model is a Python class.

Each field usually maps to a database column.

Django uses model metadata to create database queries, forms, admin screens, validation behavior, and migrations.

Models are central to Django.

Understanding models is understanding much of Django.

---

# Field Options

Django model fields include both database and validation concepts.

Example:

```python
email = models.EmailField(unique=True)
name = models.CharField(max_length=100, blank=False)
bio = models.TextField(blank=True)
```

Important distinction:

```python
null=True
```

means the database may store `NULL`.

```python
blank=True
```

means validation may allow an empty value.

They are not the same.

This distinction matters in forms, admin, validation, and database design.

Model fields are not just storage declarations.

They influence multiple layers of Django.

---

# The ORM

Django's ORM lets you query the database with Python objects.

Create:

```python
Question.objects.create(question_text="Best Python framework?")
```

Read:

```python
question = Question.objects.get(pk=1)
```

Filter:

```python
recent = Question.objects.filter(pub_date__year=2026)
```

Update:

```python
question.question_text = "Updated?"
question.save()
```

Delete:

```python
question.delete()
```

The ORM is powerful because it connects Python classes to relational data.

It can also hide expensive queries if used carelessly.

Query behavior matters.

Profiling and query inspection matter.

---

# QuerySets

A QuerySet represents a database query.

QuerySets are lazy.

Example:

```python
questions = Question.objects.filter(pub_date__year=2026)
```

This does not necessarily hit the database immediately.

Evaluation happens when results are needed:

```python
list(questions)
```

or:

```python
for question in questions:
    ...
```

Laziness is useful.

It allows query composition.

It can also surprise beginners.

If you accidentally evaluate a QuerySet inside a loop, you may cause many database queries.

Understand when queries execute.

---

# Relationships

Django models support relationships:

* `ForeignKey`
* `OneToOneField`
* `ManyToManyField`

Example:

```python
class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.CASCADE)
```

`on_delete` defines what happens when the related object is deleted.

This is not a minor detail.

It affects data integrity.

Common choices include:

* `CASCADE`
* `PROTECT`
* `SET_NULL`
* `RESTRICT`

Pick relationship behavior based on domain rules.

Do not use `CASCADE` automatically.

Deleting data is architecture.

---

# Migrations

Migrations track database schema changes.

After changing models:

```bash
python manage.py makemigrations
```

Apply migrations:

```bash
python manage.py migrate
```

Migrations are code.

They should be reviewed.

They should be committed.

They should be tested.

They represent how the database changes over time.

Bad migrations can break production data.

Schema design is not just model syntax.

It is operational design.

---

# The Admin

Django's admin is one of its most famous features.

Register a model:

```python
from django.contrib import admin

from .models import Question


admin.site.register(Question)
```

Now authorized staff can manage records through an admin interface.

The admin is excellent for:

* internal tools
* content management
* support workflows
* data inspection
* early product operations

But admin access is powerful.

Use permissions carefully.

Do not expose admin publicly without proper security.

Customize admin screens for safe workflows.

---

# Forms

Django forms handle input validation and rendering.

Example:

```python
from django import forms


class ContactForm(forms.Form):
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
```

Use:

```python
form = ContactForm(request.POST)
if form.is_valid():
    email = form.cleaned_data["email"]
```

Forms separate raw input from validated data.

This is a major security and correctness boundary.

Never trust `request.POST` directly as business input.

Validate first.

Use cleaned data.

---

# ModelForms

ModelForms generate forms from models.

Example:

```python
class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ["question_text", "pub_date"]
```

ModelForms are productive because model fields already contain validation and type information.

But be careful with:

```python
fields = "__all__"
```

This can expose fields that should not be user-editable.

Prefer explicit fields.

Input APIs should be intentional.

Forms are user-facing contracts.

---

# Authentication

Django includes an authentication system.

It provides:

* users
* groups
* permissions
* password hashing
* login and logout views
* authentication middleware
* decorators and mixins

In views:

```python
request.user
```

represents the current user.

Check authentication:

```python
if request.user.is_authenticated:
    ...
```

Django's built-in auth is a strong default for many applications.

Custom user models should be decided early.

Changing user models later can be painful.

Authentication is foundational architecture.

---

# Authorization

Authentication asks:

```text
who are you?
```

Authorization asks:

```text
what are you allowed to do?
```

Django has permissions and groups.

You can also implement domain-specific authorization.

Example:

```python
def can_view_invoice(user, invoice):
    return invoice.organization_id == user.organization_id
```

Do not rely only on templates hiding buttons.

Views and services must enforce permissions.

Authorization belongs on the server.

Security must be tested.

---

# Middleware

Middleware wraps request and response processing.

It can handle:

* security headers
* sessions
* authentication
* CSRF protection
* messages
* clickjacking protection
* custom request logging
* request IDs

Settings:

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
]
```

Middleware is powerful because it applies across requests.

Do not put ordinary business workflow in middleware.

Use it for cross-cutting request concerns.

---

# CSRF Protection

Django includes CSRF protection for forms and unsafe HTTP methods.

CSRF means Cross-Site Request Forgery.

It is an attack where a malicious site tricks a user's browser into making a request to another site where the user is authenticated.

Django's CSRF middleware and template tags help protect form submissions.

In templates:

```html
<form method="post">
  {% csrf_token %}
  ...
</form>
```

Do not disable CSRF protection casually.

If building APIs, understand how authentication and CSRF interact.

Security defaults are one of Django's strengths.

Respect them.

---

# Class-Based Views

Django supports function-based and class-based views.

Function view:

```python
def question_detail(request, question_id):
    ...
```

Class-based view:

```python
from django.views.generic import DetailView


class QuestionDetailView(DetailView):
    model = Question
    template_name = "polls/detail.html"
```

Class-based views can reduce repetition for common patterns:

* list views
* detail views
* create views
* update views
* delete views

They can also become confusing with inheritance and mixins.

Use them when they clarify.

Use function views when they are simpler.

Django gives both.

---

# Static Files

Django has a static files system.

During development:

```python
STATIC_URL = "static/"
```

Static files may live in app-level `static/` directories or project-level static directories.

For production, static files are collected:

```bash
python manage.py collectstatic
```

Deployment then serves them through a web server, CDN, or static file service.

Static file handling is part of deployment architecture.

Do not assume development behavior is production behavior.

---

# Media Files

Static files are application assets.

Media files are user-uploaded content.

They should be treated differently.

Examples:

* profile photos
* uploaded documents
* generated reports

Media files need:

* storage configuration
* access control
* size limits
* validation
* cleanup policy
* backup strategy

Do not store sensitive uploads in publicly served locations without authorization checks.

File handling is a security boundary.

---

# Testing Django

Django includes testing tools built on Python's testing ecosystem.

Run tests:

```bash
python manage.py test
```

Django creates a test database for tests that need database access.

Example:

```python
from django.test import TestCase


class QuestionTests(TestCase):
    def test_question_text(self):
        question = Question.objects.create(question_text="Hello?")
        self.assertEqual(question.question_text, "Hello?")
```

Django also provides a test client:

```python
response = self.client.get("/polls/")
```

Use tests at different layers:

* model tests
* form tests
* view tests
* service tests
* permission tests
* integration tests

Django makes database-backed tests practical.

Do not let every test become a full-stack test.

---

# Keeping Django Apps Maintainable

Django makes it easy to put code in:

```text
models.py
views.py
forms.py
admin.py
```

That is a starting point.

As apps grow, these files can become huge.

You can split them:

```text
orders/
    models/
        __init__.py
        order.py
        invoice.py
    views/
        __init__.py
        checkout.py
        history.py
    services.py
    selectors.py
    forms.py
    admin.py
```

Some teams use service layers.

Some use managers and QuerySets.

Some keep domain behavior on models.

The key is cohesion.

Put logic where future readers can find it and tests can exercise it.

---

# Fat Models and Service Layers

Django teams often debate:

```text
fat models
service layer
```

Fat models put domain behavior on model classes.

Example:

```python
class Order(models.Model):
    def mark_paid(self, payment_id):
        ...
```

Service layers coordinate workflows:

```python
def checkout(order_id, payment_gateway):
    order = Order.objects.get(pk=order_id)
    payment = payment_gateway.charge(order)
    order.mark_paid(payment.id)
    order.save()
```

Both can be good.

A model method is good for behavior tied to one model's invariants.

A service is good for workflows spanning multiple models or external systems.

Avoid both extremes:

* models with no behavior and huge procedural scripts
* models that know about every external service

Use judgment.

---

# Managers and QuerySets

Django managers and QuerySets can hold query logic.

Example:

```python
class PublishedQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status="published")


class Article(models.Model):
    objects = PublishedQuerySet.as_manager()
```

Use:

```python
Article.objects.published()
```

This keeps query concepts near the model.

It avoids scattering filter details everywhere.

Managers and QuerySets are good for reusable query behavior.

They are not the best place for every business workflow.

Keep data retrieval and application orchestration distinct when complexity grows.

---

# Signals

Django signals let parts of the system react to events.

Example:

```python
from django.db.models.signals import post_save
```

Signals can be useful for decoupled reactions.

They can also make behavior invisible.

If saving a model unexpectedly sends email, updates analytics, and modifies another object through signals, debugging becomes harder.

Use signals carefully.

Good signal use:

* simple decoupled framework integration
* cache invalidation
* audit hooks

Risky signal use:

* core business workflows
* hidden side effects
* complex ordering assumptions

Explicit service calls are often clearer.

---

# Django REST APIs

Django itself can return JSON.

Example:

```python
from django.http import JsonResponse


def health(request):
    return JsonResponse({"status": "ok"})
```

For larger APIs, many teams use Django REST Framework.

DRF provides:

* serializers
* viewsets
* routers
* authentication tools
* permissions
* browsable API

Django plus DRF is a common stack for database-backed APIs.

This chapter focuses on Django core.

But it is important to know that Django's ecosystem includes strong API tooling.

---

# Security Defaults

Django includes many security features:

* CSRF protection
* SQL injection protection through ORM parameterization
* XSS protection through template escaping
* clickjacking protection
* password hashing
* secure session handling
* host header validation through allowed hosts
* security middleware

These features help.

They do not remove responsibility.

You can still write insecure Django code.

Examples:

* marking unsafe HTML as safe
* building raw SQL unsafely
* skipping authorization checks
* exposing admin carelessly
* committing secret keys
* disabling CSRF without understanding why

Framework security is a foundation.

Application security still belongs to you.

---

# Deployment Checklist

Production Django needs careful settings.

Important checks include:

* `DEBUG = False`
* strong `SECRET_KEY`
* correct `ALLOWED_HOSTS`
* secure database configuration
* static file serving
* media file serving
* HTTPS
* secure cookies
* logging
* error reporting
* migrations
* environment-specific settings

Django provides deployment guidance because development settings are not production settings.

Never deploy with debug mode enabled.

Debug pages can expose sensitive information.

Production readiness is part of engineering, not an afterthought.

---

# Django vs Flask

Flask is smaller and more flexible.

Django is larger and more integrated.

Choose Flask when:

* you want a small core
* architecture is highly custom
* you are building focused services
* you want to choose most components yourself

Choose Django when:

* database-backed app conventions help
* admin is valuable
* built-in auth and forms help
* ORM and migrations are central
* team speed benefits from shared structure

Neither framework is universally better.

Django gives you more decisions up front.

Flask asks you to make more decisions yourself.

The right choice depends on project shape and team needs.

---

# Django vs FastAPI

Django and FastAPI optimize for different experiences.

Django is excellent for full-stack web applications and database-backed systems.

FastAPI is excellent for type-hint-driven APIs, automatic validation, OpenAPI generation, and async-friendly service development.

Django can build APIs.

FastAPI can be part of larger systems.

But the defaults differ.

Django starts with:

```text
models, views, templates, admin, auth, settings
```

FastAPI starts with:

```text
path operations, type hints, validation, OpenAPI
```

Chapter 87 will study FastAPI next.

---

# Common Django Mistakes

Common mistakes include:

* putting too much logic in views
* making models either too empty or too powerful
* ignoring QuerySet laziness
* causing N+1 queries
* treating migrations casually
* using `fields = "__all__"` in unsafe forms
* disabling CSRF without understanding the risk
* exposing admin without proper controls
* leaving `DEBUG=True` in production
* committing `SECRET_KEY`
* spreading settings and environment reads everywhere
* using signals for hidden core workflows
* not testing permissions

Django gives strong tools.

Strong tools can still be misused.

---

# A Django Checklist

When building a Django app, ask:

* Is this logic in the right layer?
* Are views thin enough?
* Are model invariants protected?
* Are QuerySets efficient?
* Are migrations reviewed?
* Are forms explicit about fields?
* Are permissions enforced server-side?
* Are admin screens safe for staff workflows?
* Are settings separated by environment?
* Is `DEBUG` off in production?
* Are static and media files handled correctly?
* Are tests covering models, views, forms, and permissions?
* Are signals used only where they clarify?

This checklist keeps Django's convenience from becoming accidental complexity.

---

# Chapter Summary

Django is a full-stack Python web framework with strong conventions and many built-in tools.

It is especially strong for database-backed web applications.

Django uses a Model-Template-View style.

Projects contain configuration for a whole site or service.

Apps provide focused units of functionality.

`manage.py` runs project commands.

Settings configure installed apps, middleware, databases, templates, static files, security, and more.

URLconf maps paths to views.

Views receive requests and return responses.

Templates render presentation.

Models define data structure and database mapping.

The ORM provides a Python API for querying and changing relational data.

QuerySets are lazy and must be understood to avoid performance surprises.

Migrations track database schema changes and must be treated as production code.

The admin interface is powerful for internal data management.

Forms and ModelForms validate and process user input.

Django includes authentication, permissions, sessions, middleware, CSRF protection, static file tooling, testing tools, and deployment guidance.

Django's strength is integration.

Its risk is assuming the framework's structure replaces architectural judgment.

The central lesson is:

```text
Django gives you conventions that make common web applications fast to build
```

Use those conventions with understanding.

---

# Exercises

1. Create a Django project and one app.

2. Add a URL route and function-based view that returns a simple response.

3. Create a model with two fields and generate a migration.

4. Apply migrations and create an object through the Django shell.

5. Register the model in the admin.

6. Write a template-backed view that renders a list of objects.

7. Create a form that validates user input.

8. Write a test for a model method and a view response.

9. Identify one QuerySet that could cause an N+1 query problem and explain how to investigate it.

10. Review a production settings file using Django's deployment concerns.

---

# Preview of Chapter 87

Chapter 86 studied Django.

We saw how Django provides an integrated framework for database-backed web applications through projects, apps, settings, URLconf, views, templates, models, ORM queries, migrations, forms, admin, authentication, middleware, testing, and deployment practices.

Next we study FastAPI.

FastAPI takes a different approach.

It is built for APIs and leans heavily on:

* type hints
* request validation
* response models
* OpenAPI generation
* dependency injection
* async support
* modern Python syntax

The transition is:

```text
Django provides integrated conventions for full-stack web apps
FastAPI provides type-driven tools for building APIs
```

Chapter 87 will show how FastAPI connects Python's type system with web API design.
