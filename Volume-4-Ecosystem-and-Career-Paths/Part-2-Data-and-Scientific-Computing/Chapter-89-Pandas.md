# Chapter 89 — Pandas

Pandas is the most widely used Python library for working with labeled tabular data.

NumPy taught us to think in arrays.

Pandas teaches us to think in tables with names.

That is the shift.

NumPy is excellent when the data is mostly homogeneous and numerical.

Pandas is excellent when the data looks like real-world records:

* rows
* columns
* labels
* missing values
* dates
* categories
* mixed types
* files
* joins
* grouping
* cleaning
* summaries
* exports

Real datasets are rarely clean arrays.

They arrive as CSV files, Excel workbooks, database query results, JSON exports, logs, reports, spreadsheets, and messy human-maintained tables.

Columns have names.

Rows may have meaningful indexes.

Values may be missing.

Numbers may be stored as strings.

Dates may have inconsistent formats.

Categories may have typos.

Columns may need to be derived from other columns.

Several tables may need to be combined.

Pandas is built for this world.

Its two central data structures are:

* `Series`
* `DataFrame`

A `Series` is a one-dimensional labeled array.

A `DataFrame` is a two-dimensional labeled table.

The word labeled is important.

In NumPy, position is central.

In Pandas, labels are central.

That single difference changes how data analysis feels.

---

# Why Pandas Matters

Pandas matters because most practical data work begins before modeling, plotting, or advanced statistics.

It begins with questions like:

* Where is the data?
* What columns exist?
* Which rows are valid?
* Which values are missing?
* Which fields should be numeric?
* Which fields should be dates?
* Which categories are inconsistent?
* Which rows are duplicates?
* How should tables be joined?
* What should be grouped?
* What summary answers the business question?

These are not glamorous questions.

They are the work.

Pandas is the workbench for that work.

It is common in:

* data analysis
* reporting
* data cleaning
* business analytics
* exploratory data analysis
* machine learning preprocessing
* financial analysis
* experiment analysis
* operations dashboards
* one-off automation scripts
* research notebooks

Pandas belongs after NumPy because it builds on the same computational foundation but adds names, alignment, missing-data tools, and table operations.

The central lesson is:

```text
NumPy gives Python numerical arrays
Pandas gives Python labeled analytical tables
```

---

# Importing Pandas

The standard convention is:

```python
import pandas as pd
```

You will see this everywhere.

Do not fight this convention.

It is short, readable, and shared by the community.

Most examples also import NumPy:

```python
import numpy as np
import pandas as pd
```

NumPy and Pandas often appear together because Pandas uses array concepts internally and interoperates with NumPy.

But your mental model should change when you enter Pandas.

You are no longer only asking:

```text
what is the shape?
```

You also ask:

```text
what are the column names?
what is the index?
what do the labels mean?
are values aligned by position or by label?
```

Those questions are Pandas questions.

---

# Series

A `Series` is a one-dimensional labeled data structure.

Create one from a list:

```python
import pandas as pd

scores = pd.Series([80, 95, 70])
```

The result has values and an index.

If you do not provide an index, Pandas creates a default integer index:

```text
0    80
1    95
2    70
dtype: int64
```

Create a `Series` with explicit labels:

```python
scores = pd.Series(
    [80, 95, 70],
    index=["Anika", "Ravi", "Meera"],
)
```

Now the labels matter:

```python
scores["Ravi"]
```

Result:

```python
95
```

This is why a `Series` feels partly like an array and partly like a dictionary.

It has ordered values.

It also has labels.

The index is not decoration.

The index participates in selection, alignment, grouping, joining, and many other operations.

---

# DataFrame

A `DataFrame` is a two-dimensional labeled table.

Create one from a dictionary:

```python
df = pd.DataFrame({
    "name": ["Anika", "Ravi", "Meera"],
    "score": [80, 95, 70],
    "passed": [True, True, True],
})
```

The columns are:

```python
df.columns
```

The index is:

```python
df.index
```

The shape is:

```python
df.shape
```

Result:

```python
(3, 3)
```

The first number is rows.

The second number is columns.

A DataFrame is not merely a list of lists.

It is a table with column labels, row labels, dtypes, and many methods for analysis.

It is the central Pandas object.

Most Pandas work is DataFrame work.

---

# Inspecting Data

When you receive a DataFrame, inspect it before transforming it.

Useful methods and attributes include:

```python
df.head()
df.tail()
df.shape
df.columns
df.index
df.dtypes
df.info()
df.describe()
```

`head()` shows the first rows.

`tail()` shows the last rows.

`shape` tells you row and column count.

`columns` shows column names.

`dtypes` shows data types.

`info()` summarizes non-null counts and dtypes.

`describe()` computes summary statistics for suitable columns.

This first inspection is not optional in real work.

It prevents avoidable mistakes.

Before you clean, group, join, or model data, know what you have.

Professional data work starts with looking.

---

# Reading Data

Pandas has many reader functions.

Common examples:

```python
pd.read_csv("sales.csv")
pd.read_excel("sales.xlsx")
pd.read_json("events.json")
pd.read_parquet("events.parquet")
pd.read_sql(query, connection)
```

The matching writers are often DataFrame methods:

```python
df.to_csv("clean_sales.csv", index=False)
df.to_excel("report.xlsx", index=False)
df.to_json("events.json")
df.to_parquet("events.parquet")
df.to_sql("sales", connection)
```

CSV is common because it is simple and portable.

But CSV has limitations.

It does not preserve rich type information.

A CSV file does not truly know which columns are dates, categories, nullable integers, or precise decimals.

When reading CSV data, be deliberate:

```python
df = pd.read_csv(
    "sales.csv",
    usecols=["order_id", "date", "region", "amount"],
    parse_dates=["date"],
    dtype={"order_id": "string", "region": "string"},
)
```

This says:

* read only the columns we need
* parse `date` as dates
* preserve IDs as strings
* preserve region as strings

Do not assume Pandas will infer everything correctly.

Inference is convenient.

Explicit types are safer when correctness matters.

---

# Columns

Selecting one column returns a `Series`:

```python
df["score"]
```

Selecting multiple columns returns a `DataFrame`:

```python
df[["name", "score"]]
```

This difference matters.

A `Series` is one-dimensional.

A `DataFrame` is two-dimensional.

You can create a new column:

```python
df["score_percent"] = df["score"] / 100
```

You can derive columns from multiple columns:

```python
df["revenue"] = df["price"] * df["quantity"]
```

Pandas encourages column-wise thinking.

Instead of looping over rows, first ask:

```text
can I express this as an operation on columns?
```

Often, the answer is yes.

That is the Pandas equivalent of NumPy vectorization.

---

# Selecting Rows and Columns

Pandas has several selection styles.

The two most important are:

* `.loc`
* `.iloc`

Use `.loc` for label-based selection.

Use `.iloc` for position-based selection.

Example:

```python
df = pd.DataFrame(
    {
        "city": ["Delhi", "Mumbai", "Bengaluru"],
        "population": [32, 21, 13],
    },
    index=["a", "b", "c"],
)
```

Label-based:

```python
df.loc["b", "city"]
```

Position-based:

```python
df.iloc[1, 0]
```

Both return:

```python
"Mumbai"
```

But the meaning is different.

`.loc["b", "city"]` means:

```text
row label b, column label city
```

`.iloc[1, 0]` means:

```text
row position 1, column position 0
```

Professional Pandas code should be clear about which meaning it wants.

---

# Boolean Filtering

Filtering rows is one of the most common Pandas operations.

```python
df[df["score"] >= 80]
```

This creates a boolean mask:

```python
df["score"] >= 80
```

Then uses it to select rows.

For multiple conditions, use `&`, `|`, and `~`, with parentheses:

```python
high_scores = df[(df["score"] >= 80) & (df["passed"])]
```

Do not use `and` and `or` for element-wise Pandas conditions.

This mirrors NumPy.

The mask should have the same index as the DataFrame.

That index alignment is important.

Pandas aligns data by labels in many operations.

This can protect you.

It can also surprise you if you assume everything is purely positional.

---

# Assignment with loc

When updating selected rows, use `.loc`.

```python
df.loc[df["score"] < 60, "passed"] = False
```

This says:

```text
for rows where score is less than 60, set passed to False
```

Avoid chained assignment:

```python
df["passed"][df["score"] < 60] = False
```

In modern Pandas, Copy-on-Write makes chained assignment the wrong mental model.

The update should happen through the object you intend to update.

Use one `.loc` operation.

That is clearer and safer.

The pattern is:

```python
df.loc[row_condition, column_name] = new_value
```

Learn this early.

It prevents many frustrating bugs.

---

# Copy-on-Write

Pandas 3.0 uses Copy-on-Write as the default behavior.

The goal is to make mutation more predictable.

Older Pandas code often suffered from confusion around whether a selection returned a view or a copy.

Copy-on-Write changes the practical habit:

```text
only the object you directly modify should be modified
```

If an object shares data with another object, Pandas can defer copying until a mutation requires separation.

The important beginner rule is:

```text
do not rely on modifying a selected subset to mutate the original DataFrame
```

Instead of:

```python
subset = df["score"]
subset.iloc[0] = 100
```

write:

```python
df.loc[df.index[0], "score"] = 100
```

Instead of chained assignment:

```python
df["score"][df["score"] < 0] = 0
```

write:

```python
df.loc[df["score"] < 0, "score"] = 0
```

This is not only about avoiding warnings.

It is about expressing intent.

You are updating `df`.

Say so directly.

---

# Data Types

Every column in a DataFrame has a dtype.

Inspect them:

```python
df.dtypes
```

Common dtypes include:

* integer
* floating point
* boolean
* string
* datetime
* category
* object
* nullable integer
* nullable boolean

The dtype affects:

* memory usage
* missing value behavior
* sorting
* comparisons
* arithmetic
* grouping
* joins
* performance

Do not ignore dtypes.

A column that looks numeric may actually be strings:

```python
df["amount"].dtype
```

If it is a string-like dtype, arithmetic will not mean what you expect.

Convert intentionally:

```python
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")
```

For dates:

```python
df["date"] = pd.to_datetime(df["date"], errors="coerce")
```

`errors="coerce"` turns invalid values into missing values.

That is useful, but it should be followed by inspection.

Conversion can hide data quality problems if you never check what became missing.

---

# Missing Data

Missing data is central to Pandas.

Detect missing values:

```python
df.isna()
df["amount"].isna()
```

Count missing values:

```python
df.isna().sum()
```

Drop rows with missing values:

```python
df.dropna()
```

Fill missing values:

```python
df["amount"] = df["amount"].fillna(0)
```

Missing data is not merely a technical inconvenience.

It is a data meaning problem.

Before filling missing values, ask:

* Does missing mean unknown?
* Does missing mean not applicable?
* Does missing mean zero?
* Does missing mean the data failed to load?
* Does missing mean the customer skipped a field?
* Does missing mean the event did not happen?

Filling missing revenue with zero may be correct in one dataset and dangerously wrong in another.

Pandas gives you methods.

You must supply interpretation.

---

# Cleaning Strings

Real data often contains messy text.

Pandas string methods live under `.str`.

Examples:

```python
df["region"] = df["region"].str.strip()
df["region"] = df["region"].str.lower()
df["email_domain"] = df["email"].str.split("@").str[-1]
```

You can check containment:

```python
df[df["email"].str.contains("@", na=False)]
```

Text cleaning often includes:

* stripping whitespace
* normalizing case
* replacing inconsistent labels
* extracting parts
* detecting invalid values
* handling missing strings

Be careful with text cleaning.

Do not normalize away meaningful differences without understanding the domain.

For example, product codes may be case-sensitive.

Names may contain punctuation that matters.

String operations are easy to write.

Data semantics are harder.

---

# Sorting

Sort rows by a column:

```python
df.sort_values("amount")
```

Descending:

```python
df.sort_values("amount", ascending=False)
```

Sort by multiple columns:

```python
df.sort_values(["region", "amount"], ascending=[True, False])
```

Sort by index:

```python
df.sort_index()
```

Sorting is often part of analysis and reporting.

But remember that sorting returns a new DataFrame unless assigned.

```python
df = df.sort_values("amount")
```

Pandas methods often return transformed objects.

Use assignment or method chaining when you want to keep the result.

---

# GroupBy

GroupBy is one of the most powerful Pandas features.

It follows the split-apply-combine pattern:

```text
split data into groups
apply a calculation to each group
combine the results
```

Example:

```python
sales = pd.DataFrame({
    "region": ["North", "South", "North", "South"],
    "amount": [100, 200, 150, 50],
})

sales.groupby("region")["amount"].sum()
```

Result:

```text
region
North    250
South    250
Name: amount, dtype: int64
```

GroupBy lets you answer questions like:

* total sales by region
* average score by class
* number of orders by customer
* maximum temperature by month
* conversion rate by campaign
* median latency by service

The group column defines the categories.

The aggregation defines the summary.

Together they answer an analytical question.

---

# Aggregating Groups

You can compute multiple aggregations:

```python
sales.groupby("region")["amount"].agg(["count", "sum", "mean"])
```

For multiple columns:

```python
sales.groupby("region").agg(
    orders=("amount", "count"),
    total_amount=("amount", "sum"),
    average_amount=("amount", "mean"),
)
```

Named aggregation is often clearer in production code.

The output column names describe what they mean.

Avoid mysterious outputs like:

```text
amount_count
amount_sum
amount_mean
```

when business readers need:

```text
orders
total_amount
average_amount
```

Data analysis is communication.

Name results accordingly.

---

# Transform

Aggregation reduces groups.

Transform returns values aligned to the original rows.

Example:

```python
sales["region_total"] = sales.groupby("region")["amount"].transform("sum")
sales["share_of_region"] = sales["amount"] / sales["region_total"]
```

Now each row knows its group's total.

This is useful when you need row-level data enriched with group-level statistics.

Common transform uses:

* percent of group total
* group mean centered values
* group ranks
* group-level flags
* normalization within category

The difference is:

```text
agg returns one row per group
transform returns one value per original row
```

That distinction is important.

---

# Joining Tables

Real analysis often uses multiple tables.

Pandas provides `merge` for database-style joins.

Example:

```python
orders = pd.DataFrame({
    "order_id": [1, 2, 3],
    "customer_id": [10, 20, 10],
    "amount": [100, 200, 50],
})

customers = pd.DataFrame({
    "customer_id": [10, 20],
    "name": ["Anika", "Ravi"],
})

result = orders.merge(customers, on="customer_id", how="left")
```

Join types include:

* `inner`
* `left`
* `right`
* `outer`
* `cross`

A left join keeps all rows from the left table.

An inner join keeps only matching rows.

An outer join keeps rows from both sides.

Join choice is a business decision, not just a syntax decision.

If a customer is missing, should the order disappear?

Usually not.

That suggests a left join.

Always check row counts before and after joins.

Unexpected row multiplication is one of the most common data mistakes.

---

# Concatenation

Concatenation stacks or combines objects.

Stack rows:

```python
combined = pd.concat([january, february, march], axis=0)
```

Combine columns:

```python
wide = pd.concat([features, labels], axis=1)
```

When stacking rows, indexes may repeat.

Often you want:

```python
combined = pd.concat([january, february, march], ignore_index=True)
```

This creates a new clean integer index.

Use concatenation when objects have the same kind of columns and should be appended.

Use merge when objects should be matched by keys.

These are different operations.

Confusing them leads to wrong datasets.

---

# Reshaping

Data can be wide or long.

Wide data:

```text
city      Jan  Feb  Mar
Delhi     10   12   15
Mumbai    20   18   22
```

Long data:

```text
city    month  value
Delhi   Jan    10
Delhi   Feb    12
Delhi   Mar    15
Mumbai  Jan    20
Mumbai  Feb    18
Mumbai  Mar    22
```

Pandas can reshape between forms.

Use `melt` to go from wide to long:

```python
long = df.melt(
    id_vars="city",
    var_name="month",
    value_name="value",
)
```

Use `pivot` or `pivot_table` to go from long to wide:

```python
wide = long.pivot(index="city", columns="month", values="value")
```

Reshaping is not just presentation.

Some operations are easier in long form.

Some reports are easier in wide form.

Know both.

---

# Pivot Tables

Pivot tables summarize data across categories.

Example:

```python
summary = sales.pivot_table(
    index="region",
    columns="product",
    values="amount",
    aggfunc="sum",
    fill_value=0,
)
```

This asks:

```text
for each region and product, what is the total amount?
```

Pivot tables are useful for:

* reports
* cross-tabulations
* dashboards
* spreadsheet-style summaries
* category comparisons

If there may be multiple rows per index-column pair, use `pivot_table`.

If each pair is unique, `pivot` can work.

The distinction matters because duplicate combinations require aggregation.

---

# Time Series

Pandas has strong time series support.

Convert strings to datetimes:

```python
df["timestamp"] = pd.to_datetime(df["timestamp"])
```

Set a datetime index:

```python
df = df.set_index("timestamp")
```

Resample by day:

```python
daily = df["amount"].resample("D").sum()
```

Extract date parts:

```python
df["month"] = df.index.month
df["weekday"] = df.index.day_name()
```

Time series work includes:

* parsing dates
* handling time zones
* resampling
* rolling windows
* shifting values
* calculating changes over time
* aligning observations by timestamp

Dates are deceptively difficult.

Be explicit about time zones when they matter.

Be careful when aggregating across daylight saving transitions.

Be careful when mixing local time and UTC.

Pandas gives strong tools, but time itself remains tricky.

---

# Rolling Windows

Rolling operations compute statistics over a moving window.

Example:

```python
df["seven_day_average"] = df["amount"].rolling(window=7).mean()
```

This is common for smoothing time series.

Other rolling operations:

```python
rolling.sum()
rolling.min()
rolling.max()
rolling.std()
```

Rolling windows are useful for:

* moving averages
* anomaly detection
* trend smoothing
* finance indicators
* operational metrics

Be clear about what a window means.

Is it seven rows?

Seven days?

Seven business days?

Those are different.

Pandas can support time-aware rolling windows, but you must choose correctly.

---

# Method Chaining

Pandas transformations can be written as a sequence of assignments:

```python
df = pd.read_csv("sales.csv")
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")
df = df.dropna(subset=["amount"])
df["region"] = df["region"].str.strip().str.title()
summary = df.groupby("region")["amount"].sum()
```

They can also be written as a chain:

```python
summary = (
    pd.read_csv("sales.csv")
    .assign(amount=lambda d: pd.to_numeric(d["amount"], errors="coerce"))
    .dropna(subset=["amount"])
    .assign(region=lambda d: d["region"].str.strip().str.title())
    .groupby("region")["amount"]
    .sum()
)
```

Method chaining can be elegant.

It can also become unreadable if overused.

Use chaining when each step reads naturally.

Break the chain when:

* a step needs explanation
* intermediate validation is required
* debugging becomes difficult
* the transformation has multiple conceptual phases

Readable data pipelines are more important than showing off compact syntax.

---

# Apply and Vectorization

Pandas has `apply`.

It is useful.

It is also overused.

Example:

```python
df["label"] = df["score"].apply(lambda x: "high" if x >= 80 else "low")
```

This works.

But many operations are better expressed with vectorized methods:

```python
df["label"] = np.where(df["score"] >= 80, "high", "low")
```

Or:

```python
df.loc[df["score"] >= 80, "label"] = "high"
df.loc[df["score"] < 80, "label"] = "low"
```

`apply` often loops at the Python level.

That can be slow on large data.

Before using `apply`, ask:

```text
is there a built-in vectorized Pandas or NumPy operation?
```

Use `apply` when the logic is genuinely row-wise or custom and the dataset size makes it acceptable.

Do not use it as your first reflex.

---

# Index Alignment

Pandas aligns by labels.

This is powerful and sometimes surprising.

Example:

```python
a = pd.Series([10, 20, 30], index=["x", "y", "z"])
b = pd.Series([1, 2, 3], index=["z", "y", "x"])

a + b
```

The result aligns labels:

```text
x    13
y    22
z    31
dtype: int64
```

It does not simply add position by position.

This protects you when data arrives in different orders.

But if labels do not match, you may get missing values:

```python
a = pd.Series([10, 20], index=["x", "y"])
b = pd.Series([1, 2], index=["y", "z"])

a + b
```

Result:

```text
x     NaN
y    21.0
z     NaN
dtype: float64
```

This is not a bug.

It is alignment.

When results contain unexpected missing values, inspect indexes.

---

# The Index

The index is the row label system.

Sometimes the default integer index is fine.

Sometimes a meaningful index helps.

Set an index:

```python
df = df.set_index("customer_id")
```

Reset an index:

```python
df = df.reset_index()
```

Use an index when it represents how you naturally identify rows.

But do not make every identifier an index automatically.

Columns are often easier for joins, exports, and clear data pipelines.

Indexes are powerful, but they can also hide important data from ordinary column operations.

A practical rule:

```text
use indexes deliberately, not habitually
```

---

# Duplicate Data

Detect duplicate rows:

```python
df.duplicated()
```

Drop duplicate rows:

```python
df.drop_duplicates()
```

Check duplicates by subset:

```python
df.duplicated(subset=["email"])
```

Duplicate handling is a domain question.

Two rows with the same email may be duplicates.

Or they may be two orders from the same customer.

Two identical event rows may indicate duplicate ingestion.

Or they may represent repeated events.

Pandas can detect patterns.

You decide meaning.

Never remove duplicates blindly from important data.

---

# Categorical Data

Categorical columns represent values from a limited set.

Examples:

* region
* plan type
* product category
* status
* priority
* experiment group

Pandas supports categorical dtype:

```python
df["status"] = df["status"].astype("category")
```

Categoricals can reduce memory and make category semantics explicit.

For ordered categories:

```python
from pandas.api.types import CategoricalDtype

priority_type = CategoricalDtype(
    categories=["low", "medium", "high"],
    ordered=True,
)

df["priority"] = df["priority"].astype(priority_type)
```

Now Pandas knows the order.

This is better than relying on alphabetical sorting.

Categorical data is common in analytics.

Treat it as data with structure, not just arbitrary strings.

---

# Working with NumPy

Pandas and NumPy interoperate.

Convert a Series to a NumPy array:

```python
arr = df["amount"].to_numpy()
```

Convert a DataFrame to a NumPy array:

```python
matrix = df[["height", "weight", "age"]].to_numpy()
```

This is common before passing data to numerical libraries or machine learning models.

But remember:

```text
NumPy arrays do not preserve Pandas column labels and index labels
```

Once you convert to NumPy, you lose table metadata.

That may be fine.

But it should be deliberate.

Before converting, choose columns explicitly and preserve any labels you may need later.

---

# Performance and Scale

Pandas is powerful, but it is not infinite.

Performance depends on:

* data size
* dtypes
* memory
* file format
* algorithm choice
* vectorization
* indexes
* joins
* groupby cardinality
* string operations

Good habits:

* read only needed columns with `usecols`
* specify dtypes when useful
* parse dates deliberately
* prefer vectorized operations
* avoid row-by-row loops
* use categorical dtype for repeated labels when appropriate
* use Parquet for efficient columnar storage when possible
* process in chunks for large CSV files
* measure memory usage

For very large datasets, Pandas may not be enough on one machine.

Options include:

* chunked Pandas processing
* Polars
* Dask
* DuckDB
* Spark
* databases
* data warehouses

The professional choice is not always "force Pandas harder."

Choose the tool that matches data size, workflow, and operational needs.

This connects directly to the next chapter on Polars.

---

# A Practical Workflow

A typical Pandas analysis might look like:

```python
import pandas as pd

raw = pd.read_csv(
    "orders.csv",
    usecols=["order_id", "customer_id", "order_date", "region", "amount"],
    parse_dates=["order_date"],
    dtype={
        "order_id": "string",
        "customer_id": "string",
        "region": "string",
    },
)

clean = (
    raw.dropna(subset=["order_id", "customer_id", "amount"])
    .assign(
        region=lambda d: d["region"].str.strip().str.title(),
        amount=lambda d: pd.to_numeric(d["amount"], errors="coerce"),
    )
    .dropna(subset=["amount"])
)

summary = (
    clean.groupby("region")
    .agg(
        orders=("order_id", "count"),
        customers=("customer_id", "nunique"),
        revenue=("amount", "sum"),
        average_order=("amount", "mean"),
    )
    .sort_values("revenue", ascending=False)
)

summary.to_csv("regional_summary.csv")
```

This workflow includes:

* reading only needed columns
* parsing dates
* preserving IDs as strings
* cleaning text
* converting numeric values
* dropping invalid rows deliberately
* grouping
* naming aggregations
* sorting
* exporting results

This is Pandas in real life.

Not just one method.

A sequence of careful data decisions.

---

# Pandas Versus Spreadsheets

Pandas overlaps with spreadsheets.

Both can:

* hold tables
* filter rows
* compute derived columns
* summarize data
* join related information
* export reports

But Pandas has different strengths:

* reproducible scripts
* version-controlled transformations
* larger workflows
* automation
* tests
* integration with Python libraries
* repeatable cleaning steps

Spreadsheets are excellent for human inspection, lightweight editing, and sharing with business users.

Pandas is excellent when the workflow must be repeated, audited, automated, or integrated into a larger system.

A professional data workflow may use both.

For example:

```text
Pandas reads raw files, cleans data, produces summaries
Excel receives a polished report for non-technical stakeholders
```

Tools are not enemies.

Use each where it fits.

---

# Pandas Versus SQL

Pandas also overlaps with SQL.

SQL is excellent for querying relational databases.

Pandas is excellent for in-memory data manipulation in Python.

SQL strengths:

* database-scale filtering
* joins near the data
* indexes
* transactions
* permissions
* shared query infrastructure
* persistent storage

Pandas strengths:

* flexible local analysis
* complex Python transformations
* exploratory workflows
* file-based data cleaning
* integration with plotting and ML libraries

Do not pull a billion database rows into Pandas because you do not want to write a `WHERE` clause.

Filter and aggregate in the database when appropriate.

Use Pandas for the part that benefits from Python-side analysis.

The best data engineers and analysts know both.

---

# Common Mistakes

The first common mistake is ignoring dtypes.

Wrong dtypes create wrong analysis.

The second common mistake is using chained assignment.

Use `.loc` for intentional updates.

The third common mistake is treating missing values as merely technical.

Missingness has meaning.

The fourth common mistake is using `apply` for everything.

Look for vectorized operations first.

The fifth common mistake is joining without checking row counts.

Joins can multiply rows unexpectedly.

The sixth common mistake is dropping duplicates blindly.

Duplicates may be meaningful.

The seventh common mistake is assuming index alignment is positional alignment.

Pandas aligns by labels.

The eighth common mistake is reading entire large files when only a few columns are needed.

Use `usecols`, chunks, or better file formats.

The ninth common mistake is converting IDs to numbers.

IDs are often labels, not quantities.

Preserve them as strings when arithmetic is not meaningful.

The tenth common mistake is doing a long manual notebook analysis without preserving the cleaning logic.

If the work matters, turn it into a reproducible script or pipeline.

---

# Professional Pandas Checklist

When working with Pandas, practice these habits:

* Import Pandas as `pd`.
* Inspect `head`, `shape`, `columns`, `dtypes`, and `info`.
* Read only the columns you need when possible.
* Specify important dtypes explicitly.
* Parse dates deliberately.
* Preserve identifiers as strings.
* Use `.loc` for label-based selection and assignment.
* Use `.iloc` for position-based selection.
* Avoid chained assignment.
* Check missing values before filling or dropping them.
* Use vectorized operations before `apply`.
* Name aggregation outputs clearly.
* Check row counts before and after joins.
* Use `ignore_index=True` when concatenating row batches that need a clean index.
* Use categorical dtype for repeated labels when appropriate.
* Convert to NumPy only when you no longer need labels.
* Use efficient file formats such as Parquet when they fit the workflow.
* Move heavy filtering or joining to SQL when the data lives in a database.
* Consider Polars or distributed tools when data exceeds comfortable Pandas scale.

These habits make Pandas code clearer, safer, and more professional.

---

# Summary

Pandas is Python's primary library for labeled tabular data analysis.

It builds on the array concepts introduced by NumPy but adds indexes, column labels, mixed dtypes, missing-data handling, grouping, joining, reshaping, time series tools, and file I/O.

A `Series` is a one-dimensional labeled array.

A `DataFrame` is a two-dimensional labeled table.

Selection should distinguish label-based `.loc` from position-based `.iloc`.

Column-wise operations are usually preferred over row loops.

Missing values require interpretation, not just method calls.

GroupBy follows split-apply-combine.

Merge joins tables by keys.

Concat combines objects along an axis.

Reshaping changes data between wide and long forms.

Time series tools help parse, index, resample, roll, and analyze time-based data.

Copy-on-Write in Pandas 3.0 makes mutation more predictable and strengthens the habit of updating the intended object directly.

The central lesson is:

```text
Pandas turns raw tables into reproducible analytical workflows
```

Used well, Pandas is not just a library for quick notebooks.

It is a tool for turning messy data into clear, repeatable, explainable results.

---

# Exercises

1. Create a DataFrame with columns `name`, `age`, `city`, and `score`.

2. Inspect its shape, columns, dtypes, first rows, and summary statistics.

3. Select one column as a Series and two columns as a DataFrame.

4. Filter rows where `score` is at least `80`.

5. Use `.loc` to update a column based on a condition.

6. Add a derived column called `score_percent`.

7. Create a DataFrame with missing values and count missing values per column.

8. Fill one missing column and drop rows missing another column.

9. Convert a string date column using `pd.to_datetime`.

10. Convert a numeric-looking string column using `pd.to_numeric`.

11. Group a sales DataFrame by region and compute count, sum, and mean.

12. Use `transform` to compute each row's share of its group total.

13. Merge an orders table with a customers table.

14. Check row counts before and after the merge.

15. Concatenate three monthly DataFrames with `ignore_index=True`.

16. Use `melt` to convert a wide table into long form.

17. Use `pivot_table` to summarize data by two categories.

18. Resample a datetime-indexed DataFrame by day.

19. Replace an `apply` solution with a vectorized solution.

20. Explain when you would choose Pandas, SQL, NumPy, or Polars for a data task.

---

# Preview of Chapter 90

Chapter 89 studied Pandas.

We learned how Python handles labeled tabular data through Series, DataFrames, indexes, dtypes, selection, assignment, missing values, string cleaning, grouping, joins, reshaping, time series operations, Copy-on-Write, and reproducible workflows.

Next we study Polars.

Polars is a newer DataFrame library designed around performance, lazy execution, expression-based queries, and Apache Arrow memory concepts.

The transition is:

```text
Pandas gives Python a mature labeled data analysis workbench
Polars explores a faster expression-oriented DataFrame model
```

Chapter 90 will show why Polars has become important for modern data processing and when it can be a better fit than Pandas.
