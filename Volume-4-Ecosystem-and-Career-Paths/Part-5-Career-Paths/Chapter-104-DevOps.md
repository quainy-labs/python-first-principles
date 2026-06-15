# Chapter 104 — DevOps

DevOps is the discipline of making software delivery reliable.

That sounds simple.

It is not.

Writing code is one part of software engineering.

Getting that code into production safely is another.

Keeping it running is another.

Recovering when it breaks is another.

Learning from failures is another.

DevOps connects all of those responsibilities.

It asks a very practical question:

```text
how does this code become a dependable running system?
```

That question includes:

```text
builds
tests
packaging
configuration
deployment
infrastructure
monitoring
alerts
logs
incidents
rollbacks
security
cost
scaling
ownership
```

DevOps is not only a job title.

It is also a way of thinking about the relationship between development and operations.

Developers should understand how their software runs.

Operations engineers should understand how software changes.

Teams should not throw code over a wall and hope someone else can make it work.

The wall is the problem.

DevOps removes the wall.

---

# Why Python Developers Need DevOps

Python developers often begin with local programs.

They write scripts.

They build command-line tools.

They create APIs.

They build data pipelines.

They train models.

They create web applications.

At first, running a program means:

```text
python app.py
```

That is enough for learning.

It is not enough for production.

Production asks harder questions:

```text
where does the application run?
how is Python installed?
how are dependencies installed?
who starts the process?
what happens if it crashes?
where do logs go?
how are secrets provided?
how are migrations run?
how are deployments rolled back?
how are errors detected?
how are users protected during failures?
```

DevOps gives Python developers the vocabulary and habits needed to answer those questions.

Without DevOps, a developer can write code that works only on their laptop.

With DevOps, a developer learns how to ship code that survives the real world.

---

# DevOps as a Career Path

DevOps roles vary across companies.

Some companies use the title DevOps Engineer.

Some say Site Reliability Engineer.

Some say Platform Engineer.

Some say Infrastructure Engineer.

Some say Cloud Engineer.

Some say Release Engineer.

Some say Production Engineer.

The titles overlap.

The shared responsibility is:

```text
make software delivery and operation dependable
```

A DevOps engineer may build CI/CD pipelines.

They may containerize applications.

They may manage Kubernetes clusters.

They may write infrastructure as code.

They may improve monitoring.

They may respond to incidents.

They may automate repetitive operations.

They may define deployment standards.

They may help developers ship more safely.

The best DevOps engineers are not merely tool operators.

They understand systems.

They understand failure.

They understand people.

They understand that a deployment process is both technical and social.

---

# Development and Operations

To understand DevOps, first understand the old split.

Development traditionally meant:

```text
write features
fix bugs
review code
merge changes
```

Operations traditionally meant:

```text
provision servers
install software
configure networks
monitor systems
respond to outages
keep production stable
```

The conflict was predictable.

Developers wanted change.

Operations wanted stability.

Both were right.

Users need new features.

Users also need the product to stay available.

DevOps says the answer is not to choose between speed and stability.

The answer is to build a delivery system where change becomes safer.

That requires automation, testing, observability, shared ownership, and feedback.

---

# The DevOps Loop

A useful mental model is:

```text
plan
    -> code
    -> build
    -> test
    -> release
    -> deploy
    -> operate
    -> monitor
    -> learn
    -> improve
```

This loop never ends.

Each production issue teaches something.

Each deployment teaches something.

Each slow release teaches something.

Each confusing alert teaches something.

Each rollback teaches something.

DevOps turns those lessons into system improvements.

The goal is not heroic firefighting.

The goal is fewer fires.

And when fires happen, the goal is calm, prepared response.

---

# From Script to Service

A Python script becomes a production service through several transitions.

At first, it may run manually:

```text
python report.py
```

Then it may run on a schedule:

```text
cron runs report.py every night
```

Then it may become a service:

```text
gunicorn app:app
```

Then it may be containerized:

```text
docker run company/app:1.4.2
```

Then it may be deployed through orchestration:

```text
Kubernetes Deployment manages replicas of the app
```

Then it may be part of a larger system:

```text
load balancer
    -> API service
    -> queue
    -> workers
    -> database
    -> cache
    -> monitoring
```

Each step adds operational concerns.

The DevOps mindset is to make those concerns explicit.

---

# Environments

Software runs in environments.

Common environments include:

```text
local development
test
staging
production
preview environments
disaster recovery environments
```

Each environment has a purpose.

Local development helps developers move quickly.

Test environments run automated checks.

Staging resembles production.

Production serves real users.

Preview environments let teams review changes before merge.

Disaster recovery environments help recover from major failures.

The danger is environment drift.

Environment drift happens when environments differ in hidden ways.

For example:

```text
local Python version differs from production
staging uses a smaller database
production has different environment variables
test uses fake services
dependencies are installed differently
file paths differ
operating system packages differ
```

DevOps reduces drift with automation, containers, infrastructure as code, and clear configuration.

---

# Configuration

Configuration is everything that changes between environments without changing application code.

Examples include:

```text
database URL
API keys
feature flags
log level
service endpoints
queue names
timeout values
allowed origins
worker counts
```

Hard-coded configuration is dangerous.

For example:

```python
DATABASE_URL = "postgres://production-db"
```

This makes the application harder to move and easier to break.

A better pattern is:

```python
import os

DATABASE_URL = os.environ["DATABASE_URL"]
```

The application reads configuration from the environment.

The deployment system provides the environment.

This separates code from deployment settings.

But configuration still needs discipline.

DevOps teams must know:

```text
which configuration keys exist
which are required
which are secret
which vary by environment
which can be changed safely
which require restart
```

Configuration should be documented, validated, and versioned where appropriate.

---

# Secrets

Secrets are sensitive values.

Examples include:

```text
database passwords
API tokens
private keys
OAuth client secrets
signing keys
cloud credentials
webhook secrets
```

Secrets must not be committed to Git.

They must not be printed in logs.

They must not be copied casually into chat tools.

They must not be stored in plain text configuration files.

Secret management includes:

```text
secure storage
limited access
rotation
audit logs
environment-specific values
emergency revocation
```

For Python developers, the most important habit is simple:

```text
never hard-code secrets
```

Use environment variables, secret managers, CI/CD secret storage, or platform-provided secret mechanisms.

Also remember that logs can leak secrets.

If a failed request prints headers, tokens may appear.

If a traceback prints configuration, credentials may appear.

Operational security begins with careful defaults.

---

# Build

A build turns source code into something that can be tested and deployed.

For a Python package, a build might produce a wheel.

For a web service, a build might produce a container image.

For a frontend, a build might produce static assets.

For infrastructure, a build might produce a plan or artifact.

The key idea is:

```text
build once, deploy the same artifact
```

If staging and production build separately, they may not run the same code.

Dependencies may resolve differently.

Build scripts may behave differently.

Environment variables may affect output.

A safer pipeline builds an artifact once, tags it, tests it, and promotes it.

For example:

```text
commit
    -> build container image
    -> run tests
    -> push image to registry
    -> deploy image to staging
    -> promote same image to production
```

The artifact becomes the unit of release.

---

# Continuous Integration

Continuous integration means changes are regularly integrated into the shared codebase and checked automatically.

In practical terms, CI usually runs on pull requests and commits.

It may run:

```text
format checks
linters
type checks
unit tests
integration tests
security scans
dependency checks
build checks
```

GitHub Actions is one common CI/CD platform.

It organizes automation into workflows, events, jobs, steps, actions, and runners.

That terminology is useful even when using another platform.

The concept is:

```text
event triggers workflow
workflow runs jobs
jobs run steps
steps execute scripts or reusable actions
```

For Python projects, a simple CI workflow might:

```text
checkout code
install Python
install dependencies
run formatting check
run tests
run type checker
build package
```

CI gives the team fast feedback.

It does not guarantee production success.

But without CI, teams often discover problems too late.

---

# Continuous Delivery and Continuous Deployment

Continuous delivery and continuous deployment are related but not identical.

Continuous delivery means the software is always in a releasable state.

Deployment may still require manual approval.

Continuous deployment means every passing change can be deployed automatically.

The difference is:

```text
continuous delivery:
    ready to deploy

continuous deployment:
    automatically deployed
```

Not every organization should deploy every change automatically.

The right choice depends on product risk, team maturity, compliance, test quality, and user impact.

High-risk systems may need approvals.

Internal tools may deploy automatically.

Consumer web apps may use gradual rollout.

The professional DevOps question is not:

```text
are we deploying as fast as possible?
```

The better question is:

```text
can we deploy safely, repeatedly, and recover quickly?
```

---

# Pipeline Design

A deployment pipeline is a sequence of checks and actions that moves code toward production.

A simple pipeline may look like:

```text
pull request
    -> lint
    -> unit tests
    -> integration tests
    -> build image
    -> scan image
    -> deploy staging
    -> smoke test staging
    -> approve production
    -> deploy production
    -> monitor rollout
```

Each stage should answer a question.

Lint asks:

```text
is the code consistent and free of obvious mistakes?
```

Unit tests ask:

```text
do small pieces behave correctly?
```

Integration tests ask:

```text
do components work together?
```

Build asks:

```text
can we create a deployable artifact?
```

Security scan asks:

```text
does this artifact contain known obvious risks?
```

Smoke tests ask:

```text
does the deployed service basically work?
```

Monitoring asks:

```text
is production healthy after the change?
```

A pipeline should not be ceremonial.

Every stage should earn its place.

---

# Containers

Containers package an application with the environment needed to run it.

Docker's documentation describes Docker as a platform for developing, shipping, and running applications, with containers providing isolated environments that include what an application needs.

For Python developers, containers solve a familiar problem:

```text
it works on my machine
```

A container image can include:

```text
base operating system files
Python runtime
system packages
Python dependencies
application code
entrypoint command
```

A container is a running instance of an image.

The distinction matters:

```text
image:
    packaged template

container:
    running process created from the image
```

Containers do not remove the need to understand dependencies.

They make dependencies explicit.

---

# Dockerfile Basics

A Dockerfile describes how to build an image.

A simple Python API Dockerfile might look like:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen

COPY . .

CMD ["uv", "run", "gunicorn", "app:app", "--bind", "0.0.0.0:8000"]
```

This is simplified.

Production Dockerfiles often need more care.

They may use:

```text
multi-stage builds
non-root users
small base images
dependency caching
locked dependencies
health checks
explicit entrypoints
minimal copied files
```

A Dockerfile is operational code.

It deserves review.

It affects security, startup time, image size, and reproducibility.

---

# Container Registries

Container images are usually stored in registries.

Examples include:

```text
Docker Hub
GitHub Container Registry
Amazon Elastic Container Registry
Google Artifact Registry
Azure Container Registry
self-hosted registries
```

A registry lets the pipeline push an image and the deployment platform pull it.

Images should be tagged carefully.

Weak tagging:

```text
latest
```

Better tagging:

```text
app:1.8.4
app:git-3f4a91c
app:2026-06-15-1430
```

The exact format matters less than traceability.

When a production incident happens, the team should be able to answer:

```text
which image is running?
which commit built it?
which pipeline built it?
which dependencies are inside it?
```

Traceability is a DevOps superpower.

---

# Orchestration

Running one container manually is easy.

Running many containers reliably is harder.

Production systems need:

```text
restart on failure
multiple replicas
service discovery
load balancing
configuration
secrets
rollouts
rollbacks
resource limits
health checks
autoscaling
```

Container orchestration handles these concerns.

Kubernetes is the dominant orchestration platform in many organizations.

Kubernetes manages containerized workloads and services using declarative configuration and automation.

It can manage scaling, failover, service discovery, load balancing, rollouts, rollbacks, secrets, and configuration.

But Kubernetes is not magic.

It is a powerful platform with its own complexity.

A DevOps engineer must know when Kubernetes is appropriate and when it is unnecessary.

A small internal service may be fine on a simpler platform.

A large multi-service environment may benefit from Kubernetes.

Professional judgment means matching the tool to the operational problem.

---

# Desired State

Kubernetes and many DevOps tools use the idea of desired state.

Instead of manually telling the system every step, you declare what should be true.

For example:

```text
run three replicas of this application image
expose them through this service
mount this configuration
use this resource limit
restart unhealthy containers
```

The platform then tries to make actual state match desired state.

This is different from a manual script:

```text
ssh into server
pull code
restart process
hope it worked
```

Desired state is powerful because it makes infrastructure inspectable and repeatable.

But it also requires careful definitions.

If the desired state is wrong, automation will faithfully make the wrong thing happen.

Automation magnifies both good design and bad design.

---

# Infrastructure as Code

Infrastructure as code means defining infrastructure using files that can be versioned, reviewed, tested, and applied.

Examples of infrastructure include:

```text
servers
networks
load balancers
databases
queues
DNS records
permissions
storage buckets
Kubernetes resources
monitoring rules
```

Infrastructure as code changes the workflow from:

```text
click in cloud console
```

to:

```text
review a pull request
apply a versioned change
```

This improves traceability.

It makes environments easier to recreate.

It reduces hidden manual changes.

It also raises the quality bar.

Infrastructure code can break production.

It needs review, planning, and rollback strategies.

---

# Immutable Infrastructure

Immutable infrastructure means replacing running units instead of modifying them in place.

For example, instead of SSHing into a server and editing files, you build a new image and deploy it.

The idea is:

```text
do not patch the running snowflake
replace it with a known artifact
```

This reduces configuration drift.

It makes rollback clearer.

It makes environments more reproducible.

Containers encourage immutable infrastructure because container images are built artifacts.

But the principle applies more broadly.

If production state is created by manual edits, the team eventually loses track of reality.

DevOps prefers reality that can be rebuilt.

---

# Deployment Strategies

Deploying means replacing one version of software with another.

There are several strategies.

A rolling deployment updates instances gradually.

A blue-green deployment runs two environments and switches traffic from old to new.

A canary deployment sends a small amount of traffic to the new version before expanding.

A shadow deployment sends copied traffic to the new version without showing its response to users.

A feature flag releases behavior separately from deployment.

Each strategy has tradeoffs.

Rolling deployments are common and efficient.

Blue-green deployments provide clean rollback but may require duplicate infrastructure.

Canary deployments reduce blast radius.

Shadow deployments help test production-like traffic.

Feature flags allow controlled exposure but add code complexity.

The DevOps engineer chooses based on risk.

The higher the risk, the more gradual and observable the rollout should be.

---

# Rollback

Every deployment plan needs a rollback plan.

Rollback means returning to a previous safe state.

A rollback plan should answer:

```text
what signal tells us to rollback?
who can trigger rollback?
how long does rollback take?
does database state allow rollback?
does configuration allow rollback?
does the previous image still exist?
will clients tolerate the change?
```

Code rollback is often easy.

Data rollback is often hard.

For example, suppose a deployment changes a database schema.

If the new version writes data in a format the old version cannot read, rolling back code may not fix the issue.

This is why database migrations require special care.

Rollback is not an afterthought.

It is part of deployment design.

---

# Database Migrations

Python web applications often use relational databases.

Changing the database schema is a production operation.

Migration tools can apply changes such as:

```text
create table
add column
rename column
create index
change constraint
backfill data
```

Dangerous migrations include:

```text
dropping columns
renaming columns used by old code
locking large tables
rewriting huge tables
changing data formats without compatibility
```

A safer pattern for many changes is expand and contract.

For example:

```text
1. add new nullable column
2. deploy code that writes both old and new column
3. backfill old data
4. deploy code that reads new column
5. stop writing old column
6. later remove old column
```

This is slower.

It is also safer.

DevOps teaches patience where impatience breaks production.

---

# Monitoring

Monitoring answers:

```text
what is happening?
```

Prometheus is a common open-source monitoring and alerting toolkit.

It collects and stores metrics as time series data, often with labels, and supports querying and alerting.

Metrics are numerical measurements over time.

Examples include:

```text
request count
error count
request latency
CPU usage
memory usage
queue depth
database connections
cache hit rate
worker retries
```

Monitoring is not only infrastructure monitoring.

Applications should expose meaningful business and product signals too.

For a Python API, useful metrics may include:

```text
requests by endpoint
error rate by endpoint
latency percentiles
background job duration
failed login attempts
payment failures
email delivery failures
model inference latency
```

If a system matters, it should tell you how it is doing.

---

# Logs

Logs are records of events.

A log line might say:

```text
user 1842 created invoice 991
```

Or:

```text
payment provider timeout after 3000 ms
```

Logs help explain what happened.

Good logs are structured.

Instead of:

```text
Something went wrong
```

Prefer:

```json
{
  "level": "error",
  "event": "payment_provider_timeout",
  "request_id": "req_123",
  "provider": "stripe",
  "timeout_ms": 3000
}
```

Structured logs are easier to search, filter, aggregate, and alert on.

Python applications can use the standard `logging` module, but production teams often configure JSON logging, request IDs, and centralized log storage.

Logs should not leak secrets.

Logs should not contain unnecessary private data.

Operational visibility and privacy must be balanced.

---

# Traces

Traces show how a request moves through a distributed system.

In a simple monolith, a request may stay inside one process.

In a distributed system, a request may involve:

```text
API gateway
authentication service
application service
database
cache
message queue
worker
external payment provider
```

When the request is slow, logs and metrics may not be enough.

A trace can show which span took time.

For example:

```text
GET /checkout
    -> authenticate user: 12 ms
    -> load cart: 20 ms
    -> calculate tax: 900 ms
    -> create payment intent: 180 ms
```

Now the problem is visible.

Tracing is especially useful for microservices, async jobs, API calls, and AI systems with multiple model/tool steps.

---

# The Three Pillars and Beyond

People often talk about three pillars of observability:

```text
metrics
logs
traces
```

This is useful.

But observability is not just collecting data.

Observability means the system can answer new questions during failure.

For example:

```text
which version introduced the error?
which users are affected?
which region is unhealthy?
which dependency slowed down?
which queue is backing up?
which endpoint changed latency?
which feature flag is enabled?
```

A pile of logs is not observability.

A dashboard nobody uses is not observability.

An alert nobody trusts is not observability.

Observability is operational understanding.

---

# Alerts

Alerts tell humans when action may be needed.

Bad alerts create noise.

Good alerts protect users.

A bad alert says:

```text
CPU is 82 percent
```

This may or may not matter.

A better alert says:

```text
checkout error rate is above 5 percent for 10 minutes
```

This is closer to user impact.

Alerts should be:

```text
actionable
owned
documented
prioritized
tied to user impact where possible
```

Every alert should have a reasonable response.

If no one knows what to do, the alert needs improvement.

If an alert fires often and does not require action, it trains people to ignore alerts.

Alert fatigue is a production risk.

---

# SLOs

SLO stands for service level objective.

An SLO defines a target for reliability.

For example:

```text
99.9 percent of checkout requests complete successfully within 500 ms over 30 days
```

An SLO is useful because it connects engineering work to user experience.

Without SLOs, reliability conversations become vague.

With SLOs, the team can reason:

```text
are users receiving acceptable service?
are we spending too much error budget?
should we slow feature work to improve reliability?
```

Not every small project needs a formal SLO program.

But every production team needs some idea of acceptable reliability.

DevOps makes reliability explicit.

---

# Incident Response

An incident is a production event that harms or threatens the service.

Examples include:

```text
site outage
high error rate
database failure
security exposure
data corruption
payment failures
queue backlog
bad deployment
third-party provider outage
```

Incident response should be practiced.

During an incident, teams need:

```text
clear roles
communication channel
status updates
timeline
mitigation plan
rollback option
customer communication
post-incident review
```

The first goal is mitigation.

Stop the harm.

Restore service.

Reduce blast radius.

Detailed root-cause analysis can come later.

During the incident, calm beats cleverness.

---

# Post-Incident Reviews

After an incident, the team should learn.

A good post-incident review asks:

```text
what happened?
how did we detect it?
how did we respond?
what went well?
what made response harder?
what user impact occurred?
what technical factors contributed?
what process factors contributed?
what will we change?
```

Blame is usually unhelpful.

Production systems fail through combinations of technical, process, and communication problems.

A useful review focuses on improving the system.

For example:

```text
add missing alert
improve rollback script
add migration safety check
update runbook
add dashboard panel
change deployment approval
fix confusing error message
```

The incident is not complete when the service recovers.

It is complete when the team has learned and improved.

---

# Runbooks

A runbook is a written procedure for operating or repairing a system.

Examples include:

```text
how to rollback a deployment
how to restart workers
how to rotate a secret
how to restore a database backup
how to drain a node
how to investigate high latency
how to handle queue backlog
```

Runbooks should be clear enough to use during stress.

They should include:

```text
symptoms
impact
commands
dashboards
logs to inspect
decision points
rollback steps
escalation contacts
```

A runbook that only one person understands is fragile.

DevOps spreads operational knowledge.

---

# Backups and Recovery

Backups are not useful unless recovery works.

Many teams have backups.

Fewer teams regularly test restores.

A recovery plan should answer:

```text
what data is backed up?
how often?
where is it stored?
who can access it?
how long is it retained?
how do we restore it?
how long does restore take?
how much data loss is acceptable?
```

Two important terms are:

```text
RPO: recovery point objective
RTO: recovery time objective
```

RPO asks:

```text
how much data can we afford to lose?
```

RTO asks:

```text
how long can we afford to be down?
```

These are business questions expressed as technical requirements.

---

# Scaling

Scaling means handling more load.

There are two broad forms:

```text
vertical scaling:
    give one machine more CPU, memory, or disk

horizontal scaling:
    run more instances
```

Python web services often scale horizontally behind a load balancer.

But scaling requires design.

If an application stores session state only in local memory, adding more instances may break sessions.

If background jobs are not idempotent, retries may create duplicates.

If a database is the bottleneck, adding application servers may not help.

Scaling questions include:

```text
what is the bottleneck?
is the application stateless?
can work be parallelized?
does the database scale?
are queues needed?
are caches safe?
are rate limits in place?
```

Scaling is not just adding servers.

Scaling is understanding where pressure accumulates.

---

# Performance

Performance is part of operations.

Users care whether the system is fast enough.

Performance work includes:

```text
profiling code
optimizing database queries
adding indexes
reducing network calls
using caches
parallelizing work
moving slow work to background jobs
reducing payload size
choosing better algorithms
```

DevOps connects performance to production signals.

Instead of guessing, measure:

```text
p50 latency
p95 latency
p99 latency
throughput
error rate
queue wait time
database query duration
external API latency
```

Average latency can hide pain.

The slowest requests often matter most.

That is why percentiles are useful.

---

# Capacity Planning

Capacity planning means ensuring the system has enough resources for expected demand.

Resources include:

```text
CPU
memory
disk
network bandwidth
database connections
queue throughput
API rate limits
cloud quotas
third-party limits
```

Capacity planning asks:

```text
what is normal load?
what is peak load?
what happens during growth?
what happens during marketing events?
what happens when a dependency slows down?
what is our headroom?
```

Cloud platforms make it easy to add resources.

They also make it easy to spend money accidentally.

Good DevOps balances capacity, reliability, and cost.

---

# Cost Awareness

Production systems cost money.

Cloud bills come from:

```text
compute
storage
network egress
managed databases
logs
metrics
traces
load balancers
containers
serverless functions
AI model calls
data transfer
backup retention
```

Cost awareness does not mean making every system cheap.

It means understanding tradeoffs.

For example:

```text
more replicas may improve reliability but increase cost
more logs may improve debugging but increase storage cost
more detailed traces may improve diagnosis but increase ingestion cost
larger databases may improve performance but increase spend
```

DevOps engineers help teams see these tradeoffs clearly.

Cost surprises are operational failures too.

---

# Linux Fundamentals

Many production systems run on Linux.

A DevOps engineer should understand Linux basics:

```text
processes
signals
file permissions
users and groups
system services
environment variables
network sockets
logs
package managers
disk usage
memory usage
CPU usage
```

Python developers benefit from knowing commands such as:

```text
ps
top
df
du
free
curl
ss
journalctl
systemctl
grep
awk
sed
tail
```

The goal is not to memorize every flag.

The goal is to investigate a running system without panic.

Linux is the floor under much of modern infrastructure.

---

# Networking Fundamentals

DevOps requires networking basics.

Important concepts include:

```text
IP addresses
ports
DNS
HTTP
TLS
load balancers
proxies
firewalls
subnets
NAT
VPNs
timeouts
retries
```

Many production issues are networking issues wearing another costume.

For example:

```text
service cannot resolve database hostname
TLS certificate expired
load balancer health check path is wrong
firewall blocks outbound traffic
DNS cache points to old address
client timeout is shorter than server processing time
```

Python developers who understand networking debug faster.

They know where to look.

They know which layer may be failing.

---

# Queues and Background Workers

Many systems should not do all work during the request-response cycle.

Slow or unreliable work can move to background jobs.

Examples include:

```text
sending emails
processing uploads
generating reports
calling slow third-party APIs
running ML inference
retrying failed payments
syncing external systems
```

Queues improve responsiveness and resilience.

They also introduce operational concerns:

```text
queue depth
retry behavior
dead-letter queues
idempotency
worker scaling
job timeouts
poison messages
duplicate execution
ordering
```

DevOps engineers monitor queues carefully.

A growing queue may mean workers are down, dependencies are slow, or traffic increased.

The queue tells a story.

The system must be able to hear it.

---

# Idempotency

Idempotency means an operation can be repeated without causing unintended additional effects.

For example:

```text
safe:
    mark order 123 as paid

dangerous if repeated:
    charge customer 123 ten dollars
```

Distributed systems retry.

Networks fail.

Workers crash.

APIs time out after the server already completed the action.

If operations are not idempotent, retries can cause duplicate side effects.

Python developers building production systems should ask:

```text
what happens if this function runs twice?
what happens if the client retries?
what happens if the worker crashes halfway?
what unique key prevents duplicates?
```

DevOps and application design meet here.

Reliability is not only infrastructure.

It is also code behavior.

---

# Release Management

Release management controls how software versions reach users.

It includes:

```text
versioning
changelogs
approvals
release notes
deployment windows
feature flags
rollback plans
communication
```

Small teams may release continuously.

Larger organizations may coordinate releases across many services.

Regulated industries may require approvals and audit trails.

The goal is not bureaucracy.

The goal is controlled change.

Every release changes production reality.

DevOps makes that change visible.

---

# Feature Flags

Feature flags allow code to be deployed while behavior remains disabled or limited.

For example:

```python
if feature_enabled("new_checkout", user):
    return new_checkout_flow(user)
else:
    return old_checkout_flow(user)
```

Feature flags help with:

```text
gradual rollout
A/B testing
quick disablement
internal testing
customer-specific enablement
risk reduction
```

But flags also add complexity.

Old flags should be removed.

Flag combinations should be tested.

Flag ownership should be clear.

A forgotten feature flag can become hidden technical debt.

---

# Security in DevOps

Security should be part of delivery.

This is often called DevSecOps.

Security checks may include:

```text
dependency scanning
secret scanning
container image scanning
static analysis
infrastructure policy checks
permission reviews
runtime monitoring
audit logging
```

Security should not appear only at the end.

If security review happens after everything is built, teams face painful rewrites.

Better pipelines catch common issues early.

Better platform defaults make secure behavior easier.

Better secrets management prevents accidental leaks.

Better access controls reduce blast radius.

Chapter 105 will study Cybersecurity directly.

Here, the DevOps lesson is:

```text
secure delivery is part of reliable delivery
```

---

# Access Control

Production access should be limited.

Not every developer needs direct database access.

Not every service needs admin permissions.

Not every CI job needs deployment credentials.

DevOps teams apply least privilege.

Least privilege means:

```text
give only the permissions needed
for only the time needed
to only the identity that needs them
```

Access control should include:

```text
human access
service accounts
CI/CD identities
cloud roles
database roles
Kubernetes permissions
secret access
```

Strong access control reduces the damage of mistakes and compromise.

---

# CI/CD Security

CI/CD pipelines are powerful.

They can build code, access secrets, publish artifacts, deploy infrastructure, and change production.

That means they are security-sensitive.

Pipeline risks include:

```text
untrusted pull requests accessing secrets
third-party actions with too much privilege
long-lived cloud credentials
overbroad deployment tokens
logs exposing secrets
mutable dependencies
compromised build artifacts
```

Safer practices include:

```text
pin action versions
limit token permissions
separate pull request checks from deployment
use short-lived credentials
protect production environments
review workflow changes carefully
scan for secrets
store artifacts immutably
```

A CI pipeline is not just automation.

It is part of the production attack surface.

---

# Platform Engineering

Platform engineering is closely related to DevOps.

It focuses on building internal platforms that help developers ship safely and quickly.

Examples include:

```text
service templates
deployment pipelines
logging standards
monitoring defaults
secret management
preview environments
database provisioning
Kubernetes abstractions
self-service deployment
```

The platform team is not there to control developers.

It is there to make good practices easy.

The best platform feels like a paved road:

```text
follow the path and you get sensible defaults
```

Developers can still do custom work when needed.

But the common path should be safe, observable, and fast.

---

# Site Reliability Engineering

Site Reliability Engineering, or SRE, is a discipline closely related to DevOps.

SRE emphasizes reliability engineering, automation, SLOs, error budgets, incident response, and operational excellence.

Where DevOps is a broad cultural and technical movement, SRE is often more explicitly focused on reliability as an engineering problem.

The two share many practices:

```text
automation
monitoring
incident response
capacity planning
post-incident reviews
reliability targets
production ownership
```

In many companies, DevOps and SRE roles overlap.

The difference matters less than the habits:

```text
measure reliability
reduce toil
automate safely
learn from incidents
protect users
```

---

# Toil

Toil is repetitive operational work that does not create lasting value.

Examples include:

```text
manually restarting services
manually provisioning standard resources
copying logs into reports
repeating the same deployment checklist
hand-editing configuration
manually rotating routine credentials
```

Some manual work is necessary.

But too much toil consumes engineering time and increases mistakes.

DevOps engineers reduce toil with automation.

But automation should be thoughtful.

Automating a bad process can make bad outcomes happen faster.

First understand the work.

Then simplify it.

Then automate it.

---

# Python in DevOps

Python is useful in DevOps.

It is used for:

```text
automation scripts
deployment helpers
cloud API clients
log processing
monitoring checks
data cleanup
release tooling
infrastructure validation
incident analysis
internal CLIs
```

Python is especially good when the task needs:

```text
readable code
API integration
JSON/YAML processing
filesystem automation
HTTP requests
data transformation
quick iteration
```

But DevOps Python should be written carefully.

Operational scripts need:

```text
clear arguments
dry-run modes
logging
timeouts
retries
error handling
idempotency
safe defaults
```

A script that changes production should be designed like production code.

---

# Shell and Python

DevOps engineers use both shell and Python.

Shell is excellent for:

```text
running commands
connecting tools
quick inspection
small glue tasks
interactive debugging
```

Python is better for:

```text
complex logic
structured data
API calls
tests
reusable tools
error handling
large automation
```

A healthy rule is:

```text
use shell for simple command composition
use Python when logic becomes real software
```

Many fragile operations begin as shell snippets that slowly become unreadable programs.

When the logic grows, give it a proper language.

---

# YAML

DevOps engineers read a lot of YAML.

YAML appears in:

```text
GitHub Actions workflows
Kubernetes manifests
Docker Compose files
Ansible playbooks
Helm values
configuration files
```

YAML looks simple.

It can be surprisingly tricky.

Indentation matters.

Booleans can surprise you.

Large files become hard to reason about.

Copy-paste creates drift.

Templates can hide complexity.

The professional skill is not merely writing YAML.

It is keeping configuration understandable.

When configuration becomes too clever, operations become harder.

---

# Cloud Platforms

Modern DevOps often uses cloud platforms.

Common services include:

```text
virtual machines
containers
managed databases
object storage
load balancers
DNS
identity and access management
secret managers
message queues
serverless functions
monitoring
container registries
```

Cloud providers differ.

The core concepts are similar.

A DevOps engineer should understand the abstractions:

```text
compute
storage
networking
identity
observability
managed services
regions
availability zones
quotas
pricing
```

Cloud skill is not memorizing every service.

It is knowing how to assemble reliable systems from managed building blocks.

---

# Serverless

Serverless platforms run code without the team managing servers directly.

Examples include functions-as-a-service and managed container platforms.

Serverless can be excellent for:

```text
event-driven tasks
webhooks
small APIs
scheduled jobs
bursty workloads
background processing
```

It has tradeoffs:

```text
cold starts
execution time limits
vendor constraints
observability challenges
local testing differences
cost surprises at high volume
```

Serverless does not remove operations.

It changes operations.

The team still owns reliability, cost, security, and behavior.

Managed infrastructure is still infrastructure.

---

# Monoliths and Microservices

DevOps work changes depending on architecture.

A monolith is one deployable application.

Microservices split behavior across multiple services.

Microservices can help teams scale ownership and independent deployment.

They also add operational complexity.

They introduce:

```text
network calls
distributed tracing
service discovery
version compatibility
partial failure
more deployments
more dashboards
more access policies
more incident paths
```

A modular monolith can be easier to operate than premature microservices.

DevOps judgment asks:

```text
does this architecture make production easier or harder?
```

Architecture is not only code structure.

It is operational reality.

---

# Python Web Service Example

Consider a small FastAPI service.

Locally, the developer runs:

```text
uvicorn app:app --reload
```

Production may require:

```text
container image
non-root runtime user
environment configuration
database migration step
gunicorn or process manager
health endpoint
readiness check
structured logs
metrics endpoint
CI pipeline
deployment manifest
secret injection
rollback plan
```

The application code may be simple.

The production system is not.

DevOps does not make this complexity disappear.

It makes the complexity visible, repeatable, and manageable.

---

# Health Checks

Health checks tell the platform whether an application is alive and ready.

There are different kinds.

A liveness check asks:

```text
is the process alive enough to restart if broken?
```

A readiness check asks:

```text
is the service ready to receive traffic?
```

A startup check asks:

```text
has the application finished starting?
```

These should not be confused.

For example, a service may be alive but not ready because it cannot connect to the database.

If traffic reaches it too early, users see errors.

Health checks help deployment platforms route traffic safely.

Poor health checks cause false confidence or unnecessary restarts.

---

# Graceful Shutdown

Production processes are stopped and restarted.

Deployments, autoscaling, node maintenance, and failures all stop processes.

A Python service should handle shutdown gracefully.

Graceful shutdown means:

```text
stop accepting new work
finish in-flight requests when possible
release resources
close database connections
stop workers cleanly
avoid corrupting state
```

If a process ignores termination signals, the platform may kill it forcefully.

That can interrupt requests or jobs.

DevOps-aware application design includes lifecycle behavior.

Startup and shutdown are part of correctness.

---

# Timeouts and Retries

Timeouts and retries are central to reliability.

Without timeouts, a service can hang forever waiting for a dependency.

Without retries, temporary failures may become user-visible failures.

But retries can also make outages worse.

If every client retries aggressively, a struggling service receives even more traffic.

Good retry design includes:

```text
reasonable timeouts
limited retry count
exponential backoff
jitter
idempotency
circuit breakers where appropriate
clear error handling
```

This is where application engineering and operations meet.

Reliable systems are built from small careful choices.

---

# Dependency Management

Python dependency management is an operational concern.

Dependencies affect:

```text
security
reproducibility
build speed
image size
runtime behavior
compatibility
```

A production Python project should use locked dependencies.

It should be able to reproduce the environment.

It should update dependencies deliberately.

It should scan for known vulnerabilities where appropriate.

It should avoid installing unnecessary packages in production images.

Dependency chaos becomes deployment chaos.

DevOps prefers controlled builds.

---

# Testing in Production

Testing in production sounds dangerous.

But every deployment is tested in production eventually.

The question is whether it is done safely.

Safe production testing includes:

```text
canary releases
feature flags
synthetic checks
shadow traffic
limited cohorts
read-only verification
automatic rollback signals
```

Unsafe production testing means:

```text
deploy to everyone
watch users complain
manually guess what broke
```

Production is the only environment exactly like production.

DevOps uses that fact carefully.

---

# Documentation

Operational documentation matters.

Important documents include:

```text
architecture diagrams
deployment process
runbooks
incident response process
service ownership
environment variables
secret rotation process
backup and restore process
on-call expectations
SLO definitions
```

Documentation should be close to the work.

If deployment changes, deployment docs should change.

If a runbook is wrong during an incident, fix it afterward.

Docs are part of the system.

Stale docs are operational debt.

---

# Team Culture

DevOps is as much culture as tooling.

A team with excellent tools can still have poor DevOps culture.

Warning signs include:

```text
developers do not know how production works
operations learns about releases after they happen
incidents become blame sessions
deployments require heroics
only one person can fix common failures
alerts are ignored
manual changes are undocumented
```

Healthy DevOps culture includes:

```text
shared ownership
clear communication
automation
learning from failure
small safe changes
respect for production
respect for developer flow
```

The best DevOps work often feels boring.

Deployments become routine.

Incidents become rarer.

Recovery becomes practiced.

That kind of boring is valuable.

---

# DevOps for AI and Data Systems

DevOps also matters for AI and data systems.

AI systems add concerns such as:

```text
model versioning
prompt versioning
eval pipelines
GPU capacity
vector indexes
model latency
model cost
guardrail monitoring
feedback loops
```

Data systems add concerns such as:

```text
pipeline schedules
data freshness
schema changes
backfills
data quality checks
lineage
warehouse cost
```

The principles remain familiar:

```text
automate delivery
observe behavior
control change
recover safely
learn from failure
```

DevOps is not only for web APIs.

It applies anywhere software must run reliably.

---

# A Day in the Life

A DevOps engineer's day might begin with a deployment failure.

They inspect the CI logs and find that dependency caching is stale.

They fix the workflow and add a clearer error message.

Later, an application team asks for a new staging database.

The DevOps engineer reviews the infrastructure change and ensures backup settings are correct.

Then an alert fires for high API latency.

They check dashboards, inspect traces, and find a slow third-party dependency.

They help the application team adjust timeouts and add a fallback.

In the afternoon, they improve the deployment pipeline so container images are scanned before release.

They update a runbook after noticing that an incident step was unclear.

They end the day reviewing cloud costs and finding an unused test cluster.

The job is varied.

It rewards curiosity, calm, and systems thinking.

---

# Skill Map

A DevOps skill map includes:

```text
Linux
networking
Git
CI/CD
containers
Kubernetes or another deployment platform
cloud fundamentals
infrastructure as code
monitoring
logging
tracing
security basics
database operations
scripting
incident response
documentation
communication
```

For Python developers, the most useful path is:

```text
learn to deploy one Python app well
    -> add CI
    -> add a container
    -> add configuration and secrets
    -> add logs and metrics
    -> add a database
    -> add migrations
    -> add rollback
    -> add monitoring and alerts
```

You do not need to learn every tool at once.

Learn the lifecycle.

Tools will make more sense afterward.

---

# Portfolio Projects

A DevOps portfolio should show operational maturity.

Good projects include:

```text
containerized Python API with CI/CD
Kubernetes deployment with health checks
infrastructure as code for a small web app
monitoring dashboard and alert rules
backup and restore practice for a database
blue-green or canary deployment demo
internal CLI for deployment automation
log aggregation setup with structured logs
```

The project should explain:

```text
how it builds
how it deploys
how configuration works
how secrets are handled
how rollback works
how logs and metrics are viewed
what failure modes were considered
```

A DevOps portfolio is strongest when it shows thinking, not just screenshots of tools.

---

# Interview Expectations

DevOps interviews often test practical reasoning.

Questions may include:

```text
How would you deploy a Python API?
How would you design a CI/CD pipeline?
How would you containerize this application?
How would you rollback a failed deployment?
How would you investigate high latency?
How would you manage secrets?
How would you monitor this system?
How would you reduce alert noise?
How would you safely run a database migration?
How would you respond to an outage?
```

Strong answers are specific.

Weak answer:

```text
I would use Kubernetes.
```

Stronger answer:

```text
I would first define the service's runtime needs, health checks, configuration, secrets, resource limits, deployment strategy, rollback path, logs, metrics, and database migration process. Kubernetes may be the right platform if we need orchestration, scaling, and standardized deployments, but I would not start there unless the operational needs justify it.
```

DevOps interviews reward judgment.

Tools matter.

Tradeoffs matter more.

---

# Common Beginner Mistakes

Beginners often confuse DevOps with tool collection.

They try to learn Docker, Kubernetes, Terraform, Prometheus, Grafana, Helm, Ansible, and every cloud service at once.

This becomes overwhelming.

The better approach is to understand problems first:

```text
how do we build?
how do we test?
how do we package?
how do we configure?
how do we deploy?
how do we observe?
how do we recover?
```

Other common mistakes include:

```text
putting secrets in Git
using latest tags in production
deploying without rollback
ignoring database migration risk
creating noisy alerts
logging private data
not setting timeouts
not testing restore procedures
making manual production changes
overusing Kubernetes for small problems
```

Each mistake is avoidable.

DevOps maturity grows through careful repetition.

---

# Professional Habits

Professional DevOps engineers build habits.

They automate repeated work.

They review infrastructure changes.

They document operational procedures.

They keep secrets out of code.

They make deployments boring.

They measure user impact.

They reduce alert noise.

They test backups.

They plan rollback before deployment.

They communicate clearly during incidents.

They learn from failures.

They design for the next engineer who will be tired at 3 a.m.

That last point matters.

Production systems are operated by humans.

Kindness to future operators is an engineering virtue.

---

# The Future of DevOps

DevOps continues to evolve.

Platform engineering is becoming more common.

Managed cloud services continue to reduce some operational burdens.

Security is moving earlier into pipelines.

AI is entering operations through log analysis, incident summarization, code review, deployment assistance, and automated remediation.

But the fundamentals remain.

Systems still fail.

Deployments still change reality.

Users still need reliability.

Teams still need feedback.

Automation still needs judgment.

The tools will change.

The responsibility will not.

---

# Practical Checklist

Before calling a Python service production-ready, ask:

```text
Can it be built reproducibly?
Are dependencies locked?
Is configuration externalized?
Are secrets managed safely?
Is there a CI pipeline?
Are tests run automatically?
Is the deployable artifact versioned?
Is deployment automated?
Is rollback documented?
Are database migrations safe?
Are logs structured?
Are metrics available?
Are traces useful if needed?
Are alerts actionable?
Are health checks correct?
Are timeouts configured?
Are retries safe?
Are backups tested?
Is access limited?
Is production ownership clear?
```

If many answers are no, the service may still run.

But it is not yet operationally mature.

---

# Summary

DevOps is the discipline of reliable software delivery and operation.

It connects development, deployment, infrastructure, observability, incident response, security, and continuous improvement.

For Python developers, DevOps explains how a local program becomes a production service.

It covers CI/CD, containers, configuration, secrets, infrastructure as code, orchestration, monitoring, logs, traces, alerts, SLOs, rollbacks, migrations, scaling, backups, access control, and team culture.

The central lesson is:

```text
production is not where software ends
production is where software proves itself
```

DevOps gives teams the practices needed to let software prove itself safely.

---

# Exercises

1. Choose a Python web application and list everything it needs to run in production.

2. Define local, staging, and production environments for that application.

3. List all configuration values the app needs.

4. Mark which configuration values are secrets.

5. Design a simple CI pipeline for the app.

6. Add linting, tests, dependency installation, and build steps to the pipeline design.

7. Write a Dockerfile outline for the app.

8. Define an image tagging strategy.

9. Design a deployment pipeline from pull request to production.

10. Choose a deployment strategy and explain why it fits the app.

11. Define rollback criteria.

12. Design a safe database migration plan for adding a new column.

13. Define five metrics the app should expose.

14. Write three useful structured log events.

15. Define one actionable alert tied to user impact.

16. Write a runbook outline for high latency.

17. Define backup and restore expectations for the database.

18. Identify three security risks in the CI/CD pipeline.

19. Write a checklist for production readiness.

20. Explain how DevOps differs from simply learning cloud tools.

---

# Preview of Chapter 105

Chapter 104 studied DevOps as a career path.

We learned how Python software moves from local code to production systems through builds, CI/CD, containers, configuration, secrets, infrastructure as code, orchestration, deployment strategies, rollback, migrations, monitoring, logs, traces, alerts, SLOs, incident response, backups, scaling, security, and operational culture.

Next we study Cybersecurity.

Cybersecurity is the discipline of protecting systems, data, users, and organizations from misuse, compromise, and harm.

The transition is direct:

```text
DevOps makes systems run reliably
Cybersecurity makes systems run defensibly
```

Chapter 105 will show how Python developers should understand threats, vulnerabilities, authentication, authorization, encryption, secure coding, dependency risk, secrets, web security, cloud security, incident response, and the security mindset.
