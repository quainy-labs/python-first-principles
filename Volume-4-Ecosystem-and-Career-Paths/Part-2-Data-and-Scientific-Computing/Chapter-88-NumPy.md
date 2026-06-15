# Chapter 88 — NumPy

NumPy is the foundation of scientific computing in Python.

It gives Python an efficient multidimensional array object and a large collection of functions for working with numerical data.

That sentence sounds simple.

It is not.

NumPy changed what Python could be used for.

Plain Python is excellent for expressing ideas.

It is readable, flexible, dynamic, and friendly.

But plain Python lists are not ideal for large numerical computation.

If you have a few numbers, a list is fine.

If you have millions of numbers and need to add, multiply, filter, reshape, aggregate, transform, simulate, or feed them into a machine learning algorithm, the story changes.

NumPy exists because numerical computing needs a different kind of container.

The core container is called `ndarray`.

The name means:

```text
N-dimensional array
```

An array can be:

* one-dimensional
* two-dimensional
* three-dimensional
* higher-dimensional

It can represent:

* a vector of numbers
* a table of values
* an image
* a time series
* a batch of examples
* a matrix
* a tensor-like block of data
* simulation state
* model parameters

NumPy matters because many other Python libraries build on its ideas.

Pandas uses NumPy heavily.

SciPy builds scientific algorithms on top of NumPy arrays.

scikit-learn accepts NumPy-style data.

PyTorch and TensorFlow use tensor concepts that feel familiar after NumPy.

Even when a library does not directly expose NumPy arrays everywhere, the mental model of shape, dtype, axis, vectorization, broadcasting, and memory layout keeps reappearing.

This chapter is not only about syntax.

It is about learning to think in arrays.

---

# Part II Opening

Part II of Volume IV is about data and scientific computing.

The previous part focused on web frameworks.

There, Python communicated with users, browsers, clients, servers, APIs, databases, and deployed systems.

Now the focus changes.

We move from:

```text
request-response systems
```

to:

```text
structured data and computation
```

This is a major part of Python's success.

Python became popular in science, analytics, engineering, finance, machine learning, automation, and research because it combined two strengths:

* a high-level language humans can work with comfortably
* fast lower-level libraries that perform serious computation efficiently

NumPy is one of the most important examples of that combination.

You write Python code.

But many array operations are executed by optimized native code underneath.

That means you can express large numerical operations clearly without writing explicit Python loops for every element.

This is the central shift:

```text
from looping over individual values
to describing operations over whole arrays
```

That shift takes practice.

Once it clicks, a large part of the Python data ecosystem becomes easier to understand.

---

# Why Plain Lists Are Not Enough

Python lists are general-purpose containers.

They can store anything:

```python
values = [1, "hello", 3.14, None, {"x": 10}]
```

This flexibility is useful.

But numerical computing usually has different needs.

If you are processing a million floating-point values, you usually do not want a container that can hold arbitrary objects.

You want:

* same-type values
* compact memory
* fast element-wise operations
* efficient slicing
* predictable shape
* numerical functions
* vectorized computation

Consider two Python lists:

```python
a = [1, 2, 3]
b = [10, 20, 30]
```

If you write:

```python
a + b
```

Python concatenates the lists:

```python
[1, 2, 3, 10, 20, 30]
```

That is correct list behavior.

But for numerical computing, you often want element-wise addition:

```text
[1 + 10, 2 + 20, 3 + 30]
```

With NumPy:

```python
import numpy as np

a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

result = a + b
```

The result is:

```python
array([11, 22, 33])
```

That is the NumPy mindset.

Operations apply to whole arrays.

You describe the computation.

NumPy handles the element-wise work efficiently.

---

# Importing NumPy

The standard convention is:

```python
import numpy as np
```

This convention is everywhere.

You will see `np.array`, `np.zeros`, `np.arange`, `np.mean`, `np.linalg`, and many other names.

Do not invent your own alias in normal code.

This is technically valid:

```python
import numpy as banana
```

But it makes your code harder for other Python programmers to read.

The convention `np` is short, clear, and shared across the ecosystem.

Good code is not only code the computer accepts.

Good code is code other humans can understand quickly.

---

# Creating Arrays

The most direct way to create an array is with `np.array`.

```python
import numpy as np

numbers = np.array([1, 2, 3, 4])
```

This creates a one-dimensional array.

You can inspect it:

```python
numbers
```

The display usually looks like:

```python
array([1, 2, 3, 4])
```

A two-dimensional array can be created from nested lists:

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])
```

This represents two rows and three columns.

Its shape is:

```python
matrix.shape
```

Result:

```python
(2, 3)
```

The shape is one of the most important pieces of NumPy information.

If you do not know the shape, you do not really know the array.

---

# The ndarray Object

The central NumPy object is `ndarray`.

An `ndarray` has:

* data
* shape
* number of dimensions
* dtype
* strides
* methods

You can inspect common attributes:

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.ndim
arr.shape
arr.size
arr.dtype
```

`ndim` is the number of dimensions.

For this array:

```python
arr.ndim
```

Result:

```python
2
```

`shape` describes the length along each dimension:

```python
arr.shape
```

Result:

```python
(2, 3)
```

`size` is the total number of elements:

```python
arr.size
```

Result:

```python
6
```

`dtype` is the data type of the elements:

```python
arr.dtype
```

Possible result:

```python
dtype('int64')
```

These attributes should become automatic.

When debugging NumPy code, ask:

```text
what is the shape?
what is the dtype?
how many dimensions does this array have?
is this a view or a copy?
```

Those questions solve many problems before they become mysterious.

---

# Dimensions and Axes

NumPy uses the word axis to refer to a dimension of an array.

For a one-dimensional array:

```python
arr = np.array([10, 20, 30])
```

The shape is:

```python
(3,)
```

There is one axis.

Axis `0` has length `3`.

For a two-dimensional array:

```python
arr = np.array([
    [1, 2, 3],
    [4, 5, 6],
])
```

The shape is:

```python
(2, 3)
```

There are two axes.

Axis `0` has length `2`.

Axis `1` has length `3`.

A useful way to read a shape is:

```text
(rows, columns)
```

for a two-dimensional array.

But do not trap yourself into thinking NumPy is only rows and columns.

A three-dimensional array might have shape:

```python
(10, 64, 64)
```

This might represent:

```text
10 images, each 64 by 64 pixels
```

A four-dimensional array might have shape:

```python
(32, 224, 224, 3)
```

This might represent:

```text
32 images, each 224 by 224 pixels, with 3 color channels
```

The names depend on the problem.

NumPy only knows axes.

You supply the meaning.

---

# Dtype

NumPy arrays are usually homogeneous.

That means the elements have the same data type.

The dtype controls how values are stored and interpreted.

Examples:

```python
np.array([1, 2, 3]).dtype
np.array([1.0, 2.0, 3.0]).dtype
np.array([True, False, True]).dtype
```

Common dtypes include:

* `int64`
* `int32`
* `float64`
* `float32`
* `bool`
* string dtypes
* datetime-related dtypes

Dtype matters because it affects:

* memory usage
* numerical precision
* performance
* overflow behavior
* compatibility with other libraries

For example:

```python
a = np.array([1, 2, 3], dtype=np.int32)
```

This explicitly requests 32-bit integers.

For machine learning, `float32` is common because it uses less memory than `float64` and is often faster on specialized hardware.

For scientific computation, `float64` is often used because precision matters.

There is no universally correct dtype.

The correct dtype depends on the problem.

---

# Array Creation Helpers

NumPy provides many ways to create arrays.

Zeros:

```python
np.zeros(5)
```

Result:

```python
array([0., 0., 0., 0., 0.])
```

Ones:

```python
np.ones((2, 3))
```

Result:

```python
array([[1., 1., 1.],
       [1., 1., 1.]])
```

A filled array:

```python
np.full((2, 3), 7)
```

Result:

```python
array([[7, 7, 7],
       [7, 7, 7]])
```

A range:

```python
np.arange(0, 10, 2)
```

Result:

```python
array([0, 2, 4, 6, 8])
```

Evenly spaced values:

```python
np.linspace(0, 1, 5)
```

Result:

```python
array([0.  , 0.25, 0.5 , 0.75, 1.  ])
```

Identity matrix:

```python
np.eye(3)
```

Result:

```python
array([[1., 0., 0.],
       [0., 1., 0.],
       [0., 0., 1.]])
```

Use the creation function that communicates intent.

`np.zeros((100, 4))` says:

```text
I need a 100 by 4 numeric array initialized to zero
```

That is clearer than building nested lists manually.

---

# Empty Arrays

NumPy also has `np.empty`.

```python
arr = np.empty((2, 3))
```

This allocates memory without initializing values.

The contents may look random.

That is not a bug.

It is old memory being viewed as array data.

Use `empty` only when you will fill every element before reading from it.

This is a performance tool.

It is not a default beginner tool.

If you need zeros, use `np.zeros`.

If you need ones, use `np.ones`.

If you need uninitialized memory because you are about to write every value, use `np.empty`.

---

# Vectorization

Vectorization is one of the most important NumPy ideas.

It means expressing operations over whole arrays instead of writing Python loops over individual elements.

Plain Python:

```python
values = [1, 2, 3, 4]
result = []

for value in values:
    result.append(value * 10)
```

NumPy:

```python
values = np.array([1, 2, 3, 4])
result = values * 10
```

The NumPy version is shorter.

But the bigger point is not just fewer lines.

The operation can run in optimized lower-level code.

This reduces Python interpreter overhead.

Python loops execute one iteration at a time at the Python level.

NumPy operations often push the loop into compiled code.

The professional NumPy question is:

```text
can I describe this operation as an array operation?
```

If yes, the code is usually clearer and faster.

---

# Element-Wise Operations

Arithmetic with arrays is usually element-wise.

```python
a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

a + b
a - b
a * b
b / a
```

Results:

```python
array([11, 22, 33])
array([ -9, -18, -27])
array([10, 40, 90])
array([10., 10., 10.])
```

Comparisons are also element-wise:

```python
a > 1
```

Result:

```python
array([False,  True,  True])
```

This boolean array can be used for filtering.

Element-wise thinking is essential.

When you write:

```python
arr * 2
```

Do not think:

```text
call multiplication once on the array
```

Think:

```text
multiply every element by 2 and produce an array-shaped result
```

---

# Universal Functions

NumPy universal functions are often called ufuncs.

They are functions that operate element-wise on arrays.

Examples:

```python
np.sqrt(arr)
np.sin(arr)
np.exp(arr)
np.log(arr)
np.abs(arr)
```

For example:

```python
values = np.array([1, 4, 9, 16])
np.sqrt(values)
```

Result:

```python
array([1., 2., 3., 4.])
```

The function is applied to each element.

Ufuncs are fast and composable.

You can write:

```python
scores = np.array([0.2, 0.5, 0.9])
adjusted = np.sqrt(scores) * 100
```

This describes the transformation directly.

Avoid rewriting basic numerical operations yourself.

Use NumPy's functions.

They are tested, optimized, and familiar to other readers.

---

# Broadcasting

Broadcasting is NumPy's rule system for operating on arrays with different shapes.

Simple example:

```python
arr = np.array([1, 2, 3])
arr + 10
```

Result:

```python
array([11, 12, 13])
```

The scalar `10` is treated as if it can apply to each element.

That is broadcasting.

More interesting example:

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

offsets = np.array([10, 20, 30])

matrix + offsets
```

Result:

```python
array([[11, 22, 33],
       [14, 25, 36]])
```

The shape of `matrix` is:

```python
(2, 3)
```

The shape of `offsets` is:

```python
(3,)
```

NumPy aligns shapes from the right.

The trailing dimensions match:

```text
(2, 3)
    (3)
```

So the one-dimensional array can be broadcast across the rows.

Broadcasting lets you avoid manual loops and repeated data.

But it can also create confusion.

When broadcasting fails, the error is often about shapes not being compatible.

The first debugging step is always:

```python
print(a.shape)
print(b.shape)
```

---

# Broadcasting Rules

The practical broadcasting rule is:

```text
compare shapes from right to left
dimensions are compatible when they are equal or one of them is 1
```

Example:

```text
A shape: (8, 1, 6, 1)
B shape:    (7, 1, 5)
```

Align from the right:

```text
(8, 1, 6, 1)
   (7, 1, 5)
```

Compare:

```text
1 and 5 are compatible because one is 1
6 and 1 are compatible because one is 1
1 and 7 are compatible because one is 1
8 has no matching dimension, so it remains
```

The result shape is:

```text
(8, 7, 6, 5)
```

You do not need to memorize exotic examples immediately.

But you should understand the common cases:

* scalar with array
* row vector with matrix
* column vector with matrix
* per-feature values across a batch
* per-row values across columns

Broadcasting is powerful because it lets you express:

```text
apply this smaller shape across that larger shape
```

without physically copying data yourself.

---

# Row and Column Broadcasting

Two-dimensional broadcasting often confuses beginners because row and column shapes look similar in ordinary writing.

Consider:

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])
```

Shape:

```python
(2, 3)
```

A row-like offset:

```python
row_offsets = np.array([10, 20, 30])
```

Shape:

```python
(3,)
```

This works:

```python
matrix + row_offsets
```

It adds across columns.

A column-like offset needs shape `(2, 1)`:

```python
column_offsets = np.array([[100], [200]])
```

Shape:

```python
(2, 1)
```

This works:

```python
matrix + column_offsets
```

Result:

```python
array([[101, 102, 103],
       [204, 205, 206]])
```

The shape `(2,)` is not the same as `(2, 1)`.

This matters constantly.

When broadcasting surprises you, reshape intentionally:

```python
values[:, None]
```

or:

```python
values.reshape(-1, 1)
```

Shape is not cosmetic.

Shape is meaning.

---

# Indexing

NumPy indexing starts with familiar Python ideas.

```python
arr = np.array([10, 20, 30, 40])

arr[0]
arr[1]
arr[-1]
```

Results:

```python
10
20
40
```

For two-dimensional arrays, use comma-separated indices:

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

matrix[0, 0]
matrix[0, 2]
matrix[1, 1]
```

Results:

```python
1
3
5
```

This is different from nested-list indexing:

```python
matrix[0][2]
```

That can work, but `matrix[0, 2]` is the idiomatic NumPy style.

It says:

```text
select row 0 and column 2
```

in one indexing operation.

---

# Slicing

Slicing works with NumPy arrays too.

```python
arr = np.array([10, 20, 30, 40, 50])

arr[1:4]
```

Result:

```python
array([20, 30, 40])
```

For two-dimensional arrays:

```python
matrix = np.array([
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
])
```

First two rows:

```python
matrix[:2, :]
```

First two columns:

```python
matrix[:, :2]
```

Middle block:

```python
matrix[1:, 1:3]
```

The colon means:

```text
take the full range along this axis
```

Slicing is one of the main ways to express data selection without loops.

---

# Views and Copies

NumPy slicing often returns a view.

A view refers to the same underlying data as the original array.

Example:

```python
arr = np.array([10, 20, 30, 40])
part = arr[1:3]

part[0] = 999
```

Now `arr` is:

```python
array([ 10, 999,  30,  40])
```

This surprises many people.

Python list slicing creates a new list.

NumPy array slicing often creates a view.

Views are efficient because they avoid copying data.

But they can be dangerous if you do not realize two arrays share memory.

If you want an independent array, copy explicitly:

```python
part = arr[1:3].copy()
```

The professional habit is:

```text
when mutation matters, know whether you have a view or a copy
```

This is especially important in data cleaning, image processing, simulation, and machine learning preprocessing.

---

# Boolean Indexing

Element-wise comparisons produce boolean arrays.

Those boolean arrays can filter data.

```python
scores = np.array([35, 80, 62, 90, 47])

mask = scores >= 60
```

`mask` is:

```python
array([False,  True,  True,  True, False])
```

Use it to select passing scores:

```python
passing = scores[mask]
```

Result:

```python
array([80, 62, 90])
```

You can inline the condition:

```python
scores[scores >= 60]
```

Boolean indexing is one of the cleanest NumPy patterns.

It replaces loops like:

```python
passing = []

for score in scores:
    if score >= 60:
        passing.append(score)
```

with an array expression.

The mental model is:

```text
build a mask
use the mask to select values
```

---

# Combining Conditions

Use `&`, `|`, and `~` for element-wise boolean logic.

Do not use `and`, `or`, and `not` with arrays.

Example:

```python
scores = np.array([35, 80, 62, 90, 47])

selected = scores[(scores >= 60) & (scores <= 85)]
```

Result:

```python
array([80, 62])
```

The parentheses are important.

This is wrong:

```python
scores[scores >= 60 & scores <= 85]
```

Operator precedence will not mean what you intend.

Use:

```python
(condition_one) & (condition_two)
```

For "or":

```python
(scores < 40) | (scores > 90)
```

For negation:

```python
~(scores >= 60)
```

Array boolean logic is element-wise.

Python boolean logic is object-level.

That distinction matters.

---

# Fancy Indexing

Fancy indexing means selecting elements using arrays or lists of indices.

```python
arr = np.array([10, 20, 30, 40, 50])

arr[[0, 2, 4]]
```

Result:

```python
array([10, 30, 50])
```

For two-dimensional arrays:

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
])

matrix[[0, 2], [1, 2]]
```

This selects:

```text
matrix[0, 1]
matrix[2, 2]
```

Result:

```python
array([2, 9])
```

Fancy indexing is powerful.

It also often returns copies rather than views.

That is another reason to be careful when mutating selected data.

Simple slicing and fancy indexing may look similar, but their memory behavior can differ.

---

# Reshaping

Reshaping changes the shape of an array without changing its data.

```python
arr = np.arange(12)
reshaped = arr.reshape(3, 4)
```

Result:

```python
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
```

The total number of elements must remain the same.

This works:

```python
np.arange(12).reshape(2, 6)
np.arange(12).reshape(3, 4)
np.arange(12).reshape(4, 3)
```

This does not:

```python
np.arange(12).reshape(5, 5)
```

You cannot reshape 12 elements into 25 elements.

Use `-1` to ask NumPy to infer one dimension:

```python
np.arange(12).reshape(3, -1)
```

Result shape:

```python
(3, 4)
```

This is common and useful.

But do not overuse it when explicit shape would be clearer.

---

# Adding and Removing Dimensions

Sometimes you need to add a dimension for broadcasting or model input.

Given:

```python
arr = np.array([10, 20, 30])
```

Shape:

```python
(3,)
```

Convert to a column:

```python
arr[:, None]
```

Shape:

```python
(3, 1)
```

Convert to a row:

```python
arr[None, :]
```

Shape:

```python
(1, 3)
```

You may also see:

```python
np.expand_dims(arr, axis=0)
np.expand_dims(arr, axis=1)
```

Removing dimensions of length one can be done with:

```python
np.squeeze(arr)
```

These operations are common in machine learning, image processing, and broadcasting.

Shape transformations are not incidental.

They are part of the computation.

---

# Transpose

Transposing changes the order of axes.

For a two-dimensional array:

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

matrix.T
```

Result:

```python
array([[1, 4],
       [2, 5],
       [3, 6]])
```

Shape changes from:

```python
(2, 3)
```

to:

```python
(3, 2)
```

For higher-dimensional arrays, transpose can reorder axes:

```python
arr.transpose(1, 0, 2)
```

This is common when working with image data or machine learning frameworks.

For example, one library may expect:

```text
(batch, height, width, channels)
```

Another may expect:

```text
(batch, channels, height, width)
```

Changing axis order correctly is essential.

Guessing is dangerous.

Always inspect shape before and after.

---

# Aggregations

Aggregations reduce data.

Common aggregations include:

```python
arr.sum()
arr.mean()
arr.min()
arr.max()
arr.std()
arr.var()
```

Example:

```python
scores = np.array([80, 90, 75, 60])

scores.mean()
```

Result:

```python
76.25
```

For two-dimensional arrays, aggregations can happen across axes.

```python
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])
```

Sum all elements:

```python
matrix.sum()
```

Result:

```python
21
```

Sum down axis `0`:

```python
matrix.sum(axis=0)
```

Result:

```python
array([5, 7, 9])
```

Sum across axis `1`:

```python
matrix.sum(axis=1)
```

Result:

```python
array([ 6, 15])
```

Understanding `axis` is essential.

Without it, NumPy aggregations feel like guessing.

---

# Thinking About Axis

For a two-dimensional array with shape:

```python
(2, 3)
```

Axis `0` is the first dimension.

Axis `1` is the second dimension.

When you reduce along an axis, that axis disappears from the result unless you keep it.

For:

```python
matrix.sum(axis=0)
```

You reduce the rows.

The row dimension disappears.

The result keeps one value per column:

```python
array([5, 7, 9])
```

For:

```python
matrix.sum(axis=1)
```

You reduce the columns.

The column dimension disappears.

The result keeps one value per row:

```python
array([6, 15])
```

A useful phrase is:

```text
axis is the dimension being collapsed
```

If you want to preserve the reduced dimension, use `keepdims=True`:

```python
matrix.sum(axis=1, keepdims=True)
```

Result shape:

```python
(2, 1)
```

This is often useful for broadcasting later.

---

# Sorting and Searching

NumPy can sort arrays:

```python
arr = np.array([3, 1, 4, 2])

np.sort(arr)
```

Result:

```python
array([1, 2, 3, 4])
```

`np.sort` returns a sorted copy.

The original array is unchanged unless you assign the result or use an in-place method.

Arguments can control sorting along an axis for multidimensional arrays.

NumPy also provides:

* `np.argsort`
* `np.argmin`
* `np.argmax`
* `np.searchsorted`
* `np.partition`

Example:

```python
scores = np.array([70, 95, 80])

scores.argmax()
```

Result:

```python
1
```

The highest score is at index `1`.

This is often useful when you need both a value and its position.

---

# Concatenation and Stacking

Arrays can be combined.

```python
a = np.array([1, 2])
b = np.array([3, 4])

np.concatenate([a, b])
```

Result:

```python
array([1, 2, 3, 4])
```

For two-dimensional arrays:

```python
x = np.array([[1, 2], [3, 4]])
y = np.array([[5, 6]])

np.concatenate([x, y], axis=0)
```

Result:

```python
array([[1, 2],
       [3, 4],
       [5, 6]])
```

Stacking creates a new axis.

```python
np.stack([a, b])
```

Result:

```python
array([[1, 2],
       [3, 4]])
```

Concatenation joins along an existing axis.

Stacking creates a new axis.

That distinction matters.

If the result shape surprises you, inspect it immediately:

```python
result.shape
```

---

# Missing Values and NaN

Numerical data often has missing or invalid values.

Floating-point arrays can use `np.nan`.

```python
values = np.array([1.0, 2.0, np.nan, 4.0])
```

Normal aggregations may produce `nan`:

```python
values.mean()
```

Result:

```python
nan
```

NumPy provides nan-aware functions:

```python
np.nanmean(values)
np.nansum(values)
np.nanmin(values)
np.nanmax(values)
```

To test for NaN, use:

```python
np.isnan(values)
```

Do not use:

```python
values == np.nan
```

NaN does not compare equal to itself.

This is floating-point behavior, not a NumPy joke.

Missing data becomes more central in Pandas.

But NumPy's handling of NaN is already important.

---

# Random Numbers

NumPy includes tools for random number generation.

Modern NumPy code commonly uses a generator:

```python
rng = np.random.default_rng(seed=42)
```

Then:

```python
rng.integers(0, 10, size=5)
rng.normal(loc=0, scale=1, size=(2, 3))
rng.choice(["red", "green", "blue"], size=4)
```

Using a generator is better than relying on global random state.

It makes code easier to test and reproduce.

The seed controls reproducibility:

```python
rng = np.random.default_rng(seed=42)
```

If you rerun the same operations from the same generator state, you get the same sequence.

This matters for:

* simulations
* tests
* examples
* machine learning experiments
* debugging probabilistic behavior

Randomness in professional code should be controlled deliberately.

---

# Linear Algebra

NumPy includes linear algebra tools under `np.linalg`.

Examples:

```python
A = np.array([
    [1.0, 2.0],
    [3.0, 4.0],
])

b = np.array([5.0, 6.0])
```

Matrix-vector multiplication:

```python
A @ b
```

Matrix multiplication uses the `@` operator.

This is different from element-wise multiplication:

```python
A * A
```

Linear algebra functions:

```python
np.linalg.inv(A)
np.linalg.det(A)
np.linalg.solve(A, b)
np.linalg.norm(b)
```

Prefer solving systems with `np.linalg.solve` rather than manually computing an inverse when appropriate.

This is both clearer and often numerically better.

Numerical linear algebra is a deep field.

NumPy gives access to useful tools.

It does not eliminate the need to understand the math behind serious numerical work.

---

# Array Memory Model

NumPy arrays store data in memory.

The shape describes how to interpret that data.

The dtype describes the size and meaning of each element.

Strides describe how many bytes to step to move along each axis.

You do not need to become a memory-layout expert immediately.

But you should understand the basic idea:

```text
an array is not just nested Python lists
it is a block of typed data plus metadata describing shape and stepping
```

This explains why arrays can be efficient.

It also explains why views are possible.

A slice can refer to the same underlying data with different metadata.

Example:

```python
arr = np.arange(10)
view = arr[::2]
```

The view does not need to copy every other value.

It can step through the original data differently.

This is powerful.

It is also why memory sharing matters.

---

# Performance Mental Model

NumPy performance comes from several ideas:

* compact typed memory
* vectorized operations
* compiled inner loops
* optimized numerical libraries
* avoiding Python-level per-element loops

This does not mean every NumPy expression is automatically fast.

Temporary arrays can cost memory.

Broadcasting can produce large conceptual outputs.

Copying large arrays can be expensive.

Wrong dtype choices can waste memory.

Repeated small NumPy calls inside Python loops can still be slow.

The goal is not to worship vectorization blindly.

The goal is to understand where work happens.

Ask:

```text
am I looping in Python over individual elements?
am I creating unnecessary temporary arrays?
am I copying data accidentally?
is the dtype larger than necessary?
is the algorithm itself appropriate?
```

NumPy can make good algorithms fast.

It cannot make a bad algorithm automatically good.

---

# A Practical Example: Normalizing Columns

Suppose you have a data matrix.

Rows are observations.

Columns are features.

```python
X = np.array([
    [10.0, 100.0],
    [20.0, 150.0],
    [30.0, 200.0],
])
```

You want each column to have mean `0` and standard deviation `1`.

Compute column means:

```python
means = X.mean(axis=0)
```

Result:

```python
array([ 20., 150.])
```

Compute column standard deviations:

```python
stds = X.std(axis=0)
```

Result:

```python
array([ 8.16496581, 40.82482905])
```

Normalize:

```python
X_scaled = (X - means) / stds
```

Broadcasting handles the subtraction and division.

`X` has shape:

```python
(3, 2)
```

`means` has shape:

```python
(2,)
```

The column means are applied across rows.

This is NumPy thinking:

```text
compute per-column statistics
broadcast them across the matrix
produce the transformed matrix
```

No explicit loop is needed.

---

# A Practical Example: Filtering Rows

Suppose each row represents a product:

```python
data = np.array([
    [1, 25, 100],
    [2, 40, 300],
    [3, 15, 150],
    [4, 60, 500],
])
```

Imagine the columns are:

```text
product_id, price, stock
```

Select products where price is at least `30` and stock is at least `200`.

```python
price = data[:, 1]
stock = data[:, 2]

mask = (price >= 30) & (stock >= 200)
selected = data[mask]
```

Result:

```python
array([[  2,  40, 300],
       [  4,  60, 500]])
```

The code reads as:

```text
take the price column
take the stock column
build a condition
select matching rows
```

This is the foundation of data filtering.

Pandas will make this more labeled and convenient.

NumPy shows the underlying array logic.

---

# A Practical Example: Image Data

An image can be represented as an array.

A grayscale image might have shape:

```text
(height, width)
```

A color image might have shape:

```text
(height, width, channels)
```

For RGB images, channels often have length `3`.

Example shape:

```python
(1080, 1920, 3)
```

This means:

```text
1080 rows
1920 columns
3 color values per pixel
```

To darken an image:

```python
darkened = image * 0.5
```

To select the red channel:

```python
red = image[:, :, 0]
```

To crop:

```python
crop = image[100:400, 200:600, :]
```

These operations are all array operations.

Image processing becomes much easier when shape and slicing are familiar.

---

# Interoperability

NumPy arrays are a common language across libraries.

Many libraries accept NumPy arrays as input.

Many libraries return NumPy arrays as output.

This makes NumPy a bridge.

Examples:

* Pandas can convert columns to NumPy arrays
* SciPy algorithms operate on arrays
* scikit-learn estimators often expect array-like inputs
* image libraries can convert images to arrays
* plotting libraries can plot arrays
* PyTorch tensors can often interoperate with NumPy arrays

This is why NumPy belongs early in the ecosystem section.

Even if you spend most of your day using Pandas or PyTorch, NumPy concepts keep appearing.

When a library says "array-like", it often means:

```text
something that behaves enough like a NumPy array for this operation
```

Understanding NumPy makes the rest of the ecosystem less mysterious.

---

# NumPy Is Not Pandas

NumPy arrays do not have column names by default.

They do not have indexes like Pandas DataFrames.

They do not automatically understand missing business values, categories, joins, groupby workflows, or labeled tabular data.

NumPy is lower-level.

It is closer to numerical arrays.

Pandas is higher-level.

It is closer to labeled data analysis.

If your data is:

```text
rectangular table with column names, mixed types, missing values, joins, grouping, and CSV files
```

Pandas may be more ergonomic.

If your data is:

```text
large homogeneous numerical arrays with mathematical operations
```

NumPy may be the right core tool.

The libraries complement each other.

Pandas often gives you human-friendly data analysis.

NumPy gives you the numerical engine and array model underneath.

---

# NumPy Is Not a Deep Learning Framework

NumPy is also not PyTorch or TensorFlow.

NumPy arrays do not provide automatic differentiation.

They do not manage GPU execution in the way deep learning frameworks do.

They do not build neural network computation graphs.

But NumPy is still essential preparation.

Deep learning tensors are conceptually close to arrays:

* shape matters
* dtype matters
* broadcasting matters
* slicing matters
* matrix multiplication matters
* axis order matters
* batching matters

If you are comfortable with NumPy, PyTorch and TensorFlow become easier to learn.

The syntax changes.

The array thinking remains.

---

# Common Mistakes

The first common mistake is ignoring shape.

If you do not know the shape, you are guessing.

The second common mistake is confusing `(n,)`, `(1, n)`, and `(n, 1)`.

These are different shapes.

They broadcast differently.

The third common mistake is using Python loops for operations NumPy can express directly.

Loops are sometimes necessary.

But many beginner loops are just hidden vector operations.

The fourth common mistake is assuming slices are independent copies.

Many slices are views.

Mutating a view can mutate the original array.

The fifth common mistake is using `and` and `or` with arrays.

Use `&` and `|` with parentheses for element-wise conditions.

The sixth common mistake is comparing to `np.nan`.

Use `np.isnan`.

The seventh common mistake is using `np.empty` and reading uninitialized values.

Use `empty` only when you will fill the array before reading.

The eighth common mistake is mixing dtypes accidentally.

Check `dtype` when results look strange or memory usage is unexpectedly high.

The ninth common mistake is treating NumPy as a database table tool.

For labeled messy tabular data, Pandas or Polars may be more appropriate.

The tenth common mistake is assuming vectorized-looking code is always memory efficient.

Large temporary arrays can be expensive.

Performance requires measurement.

---

# Professional NumPy Checklist

When working with NumPy, develop these habits:

* Import NumPy as `np`.
* Inspect `shape` frequently.
* Inspect `dtype` when precision or memory matters.
* Prefer array operations over Python element loops.
* Use broadcasting deliberately.
* Reshape explicitly when dimensions matter.
* Use boolean masks for filtering.
* Use `copy()` when independent mutation is required.
* Use `axis` intentionally for reductions.
* Use `keepdims=True` when a reduced result must broadcast later.
* Use `np.isnan` for NaN checks.
* Use `np.random.default_rng` for reproducible randomness.
* Avoid accidental giant temporary arrays.
* Measure performance before optimizing deeply.
* Choose Pandas or Polars when labels and table operations dominate.
* Choose SciPy when you need specialized scientific algorithms.
* Choose PyTorch or TensorFlow when you need automatic differentiation or deep learning infrastructure.

This checklist is practical because NumPy bugs are often shape bugs, dtype bugs, or copy/view bugs.

The sooner you learn to inspect those things, the less mysterious numerical Python becomes.

---

# Summary

NumPy is the foundational numerical array library in Python.

Its central object is `ndarray`, an N-dimensional homogeneous array.

Arrays have shape, dtype, size, dimensions, and memory behavior.

NumPy operations are often vectorized, meaning they operate over whole arrays instead of requiring Python loops over individual elements.

Universal functions apply element-wise operations efficiently.

Broadcasting allows arrays with compatible shapes to work together without manually copying data.

Indexing, slicing, boolean masks, fancy indexing, reshaping, transposing, stacking, and aggregation are core skills.

Views and copies matter because mutation can affect shared underlying data.

Axis reasoning is essential for reductions and multidimensional work.

NumPy also provides random number generation, linear algebra tools, sorting, searching, NaN-aware functions, and interoperability with the broader scientific Python ecosystem.

The central lesson is:

```text
NumPy teaches Python to think in efficient typed arrays
```

Once you can think in arrays, much of data science, scientific computing, and machine learning becomes easier to understand.

---

# Exercises

1. Create a one-dimensional NumPy array from the numbers `1` through `10`.

2. Inspect its `shape`, `ndim`, `size`, and `dtype`.

3. Create a two-dimensional array with shape `(3, 4)` using `np.arange` and `reshape`.

4. Select the first row, last column, and middle block from that array.

5. Multiply every element by `10` without writing a Python loop.

6. Create a boolean mask for values greater than `5` and use it to filter the array.

7. Combine two boolean conditions using `&`.

8. Create a `(3,)` array and reshape it into `(3, 1)`.

9. Add a row vector to a matrix using broadcasting.

10. Add a column vector to a matrix using broadcasting.

11. Compute column means and row sums for a two-dimensional array.

12. Normalize each column of a matrix by subtracting its mean and dividing by its standard deviation.

13. Create an array containing `np.nan` and compute its `nanmean`.

14. Use `np.random.default_rng` with a seed to generate reproducible random numbers.

15. Demonstrate the difference between a slice view and an explicit copy.

16. Use `np.argsort` to get the order of values in an array.

17. Use `@` for matrix multiplication and compare it with `*`.

18. Explain why `(5,)`, `(1, 5)`, and `(5, 1)` are different.

19. Write a small example where `keepdims=True` makes broadcasting easier.

20. Identify one task where NumPy is a better fit than Pandas and one task where Pandas is a better fit than NumPy.

---

# Preview of Chapter 89

Chapter 88 studied NumPy.

We learned how Python handles efficient numerical computing through arrays, shapes, dtypes, vectorized operations, broadcasting, indexing, views, copies, reductions, random numbers, and linear algebra.

Next we study Pandas.

Pandas builds on the array world but adds labels, columns, indexes, table operations, missing-data tools, grouping, joining, time series support, and data analysis workflows.

The transition is:

```text
NumPy is about efficient numerical arrays
Pandas is about labeled tabular data analysis
```

Chapter 89 will show how Pandas turns Python into a practical tool for reading, cleaning, transforming, analyzing, and exporting real-world datasets.
