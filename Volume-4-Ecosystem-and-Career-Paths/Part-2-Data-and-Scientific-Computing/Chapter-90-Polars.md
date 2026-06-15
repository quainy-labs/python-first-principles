# Chapter 90 — Polars

Polars is a modern DataFrame library for working with structured data.

It solves many of the same problems as Pandas.

But it does not think exactly like Pandas.

That is the first thing to understand.

If you approach Polars as:

```text
Pandas, but faster
```

you will miss much of its design.

Polars is built around:

* a Rust core
* columnar memory
* strict schemas
* expressions
* lazy execution
* query optimization
* parallel execution
* streaming for larger-than-memory workflows
* interoperability with the Arrow ecosystem

Pandas is a mature and flexible data analysis workbench.

Polars is a performance-oriented query engine with a DataFrame interface.

That difference matters.

In Pandas, you often write a sequence of eager transformations.

Each line runs immediately.

In Polars, especially in lazy mode, you can describe a whole query plan first.

Then Polars can optimize that plan before executing it.

This makes Polars feel closer to a database query engine than a traditional Python object manipulation library.

The central shift is:

```text
from immediate table manipulation
to expression-based query planning
```

This chapter explains that shift.

---

# Why Polars Matters

Polars matters because data workloads keep growing.

Many teams work with data that is:

* larger than comfortable Pandas memory
* stored in Parquet or cloud object storage
* processed repeatedly in pipelines
* transformed through many columns
* grouped by high-cardinality keys
* joined across multiple datasets
* used in feature engineering
* prepared for analytics or machine learning

Pandas can handle many of these workloads.

But Pandas often becomes slower or more memory-hungry as data grows.

Polars was designed with performance and query optimization as first-class goals.

It can use multiple CPU cores.

It can avoid reading unused columns.

It can push filters closer to data sources.

It can build lazy query plans.

It can process some workloads in streaming mode.

It can use efficient columnar memory concepts.

This does not mean Polars should replace Pandas everywhere.

Pandas still has a huge ecosystem, mature integrations, and a very familiar workflow for many analysts.

Polars matters because it gives Python users another model:

```text
DataFrame code that behaves more like an optimized query engine
```

That model is important for modern data engineering and performance-sensitive analytics.

---

# Polars After Pandas

The previous chapter studied Pandas.

Pandas introduced:

* Series
* DataFrames
* indexes
* labels
* dtypes
* missing data
* groupby
* joins
* reshaping
* time series
* file I/O
* reproducible data workflows

Polars also has DataFrames and Series.

But the habits are different.

Pandas uses indexes heavily.

Polars does not have a Pandas-style index.

Pandas often encourages direct column selection and assignment.

Polars encourages expressions inside contexts such as `select`, `with_columns`, `filter`, and `group_by`.

Pandas is eager by default.

Polars supports eager execution, but lazy execution is a major strength.

Pandas is very flexible about data types.

Polars is stricter.

These differences are not accidental.

They reflect a different philosophy:

```text
make queries explicit, optimizable, parallel, and predictable
```

If you bring Pandas habits unchanged into Polars, your code may work, but it may not be idiomatic.

The goal is not to translate syntax mechanically.

The goal is to learn the Polars way of expressing data transformations.

---

# Installing and Importing Polars

The standard import convention is:

```python
import polars as pl
```

This mirrors the ecosystem style:

```python
import numpy as np
import pandas as pd
import polars as pl
```

You can install Polars with:

```bash
pip install polars
```

A simple DataFrame:

```python
import polars as pl

df = pl.DataFrame({
    "name": ["Anika", "Ravi", "Meera"],
    "score": [80, 95, 70],
    "passed": [True, True, True],
})
```

Polars displays tables differently from Pandas.

It shows shape, columns, and types clearly.

That is not cosmetic.

Schema awareness is central to Polars.

The schema tells Polars what columns exist and what their types are.

---

# Series and DataFrame

Polars has `Series` and `DataFrame`.

A `Series` is a one-dimensional column:

```python
scores = pl.Series("score", [80, 95, 70])
```

A `DataFrame` is a table:

```python
df = pl.DataFrame({
    "student": ["Anika", "Ravi", "Meera"],
    "score": [80, 95, 70],
})
```

Inspect shape:

```python
df.shape
```

Inspect columns:

```python
df.columns
```

Inspect schema:

```python
df.schema
```

The schema might look like:

```python
Schema({"student": String, "score": Int64})
```

The exact display can vary by Polars version.

The idea is stable:

```text
Polars knows each column's type
```

Polars is stricter than Pandas about schemas.

This strictness may feel less forgiving at first.

But it helps prevent silent data surprises.

---

# No Pandas-Style Index

One major difference from Pandas is that Polars does not use a Pandas-style index.

Rows have positions.

Columns have names.

But there is no hidden row label object that changes query semantics.

This has consequences.

In Pandas, an index can affect:

* selection
* alignment
* joining
* resampling
* assignment
* display

In Polars, you usually keep identifiers as explicit columns.

For example:

```python
df = pl.DataFrame({
    "customer_id": ["C001", "C002", "C003"],
    "amount": [100, 250, 75],
})
```

`customer_id` stays a column.

You group by it, join on it, filter on it, and select it explicitly.

This makes queries easier to reason about because behavior does not depend on hidden index state.

The professional habit in Polars is:

```text
make keys explicit columns
```

This is one reason Polars often feels closer to SQL than Pandas.

---

# Eager Execution

Eager execution means code runs immediately.

Example:

```python
df = pl.DataFrame({
    "region": ["North", "South", "North"],
    "amount": [100, 200, 150],
})

result = df.filter(pl.col("amount") > 100)
```

The filter runs immediately.

`result` is a DataFrame.

Eager execution is good for:

* small data
* quick exploration
* teaching
* interactive work
* simple transformations

If you are learning Polars, eager examples are useful because you can see results after each step.

But Polars becomes especially powerful when you use lazy execution.

Eager mode answers:

```text
do this now
```

Lazy mode answers:

```text
remember this plan, optimize it, then run it
```

---

# Lazy Execution

Lazy execution builds a query plan instead of executing immediately.

```python
query = (
    pl.scan_csv("sales.csv")
    .filter(pl.col("amount") > 100)
    .group_by("region")
    .agg(pl.col("amount").sum().alias("total_amount"))
)
```

At this point, the query may not have read the full file yet.

It has built a plan.

To execute:

```python
result = query.collect()
```

This is a central Polars pattern.

`scan_csv` is lazy.

`read_csv` is eager.

Lazy execution gives Polars room to optimize.

It can ask:

* Which columns are actually needed?
* Can filters be pushed closer to the file scan?
* Can projections remove unused columns early?
* Can operations be reordered safely?
* Can parts of the query run in parallel?
* Can streaming execution be used?

This is why lazy mode is often recommended for serious Polars workflows.

You are not just writing transformations.

You are giving Polars a plan it can improve.

---

# Query Optimization

Query optimization is one of Polars' major strengths.

Suppose a CSV has 100 columns.

Your query uses only:

* `region`
* `amount`

In eager code, you might read the whole file first.

In lazy Polars:

```python
query = (
    pl.scan_csv("sales.csv")
    .filter(pl.col("amount") > 100)
    .group_by("region")
    .agg(pl.col("amount").sum())
)
```

Polars can see that only two columns are required.

It can avoid reading unnecessary columns when possible.

This is called projection pushdown.

If a filter can be applied early, Polars may push it closer to the data source.

This is called predicate pushdown.

These ideas also exist in databases.

That is the point.

Polars is not only a table object.

It is a query engine.

The more complete the plan Polars can see, the more opportunity it has to optimize.

---

# Expressions

Expressions are the heart of Polars.

An expression describes a computation.

It does not necessarily compute immediately by itself.

Example:

```python
pl.col("amount") * 1.18
```

This means:

```text
take the amount column and multiply it by 1.18
```

Use it inside a context:

```python
df.with_columns(
    (pl.col("amount") * 1.18).alias("amount_with_tax")
)
```

The expression describes the new column.

The context decides how it is applied.

Common expression builders:

```python
pl.col("name")
pl.lit(10)
pl.all()
pl.len()
```

Common expression operations:

```python
pl.col("amount").sum()
pl.col("name").str.to_lowercase()
pl.col("date").dt.year()
pl.col("score").mean()
pl.col("value").is_null()
```

Once you understand expressions, Polars becomes much easier.

The expression system is more than syntax.

It is what lets Polars reason about your query.

---

# Contexts

Expressions run inside contexts.

Important contexts include:

* `select`
* `with_columns`
* `filter`
* `group_by(...).agg(...)`

`select` chooses or computes output columns:

```python
df.select(
    "name",
    (pl.col("score") / 100).alias("score_percent"),
)
```

`with_columns` adds or replaces columns:

```python
df.with_columns(
    (pl.col("score") >= 60).alias("passed")
)
```

`filter` keeps rows:

```python
df.filter(pl.col("score") >= 80)
```

`group_by().agg()` summarizes groups:

```python
df.group_by("region").agg(
    pl.col("amount").sum().alias("total_amount")
)
```

The same expression can behave differently depending on context.

This is why Polars documentation often talks about expressions and contexts together.

The professional habit is:

```text
build transformations from expressions inside explicit contexts
```

---

# Selecting Columns

In Pandas, you often write:

```python
df[["name", "score"]]
```

In Polars, prefer:

```python
df.select(["name", "score"])
```

or:

```python
df.select(
    pl.col("name"),
    pl.col("score"),
)
```

You can compute while selecting:

```python
df.select(
    pl.col("name"),
    (pl.col("score") / 100).alias("score_percent"),
)
```

This is not just a different way to write column access.

It keeps the operation inside the expression system.

That makes it more consistent with lazy execution and optimization.

When writing Polars, prefer expression contexts over Pandas-like indexing habits.

---

# Filtering Rows

Use `filter`:

```python
df.filter(pl.col("score") >= 80)
```

Multiple conditions:

```python
df.filter(
    (pl.col("score") >= 80) & (pl.col("region") == "North")
)
```

Use parentheses around conditions.

Use `&` for and.

Use `|` for or.

Use `~` for not.

This part resembles NumPy and Pandas.

But Polars filtering is expression-based.

In lazy mode, filters may be optimized and pushed down.

That means it is usually better to express filters as native Polars expressions rather than Python functions.

Native expressions are visible to the optimizer.

Opaque Python functions are harder to optimize.

---

# Adding and Replacing Columns

Use `with_columns`:

```python
df = df.with_columns(
    (pl.col("price") * pl.col("quantity")).alias("revenue")
)
```

You can add multiple columns:

```python
df = df.with_columns(
    (pl.col("amount") * 1.18).alias("amount_with_tax"),
    (pl.col("amount") > 1000).alias("large_order"),
)
```

If the alias matches an existing column name, Polars replaces that column:

```python
df = df.with_columns(
    pl.col("region").str.strip_chars().str.to_titlecase().alias("region")
)
```

This is the Polars equivalent of many Pandas assignment patterns.

But the form is more declarative.

You describe the columns you want.

Polars decides how to execute the expressions.

---

# Conditional Logic

Use `pl.when(...).then(...).otherwise(...)`.

```python
df = df.with_columns(
    pl.when(pl.col("score") >= 80)
    .then(pl.lit("high"))
    .when(pl.col("score") >= 60)
    .then(pl.lit("medium"))
    .otherwise(pl.lit("low"))
    .alias("score_band")
)
```

This creates a column based on conditions.

The important detail is that string literals used as values should be wrapped with `pl.lit`.

Without `pl.lit`, Polars may interpret a string as a column name in many expression contexts.

This is a common beginner mistake.

Think:

```text
pl.col means column
pl.lit means literal value
```

That distinction is central to expression code.

---

# Grouping and Aggregation

Group data with `group_by`.

```python
sales = pl.DataFrame({
    "region": ["North", "South", "North", "South"],
    "amount": [100, 200, 150, 50],
})

summary = sales.group_by("region").agg(
    pl.col("amount").sum().alias("total_amount"),
    pl.col("amount").mean().alias("average_amount"),
    pl.len().alias("orders"),
)
```

This answers:

```text
for each region, what is total amount, average amount, and order count?
```

Aggregations are expressions too.

That makes them composable.

You can aggregate many columns:

```python
df.group_by("region").agg(
    pl.all().exclude("region").sum()
)
```

You can compute multiple summaries for one column:

```python
df.group_by("region").agg(
    pl.col("amount").min().alias("min_amount"),
    pl.col("amount").max().alias("max_amount"),
    pl.col("amount").sum().alias("total_amount"),
)
```

Good aggregation names matter.

Your output should explain itself.

---

# Window Expressions

Window expressions compute values over groups while keeping row-level output.

This is similar to Pandas `transform`.

Example:

```python
df = df.with_columns(
    pl.col("amount").sum().over("region").alias("region_total")
)
```

Now every row has its region's total.

Then:

```python
df = df.with_columns(
    (pl.col("amount") / pl.col("region_total")).alias("share_of_region")
)
```

This pattern is useful for:

* percent of group total
* ranking within groups
* group means
* group-normalized features
* row-level enrichment from group-level statistics

In Polars, `.over(...)` is a powerful way to express group-aware computations without collapsing rows.

The difference is:

```text
group_by().agg() reduces rows
expression.over(...) preserves rows
```

---

# Joins

Polars supports joins between DataFrames.

Example:

```python
orders = pl.DataFrame({
    "order_id": [1, 2, 3],
    "customer_id": [10, 20, 10],
    "amount": [100, 200, 50],
})

customers = pl.DataFrame({
    "customer_id": [10, 20],
    "name": ["Anika", "Ravi"],
})

result = orders.join(customers, on="customer_id", how="left")
```

Join types include:

* inner
* left
* full
* semi
* anti
* cross
* asof

A semi join keeps rows from the left side that have a match on the right.

An anti join keeps rows from the left side that do not have a match on the right.

An asof join is useful for time-aware nearest-key joins.

As with Pandas, joins are dangerous if you do not check cardinality.

Always ask:

* Is the key unique on the right?
* Is the key unique on the left?
* Should unmatched rows be kept?
* Can the join multiply rows?
* Do duplicate keys have meaning?

Fast DataFrame libraries can produce wrong answers quickly if join logic is wrong.

Speed does not replace data modeling.

---

# Concatenation

Use `pl.concat` to combine DataFrames.

```python
combined = pl.concat([january, february, march])
```

This is common when stacking row batches.

The schemas should be compatible.

Polars is stricter than Pandas about data types.

That is usually good.

If one file has `amount` as integer and another has `amount` as string, you should know that.

Do not casually force incompatible data together.

Clean the schema intentionally.

For production pipelines, schema drift should be treated as a signal.

It may mean the upstream data changed.

It may mean a parsing rule is wrong.

It may mean your assumptions are out of date.

Strictness helps reveal these problems early.

---

# Missing Data

Polars distinguishes null values and NaN values.

Null means missing.

NaN is a special floating-point value.

Check nulls:

```python
df.select(pl.col("amount").is_null().sum())
```

Fill nulls:

```python
df.with_columns(
    pl.col("amount").fill_null(0).alias("amount")
)
```

Drop nulls:

```python
df.drop_nulls()
```

Check NaN for floating-point data:

```python
df.select(pl.col("score").is_nan().sum())
```

Do not treat null and NaN as identical.

They often require different handling.

This distinction matters in numerical work and real-world cleaning.

As with Pandas, the hard part is not calling `fill_null`.

The hard part is deciding what missing means.

---

# String Operations

String expressions live under `.str`.

Examples:

```python
df = df.with_columns(
    pl.col("region").str.strip_chars().str.to_titlecase().alias("region")
)
```

Contains:

```python
df.filter(pl.col("email").str.contains("@"))
```

Split:

```python
df.with_columns(
    pl.col("email").str.split("@").list.last().alias("domain")
)
```

String work is common in data cleaning.

But remember:

* normalize deliberately
* preserve meaningful case when needed
* check missing values
* avoid Python row-wise functions when expression methods exist

Native Polars string expressions are visible to the query engine.

Python lambdas usually are not.

That matters for performance and optimization.

---

# Date and Time

Polars supports date and datetime operations.

Parse strings:

```python
df = df.with_columns(
    pl.col("order_date").str.to_datetime().alias("order_date")
)
```

Extract parts:

```python
df = df.with_columns(
    pl.col("order_date").dt.year().alias("year"),
    pl.col("order_date").dt.month().alias("month"),
)
```

Group by time windows:

```python
daily = (
    df.group_by_dynamic("order_date", every="1d")
    .agg(pl.col("amount").sum().alias("daily_amount"))
)
```

Time-series operations require care.

Always understand:

* timezone assumptions
* timestamp precision
* ordering
* missing timestamps
* window boundaries
* business calendar rules

Polars gives tools.

Time remains a domain problem.

---

# Reading and Writing Files

Polars can read common data formats.

Eager reads:

```python
df = pl.read_csv("sales.csv")
df = pl.read_parquet("sales.parquet")
```

Lazy scans:

```python
query = pl.scan_csv("sales.csv")
query = pl.scan_parquet("sales.parquet")
```

Write output:

```python
df.write_csv("summary.csv")
df.write_parquet("summary.parquet")
```

For small exploration, eager reads are fine.

For larger pipelines, prefer lazy scans when possible.

Lazy scans allow optimization before execution.

Parquet is especially important in modern data workflows because it is columnar.

Columnar storage pairs well with engines that can read only needed columns.

The practical lesson is:

```text
file format affects performance
```

CSV is portable.

Parquet is often better for analytical pipelines.

---

# Streaming

Streaming lets Polars process some queries without requiring the entire result to fit in memory at once.

This is useful for larger-than-memory workloads.

Not every operation can stream efficiently.

Some operations require seeing much or all of the data.

But when streaming applies, it can reduce memory pressure.

The important idea is:

```text
Polars can sometimes process data in chunks through an optimized plan
```

Streaming is not magic.

You still need to understand:

* query shape
* file format
* joins
* grouping cardinality
* sort requirements
* output size
* memory limits

Use streaming as part of a performance strategy, not as a blanket fix.

---

# Strict Schemas

Polars is strict about data types.

That may feel inconvenient when coming from Pandas.

But strictness prevents many bugs.

Suppose one dataset stores `customer_id` as string:

```text
"00123"
```

Another stores it as integer:

```text
123
```

These are not the same business value.

If a library silently converts them, you may lose leading zeros.

Polars often forces you to confront type mismatches.

You can cast explicitly:

```python
df = df.with_columns(
    pl.col("customer_id").cast(pl.String).alias("customer_id")
)
```

Explicit casting says:

```text
I know what type this column should be
```

That is better than silent guessing in production pipelines.

---

# Lazy Versus Eager in Practice

Use eager mode when:

* the data is small
* you are exploring interactively
* you need immediate feedback
* the transformation is simple

Use lazy mode when:

* reading from files
* the dataset is large
* the query has multiple steps
* you want optimization
* you want projection and predicate pushdown
* you are building repeatable pipelines

A common professional Polars pipeline looks like:

```python
summary = (
    pl.scan_parquet("orders.parquet")
    .filter(pl.col("status") == "paid")
    .with_columns(
        (pl.col("price") * pl.col("quantity")).alias("revenue")
    )
    .group_by("region")
    .agg(
        pl.len().alias("orders"),
        pl.col("revenue").sum().alias("revenue"),
        pl.col("customer_id").n_unique().alias("customers"),
    )
    .sort("revenue", descending=True)
    .collect()
)
```

This reads like a query.

That is intentional.

The pipeline describes what should happen.

Polars decides how to execute it efficiently.

---

# Avoiding Python Functions

Polars supports user-defined Python functions in some places.

But you should avoid them when a native expression exists.

Native expression:

```python
df.with_columns(
    (pl.col("amount") * 1.18).alias("amount_with_tax")
)
```

Python function style:

```python
df.with_columns(
    pl.col("amount").map_elements(lambda x: x * 1.18).alias("amount_with_tax")
)
```

The native expression is usually better.

It is visible to the optimizer.

It can be parallelized more effectively.

It avoids Python call overhead.

It communicates intent clearly.

User-defined Python functions are sometimes necessary.

But in Polars, they should not be your first tool.

The professional habit is:

```text
prefer native expressions
```

---

# Polars and SQL Thinking

Polars often feels natural to people who understand SQL.

Compare:

```sql
SELECT region, SUM(amount) AS total_amount
FROM sales
WHERE amount > 100
GROUP BY region
ORDER BY total_amount DESC;
```

Polars:

```python
result = (
    pl.scan_csv("sales.csv")
    .filter(pl.col("amount") > 100)
    .group_by("region")
    .agg(pl.col("amount").sum().alias("total_amount"))
    .sort("total_amount", descending=True)
    .collect()
)
```

Both describe a query.

Both give an engine room to optimize.

Polars also has SQL support, but the expression API is the idiomatic center of the library.

If you know SQL, use that mental model:

* select columns
* filter rows
* derive expressions
* group
* aggregate
* join
* sort
* collect results

But write it in Polars expressions.

---

# Polars Versus Pandas

Choose Pandas when:

* your workflow is small or medium-sized
* the ecosystem integration matters
* your team is already comfortable with Pandas
* you need a mature notebook analysis style
* you rely on Pandas-specific libraries
* index-based behavior is useful

Choose Polars when:

* performance matters
* memory pressure matters
* lazy query optimization helps
* your workflow is expression-oriented
* you process Parquet or large CSV files
* you want stronger schema behavior
* you want parallel execution by default
* you are building repeatable data pipelines

Do not turn this into a tribal choice.

Both libraries are useful.

Many professionals use both.

Pandas may be better for quick exploratory analysis.

Polars may be better for a large transformation pipeline.

The question is:

```text
what does this workload need?
```

That is a better question than:

```text
which library is universally better?
```

---

# Polars Versus NumPy

NumPy is about numerical arrays.

Polars is about structured DataFrames.

NumPy is ideal when:

* data is homogeneous
* operations are numerical
* shape and broadcasting dominate
* labels are not central

Polars is ideal when:

* data is tabular
* columns have names
* columns may have different types
* filtering, grouping, joining, and file I/O dominate

They can work together.

You may use Polars to read and clean structured data, then convert selected columns to NumPy for numerical algorithms:

```python
X = df.select(["height", "weight", "age"]).to_numpy()
```

But conversion loses Polars column semantics.

Convert deliberately.

---

# A Practical Pipeline

Suppose you have order data in Parquet files.

You want:

* paid orders only
* normalized region names
* revenue from price and quantity
* summary by region
* output sorted by revenue

Polars lazy pipeline:

```python
import polars as pl

summary = (
    pl.scan_parquet("orders/*.parquet")
    .filter(pl.col("status") == "paid")
    .with_columns(
        pl.col("region").str.strip_chars().str.to_titlecase().alias("region"),
        (pl.col("price") * pl.col("quantity")).alias("revenue"),
    )
    .group_by("region")
    .agg(
        pl.len().alias("orders"),
        pl.col("customer_id").n_unique().alias("customers"),
        pl.col("revenue").sum().alias("total_revenue"),
        pl.col("revenue").mean().alias("average_order_revenue"),
    )
    .sort("total_revenue", descending=True)
    .collect()
)

summary.write_csv("regional_summary.csv")
```

This is compact.

But it is also expressive.

It shows:

* source
* filter
* cleaning
* derived column
* grouping
* aggregations
* sorting
* execution boundary
* output

The call to `collect()` is the boundary between plan and result.

Before `collect`, Polars can reason about the query.

After `collect`, you have a DataFrame in memory.

---

# Debugging Polars Queries

When a Polars query fails, ask:

* Does the column exist?
* Is the dtype what I think it is?
* Is this expression inside the right context?
* Did I use `pl.col` when I meant a column?
* Did I use `pl.lit` when I meant a literal?
* Is the query lazy or eager?
* Did I call `collect()`?
* Are join key types compatible?
* Are nulls or NaNs involved?
* Is a Python function hiding logic from the optimizer?

Useful checks:

```python
df.schema
df.head()
df.glimpse()
query.collect_schema()
query.explain()
```

The exact available methods and output can vary by version, but the habit is stable:

```text
inspect schema, inspect plan, inspect small results
```

Polars errors are often schema or expression errors.

That is good.

They tend to reveal mismatches early.

---

# Common Mistakes

The first common mistake is writing Polars as if it were Pandas.

Use expressions and contexts.

The second common mistake is ignoring lazy execution.

For serious file-based workflows, lazy mode is often the better default.

The third common mistake is using Python lambdas when native expressions exist.

Native expressions are faster and more optimizable.

The fourth common mistake is forgetting `collect()`.

A lazy query is a plan until you execute it.

The fifth common mistake is confusing `pl.col("x")` and `pl.lit("x")`.

One refers to a column.

The other is a literal value.

The sixth common mistake is expecting a Pandas-style index.

Polars keeps keys as explicit columns.

The seventh common mistake is ignoring schema mismatches.

Strictness is a feature.

The eighth common mistake is assuming streaming can handle every query perfectly.

Streaming depends on the query shape.

The ninth common mistake is joining without checking cardinality.

Fast joins can still produce wrong row counts.

The tenth common mistake is choosing Polars only because it is fashionable.

Use it when its strengths match the workload.

---

# Professional Polars Checklist

When using Polars, practice these habits:

* Import Polars as `pl`.
* Keep identifiers as explicit columns.
* Inspect schemas early.
* Prefer `select`, `with_columns`, `filter`, and `group_by().agg()`.
* Use expressions instead of row-wise Python functions.
* Use `pl.col` for columns.
* Use `pl.lit` for literal values when needed.
* Prefer lazy scans for file-based pipelines.
* Call `collect()` deliberately.
* Use Parquet when columnar analytical storage fits.
* Check join cardinality.
* Cast types explicitly when schemas differ.
* Treat null and NaN separately.
* Use window expressions with `.over(...)` for row-preserving group computations.
* Inspect query plans when performance matters.
* Consider streaming for larger-than-memory workloads, but verify support for your query.
* Choose Pandas when its ecosystem or interactive ergonomics are the better fit.

These habits help you write Polars code that is not merely correct, but idiomatic.

---

# Summary

Polars is a modern DataFrame library designed for fast structured data processing.

It is written with a Rust core and focuses on performance, parallelism, strict schemas, expressions, lazy execution, query optimization, and efficient columnar workflows.

It overlaps with Pandas, but it is not just Pandas with faster syntax.

Polars does not use a Pandas-style index.

Keys should usually remain explicit columns.

The expression API is central.

Expressions are used inside contexts such as `select`, `with_columns`, `filter`, and `group_by().agg()`.

Lazy execution allows Polars to build and optimize a query plan before running it.

This enables optimizations such as reading only required columns and pushing filters closer to data sources when possible.

Polars supports joins, aggregation, window expressions, string operations, date operations, CSV and Parquet I/O, streaming, and interoperability with other data tools.

Its strictness can feel unfamiliar after Pandas, but it helps reveal schema problems early.

The central lesson is:

```text
Polars treats DataFrame work as an optimizable expression query
```

That makes it especially valuable for performance-sensitive data pipelines and larger analytical workloads.

---

# Exercises

1. Create a Polars DataFrame with columns `name`, `score`, and `region`.

2. Inspect its shape, columns, and schema.

3. Use `select` to return only two columns.

4. Use `with_columns` to create a `score_percent` column.

5. Use `filter` to keep rows where score is at least `80`.

6. Create a conditional `score_band` column using `pl.when`.

7. Group by `region` and compute count, mean score, and maximum score.

8. Use a window expression with `.over("region")` to add each row's regional average score.

9. Join an orders DataFrame with a customers DataFrame.

10. Use `pl.concat` to combine multiple DataFrames with the same schema.

11. Create a column with null values and fill the nulls.

12. Create a floating-point column with NaN values and detect them.

13. Clean a string column using `.str.strip_chars()` and `.str.to_titlecase()`.

14. Parse a date string column and extract the year and month.

15. Write an eager CSV read with `pl.read_csv`.

16. Write a lazy CSV query with `pl.scan_csv` and `collect`.

17. Explain why lazy mode can read fewer columns than an eager full read.

18. Replace a Python lambda transformation with a native Polars expression.

19. Explain the difference between `pl.col("status")` and `pl.lit("status")`.

20. Compare when you would choose Pandas and when you would choose Polars.

---

# Preview of Chapter 91

Chapter 90 studied Polars.

We learned how Polars approaches DataFrame work through expressions, contexts, strict schemas, lazy execution, query optimization, parallelism, joins, aggregation, window functions, file I/O, missing data, and streaming.

Next we study SciPy.

SciPy builds on NumPy to provide algorithms for scientific and technical computing.

Where NumPy gives arrays and foundational numerical operations, SciPy adds higher-level tools for optimization, interpolation, integration, signal processing, statistics, sparse matrices, spatial algorithms, and more.

The transition is:

```text
Polars helps process structured analytical data efficiently
SciPy helps solve scientific and mathematical computing problems
```

Chapter 91 will show how SciPy extends Python from array manipulation into a broad scientific computing toolkit.
