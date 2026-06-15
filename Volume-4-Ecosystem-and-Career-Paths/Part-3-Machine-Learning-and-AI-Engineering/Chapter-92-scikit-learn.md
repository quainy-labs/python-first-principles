# Chapter 92 — scikit-learn

scikit-learn is Python's most important general-purpose machine learning library for classical machine learning.

It is not a deep learning framework.

It is not a data cleaning library.

It is not a deployment platform.

scikit-learn is a practical toolkit for building, evaluating, and composing machine learning models using a consistent API.

It includes tools for:

* classification
* regression
* clustering
* dimensionality reduction
* preprocessing
* feature extraction
* model selection
* cross-validation
* metrics
* pipelines
* hyperparameter search

The key phrase is:

```text
consistent API
```

Many scikit-learn objects behave in similar ways.

You create an estimator.

You call `fit`.

You call `predict`, `transform`, or `score`, depending on the object.

This consistency makes scikit-learn one of the best libraries for learning practical machine learning.

It teaches a workflow:

```text
prepare data
split data
fit model
evaluate model
tune model
validate assumptions
use the model carefully
```

This chapter is not about memorizing every algorithm.

It is about understanding the workflow that keeps machine learning honest.

---

# Part III Opening

Part III of Volume IV is about Machine Learning and AI Engineering.

The previous part studied data and scientific computing.

NumPy gave us arrays.

Pandas and Polars gave us table workflows.

SciPy gave us scientific algorithms.

Now we move into systems that learn patterns from data.

This is a major shift.

Traditional programming often looks like:

```text
rules + input -> output
```

Machine learning often looks like:

```text
data + objective -> learned model
model + new input -> prediction
```

The program is no longer only a hand-written set of rules.

Part of the behavior is learned from examples.

That makes machine learning powerful.

It also makes it dangerous when misunderstood.

A model can produce numbers confidently while being wrong.

A model can perform well on training data and fail on new data.

A model can learn shortcuts instead of useful structure.

A model can encode bias from its training data.

A model can look excellent because evaluation was flawed.

Part III begins with scikit-learn because it teaches the discipline of model building before the complexity of deep learning and modern AI systems.

The professional lesson is:

```text
machine learning is not only fitting models
it is designing reliable evaluation workflows
```

---

# Why scikit-learn Matters

scikit-learn matters because many real-world machine learning problems are not deep learning problems.

You may need to:

* predict customer churn
* classify support tickets
* estimate house prices
* detect fraudulent transactions
* cluster users
* rank leads
* predict equipment failure
* classify documents
* reduce feature dimensions
* build baseline models
* compare algorithms quickly

For these tasks, scikit-learn is often the right starting point.

It is mature, well documented, and built around a clear design.

It integrates naturally with:

* NumPy arrays
* SciPy sparse arrays
* Pandas DataFrames
* feature engineering workflows
* model selection tools
* evaluation metrics

It also provides many algorithms that are still professionally important:

* linear models
* logistic regression
* decision trees
* random forests
* gradient boosting
* support vector machines
* nearest neighbors
* naive Bayes
* k-means
* principal component analysis

Deep learning gets much attention.

But classical machine learning still solves many business problems very well.

scikit-learn is the library that makes those methods practical in Python.

---

# The Estimator API

The most important scikit-learn idea is the estimator API.

An estimator is an object that can learn from data.

Most estimators implement:

```python
fit(X, y)
```

Supervised estimators learn from features `X` and targets `y`.

Example:

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)
```

After fitting, many supervised models implement:

```python
predict(X)
```

Example:

```python
y_pred = model.predict(X_test)
```

Some models also implement:

```python
predict_proba(X)
decision_function(X)
score(X, y)
```

Transformers implement:

```python
fit(X, y=None)
transform(X)
fit_transform(X, y=None)
```

Examples include:

* scalers
* encoders
* imputers
* dimensionality reducers
* feature extractors

The consistency matters.

Once you understand `fit`, `predict`, and `transform`, much of scikit-learn becomes easier.

---

# Features and Targets

Machine learning data is usually separated into:

* features
* target

Features are the input variables.

Target is what we want to predict.

In scikit-learn, the common convention is:

```python
X = features
y = target
```

`X` is often a two-dimensional array-like object:

```text
(n_samples, n_features)
```

Rows are samples.

Columns are features.

`y` is often one-dimensional:

```text
(n_samples,)
```

Example:

```python
X = df[["age", "income", "visits"]]
y = df["churned"]
```

This shape discipline connects directly to NumPy.

If `X` has 1,000 rows, `y` should have 1,000 entries.

Each row in `X` corresponds to one target value in `y`.

Many scikit-learn errors are shape errors.

Before fitting, inspect:

```python
X.shape
y.shape
```

This habit never stops being useful.

---

# Supervised Learning

Supervised learning uses labeled examples.

Each training example has input features and a known target.

Two major supervised learning tasks are:

* classification
* regression

Classification predicts categories.

Examples:

* spam or not spam
* churn or not churn
* fraud or not fraud
* disease class
* document topic

Regression predicts numerical values.

Examples:

* house price
* delivery time
* revenue
* temperature
* demand

The distinction matters because it affects:

* algorithms
* metrics
* target representation
* error interpretation
* business decisions

Do not treat every prediction as just "model output."

First classify the task.

Is the target categorical?

Classification.

Is the target continuous?

Regression.

That decision shapes the workflow.

---

# Unsupervised Learning

Unsupervised learning uses data without target labels.

Common tasks include:

* clustering
* dimensionality reduction
* anomaly detection
* density estimation

Clustering groups similar examples.

Dimensionality reduction creates lower-dimensional representations.

Anomaly detection identifies unusual points.

Example algorithms include:

* KMeans
* DBSCAN
* PCA
* t-SNE
* IsolationForest

Unsupervised learning is harder to evaluate because there may be no obvious correct answer.

If a clustering algorithm produces five clusters, what proves those clusters are meaningful?

The answer depends on the domain.

Unsupervised methods are useful, but they are easier to overinterpret.

A professional habit is:

```text
connect unsupervised results back to domain evidence
```

Do not assume a cluster is real just because an algorithm returned it.

---

# Train-Test Split

The first rule of practical machine learning is:

```text
evaluate on data the model did not train on
```

If you fit and evaluate on the same data, the score can be misleading.

The model may have memorized patterns that do not generalize.

Use a train-test split:

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)
```

Training data is used to fit the model.

Test data is held back for evaluation.

`random_state` makes the split reproducible.

For classification, stratification is often useful:

```python
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)
```

Stratification preserves class proportions in train and test splits.

This is important when classes are imbalanced.

---

# Data Leakage

Data leakage is one of the most serious machine learning mistakes.

Leakage happens when information from outside the training process influences the model during training.

This can make evaluation look better than reality.

Examples:

* scaling the full dataset before train-test split
* imputing missing values using full-dataset statistics
* selecting features using the test set
* including future information in historical predictions
* using target-derived columns as features
* duplicate records appearing in both train and test
* preprocessing text vocabulary on the full dataset before splitting

Leakage is dangerous because the code still runs.

The score may look excellent.

But the model fails in production.

The professional rule is:

```text
anything learned from data must be learned from training data only
```

This is one reason pipelines are so important.

Pipelines help ensure preprocessing is fit on training folds and applied to validation or test folds correctly.

---

# A First Classification Model

Here is a compact classification workflow:

```python
from sklearn.datasets import load_breast_cancer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

data = load_breast_cancer()
X = data.data
y = data.target

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(max_iter=1000)),
])

pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)
```

This example includes:

* dataset
* split
* preprocessing
* model
* pipeline
* fitting
* prediction
* metric

The pipeline matters.

The scaler is fit only on training data.

Then the same scaling transformation is applied to test data.

This avoids a common leakage mistake.

---

# Preprocessing

Raw features often need preprocessing.

Common preprocessing tasks:

* scaling numerical features
* imputing missing values
* encoding categorical variables
* normalizing vectors
* generating polynomial features
* transforming skewed distributions
* extracting features from text

Scaling matters for many algorithms.

Examples:

* logistic regression
* support vector machines
* k-nearest neighbors
* neural networks
* PCA

Tree-based models usually care less about feature scaling.

That does not mean preprocessing is irrelevant.

Missing values, categorical variables, and text still need treatment.

The key is:

```text
preprocessing must be part of the model workflow
```

It should not be a loose notebook cell that is easy to apply differently during training and prediction.

---

# Scaling

`StandardScaler` standardizes features:

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_train)
```

It subtracts the mean and divides by the standard deviation learned from training data.

For test data:

```python
X_test_scaled = scaler.transform(X_test)
```

Notice the difference:

```text
fit_transform on training data
transform on test data
```

Do not fit the scaler on the test data.

That would leak test distribution information into evaluation.

Other scalers include:

* `MinMaxScaler`
* `RobustScaler`
* `MaxAbsScaler`

Choose scaling based on data and model.

Outliers can distort mean and standard deviation.

`RobustScaler` may be useful when outliers are significant.

---

# Encoding Categorical Features

Many machine learning algorithms require numeric input.

Categorical features must be encoded.

Example categorical column:

```text
region: North, South, East, West
```

One common encoder is `OneHotEncoder`.

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(handle_unknown="ignore")
```

One-hot encoding creates indicator columns.

For example:

```text
region_North
region_South
region_East
region_West
```

`handle_unknown="ignore"` is important when production data may contain categories not seen during training.

Without this, unseen categories can cause errors.

Encoding is not always simple.

High-cardinality categorical variables can create many columns.

Ordinal categories need different handling than nominal categories.

For example:

```text
low < medium < high
```

has order.

```text
red, blue, green
```

does not.

Encoding should respect meaning.

---

# Missing Values

Missing values need explicit handling.

scikit-learn provides imputers.

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")
```

Common strategies:

* mean
* median
* most frequent
* constant

Missing values are not merely a technical issue.

As in Pandas, missingness has meaning.

Sometimes missing means unknown.

Sometimes missing means not applicable.

Sometimes missing itself is predictive.

In some workflows, you may add an indicator:

```python
SimpleImputer(strategy="median", add_indicator=True)
```

This lets the model know which values were originally missing.

The professional habit is:

```text
impute inside the training pipeline
```

Do not compute imputation statistics on the full dataset before splitting.

---

# ColumnTransformer

Real datasets often have different column types.

Numerical columns need scaling and imputation.

Categorical columns need imputation and encoding.

Text columns may need vectorization.

`ColumnTransformer` lets you apply different transformations to different columns.

```python
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.pipeline import Pipeline

numeric_features = ["age", "income"]
categorical_features = ["region", "plan"]

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
])

categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("encoder", OneHotEncoder(handle_unknown="ignore")),
])

preprocessor = ColumnTransformer([
    ("num", numeric_pipeline, numeric_features),
    ("cat", categorical_pipeline, categorical_features),
])
```

Then combine with a model:

```python
from sklearn.linear_model import LogisticRegression

model = Pipeline([
    ("preprocessor", preprocessor),
    ("classifier", LogisticRegression(max_iter=1000)),
])
```

This is one of the most important scikit-learn patterns.

It keeps preprocessing and modeling together.

It reduces leakage.

It makes training and prediction consistent.

---

# Pipelines

A pipeline chains steps.

Intermediate steps are usually transformers.

The final step is often an estimator.

Example:

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("classifier", SVC()),
])
```

Fit the whole pipeline:

```python
pipeline.fit(X_train, y_train)
```

Predict with the whole pipeline:

```python
pipeline.predict(X_test)
```

This means the test data receives the same preprocessing learned from training data.

Pipelines also support hyperparameter search.

You can tune parameters using step names:

```python
param_grid = {
    "classifier__C": [0.1, 1, 10],
    "classifier__kernel": ["linear", "rbf"],
}
```

The double underscore connects pipeline step name to parameter name.

Pipelines are not optional polish.

They are how professional scikit-learn workflows stay coherent.

---

# Metrics

A metric measures model performance.

The right metric depends on the task.

For classification:

* accuracy
* precision
* recall
* F1 score
* ROC AUC
* average precision
* log loss
* confusion matrix

For regression:

* mean absolute error
* mean squared error
* root mean squared error
* R squared
* median absolute error

Do not choose metrics casually.

Accuracy can be misleading with imbalanced data.

Suppose only 1% of transactions are fraudulent.

A model that predicts "not fraud" for every transaction is 99% accurate.

It is also useless for fraud detection.

For imbalanced classification, precision and recall may matter more.

The metric should reflect the real cost of errors.

False positives and false negatives often have different consequences.

Metric choice is a product and domain decision, not just a technical decision.

---

# Confusion Matrix

For binary classification, the confusion matrix summarizes prediction outcomes.

It contains:

* true positives
* true negatives
* false positives
* false negatives

Example:

```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)
```

Interpretation depends on which class is positive.

For fraud detection:

* true positive: fraud correctly flagged
* false positive: legitimate transaction flagged as fraud
* false negative: fraud missed
* true negative: legitimate transaction correctly allowed

False positives annoy users.

False negatives lose money.

The best model depends on the tradeoff.

Never discuss classification performance only as one number when the error types matter.

---

# Precision and Recall

Precision answers:

```text
of the items predicted positive, how many were truly positive?
```

Recall answers:

```text
of the truly positive items, how many did we find?
```

Example:

```python
from sklearn.metrics import precision_score, recall_score, f1_score

precision = precision_score(y_test, y_pred)
recall = recall_score(y_test, y_pred)
f1 = f1_score(y_test, y_pred)
```

Precision matters when false positives are costly.

Recall matters when false negatives are costly.

F1 balances precision and recall.

But F1 is not always the best metric.

Sometimes you care much more about recall.

Sometimes precision matters more.

Sometimes probability calibration matters.

Choose the metric that matches the decision the model supports.

---

# Cross-Validation

A single train-test split can be noisy.

Cross-validation evaluates a model across multiple splits.

Example:

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(
    pipeline,
    X,
    y,
    cv=5,
    scoring="accuracy",
)
```

This trains and evaluates the pipeline five times on different folds.

Cross-validation gives a better sense of variability.

It is especially useful when datasets are not huge.

But cross-validation must still avoid leakage.

Preprocessing should be inside the pipeline.

If you scale the whole dataset before cross-validation, validation folds leak into training folds.

The safe pattern is:

```text
pipeline first, cross-validation second
```

---

# Hyperparameter Search

Models have parameters learned from data.

They also have hyperparameters chosen before training.

Examples:

* regularization strength
* tree depth
* number of estimators
* kernel type
* number of neighbors
* learning rate

Use tools such as `GridSearchCV` or `RandomizedSearchCV`.

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    "classifier__C": [0.1, 1, 10],
}

search = GridSearchCV(
    pipeline,
    param_grid=param_grid,
    cv=5,
    scoring="f1",
)

search.fit(X_train, y_train)
```

Inspect:

```python
search.best_params_
search.best_score_
```

Then evaluate the final selected model on the held-out test set.

Do not tune hyperparameters on the test set repeatedly.

The test set should remain a final estimate, not a feedback tool.

---

# Classification Algorithms

scikit-learn includes many classification algorithms.

Examples:

* logistic regression
* k-nearest neighbors
* support vector machines
* decision trees
* random forests
* gradient boosting
* naive Bayes

Each has different strengths.

Logistic regression is a strong baseline for linear classification.

K-nearest neighbors is intuitive but can struggle with high dimensions and scaling.

Support vector machines can work well but require scaling and careful parameter choice.

Decision trees are interpretable but can overfit.

Random forests reduce overfitting by averaging many trees.

Gradient boosting can be very strong on tabular data.

Naive Bayes can be effective for text and simple probabilistic classification.

Do not start with the most complex model automatically.

Start with a reasonable baseline.

Then improve with evidence.

---

# Regression Algorithms

Regression predicts continuous values.

scikit-learn regression algorithms include:

* linear regression
* ridge regression
* lasso regression
* elastic net
* k-nearest neighbors regression
* decision tree regression
* random forest regression
* gradient boosting regression
* support vector regression

Metrics include:

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
```

Mean absolute error is easy to interpret because it is in target units.

Mean squared error penalizes large errors more strongly.

R squared describes variance explained, but can be misleading if used blindly.

Regression evaluation should include residual inspection.

Ask:

* Are errors larger for certain ranges?
* Are predictions biased high or low?
* Are outliers dominating the score?
* Does the model extrapolate dangerously?
* Is the target distribution skewed?

Numbers need context.

---

# Clustering

Clustering finds groups without labels.

Common scikit-learn clustering algorithms include:

* KMeans
* DBSCAN
* AgglomerativeClustering
* GaussianMixture

KMeans requires choosing the number of clusters.

DBSCAN can find irregular clusters and noise, but depends on distance scale and parameters.

Hierarchical clustering can reveal nested structure.

Clustering is useful for:

* exploration
* segmentation
* anomaly investigation
* grouping similar items
* feature generation

But clusters are not automatically meaningful.

A clustering result should be validated with:

* domain knowledge
* stability checks
* visualization when possible
* downstream usefulness
* careful interpretation of features and scaling

Scaling matters a lot in clustering because distance matters.

If one feature has a much larger numeric range than another, it can dominate distance calculations.

---

# Dimensionality Reduction

Dimensionality reduction transforms data into fewer features.

Reasons:

* visualization
* compression
* noise reduction
* faster models
* feature extraction
* understanding structure

Common methods:

* PCA
* TruncatedSVD
* NMF
* t-SNE
* manifold learning methods

PCA is a linear method.

It finds directions of high variance.

Example:

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X_scaled)
```

PCA is sensitive to scaling.

For many datasets, scale before PCA.

t-SNE and similar methods are useful for visualization but should not be overinterpreted as proof of true clusters.

Dimensionality reduction can reveal patterns.

It can also create visual illusions.

---

# Text Features

scikit-learn includes tools for converting text into numerical features.

Examples:

```python
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
```

`CountVectorizer` creates token count features.

`TfidfVectorizer` creates term-frequency inverse-document-frequency features.

Example:

```python
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression

text_model = Pipeline([
    ("tfidf", TfidfVectorizer()),
    ("classifier", LogisticRegression(max_iter=1000)),
])
```

This can work surprisingly well for:

* spam detection
* topic classification
* sentiment baselines
* ticket routing
* document tagging

Modern NLP often uses embeddings and transformer models.

But classical text features remain useful for baselines, interpretability, speed, and small-data problems.

---

# Imbalanced Data

Many real classification problems are imbalanced.

Examples:

* fraud detection
* disease detection
* rare failure prediction
* abuse detection
* churn prediction in some settings

If one class is rare, accuracy may be misleading.

Use metrics such as:

* precision
* recall
* F1
* ROC AUC
* precision-recall curves
* average precision

Some models support class weights:

```python
LogisticRegression(class_weight="balanced")
```

Other approaches include resampling techniques, threshold tuning, and collecting more data.

But be careful.

Resampling must happen inside the training process, not before splitting in a way that leaks information.

The core question is:

```text
what mistakes are expensive?
```

That determines the evaluation strategy.

---

# Probability and Thresholds

Many classifiers can produce scores or probabilities.

Example:

```python
probabilities = model.predict_proba(X_test)[:, 1]
```

The default decision threshold is often `0.5`.

But `0.5` is not sacred.

For fraud detection, you might flag transactions above `0.2`.

For high-stakes medical screening, you may choose a threshold that prioritizes recall.

For customer outreach, you may choose a threshold based on campaign capacity.

Threshold choice should be tied to costs and decisions.

Do not confuse:

```text
predicting probabilities
```

with:

```text
choosing actions
```

The model estimates.

The system decides.

That decision should be designed.

---

# Model Interpretation

Some scikit-learn models are easier to interpret than others.

Linear model coefficients can be informative when features are properly scaled and encoded.

Decision trees can be visualized, though large trees become hard to understand.

Random forests and gradient boosting models can provide feature importance, but feature importance has caveats.

Permutation importance can help estimate how much a feature affects model performance.

Example:

```python
from sklearn.inspection import permutation_importance

result = permutation_importance(model, X_test, y_test, n_repeats=10, random_state=42)
```

Interpretation should be careful.

Correlated features can confuse importance.

Encoded categorical variables can spread importance across many columns.

Feature importance does not prove causation.

An interpretable model is not automatically a fair or correct model.

Use interpretation tools as evidence, not as final truth.

---

# Reproducibility

Machine learning workflows should be reproducible.

Set random states where appropriate:

```python
train_test_split(..., random_state=42)
RandomForestClassifier(random_state=42)
```

Track:

* dataset version
* feature list
* preprocessing steps
* model type
* hyperparameters
* random seeds
* metrics
* train-test split strategy
* package versions

Reproducibility is not only for science.

It matters in business systems too.

If a model changes behavior, you need to know why.

Was it new data?

Different preprocessing?

Different parameters?

Different package version?

Different random split?

Without tracking, debugging model behavior becomes guesswork.

---

# A Complete Tabular Pipeline

Here is a realistic shape for tabular classification.

```python
from sklearn.compose import ColumnTransformer
from sklearn.ensemble import RandomForestClassifier
from sklearn.impute import SimpleImputer
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

numeric_features = ["age", "income", "visits"]
categorical_features = ["region", "plan"]

X = df[numeric_features + categorical_features]
y = df["churned"]

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
])

categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("encoder", OneHotEncoder(handle_unknown="ignore")),
])

preprocessor = ColumnTransformer([
    ("num", numeric_pipeline, numeric_features),
    ("cat", categorical_pipeline, categorical_features),
])

model = Pipeline([
    ("preprocessor", preprocessor),
    ("classifier", RandomForestClassifier(random_state=42)),
])

model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(classification_report(y_test, y_pred))
```

This example includes many professional habits:

* explicit feature lists
* train-test split
* stratification
* numeric preprocessing
* categorical preprocessing
* missing value handling
* unknown category handling
* pipeline composition
* model fitting
* held-out evaluation

It is not the only correct structure.

But it is a strong baseline pattern.

---

# Saving Models

After training, models are often saved.

The scikit-learn ecosystem commonly uses tools such as `joblib`.

Example:

```python
import joblib

joblib.dump(model, "churn_model.joblib")
```

Load later:

```python
model = joblib.load("churn_model.joblib")
```

Saving the whole pipeline is usually better than saving only the estimator.

The pipeline includes preprocessing.

If you save only the classifier, production data may not be transformed the same way as training data.

Model files should be treated carefully.

They may not be safe to load from untrusted sources.

They are also tied to code and package versions.

For production systems, track the model artifact, code version, data version, and environment.

---

# Deployment Is Not Training

scikit-learn trains models.

It does not magically solve deployment.

Production use requires decisions about:

* input validation
* feature availability
* model artifact loading
* prediction latency
* batch versus real-time inference
* monitoring
* logging
* drift detection
* retraining
* rollback
* security
* explainability
* business decision thresholds

A model that performs well in a notebook can still fail in production.

Reasons include:

* production data has different distributions
* features are missing or delayed
* categories appear that training never saw
* preprocessing is inconsistent
* model version is mismatched
* data contracts are not enforced
* user behavior changes

Training is one phase.

Operating a model is another.

This distinction becomes central in later chapters on AI Engineering and Machine Learning Engineering.

---

# Common Mistakes

The first common mistake is evaluating on training data.

Always evaluate on data the model did not train on.

The second common mistake is data leakage.

Preprocessing and feature selection must be learned from training data only.

The third common mistake is choosing accuracy for imbalanced problems.

Use metrics that reflect the real decision.

The fourth common mistake is skipping baselines.

A simple model gives context for whether complexity is helping.

The fifth common mistake is ignoring feature meaning.

Models do not understand business semantics automatically.

The sixth common mistake is tuning on the test set.

Use validation or cross-validation for tuning, and keep the test set final.

The seventh common mistake is treating probability threshold `0.5` as universal.

Thresholds should reflect costs and actions.

The eighth common mistake is interpreting feature importance causally.

Importance is not causation.

The ninth common mistake is saving only the estimator instead of the full preprocessing pipeline.

Training and inference must use the same transformations.

The tenth common mistake is thinking model training is the whole system.

Production machine learning includes monitoring, data contracts, retraining, and operational discipline.

---

# Professional scikit-learn Checklist

When using scikit-learn, practice these habits:

* Define the task clearly: classification, regression, clustering, or transformation.
* Separate features `X` and target `y`.
* Inspect shapes before fitting.
* Split data before fitting preprocessors.
* Use stratified splits for imbalanced classification when appropriate.
* Put preprocessing inside pipelines.
* Use `ColumnTransformer` for mixed feature types.
* Choose metrics based on real error costs.
* Build a simple baseline first.
* Use cross-validation for more reliable model comparison.
* Use hyperparameter search without touching the final test set repeatedly.
* Inspect confusion matrices, residuals, or error patterns.
* Handle missing values deliberately.
* Handle unseen categories deliberately.
* Track random states and package versions.
* Save the full pipeline, not just the model.
* Treat deployment as a separate engineering problem.

This checklist protects against the most common ways ML projects fool themselves.

---

# Summary

scikit-learn is Python's core library for classical machine learning workflows.

It provides a consistent estimator API built around `fit`, `predict`, `transform`, and `score`.

It supports supervised learning, unsupervised learning, preprocessing, feature extraction, model selection, metrics, pipelines, and hyperparameter search.

The most important workflow skill is honest evaluation.

Models must be evaluated on data they did not train on.

Preprocessing must be learned from training data only.

Pipelines and `ColumnTransformer` help prevent leakage and keep training and inference consistent.

Metric choice must match the task and real-world cost of errors.

Cross-validation and hyperparameter search help compare models more reliably.

scikit-learn is powerful for tabular data, classical text features, baseline models, interpretable workflows, and many practical business prediction tasks.

Its limits matter too.

It is not a deep learning framework, not a full deployment platform, and not a replacement for data quality, domain understanding, or operational monitoring.

The central lesson is:

```text
scikit-learn turns arrays and tables into disciplined machine learning workflows
```

Used well, it teaches not only how to fit models, but how to avoid believing models too easily.

---

# Exercises

1. Load a toy dataset from `sklearn.datasets`.

2. Split the data into training and test sets with `train_test_split`.

3. Fit a simple classification model and evaluate it on the test set.

4. Build a pipeline with `StandardScaler` and `LogisticRegression`.

5. Compare evaluation with and without stratified splitting on an imbalanced dataset.

6. Create a `ColumnTransformer` for numeric and categorical columns.

7. Add `SimpleImputer` to handle missing values.

8. Add `OneHotEncoder(handle_unknown="ignore")` for categorical features.

9. Use `cross_val_score` to evaluate a pipeline.

10. Use `GridSearchCV` to tune one hyperparameter.

11. Print a confusion matrix and explain false positives and false negatives.

12. Compare accuracy, precision, recall, and F1 on the same predictions.

13. Fit a regression model and evaluate mean absolute error.

14. Inspect regression residuals and describe any pattern.

15. Fit a KMeans model and discuss how you would validate the clusters.

16. Use PCA to reduce a dataset to two dimensions.

17. Build a text classification pipeline with `TfidfVectorizer`.

18. Save a fitted pipeline with `joblib`.

19. Explain one example of data leakage and how a pipeline prevents it.

20. Design a production checklist for a scikit-learn model used in a real product.

---

# Preview of Chapter 93

Chapter 92 studied scikit-learn.

We learned how machine learning workflows are built from features, targets, estimators, transformers, train-test splits, preprocessing, pipelines, metrics, cross-validation, hyperparameter search, and careful evaluation.

Next we study PyTorch.

PyTorch is a deep learning framework built around tensors, automatic differentiation, neural network modules, optimizers, datasets, training loops, GPUs, and flexible research-to-production workflows.

The transition is:

```text
scikit-learn provides structured classical machine learning workflows
PyTorch gives Python a flexible engine for deep learning
```

Chapter 93 will show how PyTorch extends array thinking into differentiable programming and neural network training.
