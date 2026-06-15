# Chapter 91 — SciPy

SciPy is a library of scientific and technical computing algorithms built on NumPy.

NumPy gives Python efficient arrays.

SciPy gives those arrays a large collection of mathematical tools.

That is the relationship.

If NumPy is the array foundation, SciPy is the scientific algorithm layer.

SciPy helps with problems such as:

* optimization
* root finding
* numerical integration
* interpolation
* linear algebra
* sparse arrays
* statistics
* signal processing
* Fourier transforms
* spatial algorithms
* image processing
* special mathematical functions
* graph routines
* scientific constants

This is a different kind of library from Pandas or Polars.

Pandas and Polars are about tables and data workflows.

SciPy is about numerical methods.

It answers questions like:

* What input minimizes this function?
* Where does this equation equal zero?
* What is the integral of this function?
* How can I interpolate between measured points?
* How do I solve this linear system?
* How do I represent a huge mostly-empty matrix?
* How do I filter a noisy signal?
* How do I compute distances between points?
* How do I work with probability distributions?

SciPy is not one topic.

It is a toolbox of many topics.

This chapter gives you a map.

The goal is not to memorize every subpackage.

The goal is to understand when SciPy is the right tool and how to approach it professionally.

---

# Why SciPy Matters

SciPy matters because real scientific and engineering problems rarely stop at array arithmetic.

NumPy can store data and perform vectorized operations.

But many problems require established algorithms.

For example:

* fitting a model to measured data
* solving a system of equations
* minimizing an error function
* integrating a differential equation
* interpolating missing measurements
* filtering a time-domain signal
* computing a Fourier transform
* finding nearest neighbors
* working with probability distributions
* solving sparse linear systems

You could try to implement all of these yourself.

Sometimes that is educational.

Usually it is a bad professional choice.

Numerical algorithms are subtle.

They involve:

* convergence
* stability
* conditioning
* precision
* tolerances
* error estimates
* algorithmic complexity
* assumptions about input shape
* assumptions about smoothness
* assumptions about sparsity

SciPy gives access to mature implementations of many common algorithms.

That does not remove responsibility.

You still need to understand the problem.

You still need to choose the right method.

You still need to check the result.

But SciPy prevents you from reinventing decades of numerical computing badly.

---

# SciPy After NumPy, Pandas, and Polars

The previous chapters gave us three foundations.

NumPy taught arrays:

```text
shape, dtype, broadcasting, vectorization, indexing, linear algebra basics
```

Pandas taught labeled tables:

```text
Series, DataFrames, indexes, missing values, grouping, joining, time series
```

Polars taught optimized DataFrame queries:

```text
expressions, strict schemas, lazy execution, query optimization, parallelism
```

SciPy sits beside these tools rather than replacing them.

A real workflow might look like:

```text
Pandas or Polars reads and cleans tabular data
NumPy arrays hold numerical inputs
SciPy solves the mathematical problem
Matplotlib or another tool visualizes the result
```

For example:

1. Read experiment data with Pandas.
2. Extract columns as NumPy arrays.
3. Fit a curve with SciPy optimization.
4. Evaluate residuals.
5. Plot the fitted curve.

SciPy is often the algorithmic middle of a scientific workflow.

It is where data becomes computation.

---

# Importing SciPy

SciPy is organized into subpackages.

You usually import the part you need:

```python
from scipy import optimize
from scipy import integrate
from scipy import interpolate
from scipy import linalg
from scipy import sparse
from scipy import stats
```

Or import specific functions:

```python
from scipy.optimize import minimize
from scipy.integrate import solve_ivp
from scipy.interpolate import CubicSpline
```

Avoid writing code that imports every possible SciPy name into the namespace.

SciPy is broad.

Explicit imports help readers know which scientific domain the code is using.

For example:

```python
from scipy.optimize import minimize
```

immediately tells the reader:

```text
this code solves an optimization problem
```

Good scientific code should make the problem type visible.

---

# SciPy Subpackages

SciPy is divided into subpackages.

Important ones include:

* `scipy.optimize`
* `scipy.integrate`
* `scipy.interpolate`
* `scipy.linalg`
* `scipy.sparse`
* `scipy.stats`
* `scipy.signal`
* `scipy.fft`
* `scipy.spatial`
* `scipy.special`
* `scipy.ndimage`
* `scipy.constants`
* `scipy.io`

Each subpackage represents a domain.

This organization matters.

When you have a problem, first classify it.

Do you need to minimize something?

Look at `optimize`.

Do you need to compute an integral?

Look at `integrate`.

Do you need values between measured points?

Look at `interpolate`.

Do you need sparse matrix storage or solvers?

Look at `sparse`.

Do you need probability distributions or statistical tests?

Look at `stats`.

This is how SciPy becomes manageable.

It is not one giant bag of functions.

It is a collection of domain-specific numerical tools.

---

# The Scientific Computing Mindset

SciPy requires a different mindset from ordinary application programming.

In application programming, a function often either works or raises an error.

In numerical computing, a function may return a result that is:

* approximate
* sensitive to input scaling
* dependent on tolerance settings
* affected by floating-point precision
* locally optimal but not globally optimal
* accurate for some ranges but not others
* invalid because assumptions were violated

This means you must ask different questions.

Not only:

```text
did the code run?
```

But:

```text
does the result make mathematical sense?
how accurate is it?
what assumptions does the method make?
what happens if inputs change slightly?
did the algorithm converge?
is the problem well-conditioned?
```

SciPy gives powerful tools.

Those tools deserve careful interpretation.

---

# Optimization

Optimization is about finding inputs that minimize or maximize an objective function.

SciPy's optimization tools live in `scipy.optimize`.

The central idea is:

```text
define a function that measures how good or bad a candidate solution is
then use an algorithm to search for a better solution
```

Example:

```python
import numpy as np
from scipy.optimize import minimize


def objective(x):
    return (x[0] - 3) ** 2 + (x[1] + 2) ** 2


initial_guess = np.array([0.0, 0.0])
result = minimize(objective, initial_guess)
```

The minimum occurs near:

```python
x = [3, -2]
```

The `result` object contains information such as:

* solution estimate
* objective value
* success flag
* status message
* number of function evaluations
* algorithm-specific details

Do not only read `result.x`.

Also inspect:

```python
result.success
result.message
result.fun
```

Optimization is not merely calling `minimize`.

It is checking whether the search actually succeeded.

---

# Choosing an Optimization Method

Optimization methods differ.

Some use gradients.

Some do not.

Some support constraints.

Some are local.

Some attempt global search.

Some are better for smooth functions.

Some tolerate noisy functions.

Some scale better to high dimensions.

Common SciPy optimization tools include:

* `minimize`
* `minimize_scalar`
* `least_squares`
* `root`
* `root_scalar`
* `linprog`
* global optimization routines

The first question is:

```text
what kind of problem do I have?
```

Is it a scalar function of one variable?

Use `minimize_scalar` or root-finding tools.

Is it a multivariable smooth objective?

Use `minimize`.

Is it a curve-fitting or residual minimization problem?

Use `least_squares` or curve-fitting tools.

Is it a set of equations equal to zero?

Use `root`.

Is it a linear programming problem?

Use `linprog`.

Choosing the right problem formulation is often more important than choosing a fancy method.

---

# Root Finding

Root finding asks:

```text
where does f(x) = 0?
```

Example:

```python
from scipy.optimize import root_scalar


def f(x):
    return x**2 - 2


result = root_scalar(f, bracket=[0, 2])
```

The root is near:

```python
1.41421356
```

because:

```text
sqrt(2)^2 - 2 = 0
```

The bracket is important.

For bracketing methods, you provide an interval where the function changes sign.

Root finding is common in:

* physics
* engineering
* calibration
* economics
* numerical equation solving
* fixed-point problems

The professional habit is:

```text
verify the root by evaluating the function at the returned solution
```

If `f(result.root)` is not close to zero, something is wrong or tolerance expectations need review.

---

# Least Squares

Least squares problems appear everywhere.

The idea is:

```text
choose parameters that minimize residual errors
```

Suppose measured data roughly follows:

```text
y = a*x + b
```

You can fit parameters by minimizing residuals:

```python
import numpy as np
from scipy.optimize import least_squares

x = np.array([0, 1, 2, 3, 4], dtype=float)
y = np.array([1.1, 2.0, 2.9, 4.1, 5.1])


def residuals(params):
    a, b = params
    predicted = a * x + b
    return predicted - y


result = least_squares(residuals, x0=np.array([1.0, 0.0]))
```

The result estimates `a` and `b`.

Least squares is common in:

* curve fitting
* calibration
* regression-like problems
* parameter estimation
* inverse problems

Always inspect residuals.

A low numerical loss does not automatically mean the model is scientifically appropriate.

Plotting residuals can reveal patterns the fit hides.

---

# Numerical Integration

Numerical integration estimates areas, accumulated quantities, or solutions to differential equations.

SciPy's integration tools live in `scipy.integrate`.

For a definite integral:

```python
from scipy.integrate import quad


def f(x):
    return x**2


value, error = quad(f, 0, 1)
```

The exact result is:

```text
1/3
```

`quad` returns an estimated value and an error estimate.

Do not ignore the error estimate.

Numerical integration is approximate.

It is affected by:

* function smoothness
* singularities
* oscillation
* infinite limits
* tolerance settings
* discontinuities

When integration results matter, test the method on known cases and inspect sensitivity.

---

# Differential Equations

Many scientific systems are described by differential equations.

SciPy provides tools such as `solve_ivp` for initial value problems.

Example:

```python
import numpy as np
from scipy.integrate import solve_ivp


def decay(t, y):
    return -0.5 * y


solution = solve_ivp(
    decay,
    t_span=(0, 10),
    y0=[1.0],
    t_eval=np.linspace(0, 10, 100),
)
```

This models exponential decay.

The solver returns time points and solution values.

Differential equation solving requires care.

Ask:

* Is the system stiff?
* What tolerances are appropriate?
* Are units consistent?
* Does the solution conserve expected quantities?
* Does the solution behave correctly on a simple test problem?
* Are time steps fine enough for the phenomenon?

ODE solvers are powerful.

They are not a substitute for understanding the system being modeled.

---

# Interpolation

Interpolation estimates values between known data points.

SciPy interpolation tools live in `scipy.interpolate`.

Example with a cubic spline:

```python
import numpy as np
from scipy.interpolate import CubicSpline

x = np.array([0, 1, 2, 3])
y = np.array([0, 1, 0, 1])

spline = CubicSpline(x, y)

value = spline(1.5)
```

Interpolation is useful when:

* measurements are sampled at discrete points
* you need values at new positions
* grids need resampling
* curves need smoothing or approximation

But interpolation has assumptions.

Linear interpolation behaves differently from cubic splines.

Spline interpolation can overshoot.

Monotonic data may need monotonic interpolation.

Extrapolation outside the known data range is especially risky.

The professional rule is:

```text
interpolation is not prediction by magic
```

It fills between known points using assumptions.

Make those assumptions explicit.

---

# Linear Algebra

SciPy provides linear algebra tools in `scipy.linalg`.

NumPy also has `numpy.linalg`.

SciPy's linear algebra module includes NumPy-like tools plus additional routines and is built around BLAS/LAPACK support.

Common operations:

```python
import numpy as np
from scipy import linalg

A = np.array([[3.0, 1.0], [1.0, 2.0]])
b = np.array([9.0, 8.0])

x = linalg.solve(A, b)
```

This solves:

```text
A @ x = b
```

Prefer solving linear systems directly with `solve` instead of computing a matrix inverse and multiplying.

This is usually clearer and numerically better.

Linear algebra tools include:

* solving systems
* matrix decompositions
* eigenvalues
* determinants
* norms
* least squares
* matrix functions

Linear algebra is foundational for scientific computing and machine learning.

SciPy gives tools.

Mathematical understanding still matters.

---

# Sparse Arrays

Sparse arrays represent data where most entries are zero.

This matters because dense storage can be impossible.

Imagine a matrix with:

```text
1,000,000 rows and 1,000,000 columns
```

A dense representation would be enormous.

But if most values are zero, sparse storage can store only the meaningful nonzero values and structure.

SciPy sparse tools live in `scipy.sparse`.

Example:

```python
import numpy as np
from scipy import sparse

row = np.array([0, 1, 2])
col = np.array([0, 2, 1])
data = np.array([10, 20, 30])

matrix = sparse.coo_array((data, (row, col)), shape=(3, 3))
```

Sparse formats matter.

Common formats include:

* COO
* CSR
* CSC
* DIA
* LIL
* DOK

Different formats are efficient for different operations.

For example, CSR is often good for arithmetic and row slicing.

COO is often convenient for constructing sparse arrays.

Sparse computing is not just "dense arrays with zeros removed."

It has its own performance rules.

---

# Statistics

SciPy's statistics tools live in `scipy.stats`.

They include:

* probability distributions
* random variables
* descriptive statistics
* hypothesis tests
* correlation functions
* statistical utilities

Example:

```python
from scipy import stats

probability = stats.norm.cdf(1.96)
```

This computes the cumulative probability for the standard normal distribution at `1.96`.

Another example:

```python
sample = [2.1, 2.5, 2.4, 2.8, 3.0]

result = stats.ttest_1samp(sample, popmean=2.0)
```

Statistical functions are easy to call.

Statistical interpretation is harder.

Before using a test, ask:

* What are the assumptions?
* Are observations independent?
* Is the sample size appropriate?
* What is the null hypothesis?
* What does the p-value mean?
* What effect size matters?
* Are multiple comparisons involved?

SciPy gives statistical machinery.

It does not replace statistical reasoning.

---

# Signal Processing

SciPy's signal processing tools live in `scipy.signal`.

They help with:

* filtering
* convolution
* spectral analysis
* window functions
* peak finding
* digital filter design
* resampling
* continuous and discrete systems

Example:

```python
import numpy as np
from scipy import signal

t = np.linspace(0, 1, 500)
clean = np.sin(2 * np.pi * 10 * t)
noise = 0.3 * np.random.default_rng(42).normal(size=t.shape)
observed = clean + noise

filtered = signal.savgol_filter(observed, window_length=31, polyorder=3)
```

Signal processing appears in:

* audio
* sensors
* communications
* biomedical data
* finance
* industrial monitoring
* physics experiments

Be careful with filters.

Filters can distort signals.

They can introduce delay.

They can remove meaningful variation.

They can create artifacts.

Always understand what the filter assumes and what it changes.

---

# Fourier Transforms

Fourier transforms convert signals between time or space domains and frequency domains.

SciPy provides Fourier tools in `scipy.fft`.

Example:

```python
import numpy as np
from scipy.fft import fft, fftfreq

sample_rate = 1000
t = np.arange(sample_rate) / sample_rate
signal_values = np.sin(2 * np.pi * 50 * t)

spectrum = fft(signal_values)
frequencies = fftfreq(t.size, d=1 / sample_rate)
```

Fourier analysis is useful for:

* frequency detection
* filtering
* image processing
* signal analysis
* differential equations
* convolution

But frequency-domain analysis has assumptions too.

Sampling rate matters.

Aliasing matters.

Windowing matters.

Signal length matters.

The FFT is fast.

Correct interpretation is the hard part.

---

# Spatial Algorithms

SciPy's spatial tools live in `scipy.spatial`.

They include:

* distance computations
* KD-trees
* nearest-neighbor search
* Delaunay triangulation
* Voronoi diagrams
* convex hulls

Example:

```python
import numpy as np
from scipy.spatial import KDTree

points = np.array([
    [0.0, 0.0],
    [1.0, 1.0],
    [2.0, 2.0],
])

tree = KDTree(points)
distance, index = tree.query([1.2, 1.1])
```

Spatial algorithms are useful in:

* geometry
* clustering
* nearest-neighbor lookup
* geographic analysis
* simulations
* mesh processing
* computer vision

As dimensions grow, nearest-neighbor behavior can become harder.

High-dimensional distance has traps.

Do not assume Euclidean distance is meaningful for every dataset.

---

# Special Functions

SciPy's special functions live in `scipy.special`.

These include functions from advanced mathematics:

* gamma functions
* beta functions
* Bessel functions
* error functions
* orthogonal polynomials
* combinatorial functions

Example:

```python
from scipy import special

value = special.gamma(5)
```

Special functions appear in:

* probability
* physics
* engineering
* differential equations
* statistics
* signal processing

Many special functions are difficult to implement accurately across all input ranges.

Use SciPy rather than writing naive versions.

Numerical accuracy is often the whole point.

---

# Image Processing

SciPy has multidimensional image processing tools in `scipy.ndimage`.

They can perform operations such as:

* filtering
* convolution
* morphology
* measurements
* interpolation
* transformations

Example:

```python
import numpy as np
from scipy import ndimage

image = np.random.default_rng(42).normal(size=(100, 100))
smoothed = ndimage.gaussian_filter(image, sigma=2)
```

For full computer vision workflows, libraries such as scikit-image or OpenCV may be more specialized.

But `scipy.ndimage` is useful for many numerical image operations.

The NumPy chapter's array thinking helps here.

Images are arrays.

Filters are numerical operations over those arrays.

---

# Constants and Units

SciPy includes physical and mathematical constants in `scipy.constants`.

Example:

```python
from scipy import constants

speed = constants.speed_of_light
planck = constants.Planck
```

Constants help avoid magic numbers.

But constants do not solve unit consistency.

You still need to know whether your formula expects:

* meters or kilometers
* seconds or milliseconds
* radians or degrees
* kilograms or grams
* Celsius or Kelvin

Many scientific bugs are unit bugs.

Using named constants is good.

Tracking units carefully is better.

For unit-aware work, specialized libraries may be appropriate.

---

# File I/O

SciPy includes `scipy.io` for some scientific file formats.

Examples include MATLAB files and other scientific formats.

In modern workflows, you may also use:

* NumPy file formats
* HDF5 through `h5py`
* NetCDF through related libraries
* image libraries
* Pandas
* Polars
* xarray
* domain-specific file readers

The important point is:

```text
scientific data often arrives in domain-specific formats
```

Choose file tools based on the format, data size, metadata needs, and ecosystem.

Do not force everything through CSV.

CSV is useful, but it is not a scientific data format for every job.

---

# A Practical Example: Curve Fitting

Suppose measurements follow an exponential decay:

```text
y = a * exp(-b*x) + c
```

Use SciPy to estimate parameters.

```python
import numpy as np
from scipy.optimize import curve_fit


def model(x, a, b, c):
    return a * np.exp(-b * x) + c


x = np.linspace(0, 4, 50)
rng = np.random.default_rng(42)
y = model(x, 2.5, 1.3, 0.5) + 0.2 * rng.normal(size=x.size)

params, covariance = curve_fit(model, x, y, p0=[1.0, 1.0, 0.0])
```

`params` contains estimated values for `a`, `b`, and `c`.

This looks simple.

But professional fitting requires more:

* inspect residuals
* check parameter uncertainty
* choose reasonable initial guesses
* consider bounds
* validate the model form
* avoid overfitting
* understand measurement noise

SciPy can fit the curve.

You must decide whether the curve means anything.

---

# A Practical Example: Interpolation

Suppose a sensor reports values at irregular times.

You need estimates at regular time intervals.

```python
import numpy as np
from scipy.interpolate import PchipInterpolator

time = np.array([0.0, 0.7, 1.8, 3.0, 4.2])
temperature = np.array([20.0, 21.5, 22.0, 21.0, 19.5])

interp = PchipInterpolator(time, temperature)

regular_time = np.linspace(0, 4.2, 50)
regular_temperature = interp(regular_time)
```

PCHIP is often useful when preserving monotonic behavior matters more than producing a very smooth cubic spline.

The method choice depends on the data.

Never interpolate without thinking about:

* spacing of measurements
* noise
* smoothness
* monotonicity
* extrapolation
* physical constraints

The code is small.

The judgment is the real work.

---

# A Practical Example: Sparse Matrix

Suppose users rate only a few products.

A dense user-product matrix would be mostly zeros.

Sparse storage is natural.

```python
import numpy as np
from scipy import sparse

users = np.array([0, 0, 1, 2])
products = np.array([1, 3, 0, 2])
ratings = np.array([5.0, 4.0, 3.0, 4.5])

ratings_matrix = sparse.coo_array(
    (ratings, (users, products)),
    shape=(3, 4),
).tocsr()
```

Now the matrix stores only known ratings.

Sparse representations appear in:

* recommendation systems
* graph algorithms
* text vectorization
* finite element methods
* scientific simulations
* large linear systems

Sparse arrays are essential when zeros dominate.

But they are not always faster.

Use sparse storage when the structure fits the operations you need.

---

# SciPy and Machine Learning

SciPy is not a machine learning framework.

But machine learning uses many ideas SciPy supports:

* optimization
* sparse matrices
* distance computations
* statistics
* linear algebra
* interpolation
* signal processing

scikit-learn, the next chapter's topic, builds heavily on NumPy and SciPy concepts.

For example:

* many models optimize objective functions
* text features often use sparse matrices
* nearest-neighbor methods use distance computations
* preprocessing may use statistics
* dimensionality reduction relies on linear algebra

Learning SciPy helps you understand what machine learning libraries are doing underneath.

You do not need to implement every algorithm yourself.

But you should recognize the mathematical families.

---

# Numerical Reliability

Numerical computing has traps.

Floating-point arithmetic is approximate.

Small errors can accumulate.

Some problems are ill-conditioned.

Some algorithms fail silently if assumptions are violated.

Some solutions are local rather than global.

Some inputs are outside the stable range of an algorithm.

Professional SciPy use includes:

* checking convergence flags
* checking residuals
* checking error estimates
* testing on known cases
* scaling inputs
* plotting results
* comparing methods when uncertain
* reading method documentation
* understanding tolerances
* preserving units

The bad habit is:

```text
call a SciPy function and trust the number blindly
```

The good habit is:

```text
call the function, inspect diagnostics, validate the result
```

---

# Common Mistakes

The first common mistake is using SciPy without understanding the mathematical problem type.

Classify the problem first.

The second common mistake is ignoring result diagnostics.

Check success flags, messages, residuals, and error estimates.

The third common mistake is computing matrix inverses when solving systems directly is better.

Use solve routines when solving `A @ x = b`.

The fourth common mistake is treating interpolation as reliable prediction.

Interpolation depends on assumptions and is especially risky outside the data range.

The fifth common mistake is choosing an optimization method blindly.

Different methods have different assumptions and strengths.

The sixth common mistake is ignoring scale.

Poorly scaled inputs can hurt numerical methods.

The seventh common mistake is using dense arrays for naturally sparse problems.

Sparse structure can be essential.

The eighth common mistake is running statistical tests without checking assumptions.

Statistics is not only function calls.

The ninth common mistake is ignoring units.

Unit mistakes can make beautiful code produce nonsense.

The tenth common mistake is expecting SciPy to replace domain knowledge.

SciPy provides algorithms.

You provide problem understanding.

---

# Professional SciPy Checklist

When using SciPy, practice these habits:

* Start by naming the mathematical problem type.
* Choose the relevant subpackage deliberately.
* Use NumPy arrays with clear shape and dtype.
* Inspect function documentation before using unfamiliar algorithms.
* Check convergence and status outputs.
* Check residuals or error estimates.
* Test on a small problem with known answer.
* Prefer direct solvers over explicit matrix inverses when appropriate.
* Use sparse structures for genuinely sparse problems.
* Scale variables when optimization behaves poorly.
* Be explicit about tolerances.
* Avoid extrapolation unless it is justified.
* Plot data and results when possible.
* Track physical units carefully.
* Treat statistical results as evidence, not automatic truth.
* Keep algorithm choices documented in important code.

These habits help turn SciPy from a bag of functions into a professional scientific computing tool.

---

# Summary

SciPy is a broad scientific computing library built on NumPy.

It provides algorithms and utilities for optimization, root finding, least squares, numerical integration, differential equations, interpolation, linear algebra, sparse arrays, statistics, signal processing, Fourier transforms, spatial algorithms, image processing, special functions, constants, and scientific file formats.

SciPy is organized into subpackages, each focused on a problem domain.

The most important skill is learning to classify the problem before choosing a function.

Numerical methods are approximate and assumption-heavy.

Professional SciPy use requires inspecting diagnostics, validating results, checking assumptions, and understanding units, tolerances, and numerical stability.

SciPy connects directly to machine learning because many ML tools rely on optimization, sparse matrices, distance computations, statistics, and linear algebra.

The central lesson is:

```text
SciPy turns NumPy arrays into scientific and mathematical problem solving
```

Once you understand that, SciPy becomes a map of algorithm families rather than an overwhelming list of functions.

---

# Exercises

1. Import `scipy.optimize`, `scipy.integrate`, `scipy.interpolate`, `scipy.linalg`, `scipy.sparse`, and `scipy.stats`.

2. Use `minimize` to minimize a simple quadratic function.

3. Inspect the optimization result's `success`, `message`, `x`, and `fun`.

4. Use `root_scalar` to find the root of `x**2 - 2`.

5. Verify the returned root by evaluating the function at the root.

6. Use `least_squares` to fit a straight line to noisy data.

7. Plot or inspect residuals from a fitted model.

8. Use `quad` to integrate `x**2` from `0` to `1`.

9. Solve a simple differential equation with `solve_ivp`.

10. Use `CubicSpline` or `PchipInterpolator` to interpolate measured points.

11. Solve a linear system with `scipy.linalg.solve`.

12. Explain why solving directly is usually better than computing an inverse.

13. Create a sparse COO array and convert it to CSR.

14. Use a normal distribution from `scipy.stats` to compute a cumulative probability.

15. Run a simple one-sample t-test and explain what the result does and does not prove.

16. Smooth a noisy signal with a SciPy signal-processing function.

17. Compute an FFT and identify what the frequency axis means.

18. Build a KD-tree and query the nearest neighbor to a point.

19. Use a function from `scipy.special`.

20. Choose three SciPy subpackages and describe a real-world problem each one could solve.

---

# Part II Closing

Part II of Volume IV covered data and scientific computing.

NumPy introduced efficient arrays, vectorized operations, broadcasting, shapes, dtypes, indexing, reductions, random numbers, and linear algebra foundations.

Pandas introduced labeled tabular data analysis with Series, DataFrames, indexes, missing values, grouping, joining, reshaping, time series, and reproducible workflows.

Polars introduced a performance-oriented DataFrame model based on expressions, strict schemas, lazy execution, query optimization, and parallelism.

SciPy introduced scientific algorithms built on NumPy arrays.

Together, these libraries show why Python became so important for data work.

Python itself provides the language.

NumPy provides the array foundation.

Pandas and Polars provide table workflows.

SciPy provides mathematical and scientific algorithms.

This stack lets a Python programmer move from raw data to numerical computation to practical analysis.

---

# Preview of Chapter 92

Chapter 91 studied SciPy.

We learned how SciPy extends NumPy with algorithm families for optimization, integration, interpolation, linear algebra, sparse arrays, statistics, signal processing, spatial computation, Fourier transforms, special functions, and scientific workflows.

Next we begin Part III of Volume IV: Machine Learning and AI Engineering.

The first chapter in that part is scikit-learn.

scikit-learn builds on NumPy and SciPy to provide practical machine learning tools for classification, regression, clustering, preprocessing, model selection, and pipelines.

The transition is:

```text
SciPy provides scientific algorithms
scikit-learn organizes many of those numerical ideas into machine learning workflows
```

Chapter 92 will show how scikit-learn turns arrays and scientific algorithms into practical supervised and unsupervised learning systems.
