# Chapter 76 — Packaging

Packaging is how Python code becomes installable software.

You can write useful Python code in a single file.

You can organize it into modules.

You can organize modules into packages.

You can test it.

You can debug it.

You can log from it.

But professional Python code often needs one more step:

```text
make it installable, versioned, and distributable
```

That is packaging.

Packaging answers questions such as:

* what is this project called?
* what Python versions does it support?
* what dependencies does it need?
* what files belong in the distribution?
* how is it built?
* how is it installed?
* how does a command-line script get created?
* how does another project depend on it?
* how does a user know which version they have?
* how is it published?

Packaging is not just uploading code to PyPI.

Packaging is the boundary between source code and usable software.

---

# Why Packaging Matters

Without packaging, using code often means copying files around.

That does not scale.

Copying files creates problems:

* no clear version
* no dependency metadata
* no install process
* no uninstall process
* no command-line entry points
* no clear ownership
* no compatibility information
* no build artifact
* no reliable reuse

Packaging makes code portable.

It lets someone write:

```bash
python -m pip install your-package
```

and then:

```python
import your_package
```

or:

```bash
your-command
```

That is a major transformation.

The code is no longer just files in your editor.

It is a project with metadata, dependencies, and an installation contract.

---

# Packaging Vocabulary

Python packaging has vocabulary that can confuse even experienced programmers.

The most important distinction is:

```text
import package
distribution package
installed project
```

These are related.

They are not the same thing.

An import package is what Python imports:

```python
import requests
```

A distribution package is the installable thing managed by packaging tools:

```bash
python -m pip install requests
```

An installed project is the result of installing a distribution into an environment.

Usually the names are similar.

Sometimes they differ.

For example, a distribution may have a hyphenated name while the import package uses underscores.

Packaging becomes less mysterious when you keep these layers separate.

---

# Module, Package, Project, Distribution

A module is usually one `.py` file:

```text
math_tools.py
```

An import package is usually a directory containing Python modules:

```text
math_tools/
    __init__.py
    arithmetic.py
    geometry.py
```

A project is the whole repository or source tree:

```text
math-tools/
    pyproject.toml
    README.md
    src/
        math_tools/
            __init__.py
            arithmetic.py
            geometry.py
    tests/
```

A distribution archive is a build artifact:

```text
dist/
    math_tools-1.0.0.tar.gz
    math_tools-1.0.0-py3-none-any.whl
```

The project produces distribution archives.

The distribution archives are installed.

The installed files provide import packages and possibly command-line scripts.

The flow is:

```text
source project -> distribution archive -> installed project -> importable package
```

---

# Import Name vs Distribution Name

The name you install is not always exactly the name you import.

Install:

```bash
python -m pip install beautifulsoup4
```

Import:

```python
from bs4 import BeautifulSoup
```

Install:

```bash
python -m pip install PyYAML
```

Import:

```python
import yaml
```

This is not a bug.

The distribution name and import package name are different concepts.

When documenting your project, be clear:

```text
Install with: python -m pip install example-package
Import with: import example_package
```

Confusing these names is one of the most common packaging beginner problems.

---

# Project Layout

A common modern layout is the `src` layout:

```text
project-name/
    pyproject.toml
    README.md
    LICENSE
    src/
        project_name/
            __init__.py
            core.py
    tests/
        test_core.py
```

The package code lives under `src/`.

This layout helps prevent accidental imports from the source tree.

Without `src`, a project may look like:

```text
project-name/
    pyproject.toml
    project_name/
        __init__.py
        core.py
    tests/
        test_core.py
```

This is called a flat layout.

It can work.

But `src` layout often catches packaging mistakes earlier because tests must import the installed or properly configured package rather than accidentally importing local files from the project root.

For serious reusable packages, `src` layout is a strong default.

---

# A Minimal Package

Suppose we want to package a small library:

```text
greetings/
    pyproject.toml
    README.md
    LICENSE
    src/
        greetings/
            __init__.py
            message.py
    tests/
        test_message.py
```

`src/greetings/message.py`:

```python
def hello(name):
    return f"Hello, {name}"
```

`src/greetings/__init__.py`:

```python
from .message import hello


__all__ = ["hello"]
```

Now users can write:

```python
from greetings import hello
```

But the package is not fully defined until we add packaging metadata.

That metadata lives in `pyproject.toml`.

---

# pyproject.toml

`pyproject.toml` is the central configuration file for modern Python packaging.

It can define:

* build system
* project metadata
* dependencies
* optional dependencies
* command-line scripts
* tool configuration

Example:

```toml
[build-system]
requires = ["hatchling >= 1.26"]
build-backend = "hatchling.build"

[project]
name = "greetings"
version = "0.1.0"
description = "Small greeting utilities"
readme = "README.md"
requires-python = ">=3.10"
license = "MIT"
authors = [
  { name = "Narendra Nareda" }
]
dependencies = []
```

The `[build-system]` table tells build tools how to build the project.

The `[project]` table describes the project itself.

Packaging tools read this file to build, install, and publish your project.

---

# Build Frontend and Build Backend

Python packaging separates build frontends from build backends.

A build frontend is the tool you run.

Examples:

* `python -m build`
* `pip`
* packaging workflows in other tools

A build backend performs the actual build according to project configuration.

Examples:

* Hatchling
* setuptools
* Flit
* PDM backend
* uv build backend

In `pyproject.toml`:

```toml
[build-system]
requires = ["hatchling >= 1.26"]
build-backend = "hatchling.build"
```

This says:

```text
to build this project, install Hatchling and call its build backend
```

The frontend coordinates.

The backend builds.

This separation lets different projects use different build systems while tools can still interact through common standards.

---

# Build Isolation

Modern build tools usually build projects in isolated environments.

That means build dependencies are installed separately from your current environment.

This matters.

If your project needs a package at build time, it must be declared in `[build-system].requires`.

For example:

```toml
[build-system]
requires = ["setuptools >= 77.0.3", "wheel"]
build-backend = "setuptools.build_meta"
```

Do not rely on packages that happen to be installed on your machine.

Build isolation protects reproducibility.

It helps answer:

```text
can this project build in a clean environment?
```

Professional packaging assumes clean environments.

---

# Project Metadata

Metadata describes the project.

Common fields include:

```toml
[project]
name = "greetings"
version = "0.1.0"
description = "Small greeting utilities"
readme = "README.md"
requires-python = ">=3.10"
license = "MIT"
authors = [
  { name = "Narendra Nareda" }
]
keywords = ["greeting", "example"]
classifiers = [
  "Programming Language :: Python :: 3",
  "Operating System :: OS Independent",
]
```

Metadata helps:

* installers choose compatible versions
* users understand the package
* package indexes display information
* tools resolve dependencies
* downstream projects evaluate compatibility

Metadata is not decoration.

It is part of the package contract.

---

# Project Name

The project name is the distribution name.

Example:

```toml
name = "greetings"
```

This is the name users install:

```bash
python -m pip install greetings
```

Project names may use letters, numbers, dots, underscores, and hyphens.

Names are normalized by packaging tools.

In practice, avoid clever names that differ too much from the import name.

If the distribution name is:

```text
greetings-tools
```

and the import package is:

```python
import greetings
```

document that clearly.

---

# Version

The version identifies a specific release.

Example:

```toml
version = "0.1.0"
```

Versions matter because users and tools need to compare releases.

Common style:

```text
MAJOR.MINOR.PATCH
```

For example:

```text
1.4.2
```

The exact meaning depends on your project's versioning policy.

Semantic versioning is common:

* major version changes may break compatibility
* minor version adds compatible features
* patch version fixes bugs

But version numbers are promises only if the project honors them.

Choose a policy and document it.

---

# requires-python

`requires-python` declares supported Python versions.

Example:

```toml
requires-python = ">=3.10"
```

This tells installers that the package needs Python 3.10 or newer.

If a user on Python 3.9 tries to install it, tools can avoid incompatible releases.

Do not claim support for Python versions you do not test.

If your CI tests Python 3.10, 3.11, 3.12, and 3.13, your metadata can reflect that support.

Supported Python versions are part of your compatibility contract.

---

# README

The README is often shown on package index pages.

Metadata points to it:

```toml
readme = "README.md"
```

A good package README includes:

* what the project does
* installation command
* quick example
* supported Python versions
* basic configuration
* links to documentation
* license
* contribution or issue information when relevant

For libraries, include import examples.

For command-line tools, include command examples.

The README is often the first user experience of your package.

Treat it as part of the product.

---

# License Files

A package should include license information.

Example:

```toml
license = "MIT"
license-files = ["LICEN[CS]E*"]
```

The license tells users under what terms they can use, modify, and distribute the software.

Do not treat licensing casually.

If the project is internal, your organization may have policy.

If the project is public, choose a license deliberately.

If the project has dependencies, their licenses may also matter.

Packaging makes legal and operational metadata visible.

---

# Dependencies

Runtime dependencies belong in `[project].dependencies`.

Example:

```toml
[project]
dependencies = [
  "requests >= 2.32",
  "pydantic >= 2",
]
```

These are packages required when your project is installed and used.

If your package imports `requests` at runtime, then `requests` is a runtime dependency.

When users install your package, installers can install those dependencies too.

Do not hide runtime dependencies in documentation only.

Declare them.

---

# Dependency Specifiers

Dependency specifiers describe acceptable versions.

Examples:

```toml
dependencies = [
  "requests >= 2.32",
  "Django >= 5.0, < 6.0",
  "typing-extensions >= 4.12; python_version < '3.13'",
]
```

Specifiers can include:

* minimum versions
* upper bounds
* exclusions
* environment markers
* extras

A minimum version says:

```text
we need at least this version
```

An upper bound says:

```text
we are not claiming compatibility beyond this range
```

Use bounds thoughtfully.

Too loose can allow incompatible future versions.

Too tight can cause dependency conflicts.

Packaging is tradeoff management.

---

# Optional Dependencies

Optional dependencies are declared in `[project.optional-dependencies]`.

Example:

```toml
[project.optional-dependencies]
dev = [
  "pytest",
  "ruff",
]
postgres = [
  "psycopg[binary] >= 3.1",
]
```

Users can install extras:

```bash
python -m pip install "greetings[dev]"
python -m pip install "greetings[postgres]"
```

Extras are useful for:

* development tools
* optional database support
* optional cloud provider integrations
* optional performance features
* documentation dependencies
* testing dependencies

Do not put required runtime dependencies in extras.

If the package cannot work without it, it belongs in `dependencies`.

---

# Dependencies vs Requirements Files

`dependencies` in `pyproject.toml` and `requirements.txt` serve different purposes.

Package dependencies describe what your project needs:

```toml
dependencies = [
  "requests >= 2.32",
]
```

A requirements file often describes an environment:

```text
requests==2.32.4
urllib3==2.5.0
certifi==2026.1.1
```

Libraries usually specify ranges because they need to work with many environments.

Applications often pin exact versions because they deploy one environment.

The distinction is:

```text
library metadata declares compatibility
application lock files declare a deployment
```

Do not confuse them.

---

# Lock Files

A lock file records exact resolved dependency versions.

Lock files help reproduce environments.

They are especially important for applications.

A library may not commit a universal lock file for its users because the library is installed into many different dependency graphs.

An application usually should have a reproducible dependency set.

Different tools use different lock formats.

The principle is stable:

```text
dependency ranges describe what can work
locks record what will be used
```

When debugging production dependency issues, the lock file is evidence.

---

# Modern Project Managers and `uv`

Modern Python development often uses project management tools on top of the packaging standards.

The standards define concepts such as:

* project metadata
* dependencies
* build systems
* wheels
* source distributions
* virtual environments

Tools make those concepts convenient in daily work.

Common tools include:

* `pip`
* `venv`
* `pip-tools`
* `pipx`
* Poetry
* Hatch
* PDM
* `uv`

The tool landscape changes over time, but the underlying questions stay stable:

```text
how do I create the project?
how do I create the environment?
how do I add dependencies?
how do I lock dependencies?
how do I run commands?
how do I build distributions?
how do I publish artifacts?
```

`uv` is important in modern Python because it combines several workflows that were historically handled by separate tools.

It can manage projects, create virtual environments, resolve and lock dependencies, run commands, manage Python versions, run one-off tools, and provide a `pip`-compatible interface.

That makes it useful for both learning and professional projects.

For example, a small project workflow might be:

```bash
uv init greetings
cd greetings
uv add requests
uv run python -c "import requests; print(requests.__version__)"
uv lock
uv sync
```

The important ideas are:

```text
uv init creates project structure
uv add records a dependency
uv run runs inside the project environment
uv lock records the resolved dependency set
uv sync makes the environment match the lock file
```

This should feel familiar after learning virtual environments, dependencies, and lock files.

`uv` is not a separate universe.

It is a modern tool that automates familiar packaging and environment concepts.

There is also a `uv pip` interface.

It can be used in workflows that still look like `pip`, `pip-tools`, or `virtualenv`:

```bash
uv venv
uv pip install requests
uv pip freeze
```

That interface helps teams migrate gradually.

A team does not need to rewrite every workflow on day one.

The professional lesson is:

```text
learn the packaging model first
then choose tools that make the model easier to use
```

Tools change.

The model remains.

If you understand environments, dependency resolution, lock files, project metadata, wheels, and build systems, then `pip`, Poetry, Hatch, PDM, and `uv` become different interfaces to understandable ideas.

---

# Source Distribution

A source distribution, or sdist, is an archive of source files and metadata.

It often has a name like:

```text
greetings-0.1.0.tar.gz
```

An sdist lets users or tools build the package from source.

It should contain enough files to build the project.

That may include:

* Python source
* `pyproject.toml`
* README
* license
* package data
* generated files required for build

If your sdist is incomplete, installation from source may fail.

Always test your built distributions, not only your source tree.

---

# Wheel

A wheel is a built distribution.

It often has a name like:

```text
greetings-0.1.0-py3-none-any.whl
```

Wheels install faster than source distributions because they are already built.

Pure Python projects often have universal wheels.

Projects with compiled extensions may need different wheels for different:

* operating systems
* CPU architectures
* Python versions
* ABIs

For example:

```text
package-1.0.0-cp312-cp312-manylinux_x86_64.whl
```

Wheel tags communicate compatibility.

Installers use those tags to choose the right file.

---

# Building a Package

The standard build command with the `build` frontend is:

```bash
python -m build
```

Run it from the directory containing `pyproject.toml`.

It usually creates:

```text
dist/
    greetings-0.1.0.tar.gz
    greetings-0.1.0-py3-none-any.whl
```

Before building, you may need:

```bash
python -m pip install --upgrade build
```

The build command should work in a clean environment.

If it only works because your local machine has hidden packages installed, the packaging metadata is incomplete.

---

# Installing Locally

To install a local project:

```bash
python -m pip install .
```

This builds and installs the project into the current environment.

To install from a wheel:

```bash
python -m pip install dist/greetings-0.1.0-py3-none-any.whl
```

Testing installation matters.

Running code from the source tree is not the same as running installed code.

Packaging bugs often appear only after install:

* missing files
* incorrect entry points
* wrong package discovery
* missing dependencies
* metadata mistakes

Always test the installed artifact before publishing.

---

# Editable Installs

An editable install lets you install a project while still editing the source.

Command:

```bash
python -m pip install -e .
```

This is useful during development.

Changes to source files are reflected without reinstalling the package each time.

Editable installs are convenient, but they can hide packaging problems.

Before release, test a normal install from the built wheel.

Development convenience is not the same as distribution correctness.

---

# Package Data

Packages sometimes need non-Python files.

Examples:

* templates
* configuration schemas
* static assets
* type marker files
* data files
* default resources

Packaging must include them intentionally.

A common bug:

```text
tests pass locally because data files exist in the source tree
installed package fails because data files were not included
```

Use your build backend's configuration to include package data.

At runtime, use `importlib.resources` for package resources rather than assuming filesystem paths.

Installed packages may live in locations you do not expect.

Some may even be loaded from zip-like contexts.

Resource access should go through packaging-aware APIs.

---

# Command-Line Entry Points

Packages can install command-line scripts.

In `pyproject.toml`:

```toml
[project.scripts]
greet = "greetings.cli:main"
```

This creates a command:

```bash
greet
```

that calls:

```python
greetings.cli.main
```

Example `src/greetings/cli.py`:

```python
def main():
    print("Hello")
```

Better CLI functions return exit codes:

```python
import argparse


def main(argv=None):
    parser = argparse.ArgumentParser()
    parser.add_argument("name")
    args = parser.parse_args(argv)
    print(f"Hello, {args.name}")
    return 0
```

Entry points connect installed packages to executable commands.

They are one of the main reasons packaging is more powerful than copying files.

---

# The __main__ Pattern

A package can also support:

```bash
python -m greetings
```

by defining:

```text
src/
    greetings/
        __main__.py
```

Example:

```python
from .cli import main


raise SystemExit(main())
```

The `__main__.py` file runs when the package is executed as a module.

This is separate from console script entry points.

You can support both:

```bash
greet Narendra
python -m greetings Narendra
```

Both can call the same `main` function.

That keeps behavior consistent.

---

# Entry Point Groups

Entry points are not only for command-line scripts.

They can also support plugin systems.

For example:

```toml
[project.entry-points."myapp.plugins"]
csv = "my_plugin.csv:CSVPlugin"
json = "my_plugin.json:JSONPlugin"
```

Another application can discover these entry points and load plugins.

This is how some Python ecosystems support extensibility.

The general idea:

```text
packages advertise callable objects by name
other code discovers those advertised objects
```

Command-line scripts are the most common beginner-facing use.

Plugin entry points are the advanced form.

---

# Publishing

Publishing usually means uploading distributions to a package index.

The public Python index is PyPI.

TestPyPI exists for testing publishing workflows.

Typical flow:

```bash
python -m build
python -m twine upload dist/*
```

For testing:

```bash
python -m twine upload --repository testpypi dist/*
```

Do not publish experiments to real PyPI casually.

Package names are visible and releases are difficult to erase cleanly from user workflows.

Use TestPyPI for practice.

Use real PyPI when the package is ready.

---

# API Tokens

Modern publishing should use API tokens rather than passwords.

Tokens can be scoped.

For example, a token may be limited to one project.

This is safer than using an account-wide password.

Protect tokens carefully.

Do not commit them.

Do not print them in logs.

Do not paste them into public issue trackers.

Publishing credentials are supply-chain credentials.

Treat them seriously.

---

# Package Indexes

PyPI is the public package index.

Organizations may also use private indexes.

Private indexes are useful for:

* internal packages
* proprietary code
* controlled dependency mirrors
* security review workflows
* offline or regulated environments

Installers can be configured to use different indexes.

Be careful with names.

If an internal package has the same name as a public package, dependency confusion can become a security risk.

Package naming and index configuration are part of supply-chain security.

---

# Versioning Releases

A release should be immutable in practice.

If you publish version `1.2.0`, do not later publish different code under the same version.

Instead, release `1.2.1`.

Version numbers let users and tools reason about change.

Reusing versions breaks that reasoning.

Before releasing, check:

* version updated
* changelog updated if used
* tests pass
* build succeeds
* wheel installs
* metadata looks correct
* README renders correctly
* package data included
* entry points work

Releases are small ceremonies because they create artifacts other people may depend on.

---

# Pre-Releases

Python packaging supports pre-release versions.

Examples:

```text
1.0.0a1
1.0.0b1
1.0.0rc1
```

Meanings:

* alpha
* beta
* release candidate

Pre-releases let users test upcoming changes before a final release.

Installers usually avoid pre-releases unless requested or required.

Use pre-releases when compatibility risk is real and feedback matters.

---

# Local Version Labels

Local version labels can identify local builds:

```text
1.0.0+company.1
```

They are useful in controlled internal workflows.

Do not use local versions casually for public releases.

Versioning has standards because tools need predictable ordering.

Inventing version formats creates pain.

Follow packaging version rules.

---

# Wheels With Native Extensions

Packages with C extensions or other native code have more complex packaging.

They may need:

* compiler toolchains
* platform-specific wheels
* ABI compatibility decisions
* manylinux or musllinux wheels on Linux
* macOS architecture handling
* Windows wheels
* Python-version-specific builds

Pure Python packaging is mostly metadata and files.

Native packaging is also compilation and platform compatibility.

Chapter 71 explained C extensions.

Packaging is where those extensions become installable for users.

If you publish native packages, test on the platforms you claim to support.

---

# Universal Pure Python Wheels

A pure Python wheel may be compatible with many Python versions and platforms.

Example:

```text
greetings-0.1.0-py3-none-any.whl
```

This means:

```text
Python 3
no specific ABI
any platform
```

That is the simplest case.

Many libraries are pure Python and can use this kind of wheel.

If your package includes platform-specific files or compiled extensions, compatibility tags become more specific.

---

# What pip Does

At a high level, `pip install package`:

1. finds candidate distributions
2. checks version and environment compatibility
3. resolves dependencies
4. downloads or uses local files
5. builds from source if needed
6. installs files into the environment
7. records installed metadata

This is simplified.

But it explains why metadata matters.

If dependencies are wrong, resolution is wrong.

If Python version metadata is wrong, incompatible installs happen.

If package data is missing, installed code fails.

Packaging metadata is executable information for installers.

---

# Virtual Environments

Packaging almost always interacts with virtual environments.

A virtual environment gives a project its own installed packages.

Create:

```bash
python -m venv .venv
```

Activate on Unix-like shells:

```bash
source .venv/bin/activate
```

Install:

```bash
python -m pip install .
```

Virtual environments prevent one project's dependencies from colliding with another's.

They are not packaging artifacts themselves.

They are installation environments.

---

# Application Packaging vs Library Packaging

Libraries and applications have different packaging needs.

A library is installed into other projects.

It should:

* declare compatibility ranges
* avoid unnecessary dependency pins
* preserve public APIs
* document supported Python versions
* avoid surprising side effects

An application is deployed as a complete system.

It should:

* pin exact dependency versions
* use a lock file or equivalent
* configure runtime environment
* include deployment metadata
* define entry points or service commands

A library says:

```text
I can work with these versions
```

An application says:

```text
deploy exactly this environment
```

Mixing these mindsets causes problems.

---

# Internal Packages

Not every package is public.

Teams often create internal packages for shared code.

Examples:

* common authentication client
* shared data models
* internal API SDK
* company logging configuration
* reusable CLI tools
* domain-specific utilities

Internal packages need the same discipline:

* clear names
* versions
* changelogs
* dependency metadata
* tests
* release process
* private index or distribution channel

Internal code can break production too.

Do not treat internal packaging as less real.

---

# Namespace Packages

Namespace packages let multiple distributions provide modules under one shared package namespace.

Example:

```text
company.analytics
company.billing
company.auth
```

These may come from different distributions but share the `company` namespace.

Namespace packages are useful for large organizations and plugin-like ecosystems.

They also add complexity.

For most beginner and intermediate projects, regular packages with `__init__.py` are simpler.

Use namespace packages when the project structure truly needs them.

---

# Package Discovery

Build backends need to know which packages to include.

Some infer packages automatically.

Others require configuration.

With `src` layout, discovery often looks under `src/`.

Common mistake:

```text
build succeeds but installed package is empty
```

This usually means package discovery did not include your code.

Always inspect the built wheel or install it into a clean environment and import it.

Packaging correctness is verified by installation, not hope.

---

# Checking Built Artifacts

After building:

```bash
python -m build
```

check:

```text
dist/
```

Then install the wheel in a clean environment:

```bash
python -m pip install dist/greetings-0.1.0-py3-none-any.whl
```

Test:

```bash
python -c "import greetings; print(greetings.hello('Narendra'))"
```

If you provide a CLI:

```bash
greet Narendra
```

This catches real packaging errors.

Do this before publishing.

---

# Inspecting Metadata

Installed packages include metadata.

You can inspect with:

```bash
python -m pip show greetings
```

From Python:

```python
from importlib.metadata import version


print(version("greetings"))
```

Remember:

```python
version("greetings")
```

uses the distribution name, not necessarily the import package name.

`importlib.metadata` can also inspect entry points and metadata.

This is useful for plugin systems and diagnostics.

---

# py.typed

If a package includes inline type hints and wants type checkers to use them, it may include a `py.typed` marker file.

Example:

```text
src/
    greetings/
        __init__.py
        message.py
        py.typed
```

The marker tells type checkers that the package supports typing.

It must be included in the packaged files.

This connects packaging to type hints and static type checking.

Chapters 77 and 78 will continue that thread.

Typing information is useful only if it reaches the installed package.

---

# Common Packaging Mistakes

Common mistakes include:

* confusing distribution name with import name
* forgetting `pyproject.toml`
* missing runtime dependencies
* pinning library dependencies too tightly
* leaving application dependencies too loose
* forgetting package data
* testing only from source tree
* never installing the built wheel
* broken console script entry points
* missing license files
* claiming unsupported Python versions
* publishing with wrong version
* reusing a published version number
* committing publishing tokens
* assuming editable install proves release correctness

Packaging problems are often invisible until someone else installs the project.

That is why clean-environment testing matters.

---

# Packaging Checklist

Before releasing a package, check:

* Does `pyproject.toml` define the build system?
* Is project metadata correct?
* Is the package name available and appropriate?
* Is the version correct?
* Is `requires-python` accurate?
* Are runtime dependencies declared?
* Are optional dependencies grouped clearly?
* Are README and license included?
* Are package data files included?
* Do console scripts work after installation?
* Does `python -m build` succeed?
* Does the wheel install in a clean environment?
* Can the package be imported after install?
* Do tests pass against the installed artifact?
* Are tokens and secrets absent from the repository?

This checklist is not bureaucracy.

It protects users from broken releases.

---

# A Practical Packaging Flow

A practical flow for a small library:

1. Create `src` layout.
2. Add package code under `src/package_name`.
3. Add tests under `tests`.
4. Add `README.md`.
5. Add `LICENSE`.
6. Add `pyproject.toml`.
7. Declare metadata and dependencies.
8. Add console scripts if needed.
9. Run tests.
10. Build with `python -m build`.
11. Install the wheel in a clean environment.
12. Test import and CLI behavior.
13. Publish to TestPyPI if practicing.
14. Publish to PyPI or internal index when ready.

The exact tools may vary.

The ideas remain:

```text
declare -> build -> install -> verify -> publish
```

---

# Chapter Summary

Packaging turns Python source code into installable, versioned, distributable software.

Modules, import packages, projects, distribution packages, and installed projects are related but distinct.

The distribution name users install may differ from the package name they import.

Modern Python packaging centers on `pyproject.toml`.

The `[build-system]` table tells build frontends which backend to use.

The `[project]` table contains metadata such as name, version, description, README, Python version support, license, authors, dependencies, optional dependencies, and entry points.

Build frontends coordinate builds.

Build backends create distribution artifacts.

Source distributions contain source and metadata.

Wheels are built distributions and usually install faster.

Pure Python wheels can often be platform-independent.

Native-extension wheels are platform, ABI, and Python-version sensitive.

Runtime dependencies belong in project metadata.

Requirements files and lock files describe environments, especially applications.

Libraries usually declare compatibility ranges.

Applications usually pin exact deployments.

Modern project managers such as `uv`, Poetry, Hatch, and PDM help create environments, manage dependencies, lock dependency sets, run commands, and build projects while still relying on Python packaging standards.

Entry points create command-line scripts and plugin hooks.

Package data must be included intentionally.

Editable installs are useful during development but do not prove release correctness.

A package should be built, installed in a clean environment, imported, and tested before publishing.

The central lesson is:

```text
packaging is the contract between your source code and everyone who installs it
```

Good packaging makes code reusable.

Poor packaging makes good code hard to use.

---

# Exercises

1. Create a small `src` layout project with one import package and one function.

2. Write a minimal `pyproject.toml` using a modern build backend.

3. Add project metadata: name, version, description, README, license, and supported Python version.

4. Add one runtime dependency and one optional `dev` dependency group.

5. Build the project with `python -m build`.

6. Install the wheel into a clean virtual environment and import the package.

7. Add a console script entry point and verify that the command works after installation.

8. Add a non-Python data file and configure the package so it is included.

9. Compare a normal install with an editable install.

10. Create the same project with `uv init`, add one dependency with `uv add`, run a command with `uv run`, and inspect the generated lock file.

11. Write a release checklist for one of your own Python projects.

---

# Preview of Chapter 77

Chapter 76 studied packaging.

We saw how Python code becomes installable software through project layout, metadata, build systems, distributions, wheels, dependencies, entry points, and release workflows.

Next we study type hints.

Type hints let Python code express expected shapes:

* function parameter types
* return types
* variable types
* generic containers
* optional values
* protocols
* callable signatures
* type aliases
* class attributes

Packaging and type hints connect because installed packages can carry typing information for users and tools.

If a library exposes a public API, type hints help users understand that API before runtime.

The transition is:

```text
packaging makes code installable
type hints make APIs more explicit
```

Chapter 77 will explain type hints as communication first, tooling second, and runtime behavior only where Python explicitly provides it.
