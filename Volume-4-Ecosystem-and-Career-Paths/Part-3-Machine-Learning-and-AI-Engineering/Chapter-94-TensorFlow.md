# Chapter 94 — TensorFlow

TensorFlow is a machine learning and deep learning platform.

PyTorch gives you a flexible tensor-first framework that many people experience through explicit training loops.

TensorFlow also gives you tensors, automatic differentiation, neural network layers, optimizers, datasets, accelerators, and custom training loops.

But TensorFlow is often understood as a broader platform.

It includes tools and pathways for:

* model building
* Keras high-level APIs
* eager execution
* graph execution
* data input pipelines
* training loops
* model saving
* serving
* mobile and edge deployment
* browser deployment
* production ML pipelines
* distributed training
* profiling and performance

This is why TensorFlow deserves its own chapter after PyTorch.

The two frameworks overlap.

They also have different cultures and strengths.

PyTorch often feels like:

```text
write Pythonic tensor code and control the training loop
```

TensorFlow often feels like:

```text
build models inside a broad training and deployment ecosystem
```

That is a simplification, but it is a useful starting point.

This chapter explains TensorFlow's core ideas and where it fits in the Python AI ecosystem.

---

# Why TensorFlow Matters

TensorFlow matters because deep learning is not only training.

A complete machine learning system may need:

* efficient training
* repeatable data pipelines
* model checkpoints
* exported model artifacts
* model serving
* mobile inference
* browser inference
* distributed training
* monitoring
* production pipelines
* hardware acceleration

TensorFlow has historically invested heavily in this full lifecycle.

It is widely used in:

* production ML systems
* mobile ML
* edge ML
* computer vision
* NLP
* recommendation systems
* structured data models
* research
* enterprise machine learning platforms

TensorFlow 2 made the framework more approachable by emphasizing eager execution and Keras.

That means TensorFlow code can feel more immediate and Pythonic than older TensorFlow 1 code.

But TensorFlow still has a strong graph execution story.

This combination matters:

```text
eager execution helps development
graph execution helps optimization and deployment
```

Understanding that tension helps you understand TensorFlow.

---

# TensorFlow After PyTorch

The previous chapter studied PyTorch.

PyTorch taught:

* tensors
* devices
* autograd
* modules
* losses
* optimizers
* datasets
* dataloaders
* explicit training loops
* checkpoints
* GPU usage

TensorFlow has equivalent concepts.

But the most common TensorFlow beginner path often goes through Keras:

```python
model = keras.Sequential([...])
model.compile(...)
model.fit(...)
model.evaluate(...)
model.predict(...)
```

This is closer to scikit-learn in feel than a raw PyTorch training loop.

However, TensorFlow also supports custom training loops with `tf.GradientTape`.

So TensorFlow has multiple levels:

```text
Keras built-in training APIs
custom Keras models
low-level TensorFlow tensors and GradientTape
tf.function graph execution
deployment formats and tools
```

The professional skill is knowing which level you need.

Do not use low-level custom loops when the built-in Keras workflow is enough.

Do not force the built-in workflow when your model or training process genuinely needs customization.

---

# Importing TensorFlow

The standard import is:

```python
import tensorflow as tf
```

Keras is commonly used as the high-level model API:

```python
from tensorflow import keras
from tensorflow.keras import layers
```

You may also see standalone Keras imports in modern code:

```python
import keras
from keras import layers
```

Keras has evolved as its own multi-backend project, while TensorFlow also documents Keras workflows.

For this chapter, the important practical idea is:

```text
Keras provides the high-level model-building interface most TensorFlow users start with
```

TensorFlow provides the broader tensor, graph, data, training, and deployment ecosystem.

---

# Tensors

The central TensorFlow object is the tensor.

Create tensors:

```python
import tensorflow as tf

scalar = tf.constant(3.0)
vector = tf.constant([1.0, 2.0, 3.0])
matrix = tf.constant([[1.0, 2.0], [3.0, 4.0]])
```

Inspect shape:

```python
matrix.shape
```

Inspect dtype:

```python
matrix.dtype
```

TensorFlow tensors are immutable.

If you perform an operation, you create a new tensor:

```python
x = tf.constant([1.0, 2.0, 3.0])
y = x * 2
```

This is similar to the array thinking from NumPy and PyTorch.

The essential habits remain:

```text
know the shape
know the dtype
know what operation is being applied
```

Shape errors remain one of the most common sources of deep learning bugs.

---

# Variables

Model parameters need to change during training.

TensorFlow uses variables for mutable state.

```python
w = tf.Variable(1.0)
```

You can assign:

```python
w.assign(2.0)
```

Variables are used for trainable parameters such as weights and biases.

In normal Keras model code, you often do not create variables manually.

Keras layers create and manage variables for you.

Example:

```python
layer = layers.Dense(10)
```

When the layer is built, it creates variables for weights and biases.

The distinction is:

```text
Tensor is a value
Variable is mutable state
```

This distinction matters when writing custom models, custom layers, or low-level training loops.

---

# Tensor Operations

TensorFlow supports many tensor operations:

```python
x = tf.constant([[1.0, 2.0], [3.0, 4.0]])
y = tf.constant([[10.0, 20.0], [30.0, 40.0]])

x + y
x * y
tf.reduce_sum(x)
tf.reduce_mean(x)
tf.matmul(x, y)
```

The `tf.reduce_*` functions reduce dimensions:

```python
tf.reduce_sum(x, axis=0)
tf.reduce_sum(x, axis=1)
```

Matrix multiplication:

```python
tf.matmul(x, y)
```

or:

```python
x @ y
```

Broadcasting works in TensorFlow too.

The NumPy lessons still apply:

* shapes matter
* axes matter
* dtype matters
* broadcasting can help
* broadcasting can also surprise you

Deep learning frameworks did not remove array thinking.

They made it more important.

---

# Eager Execution

TensorFlow 2 uses eager execution by default.

That means operations execute immediately.

Example:

```python
x = tf.constant([1.0, 2.0, 3.0])
y = x * 2
print(y)
```

You get a result immediately.

This is easier to debug than older graph-only workflows.

Eager execution lets you:

* inspect tensors directly
* use Python debugging tools
* write natural control flow
* experiment interactively
* build models incrementally

This is one reason TensorFlow 2 is much friendlier than TensorFlow 1.

But eager execution is not the whole story.

TensorFlow can also convert Python functions into graphs with `tf.function`.

That graph capability is important for performance and deployment.

---

# Graphs and tf.function

`tf.function` converts a Python function into a TensorFlow graph function.

Example:

```python
@tf.function
def add_and_square(x, y):
    return tf.square(x + y)
```

Calling this function can execute as a graph.

Graphs can help TensorFlow:

* optimize computation
* serialize execution
* run efficiently on accelerators
* deploy models more easily
* reduce Python overhead

But graphs also introduce constraints.

Python side effects may not behave the way you expect.

Debugging can be harder.

Tensor shapes and input signatures can affect tracing behavior.

The professional rule is:

```text
write clear eager code first
then use tf.function when graph execution helps
```

Do not use `tf.function` as a magical speed sticker.

It is a compilation boundary.

Understand what crosses that boundary.

---

# Automatic Differentiation

TensorFlow uses `tf.GradientTape` for automatic differentiation.

Example:

```python
x = tf.Variable(2.0)

with tf.GradientTape() as tape:
    y = x * x + 3 * x

gradient = tape.gradient(y, x)
```

The derivative of:

```text
x^2 + 3x
```

at `x = 2` is:

```text
7
```

TensorFlow computes it.

This is the same deep learning foundation we saw in PyTorch:

```text
define computation
compute loss
differentiate loss with respect to trainable variables
update variables
```

Keras usually hides the gradient tape inside `model.fit`.

Custom training loops expose it.

---

# Keras

Keras is the high-level API most TensorFlow users meet first.

Keras provides:

* layers
* models
* losses
* optimizers
* metrics
* callbacks
* preprocessing layers
* saving and serialization tools
* built-in training loops

A simple model:

```python
from tensorflow import keras
from tensorflow.keras import layers

model = keras.Sequential([
    layers.Dense(64, activation="relu"),
    layers.Dense(1),
])
```

Keras is designed to make common deep learning workflows concise.

Instead of writing the full training loop, you can compile and fit:

```python
model.compile(
    optimizer="adam",
    loss="mse",
    metrics=["mae"],
)

model.fit(X_train, y_train, epochs=10, validation_data=(X_val, y_val))
```

This is one of TensorFlow's main beginner-friendly paths.

It is also used professionally when the built-in abstractions fit the problem.

---

# Sequential Models

`Sequential` is the simplest Keras model style.

It represents a stack of layers.

```python
model = keras.Sequential([
    layers.Input(shape=(10,)),
    layers.Dense(64, activation="relu"),
    layers.Dense(32, activation="relu"),
    layers.Dense(1),
])
```

This works well when data flows through one layer after another.

Use Sequential when the architecture is linear:

```text
input -> layer -> layer -> output
```

Sequential is not ideal for:

* multiple inputs
* multiple outputs
* skip connections
* shared layers
* complex branching
* models with unusual graph structure

For those, use the Functional API or subclassing.

Start simple.

Move to more flexible APIs only when the model needs them.

---

# Functional API

The Keras Functional API builds models as graphs of layers.

Example:

```python
inputs = keras.Input(shape=(10,))
x = layers.Dense(64, activation="relu")(inputs)
x = layers.Dense(32, activation="relu")(x)
outputs = layers.Dense(1)(x)

model = keras.Model(inputs=inputs, outputs=outputs)
```

This style makes data flow explicit.

It supports:

* multiple inputs
* multiple outputs
* branching
* merging
* shared layers
* skip connections

The Functional API is often the best professional default for nontrivial Keras models.

It is more explicit than Sequential and less open-ended than subclassing.

It also gives Keras a clear model graph, which helps with summaries, serialization, and visualization.

---

# Model Subclassing

Subclassing gives maximum flexibility.

```python
class MyModel(keras.Model):
    def __init__(self):
        super().__init__()
        self.dense1 = layers.Dense(64, activation="relu")
        self.dense2 = layers.Dense(1)

    def call(self, inputs):
        x = self.dense1(inputs)
        return self.dense2(x)
```

This resembles PyTorch's `nn.Module` style.

Subclassing is useful when:

* the forward pass needs custom Python logic
* the architecture is dynamic
* layers are reused in unusual ways
* the model does not fit a clean static graph

But subclassing can make serialization and inspection harder than Functional models.

Use it when needed.

Do not use it only because it feels more advanced.

The professional choice is:

```text
Sequential for simple stacks
Functional API for explicit model graphs
subclassing for custom behavior
```

---

# Compile

Keras `compile` configures training.

```python
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss=keras.losses.BinaryCrossentropy(from_logits=True),
    metrics=[keras.metrics.BinaryAccuracy()],
)
```

Compile answers:

* how should parameters be updated?
* what loss should be optimized?
* what metrics should be tracked?

The optimizer updates weights.

The loss drives learning.

Metrics help evaluate performance.

As with PyTorch, the loss must match the model output and target format.

If your model outputs logits, the loss must know that.

For example:

```python
keras.losses.BinaryCrossentropy(from_logits=True)
```

Mismatching logits, probabilities, labels, and losses is a common deep learning mistake.

---

# Fit, Evaluate, Predict

Keras built-in training uses:

```python
model.fit(...)
```

Evaluation:

```python
model.evaluate(...)
```

Prediction:

```python
model.predict(...)
```

Example:

```python
history = model.fit(
    X_train,
    y_train,
    epochs=20,
    batch_size=32,
    validation_data=(X_val, y_val),
)

test_metrics = model.evaluate(X_test, y_test)
predictions = model.predict(X_new)
```

The `history` object records training metrics over epochs.

This is useful for plotting learning curves.

Built-in Keras training is often enough for:

* straightforward supervised learning
* image classification
* regression
* text classification
* many transfer learning workflows
* many structured data problems

Use it when it fits.

Custom loops are not inherently more professional.

Clear simple code is professional.

---

# Callbacks

Keras callbacks run during training.

Common callbacks:

* `EarlyStopping`
* `ModelCheckpoint`
* `TensorBoard`
* `ReduceLROnPlateau`
* custom callbacks

Example:

```python
callbacks = [
    keras.callbacks.EarlyStopping(
        monitor="val_loss",
        patience=3,
        restore_best_weights=True,
    ),
    keras.callbacks.ModelCheckpoint(
        "best_model.keras",
        save_best_only=True,
    ),
]

model.fit(
    X_train,
    y_train,
    validation_data=(X_val, y_val),
    epochs=50,
    callbacks=callbacks,
)
```

Callbacks keep training behavior organized.

Early stopping helps reduce overfitting.

Model checkpointing preserves useful model states.

TensorBoard supports visualization.

Callbacks are one of Keras's strengths.

They let you customize training without rewriting the whole loop.

---

# Custom Training Loops

Sometimes built-in `fit` is not enough.

You may need:

* custom losses
* multiple optimizers
* unusual gradient behavior
* reinforcement learning
* adversarial training
* custom logging
* complex training schedules

Then use `tf.GradientTape`.

Example:

```python
optimizer = keras.optimizers.Adam(learning_rate=1e-3)
loss_fn = keras.losses.MeanSquaredError()

for x_batch, y_batch in train_dataset:
    with tf.GradientTape() as tape:
        predictions = model(x_batch, training=True)
        loss = loss_fn(y_batch, predictions)

    gradients = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
```

This resembles PyTorch's explicit training loop.

Important details:

* pass `training=True` during training
* compute loss inside the tape
* compute gradients with respect to trainable variables
* apply gradients through the optimizer

Custom loops give control.

They also give you more ways to make mistakes.

---

# tf.data

`tf.data` builds input pipelines.

It can read, transform, shuffle, batch, prefetch, and stream data efficiently.

Example:

```python
dataset = tf.data.Dataset.from_tensor_slices((X_train, y_train))
dataset = dataset.shuffle(buffer_size=1000).batch(32).prefetch(tf.data.AUTOTUNE)
```

Common pipeline steps:

* create dataset
* map preprocessing
* shuffle
* batch
* cache when appropriate
* prefetch

Example with mapping:

```python
dataset = dataset.map(preprocess, num_parallel_calls=tf.data.AUTOTUNE)
```

`tf.data` matters because training performance is often limited by data loading.

A fast model can sit idle if batches arrive slowly.

The goal is:

```text
keep the accelerator fed
```

Data input pipelines are part of model performance.

They are not an afterthought.

---

# Preprocessing Layers

Keras includes preprocessing layers.

Examples:

* normalization
* string lookup
* category encoding
* text vectorization
* image resizing
* image augmentation

Example:

```python
normalizer = layers.Normalization()
normalizer.adapt(X_train)
```

Then use it inside a model:

```python
model = keras.Sequential([
    normalizer,
    layers.Dense(64, activation="relu"),
    layers.Dense(1),
])
```

The `adapt` step learns preprocessing state from training data.

This is similar to fitting preprocessors in scikit-learn.

The same leakage rule applies:

```text
learn preprocessing state from training data only
```

Putting preprocessing inside the model can make deployment easier because preprocessing travels with the model.

---

# Saving Models

TensorFlow and Keras support model saving.

Modern Keras models can be saved with:

```python
model.save("model.keras")
```

Load:

```python
model = keras.models.load_model("model.keras")
```

TensorFlow also has SavedModel for export and serving workflows.

SavedModel preserves a TensorFlow program for deployment.

Saving matters because production systems need artifacts, not notebook state.

A model artifact should be tied to:

* code version
* data version
* preprocessing
* metrics
* training configuration
* dependency versions
* expected input schema

Saving a model is not the same as making a production system.

But it is a necessary step.

---

# TensorBoard

TensorBoard is TensorFlow's visualization toolkit.

It can show:

* scalar metrics
* training curves
* graphs
* histograms
* images
* embeddings
* profiling information

Use the Keras callback:

```python
tensorboard_callback = keras.callbacks.TensorBoard(log_dir="logs")

model.fit(
    X_train,
    y_train,
    epochs=10,
    callbacks=[tensorboard_callback],
)
```

TensorBoard helps answer:

* is training loss decreasing?
* is validation loss diverging?
* are gradients behaving strangely?
* is the input pipeline slow?
* is the model graph what I expect?

Visualization does not replace metrics.

It helps you understand the training process.

---

# Distribution Strategies

TensorFlow supports distributed training through distribution strategies.

Examples include:

* single-machine multi-GPU training
* multi-worker training
* TPU training

A common pattern:

```python
strategy = tf.distribute.MirroredStrategy()

with strategy.scope():
    model = build_model()
    model.compile(...)
```

Distribution strategies try to let you scale training without rewriting the entire model.

But distributed training introduces complexity:

* batch size changes
* learning rate tuning
* data sharding
* checkpointing
* communication overhead
* hardware differences
* failure handling

Do not distribute a broken single-device training job.

First make the model correct on one device.

Then scale.

---

# TensorFlow Lite

TensorFlow Lite is for deploying models on mobile and edge devices.

It targets environments such as:

* Android
* iOS
* embedded Linux
* microcontrollers in some workflows
* edge devices

Edge deployment has special constraints:

* limited memory
* limited compute
* battery usage
* latency
* offline inference
* model size
* hardware accelerators

TensorFlow Lite often involves conversion and optimization.

For example:

```python
converter = tf.lite.TFLiteConverter.from_saved_model("saved_model")
tflite_model = converter.convert()
```

Deployment on edge devices is not merely exporting a file.

You must test on target hardware.

The model that works on a workstation may be too slow or too large on a phone.

---

# TensorFlow Serving

TensorFlow Serving is designed for serving TensorFlow models in production.

It supports serving exported models through a dedicated serving system.

Serving concerns include:

* model versioning
* request batching
* latency
* throughput
* rollback
* monitoring
* hardware utilization
* input signatures

TensorFlow's serving ecosystem is one reason it has been attractive for production ML systems.

But serving is still engineering.

You need:

* clear input contracts
* validation
* monitoring
* rollback strategy
* performance tests
* security controls
* model governance

The model is only one part of the serving system.

---

# TensorFlow.js and TFX

TensorFlow.js brings machine learning to JavaScript environments.

It can run models in browsers or Node.js.

This is useful when:

* inference should happen client-side
* privacy benefits from local inference
* latency should avoid a server round trip
* interactive web ML experiences are needed

TFX is TensorFlow Extended.

It is a platform for production ML pipelines.

TFX concerns include:

* data ingestion
* validation
* transformation
* training
* evaluation
* serving
* pipeline orchestration

These tools show TensorFlow's platform orientation.

TensorFlow is not just a Python training library.

It is an ecosystem around training and operationalizing ML.

---

# TensorFlow Versus PyTorch

Both TensorFlow and PyTorch are major deep learning frameworks.

They can both train neural networks.

They can both use GPUs.

They can both compute gradients.

They can both build production systems.

The difference is often about workflow and ecosystem preference.

PyTorch is often praised for:

* Pythonic feel
* research flexibility
* explicit training loops
* dynamic model development
* strong adoption in research

TensorFlow is often praised for:

* Keras high-level workflows
* deployment ecosystem
* TensorFlow Lite
* TensorFlow Serving
* production pipeline tooling
* graph and platform orientation

These are tendencies, not absolute laws.

The right choice depends on:

* team expertise
* deployment target
* model type
* ecosystem dependencies
* production constraints
* existing codebase
* hardware strategy

Do not choose by slogan.

Choose by system needs.

---

# A Small Complete Example

Here is a compact Keras binary classification workflow.

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

normalizer = layers.Normalization()
normalizer.adapt(X_train)

model = keras.Sequential([
    layers.Input(shape=(X_train.shape[1],)),
    normalizer,
    layers.Dense(64, activation="relu"),
    layers.Dropout(0.3),
    layers.Dense(1),
])

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss=keras.losses.BinaryCrossentropy(from_logits=True),
    metrics=[
        keras.metrics.BinaryAccuracy(threshold=0.0),
        keras.metrics.AUC(from_logits=True),
    ],
)

callbacks = [
    keras.callbacks.EarlyStopping(
        monitor="val_loss",
        patience=3,
        restore_best_weights=True,
    ),
]

history = model.fit(
    X_train,
    y_train,
    validation_data=(X_val, y_val),
    epochs=50,
    batch_size=32,
    callbacks=callbacks,
)

test_metrics = model.evaluate(X_test, y_test)
model.save("classifier.keras")
```

This example shows:

* preprocessing layer
* model definition
* dropout
* optimizer
* loss
* metrics
* early stopping
* validation data
* test evaluation
* saving

It also shows a subtle detail:

```python
BinaryAccuracy(threshold=0.0)
```

Because the model outputs logits, the decision boundary for a binary logit is `0.0`, not `0.5`.

Details like this matter.

---

# Debugging TensorFlow Models

When a TensorFlow model does not learn, ask:

* Are input shapes correct?
* Are target shapes correct?
* Are dtypes correct?
* Does the loss match the output?
* Are logits and probabilities handled correctly?
* Is preprocessing adapted only on training data?
* Is validation data separate?
* Is the learning rate reasonable?
* Is the model too small or too large?
* Are callbacks stopping training too early?
* Is `training=True` used where needed in custom loops?
* Is `tf.function` hiding a debugging issue?
* Is the input pipeline producing the expected batches?

As with PyTorch, a powerful debugging trick is to overfit a tiny batch.

If a model cannot overfit a very small dataset, something basic is wrong.

Check the simplest possible version before adding complexity.

---

# Common Mistakes

The first common mistake is using the wrong loss for the output.

Logits and probabilities are not the same.

The second common mistake is adapting preprocessing layers on all data.

Adapt preprocessing on training data only.

The third common mistake is using `model.fit` without understanding what it does.

High-level APIs are helpful, but you still need to know the training process.

The fourth common mistake is writing custom training loops before they are needed.

Use built-in Keras training when it fits.

The fifth common mistake is using `tf.function` too early.

Debug eagerly first.

The sixth common mistake is ignoring input pipeline performance.

Slow data loading can waste expensive accelerators.

The seventh common mistake is saving only weights when deployment needs preprocessing and model structure.

Save the artifact that matches the deployment path.

The eighth common mistake is assuming TensorFlow Lite conversion guarantees good edge performance.

Test on target hardware.

The ninth common mistake is comparing TensorFlow and PyTorch as identity choices.

Choose based on project needs.

The tenth common mistake is treating deployment tools as a substitute for ML system design.

Production still needs contracts, monitoring, rollback, and governance.

---

# Professional TensorFlow Checklist

When using TensorFlow, practice these habits:

* Import TensorFlow as `tf`.
* Use Keras built-in workflows when they fit.
* Choose Sequential, Functional API, or subclassing deliberately.
* Inspect tensor shapes and dtypes.
* Match loss functions to outputs and targets.
* Use training data only for preprocessing adaptation.
* Keep validation and test roles separate.
* Use callbacks for early stopping and checkpointing.
* Use `tf.data` for scalable input pipelines.
* Use `tf.function` after eager code is correct.
* Use custom training loops only when needed.
* Save models in a format appropriate for the deployment target.
* Test exported models before trusting them.
* Use TensorBoard for training visibility.
* Profile before optimizing.
* Test TensorFlow Lite models on real target devices.
* Treat TensorFlow Serving and TFX as system components, not magic.
* Track data, code, model, and dependency versions.

These habits make TensorFlow work more reliable and less mysterious.

---

# Summary

TensorFlow is a broad machine learning platform centered on tensors, automatic differentiation, Keras, graph execution, data pipelines, accelerators, saving, serving, and deployment tooling.

TensorFlow 2 emphasizes eager execution and high-level Keras APIs, making everyday model development much more approachable.

Keras provides Sequential models, the Functional API, subclassed models, compile/fit/evaluate/predict workflows, callbacks, preprocessing layers, and serialization.

TensorFlow's lower-level APIs support custom training loops through `tf.GradientTape`.

`tf.data` helps build efficient input pipelines.

`tf.function` converts Python functions into graph functions when performance or deployment benefits justify it.

TensorFlow's ecosystem includes TensorBoard, TensorFlow Lite, TensorFlow Serving, TensorFlow.js, and TFX.

Compared with PyTorch, TensorFlow is often associated with a broader platform and deployment story, while PyTorch is often associated with flexible Pythonic research workflows.

Both are powerful.

The central lesson is:

```text
TensorFlow combines deep learning APIs with a broad path from training to deployment
```

Use it when its abstractions and ecosystem match the system you are building.

---

# Exercises

1. Create scalar, vector, and matrix tensors with TensorFlow.

2. Inspect tensor shape and dtype.

3. Create a `tf.Variable` and update it with `assign`.

4. Use `tf.GradientTape` to compute the derivative of a simple function.

5. Build a Keras Sequential model for regression.

6. Build the same model with the Functional API.

7. Compile a model with an optimizer, loss, and metric.

8. Train a model with `model.fit`.

9. Evaluate a model with `model.evaluate`.

10. Generate predictions with `model.predict`.

11. Add an `EarlyStopping` callback.

12. Save and load a model using the `.keras` format.

13. Create a `tf.data.Dataset` from tensors, batch it, and prefetch it.

14. Add a Keras `Normalization` preprocessing layer and adapt it on training data.

15. Write a custom training loop using `tf.GradientTape`.

16. Wrap a simple function with `tf.function` and explain what changes.

17. Log training with TensorBoard.

18. Explain when TensorFlow Lite would be useful.

19. Explain when TensorFlow Serving would be useful.

20. Compare a TensorFlow Keras workflow with a PyTorch training loop.

---

# Preview of Chapter 95

Chapter 94 studied TensorFlow.

We learned how TensorFlow connects tensors, variables, eager execution, graph execution, automatic differentiation, Keras models, built-in training APIs, custom training loops, `tf.data`, callbacks, preprocessing layers, saving, TensorBoard, distributed training, TensorFlow Lite, TensorFlow Serving, TensorFlow.js, and TFX.

Next we study AI Engineering.

AI Engineering is broader than training models.

It includes designing systems that use models safely, reliably, observably, and usefully inside real products and workflows.

The transition is:

```text
TensorFlow and PyTorch help build and train models
AI Engineering asks how models become dependable systems
```

Chapter 95 will show how model APIs, prompts, data pipelines, evaluation, monitoring, safety, cost, latency, and product design come together in practical AI systems.
