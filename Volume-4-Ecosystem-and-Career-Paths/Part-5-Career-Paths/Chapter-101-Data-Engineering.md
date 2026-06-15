# Chapter 101 — Data Engineering

Data engineering is the discipline of building systems that move, transform, validate, store, and serve data.

Backend engineering focuses on application behavior.

Data engineering focuses on data reliability.

That distinction matters.

A backend engineer may ask:

```text
Does this API correctly create an order?
```

A data engineer may ask:

```text
Can every order be trusted in the warehouse tomorrow morning?
```

Data engineering is about making data usable at scale.

It supports:

* analytics
* reporting
* machine learning
* experimentation
* business intelligence
* finance operations
* product decisions
* compliance
* customer insights
* operational dashboards
* AI systems

Python is widely used in data engineering because it is strong at automation, APIs, files, data formats, orchestration, validation, and glue code.

But data engineering is not only Python scripts.

It includes warehouses, lakes, pipelines, schedulers, quality checks, schemas, lineage, monitoring, ownership, and business meaning.

The central question is:

```text
can people and systems trust this data?
```

---

# Why Data Engineering Matters

Organizations make decisions from data.

They ask questions like:

* How much revenue did we make yesterday?
* Which customers are churning?
* Which features are used most?
* Which campaigns are profitable?
* Which orders failed?
* Which models need retraining?
* Which users should receive alerts?
* Which compliance reports are due?

If the data is late, wrong, incomplete, duplicated, stale, or misunderstood, decisions suffer.

A dashboard can look polished and still be wrong.

A machine learning model can be sophisticated and still train on bad data.

A finance report can be automated and still double-count revenue.

Data engineering matters because data quality is infrastructure.

Bad data creates invisible damage.

Good data creates leverage.

The professional data engineer does not only move bytes.

They protect meaning.

---

# Data Engineering Versus Data Science

Data engineering and data science overlap, but they are different.

Data science often asks:

```text
what can we learn from this data?
```

Data engineering often asks:

```text
how do we make this data available, reliable, and understandable?
```

Data scientists may build models, run experiments, and analyze patterns.

Data engineers build the pipelines and platforms that make that work possible.

Data engineers care about:

* ingestion
* transformation
* storage
* orchestration
* quality
* lineage
* performance
* cost
* governance
* reliability

A strong data scientist can be blocked by weak data infrastructure.

A strong data engineer makes many analysts, scientists, and product teams more effective.

This is why data engineering is a high-leverage career path.

---

# The Data Lifecycle

Data moves through a lifecycle.

A simplified lifecycle:

```text
source -> ingest -> store -> transform -> validate -> serve -> monitor
```

Sources may include:

* application databases
* event streams
* APIs
* files
* logs
* third-party platforms
* sensors
* spreadsheets
* SaaS tools

Ingestion brings data into the platform.

Storage keeps raw and processed data.

Transformation turns raw data into usable models.

Validation checks quality.

Serving exposes data to users or systems.

Monitoring watches freshness, failures, volume, and quality.

Every stage can fail.

Data engineering is about designing the lifecycle so failures are visible, recoverable, and limited.

---

# Batch Processing

Batch processing handles data in chunks at scheduled intervals.

Examples:

* hourly order sync
* nightly warehouse load
* daily revenue report
* weekly customer segmentation
* monthly compliance export

Batch processing is common because many business questions do not need second-by-second freshness.

A daily report can be enough.

Batch pipelines usually have:

* schedule
* input source
* transformation
* output destination
* quality checks
* logs
* retries
* alerts

Batch processing can be simple.

It can also become complex when data volume grows, dependencies multiply, and backfills are needed.

The key batch question is:

```text
what data interval does this run represent?
```

If you cannot answer that, reruns and backfills become dangerous.

---

# Streaming

Streaming processes data continuously or near real time.

Examples:

* clickstream events
* fraud detection
* IoT sensor data
* log processing
* real-time recommendations
* monitoring alerts

Streaming is useful when latency matters.

But streaming is harder than batch.

It introduces questions:

* What happens when events arrive late?
* What happens when events arrive out of order?
* How are duplicates handled?
* What is the event-time window?
* How is state stored?
* How are failures replayed?
* How are schema changes handled?

Tools may include Kafka, Flink, Spark Structured Streaming, cloud streaming services, and queue systems.

Do not choose streaming only because it sounds modern.

If daily batch is enough, daily batch is often simpler and more reliable.

Use streaming when the business needs streaming.

---

# ETL and ELT

ETL means:

```text
Extract -> Transform -> Load
```

ELT means:

```text
Extract -> Load -> Transform
```

In ETL, data is transformed before it enters the destination store.

In ELT, raw data is loaded first, then transformed inside the warehouse or lakehouse.

Modern cloud data stacks often use ELT.

Reasons:

* warehouses are powerful
* raw data can be preserved
* transformations can be versioned
* SQL can express business logic
* analysts can contribute
* lineage can be tracked

ETL is still useful when:

* data must be cleaned before storage
* source data is too large
* privacy rules require filtering early
* destination cannot handle raw shape
* transformations need specialized compute

The professional question is:

```text
where should transformation happen, and why?
```

There is no universal answer.

---

# Ingestion

Ingestion brings data from source systems into the data platform.

Sources include:

* production databases
* SaaS APIs
* event streams
* cloud storage
* logs
* partner files
* internal services

Ingestion must handle:

* authentication
* pagination
* rate limits
* incremental loading
* full refreshes
* schema changes
* retries
* idempotency
* late data
* duplicates
* deletes

A naive ingestion job says:

```text
fetch everything every night
```

That may work at small scale.

At larger scale, you need incremental strategies:

* updated timestamp
* monotonically increasing ID
* change data capture
* event logs
* source-provided cursors

Ingestion is not only extraction.

It is keeping source history trustworthy.

---

# Change Data Capture

Change Data Capture is often called CDC.

CDC captures changes from a database or source system.

Instead of copying full tables repeatedly, CDC records inserts, updates, and deletes.

CDC is useful when:

* tables are large
* freshness matters
* full reloads are expensive
* deletes must be represented
* downstream systems need change history

CDC introduces complexity:

* ordering
* exactly-once expectations
* duplicate events
* schema evolution
* source database impact
* replay
* tombstones for deletes

CDC is powerful.

It should be treated as infrastructure.

If CDC silently breaks, downstream data may look fresh while missing changes.

That is dangerous.

---

# Storage Layers

Data platforms often separate storage into layers.

Common pattern:

```text
raw -> staging -> intermediate -> marts
```

Raw data preserves source-like records.

Staging cleans and standardizes source data.

Intermediate models combine or reshape data.

Marts serve business use cases such as finance, product, sales, or marketing.

This layering helps:

* debugging
* lineage
* ownership
* reuse
* testing
* business definitions

Avoid jumping directly from raw data to many dashboards.

That creates duplicated logic and inconsistent definitions.

Good data engineering creates trusted shared models.

The goal is not only storage.

The goal is reusable meaning.

---

# Warehouses and Lakehouses

Data warehouses are optimized for analytical queries.

Examples include:

* Snowflake
* BigQuery
* Redshift
* Databricks SQL
* cloud warehouse services

Data lakes store large volumes of files, often in object storage.

Lakehouses combine lake storage with table formats, metadata, transactions, and warehouse-like behavior.

Important table/file formats include:

* Parquet
* Delta Lake
* Apache Iceberg
* Apache Hudi

The storage choice affects:

* cost
* query performance
* governance
* concurrency
* schema evolution
* streaming support
* tool ecosystem

Data engineers do not need to memorize every vendor.

They do need to understand tradeoffs.

Where data lives shapes how it can be used.

---

# Data Formats

Data engineering uses many formats.

Common formats:

* CSV
* JSON
* Avro
* Parquet
* ORC
* XML

CSV is simple and portable.

It is weak for schema, types, compression, and large analytical workloads.

JSON is flexible and common for APIs.

It can become messy for deeply nested or inconsistent records.

Parquet is columnar and efficient for analytics.

It supports typed columns, compression, and reading selected columns.

Avro is common in streaming and schema-oriented systems.

The format is not a detail.

It affects:

* storage cost
* read performance
* schema handling
* compression
* interoperability

Choose formats based on workflow, not habit.

---

# Transformations

Transformations turn raw data into usable data.

Examples:

* rename columns
* cast types
* remove duplicates
* join tables
* derive metrics
* aggregate events
* normalize categories
* calculate revenue
* build customer dimensions
* create fact tables

Transformation is where business meaning enters.

Example:

```text
revenue = paid order amount minus refunds, excluding test orders
```

That definition is not obvious from raw data.

It must be encoded somewhere.

Data engineers often work with analytics engineers, analysts, finance teams, and product teams to define such logic.

The code matters.

The definition matters more.

Wrong business logic creates wrong data products.

---

# SQL in Data Engineering

SQL is central to data engineering.

Python is important.

But SQL is unavoidable in most data engineering roles.

Data engineers use SQL for:

* selecting data
* joining tables
* aggregating metrics
* window functions
* transformations
* validation checks
* debugging
* performance analysis

Strong SQL skills include:

* joins
* grouping
* window functions
* common table expressions
* indexes or partitioning concepts
* query plans
* null semantics
* time handling
* incremental logic

Python and SQL complement each other.

Python orchestrates, integrates, validates, and handles complex workflows.

SQL expresses transformations close to analytical storage.

Data engineers should be comfortable in both.

---

# dbt

dbt is a major tool in analytics engineering and data transformation.

It lets teams write SQL models and manage transformations with software engineering practices.

dbt projects can include:

* models
* sources
* tests
* documentation
* macros
* snapshots
* exposures
* metrics or semantic definitions
* lineage

The key idea is:

```text
transform warehouse data through versioned, tested, documented SQL models
```

dbt is important because it brings practices from software engineering into analytics:

* version control
* modularity
* code review
* CI/CD
* documentation
* testing
* dependency graphs

Not every data engineering job uses dbt.

But the ideas matter widely.

Data transformations should be modular, tested, reviewed, and documented.

---

# Orchestration

Orchestration coordinates pipeline tasks.

It answers:

* what runs?
* when does it run?
* what depends on what?
* what happens when something fails?
* how are retries handled?
* where are logs?
* how are backfills run?

Apache Airflow is a major orchestration tool.

Airflow represents workflows as DAGs.

DAG means Directed Acyclic Graph.

A DAG defines tasks and dependencies.

Example:

```text
extract_orders -> transform_orders -> validate_orders -> publish_mart
```

The value of orchestration is not only scheduling.

It is visibility and control.

Without orchestration, pipelines become scattered cron jobs.

Scattered cron jobs become hard to debug.

---

# DAG Thinking

DAG thinking is important even if you do not use Airflow.

A pipeline has dependencies.

Some tasks can run in parallel.

Some tasks must wait.

Some failures should block downstream tasks.

Some failures can be skipped.

Example:

```text
load_customers -> build_customer_dimension
load_orders    -> build_order_facts
build_customer_dimension + build_order_facts -> build_revenue_mart
```

This graph makes dependencies visible.

When the revenue mart is wrong, you can trace upstream.

When a source is late, you know what it blocks.

DAGs turn pipeline logic into an inspectable structure.

Data engineers should learn to see workflows as graphs.

---

# Data Quality

Data quality means data is fit for use.

Quality dimensions include:

* completeness
* freshness
* uniqueness
* validity
* consistency
* accuracy
* timeliness
* integrity

Example checks:

* primary key is unique
* required field is not null
* order amount is nonnegative
* event timestamp is within expected range
* customer ID exists in customer table
* daily row count is within expected bounds
* dashboard table was updated today

Data quality checks should run automatically.

If a pipeline produces bad data, it should fail or warn before users make decisions.

A pipeline that always succeeds while producing wrong data is not reliable.

It is quietly dangerous.

---

# Freshness

Freshness measures how current data is.

Examples:

```text
orders table loaded through 2026-06-15 08:00
events table is 12 minutes behind
finance report last updated yesterday
```

Freshness expectations depend on use case.

A fraud system may need seconds.

A marketing report may need daily data.

A monthly finance close may need stable, audited data more than real-time freshness.

Data engineers should define freshness contracts.

Ask:

* How fresh does this data need to be?
* Who is affected if it is late?
* How will we detect lateness?
* Who gets alerted?
* Is late data better than wrong data?

Freshness is not a universal race to real time.

It is a product requirement.

---

# Lineage

Lineage shows where data came from and how it changed.

It answers:

```text
which sources feed this table?
which transformations created this metric?
which dashboards depend on this model?
what breaks if this column changes?
```

Lineage is useful for:

* debugging
* impact analysis
* documentation
* governance
* migrations
* trust

Without lineage, data platforms become mysterious.

A column appears in a dashboard.

Nobody knows who owns it.

Nobody knows how it is calculated.

Nobody knows what will break if it changes.

Lineage turns hidden dependencies into visible structure.

Tools can help.

But the team still needs discipline around naming, ownership, and documentation.

---

# Backfills

A backfill reruns a pipeline for past data.

Examples:

* recompute revenue for the last 12 months
* reload events after a bug fix
* rebuild a customer table after schema change
* fill a new column for historical records

Backfills are common and risky.

Questions:

* Is the pipeline deterministic?
* Can it run for old dates?
* Will it overwrite current data?
* Can it be paused?
* How much will it cost?
* Will downstream dashboards change?
* Should users be notified?

Design pipelines with backfills in mind.

A pipeline that only works for today is fragile.

Time is part of data engineering.

---

# Idempotency in Data Pipelines

A data pipeline should often be idempotent.

Running it twice for the same interval should not duplicate results.

Bad pattern:

```text
append all of yesterday's orders every time the job runs
```

If the job reruns, duplicates appear.

Better patterns:

* delete and replace partition
* merge on primary key
* upsert records
* use deterministic output paths
* track loaded batch IDs

Idempotency supports:

* retries
* backfills
* recovery
* confidence

In data engineering, reruns are normal.

Pipelines should expect them.

---

# Schema Evolution

Schemas change.

Sources add columns.

Sources remove columns.

Types change.

Enums gain new values.

Nested structures shift.

Schema evolution can break pipelines.

Good practices:

* validate expected columns
* tolerate additive changes when safe
* alert on breaking changes
* version contracts
* document source ownership
* test transformations
* use schema registries where appropriate

Not every schema change is bad.

But silent schema changes are dangerous.

The data platform should detect important changes before users discover broken dashboards.

---

# Data Contracts

A data contract defines expectations between producers and consumers.

It may specify:

* fields
* types
* meanings
* freshness
* nullability
* ownership
* allowed values
* change process

Example:

```text
orders.created_at must be UTC timestamp
orders.order_id must be unique
orders.amount_cents must be nonnegative integer
orders.status must be one of pending, paid, refunded, canceled
```

Data contracts help prevent upstream changes from breaking downstream users.

They also create accountability.

Without contracts, data consumers depend on guesses.

Data engineering maturity often means turning guesses into contracts.

---

# Observability

Data observability means understanding pipeline and data health.

Useful signals:

* pipeline success or failure
* run duration
* row counts
* freshness
* null rates
* duplicate counts
* schema changes
* distribution shifts
* cost
* queue lag
* source availability

Alerts should be meaningful.

Bad alert:

```text
pipeline failed
```

Better alert:

```text
daily_orders_mart failed validation: order_id uniqueness check failed; 1,238 duplicate keys found; downstream dashboards affected: revenue_daily, finance_close
```

Data observability combines pipeline health and data health.

A pipeline can succeed while data is wrong.

Observe both.

---

# Governance

Data governance means managing data responsibly.

Concerns include:

* ownership
* access control
* privacy
* retention
* lineage
* documentation
* quality
* compliance
* sensitive data handling
* auditability

Data engineers often work with security, legal, compliance, and business teams.

Governance is not only bureaucracy.

It prevents harm.

It answers:

* Who can see this data?
* How long should it be kept?
* Is it personal data?
* Can it be used for machine learning?
* Who owns the definition?
* What happens when a user requests deletion?

Data platforms without governance become data swamps.

Useful data requires responsibility.

---

# Python in Data Engineering

Python appears throughout data engineering.

Common uses:

* API ingestion scripts
* file parsing
* orchestration code
* validation checks
* custom transformations
* data quality tests
* command-line tools
* Spark jobs
* Airflow DAGs
* warehouse utilities
* metadata automation
* report generation
* ML data preparation

Useful Python libraries include:

* pandas
* polars
* pyarrow
* requests
* httpx
* sqlalchemy
* pydantic
* great expectations or similar validation tools
* airflow-related packages
* cloud SDKs

Python's value is not only data manipulation.

It is orchestration and integration.

Python can talk to APIs, databases, files, clouds, queues, and command-line tools.

That makes it a powerful data engineering glue language.

---

# Data Engineering Skills

A data engineer should develop skill in several areas.

Programming:

* Python
* SQL
* testing
* packaging
* command-line tools

Data systems:

* warehouses
* lakes
* file formats
* partitions
* schemas
* indexes or clustering

Pipelines:

* orchestration
* scheduling
* retries
* backfills
* idempotency
* incremental loads

Quality:

* validation
* freshness
* uniqueness
* reconciliation
* contracts

Operations:

* monitoring
* alerts
* cost
* incident response
* documentation

Business:

* metric definitions
* stakeholder communication
* ownership
* trust

Data engineering is technical and social.

Data meaning is negotiated with people.

Data reliability is enforced with systems.

---

# Growing as a Data Engineer

Early data engineering work may involve writing ingestion scripts or SQL transformations.

Then responsibility grows.

You begin to own:

* pipeline reliability
* data quality
* warehouse design
* orchestration patterns
* cost control
* data contracts
* lineage
* platform conventions
* incident response
* stakeholder trust

The growth path is from:

```text
can move data
```

to:

```text
can make data trustworthy
```

Senior data engineers think about systems.

They ask:

* What happens when this source changes?
* Can this be backfilled?
* Who owns this metric?
* How will we detect bad data?
* What downstream users depend on this?
* What is the cost of this pipeline?
* Can this be operated by someone else?

That is the difference between writing pipelines and owning data platforms.

---

# Common Mistakes

The first common mistake is treating data movement as the whole job.

Moved data is not necessarily trusted data.

The second common mistake is ignoring idempotency.

Reruns should not duplicate records.

The third common mistake is building pipelines that cannot backfill.

History matters.

The fourth common mistake is skipping data quality checks.

A successful run can still produce bad data.

The fifth common mistake is ignoring schema changes.

Source systems evolve.

The sixth common mistake is creating many duplicate metric definitions.

Business definitions need ownership.

The seventh common mistake is overusing streaming when batch would work.

Real time adds complexity.

The eighth common mistake is not monitoring freshness.

Late data can be as harmful as failed data.

The ninth common mistake is hiding transformations in notebooks or one-off scripts.

Production logic should be versioned and reviewable.

The tenth common mistake is ignoring cost.

Data platforms can become expensive quietly.

---

# Data Engineering Checklist

For a data pipeline, ask:

* What source does it read?
* Who owns the source?
* Is the load full, incremental, or CDC?
* What interval does each run represent?
* Is the pipeline idempotent?
* Can it backfill?
* What schema is expected?
* What happens when the schema changes?
* What quality checks run?
* What freshness is required?
* What downstream tables or dashboards depend on it?
* What happens when it fails?
* Who is alerted?
* Are logs and metrics useful?
* Is sensitive data protected?
* Are costs understood?
* Is lineage visible?
* Is documentation available?
* Is ownership clear?

This checklist turns a pipeline into a data product.

---

# Summary

Data engineering is the work of making data available, reliable, understandable, and usable.

Python is important in data engineering because it is excellent for ingestion, automation, validation, orchestration, file handling, APIs, and custom workflows.

Data engineers work with batch processing, streaming, ETL, ELT, ingestion, CDC, storage layers, warehouses, lakehouses, data formats, transformations, SQL, dbt, orchestration, Airflow, data quality, freshness, lineage, backfills, idempotency, schema evolution, data contracts, observability, and governance.

The role is not only technical.

It also requires communication with stakeholders about definitions, trust, ownership, and impact.

The central lesson is:

```text
data engineering is production ownership of data reliability
```

If people use data to make decisions or power systems, someone must ensure that data can be trusted.

That someone is often the data engineer.

---

# Exercises

1. Design a daily batch pipeline that loads orders from an API into a warehouse.

2. Define whether the pipeline is full refresh, incremental, or CDC.

3. Identify the run interval for each pipeline execution.

4. Make the pipeline idempotent.

5. Design a backfill strategy for the last six months.

6. List required fields and types for the orders data contract.

7. Add freshness expectations for the orders table.

8. Add data quality checks for uniqueness, nulls, and nonnegative amounts.

9. Design a raw, staging, intermediate, and mart layer for ecommerce data.

10. Write a SQL transformation that calculates daily revenue.

11. Identify how refunds should affect the revenue definition.

12. Design an Airflow DAG for extract, load, transform, validate, and publish steps.

13. Identify what should happen if validation fails.

14. Choose whether a use case needs batch or streaming and explain why.

15. Compare CSV, JSON, and Parquet for an analytical workload.

16. Define lineage for a dashboard metric.

17. Design an alert for stale data.

18. Identify sensitive fields and propose access controls.

19. Estimate cost drivers for a large warehouse transformation.

20. Write a one-page data product contract for a customer dimension table.

---

# Preview of Chapter 102

Chapter 101 studied Data Engineering.

We learned how Python supports data pipelines through ingestion, batch processing, streaming, ETL, ELT, CDC, storage layers, warehouses, file formats, transformations, SQL, dbt, orchestration, Airflow, data quality, freshness, lineage, backfills, idempotency, schema evolution, data contracts, observability, governance, and production ownership of data reliability.

Next we study Machine Learning Engineering.

Machine Learning Engineering focuses on turning models into reliable systems that can be trained, evaluated, deployed, monitored, and improved over time.

The transition is:

```text
data engineering makes data reliable
machine learning engineering makes model behavior reliable
```

Chapter 102 will show how Python fits into feature pipelines, training systems, experiment tracking, model registries, deployment, monitoring, drift detection, and retraining workflows.
