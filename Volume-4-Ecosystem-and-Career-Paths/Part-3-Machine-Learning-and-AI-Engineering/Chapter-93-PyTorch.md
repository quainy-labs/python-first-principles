# Chapter 93 — PyTorch

PyTorch is a deep learning framework for Python.

It gives Python a flexible system for working with tensors, automatic differentiation, neural network layers, optimizers, datasets, GPUs, and training loops.

scikit-learn gave us a structured machine learning workflow.

PyTorch gives us a lower-level and more flexible engine for deep learning.

That difference matters.

In scikit-learn, you usually choose an estimator and call:

```python
model.fit(X_train, y_train)
```

In PyTorch, you often write the training loop yourself.

You define:

* the model
* the loss function
* the optimizer
* the forward pass
* the backward pass
* the parameter update
* the data loading loop
* the evaluation loop
* the checkpointing behavior

This is more work.

It is also more control.

That control is why PyTorch became so important in deep learning research and engineering.

The central shift is:

```text
from using a fixed estimator interface
to building differentiable programs
```

PyTorch lets you write ordinary Python code that performs tensor computation and tracks gradients.

This chapter explains that foundation.

---

# Why PyTorch Matters

PyTorch matters because modern deep learning is built from tensors, gradients, and optimization.

Deep learning models can include:

* linear layers
* convolutions
* embeddings
* recurrent networks
* transformers
* normalization layers
* attention mechanisms
* activation functions
* loss functions
* optimizers
* data pipelines

PyTorch provides the building blocks.

It is widely used for:

* computer vision
* natural language processing
* speech
* recommendation systems
* reinforcement learning
* generative models
* scientific machine learning
* research prototypes
* production training systems
* model fine-tuning

PyTorch became popular partly because it feels like Python.

You can use normal Python control flow.

You can debug with ordinary tools.

You can build models dynamically.

You can inspect tensors directly.

This makes PyTorch friendly for experimentation.

But flexibility has a cost.

It is easier to write a broken training loop in PyTorch than in scikit-learn.

The code can run while silently training poorly.

Professional PyTorch use requires discipline.

---

# PyTorch After scikit-learn

The previous chapter studied scikit-learn.

scikit-learn taught:

* estimators
* transformers
* pipelines
* train-test splits
* cross-validation
* metrics
* preprocessing
* leakage prevention
* model selection

Those ideas still matter.

PyTorch does not cancel them.

You still need:

* clean data
* honest evaluation
* train-validation-test separation
* metrics that match the task
* reproducibility
* monitoring
* deployment discipline

What changes is the modeling layer.

scikit-learn hides much of the training loop.

PyTorch exposes it.

That exposure is useful when you need custom neural network architectures, custom losses, custom training behavior, GPU acceleration, or research flexibility.

The professional relationship is:

```text
scikit-learn teaches ML workflow discipline
PyTorch teaches differentiable model construction and training control
```

Do not abandon the workflow discipline when you enter PyTorch.

You need it even more.

---

# Importing PyTorch

The standard import is:

```python
import torch
```

Neural network tools are under `torch.nn`:

```python
from torch import nn
```

Data loading tools are under `torch.utils.data`:

```python
from torch.utils.data import DataLoader, Dataset
```

Optimizers are under `torch.optim`:

```python
from torch import optim
```

In many examples, you will see:

```python
import torch
from torch import nn
from torch.utils.data import DataLoader
```

These imports already reveal the main structure:

* tensors come from `torch`
* models come from `nn`
* data batches come from `DataLoader`
* training updates come from optimizers

---

# Tensors

The central PyTorch object is the tensor.

A tensor is similar in spirit to a NumPy array.

It can hold numbers in one or more dimensions.

Examples:

```python
scalar = torch.tensor(3.0)
vector = torch.tensor([1.0, 2.0, 3.0])
matrix = torch.tensor([[1.0, 2.0], [3.0, 4.0]])
```

Inspect shape:

```python
matrix.shape
```

Result:

```python
torch.Size([2, 2])
```

Inspect dtype:

```python
matrix.dtype
```

Inspect device:

```python
matrix.device
```

PyTorch tensor habits are close to NumPy habits:

```text
know the shape
know the dtype
know the device
```

The device is especially important in PyTorch.

NumPy usually works on CPU.

PyTorch tensors can live on CPU, CUDA GPUs, Apple's Metal Performance Shaders backend, and other supported devices.

Operations generally require tensors to be on the same device.

---

# Tensor Creation

Common creation functions:

```python
torch.zeros(3, 4)
torch.ones(3, 4)
torch.empty(3, 4)
torch.arange(0, 10)
torch.linspace(0, 1, steps=5)
torch.randn(2, 3)
torch.rand(2, 3)
```

Like NumPy, `empty` allocates memory without initializing values.

Use it only when you will fill the tensor before reading it.

Random tensors are common in neural networks:

```python
x = torch.randn(32, 10)
```

This might represent:

```text
32 samples, each with 10 features
```

The first dimension is often batch size.

Batch thinking is central to deep learning.

Models usually process many examples at once.

That improves hardware utilization and makes optimization more stable.

---

# Tensor Operations

PyTorch supports familiar tensor operations:

```python
a = torch.tensor([1.0, 2.0, 3.0])
b = torch.tensor([10.0, 20.0, 30.0])

a + b
a * b
a.mean()
a.sum()
```

Matrix multiplication uses `@`:

```python
x = torch.randn(32, 10)
w = torch.randn(10, 5)

y = x @ w
```

The result shape is:

```text
(32, 5)
```

Broadcasting works similarly to NumPy:

```python
x = torch.randn(32, 10)
bias = torch.randn(10)

y = x + bias
```

The bias is broadcast across the batch.

Shape reasoning from NumPy carries directly into PyTorch.

If a PyTorch model fails, the first debugging question is often:

```text
what are the tensor shapes at each step?
```

---

# Devices

PyTorch tensors live on a device.

Common device selection:

```python
device = "cuda" if torch.cuda.is_available() else "cpu"
```

Then:

```python
model = model.to(device)
x = x.to(device)
y = y.to(device)
```

All tensors involved in the same operation usually need to be on the same device.

This error is common:

```text
expected all tensors to be on the same device
```

It means part of your computation is on CPU and part is on GPU.

Professional habit:

```text
move model and batches to the same device deliberately
```

Do not scatter `.to(device)` randomly.

Have a clear training loop pattern.

Device movement can be expensive.

Moving tensors between CPU and GPU repeatedly inside tight loops can slow training badly.

---

# PyTorch and NumPy

PyTorch tensors and NumPy arrays can interoperate.

Create tensor from NumPy:

```python
import numpy as np

array = np.array([1.0, 2.0, 3.0])
tensor = torch.from_numpy(array)
```

Convert tensor to NumPy:

```python
array = tensor.numpy()
```

If the tensor is on GPU, move it to CPU first:

```python
array = tensor.cpu().numpy()
```

If the tensor tracks gradients, detach it first:

```python
array = tensor.detach().cpu().numpy()
```

That line appears often.

It means:

```text
remove from autograd graph
move to CPU
convert to NumPy
```

Do not call `.numpy()` blindly on tensors involved in training.

Understand whether gradients, devices, and memory sharing matter.

---

# Automatic Differentiation

Automatic differentiation is the heart of PyTorch.

PyTorch can track operations on tensors and compute gradients automatically.

Example:

```python
x = torch.tensor(2.0, requires_grad=True)
y = x * x + 3 * x

y.backward()

x.grad
```

The function is:

```text
y = x^2 + 3x
```

The derivative is:

```text
2x + 3
```

At `x = 2`, the derivative is:

```text
7
```

PyTorch computes that gradient.

This is what makes neural network training practical.

You define a computation.

PyTorch records the operations.

You compute a loss.

You call `backward`.

PyTorch computes gradients for parameters that require gradients.

---

# The Computation Graph

When tensors with `requires_grad=True` participate in operations, PyTorch builds a computation graph.

The graph records how outputs depend on inputs.

Example:

```python
x = torch.tensor([1.0, 2.0, 3.0], requires_grad=True)
y = (x * 2).sum()
```

The output `y` depends on `x`.

Calling:

```python
y.backward()
```

computes:

```python
x.grad
```

For most training loops, you do not manually inspect the graph.

But you must understand the lifecycle:

```text
forward pass builds graph
loss is computed
backward pass computes gradients
optimizer uses gradients
gradients are cleared before the next step
```

If you accidentally keep references to computation graphs, memory usage can grow.

If you detach tensors accidentally, gradients may stop flowing.

Autograd is powerful, but it follows the graph you actually build, not the graph you intended.

---

# requires_grad

The flag `requires_grad` controls whether PyTorch tracks gradients for a tensor.

Model parameters usually require gradients.

Input data usually does not.

Example:

```python
w = torch.randn(10, 1, requires_grad=True)
```

If a tensor does not require gradients, PyTorch does not need to track operations for gradient computation.

This saves memory and work.

During evaluation, disable gradient tracking:

```python
with torch.no_grad():
    predictions = model(x)
```

This is important.

Evaluation does not need gradients.

Using `torch.no_grad()` reduces memory usage and speeds inference.

Modern PyTorch also provides inference-focused contexts, but `no_grad` remains a fundamental idea for beginners.

---

# Neural Network Modules

PyTorch neural networks are usually built with `nn.Module`.

Example:

```python
from torch import nn


class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.layers = nn.Sequential(
            nn.Linear(10, 32),
            nn.ReLU(),
            nn.Linear(32, 1),
        )

    def forward(self, x):
        return self.layers(x)
```

Create the model:

```python
model = NeuralNetwork()
```

Call it:

```python
output = model(x)
```

You usually do not call `forward` directly.

Use:

```python
model(x)
```

That lets PyTorch's module machinery run correctly.

`nn.Module` tracks submodules and parameters.

When you assign layers inside `__init__`, PyTorch registers them.

Then:

```python
model.parameters()
```

can find trainable parameters for the optimizer.

---

# Layers

Common layers include:

```python
nn.Linear
nn.Conv2d
nn.Embedding
nn.LayerNorm
nn.BatchNorm1d
nn.Dropout
nn.ReLU
nn.GELU
nn.Softmax
```

A linear layer:

```python
layer = nn.Linear(in_features=10, out_features=5)
```

It expects input whose last dimension is `10`.

It produces output whose last dimension is `5`.

Shape example:

```text
input:  (32, 10)
output: (32, 5)
```

The batch dimension stays.

The feature dimension changes.

Understanding layers means understanding what shape they expect and what shape they return.

When building models, track shapes layer by layer.

Many PyTorch errors are shape mismatches wearing dramatic clothing.

---

# Activation Functions

Neural networks need nonlinear activation functions.

Without nonlinearities, stacking linear layers would still produce a linear function.

Common activations:

* ReLU
* GELU
* sigmoid
* tanh
* softmax

Example:

```python
nn.Sequential(
    nn.Linear(10, 32),
    nn.ReLU(),
    nn.Linear(32, 1),
)
```

ReLU is common in many feedforward networks.

GELU is common in transformer-style architectures.

Sigmoid is often used for binary probabilities, though many loss functions expect raw logits instead.

Softmax converts logits to class probabilities across classes.

Be careful:

```text
loss functions often expect logits, not probabilities
```

For example, `nn.CrossEntropyLoss` expects raw class scores.

Do not apply softmax before it unless the documentation and your setup specifically require it.

---

# Loss Functions

A loss function measures how wrong the model is.

Common losses:

```python
nn.MSELoss()
nn.L1Loss()
nn.CrossEntropyLoss()
nn.BCEWithLogitsLoss()
```

Regression often uses:

```python
loss_fn = nn.MSELoss()
```

Multi-class classification often uses:

```python
loss_fn = nn.CrossEntropyLoss()
```

Binary classification often uses:

```python
loss_fn = nn.BCEWithLogitsLoss()
```

The loss must match:

* task type
* model output shape
* target shape
* target dtype
* whether outputs are logits or probabilities

This is a common source of bugs.

For `CrossEntropyLoss`, targets are usually class indices, not one-hot vectors.

For `BCEWithLogitsLoss`, outputs are raw logits and targets are floating values such as `0.0` or `1.0`.

Read the loss function expectations carefully.

---

# Optimizers

An optimizer updates model parameters using gradients.

Common optimizers:

```python
torch.optim.SGD
torch.optim.Adam
torch.optim.AdamW
```

Example:

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)
```

The training step usually follows this pattern:

```python
optimizer.zero_grad()
predictions = model(x)
loss = loss_fn(predictions, y)
loss.backward()
optimizer.step()
```

The order matters.

Clear old gradients.

Run forward pass.

Compute loss.

Backpropagate.

Update parameters.

Gradients accumulate in PyTorch by default.

That is why `zero_grad()` is necessary.

Forgetting it is a classic mistake.

---

# Dataset

PyTorch data pipelines often use `Dataset`.

A dataset knows how to return one sample.

Custom dataset:

```python
from torch.utils.data import Dataset


class TabularDataset(Dataset):
    def __init__(self, features, targets):
        self.features = torch.as_tensor(features, dtype=torch.float32)
        self.targets = torch.as_tensor(targets, dtype=torch.float32)

    def __len__(self):
        return len(self.features)

    def __getitem__(self, index):
        return self.features[index], self.targets[index]
```

The two required methods are:

```python
__len__
__getitem__
```

`__len__` returns dataset size.

`__getitem__` returns one example.

This design lets PyTorch build flexible data loading systems.

The dataset describes data access.

The dataloader handles batching, shuffling, and parallel loading.

---

# DataLoader

`DataLoader` turns a dataset into batches.

```python
from torch.utils.data import DataLoader

train_loader = DataLoader(
    train_dataset,
    batch_size=32,
    shuffle=True,
)
```

During training:

```python
for x_batch, y_batch in train_loader:
    ...
```

Batching matters because deep learning training usually updates parameters based on mini-batches rather than one example at a time.

`shuffle=True` is common for training.

For validation and testing, shuffling is usually unnecessary:

```python
test_loader = DataLoader(test_dataset, batch_size=32, shuffle=False)
```

DataLoader can also use multiple workers for loading data.

That can improve throughput when data loading is a bottleneck.

But more workers are not always better.

Measure.

---

# A Basic Training Loop

A training loop brings together model, data, loss, optimizer, and device.

```python
def train_one_epoch(model, dataloader, loss_fn, optimizer, device):
    model.train()

    total_loss = 0.0

    for x_batch, y_batch in dataloader:
        x_batch = x_batch.to(device)
        y_batch = y_batch.to(device)

        optimizer.zero_grad()
        predictions = model(x_batch)
        loss = loss_fn(predictions, y_batch)
        loss.backward()
        optimizer.step()

        total_loss += loss.item() * x_batch.size(0)

    return total_loss / len(dataloader.dataset)
```

Important details:

* `model.train()` enables training behavior
* batches move to device
* gradients are cleared
* forward pass computes predictions
* loss measures error
* backward pass computes gradients
* optimizer updates parameters
* `loss.item()` extracts a Python number

Training loops are where many bugs live.

Keep them boring and explicit.

---

# Evaluation Loop

Evaluation should not update model parameters.

```python
def evaluate(model, dataloader, loss_fn, device):
    model.eval()

    total_loss = 0.0

    with torch.no_grad():
        for x_batch, y_batch in dataloader:
            x_batch = x_batch.to(device)
            y_batch = y_batch.to(device)

            predictions = model(x_batch)
            loss = loss_fn(predictions, y_batch)

            total_loss += loss.item() * x_batch.size(0)

    return total_loss / len(dataloader.dataset)
```

Important details:

* `model.eval()` changes behavior for layers such as dropout and batch normalization
* `torch.no_grad()` disables gradient tracking
* no optimizer step occurs

Do not evaluate in training mode.

Do not train in evaluation mode.

This is a common source of confusing results.

Use:

```python
model.train()
```

for training.

Use:

```python
model.eval()
```

for validation and testing.

---

# Training and Validation

A typical training process runs for multiple epochs.

```python
for epoch in range(num_epochs):
    train_loss = train_one_epoch(model, train_loader, loss_fn, optimizer, device)
    val_loss = evaluate(model, val_loader, loss_fn, device)

    print(f"epoch={epoch} train_loss={train_loss:.4f} val_loss={val_loss:.4f}")
```

Training loss tells you how well the model fits training data.

Validation loss tells you how well it performs on held-out data during development.

Patterns matter:

* training loss high, validation loss high: underfitting
* training loss low, validation loss high: overfitting
* both improving: learning
* validation unstable: data, learning rate, batch size, or metric issues
* loss becomes NaN: numerical instability

Do not stare at final accuracy only.

Watch learning curves.

They often tell the story.

---

# Overfitting

Overfitting happens when a model learns training data patterns that do not generalize.

Deep networks can overfit easily.

Common defenses:

* more data
* data augmentation
* weight decay
* dropout
* early stopping
* smaller models
* better validation
* regularization
* simpler features

Dropout:

```python
nn.Dropout(p=0.5)
```

Weight decay with AdamW:

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-2)
```

Early stopping monitors validation performance and stops when improvement stalls.

Overfitting is not solved by one magic trick.

It is diagnosed by comparing training and validation behavior.

---

# Learning Rate

The learning rate controls update size.

If it is too high, training may diverge.

If it is too low, training may be painfully slow or get stuck.

Example:

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)
```

Learning rate is one of the most important hyperparameters.

Symptoms:

* loss explodes: learning rate may be too high
* loss becomes NaN: learning rate, data, or numerical issue
* loss barely changes: learning rate may be too low
* validation jumps wildly: learning rate may be unstable

Schedulers adjust learning rate during training:

```python
scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.1)
```

Then:

```python
scheduler.step()
```

Schedulers are useful, but first understand the basic learning rate.

---

# Saving and Loading

PyTorch commonly saves model parameters using a state dictionary.

Save:

```python
torch.save(model.state_dict(), "model.pt")
```

Load:

```python
model = NeuralNetwork()
model.load_state_dict(torch.load("model.pt"))
model.eval()
```

The `state_dict` contains learned parameters and buffers.

It does not contain the Python class definition.

You need the model code to recreate the architecture before loading weights.

For checkpointing during training, save more:

```python
torch.save(
    {
        "epoch": epoch,
        "model_state_dict": model.state_dict(),
        "optimizer_state_dict": optimizer.state_dict(),
        "val_loss": val_loss,
    },
    "checkpoint.pt",
)
```

This lets you resume training with optimizer state.

Treat model files carefully.

Do not load untrusted serialized objects.

Model artifacts are part of your software supply chain.

---

# A Small Complete Example

Here is a compact binary classification example for tabular data.

```python
import torch
from torch import nn
from torch.utils.data import DataLoader, TensorDataset

device = "cuda" if torch.cuda.is_available() else "cpu"

X_train_tensor = torch.as_tensor(X_train, dtype=torch.float32)
y_train_tensor = torch.as_tensor(y_train, dtype=torch.float32).view(-1, 1)
X_val_tensor = torch.as_tensor(X_val, dtype=torch.float32)
y_val_tensor = torch.as_tensor(y_val, dtype=torch.float32).view(-1, 1)

train_dataset = TensorDataset(X_train_tensor, y_train_tensor)
val_dataset = TensorDataset(X_val_tensor, y_val_tensor)

train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
val_loader = DataLoader(val_dataset, batch_size=32)


class BinaryClassifier(nn.Module):
    def __init__(self, input_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 64),
            nn.ReLU(),
            nn.Linear(64, 1),
        )

    def forward(self, x):
        return self.net(x)


model = BinaryClassifier(input_dim=X_train_tensor.shape[1]).to(device)
loss_fn = nn.BCEWithLogitsLoss()
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)

for epoch in range(10):
    model.train()

    for x_batch, y_batch in train_loader:
        x_batch = x_batch.to(device)
        y_batch = y_batch.to(device)

        optimizer.zero_grad()
        logits = model(x_batch)
        loss = loss_fn(logits, y_batch)
        loss.backward()
        optimizer.step()

    model.eval()
    val_loss = 0.0

    with torch.no_grad():
        for x_batch, y_batch in val_loader:
            x_batch = x_batch.to(device)
            y_batch = y_batch.to(device)

            logits = model(x_batch)
            loss = loss_fn(logits, y_batch)
            val_loss += loss.item() * x_batch.size(0)

    val_loss /= len(val_loader.dataset)
    print(f"epoch={epoch} val_loss={val_loss:.4f}")
```

This example shows the core pattern:

* tensors
* dataset
* dataloader
* module
* loss
* optimizer
* device movement
* training mode
* evaluation mode
* no-gradient evaluation

Once you understand this, larger PyTorch systems become less mysterious.

---

# Metrics

Loss is not always the metric you care about.

For classification, you may need:

* accuracy
* precision
* recall
* F1
* ROC AUC
* average precision

For regression:

* MAE
* RMSE
* R squared

PyTorch itself focuses on tensor computation and training mechanics.

For many metrics, you may use:

* scikit-learn metrics
* TorchMetrics
* custom metric code

When using scikit-learn metrics, convert tensors safely:

```python
y_true_np = y_true.detach().cpu().numpy()
y_pred_np = y_pred.detach().cpu().numpy()
```

Remember:

```text
training loss optimizes the model
evaluation metrics judge usefulness
```

They are related, but not identical.

---

# Reproducibility

PyTorch workflows can involve randomness in many places:

* parameter initialization
* data shuffling
* dropout
* data augmentation
* GPU kernels
* train-validation split

Set seeds:

```python
torch.manual_seed(42)
```

If using NumPy and Python random:

```python
import random
import numpy as np

random.seed(42)
np.random.seed(42)
torch.manual_seed(42)
```

Full reproducibility can be difficult, especially on GPUs.

Some operations may be nondeterministic depending on backend and settings.

The professional habit is to track:

* code version
* data version
* package versions
* seed values
* hardware
* hyperparameters
* training configuration
* checkpoint path

Reproducibility is not only about setting one seed.

It is about making the experiment understandable later.

---

# GPUs

GPUs accelerate many tensor operations.

But GPU use is not automatic magic.

To benefit from a GPU:

* model must be on GPU
* data batches must be on GPU
* operations must be large enough to amortize overhead
* data loading must keep up
* memory must fit

Common pattern:

```python
device = "cuda" if torch.cuda.is_available() else "cpu"
model.to(device)
```

Then move each batch:

```python
x_batch = x_batch.to(device)
y_batch = y_batch.to(device)
```

GPU memory is limited.

If you run out of memory:

* reduce batch size
* reduce model size
* use mixed precision when appropriate
* avoid storing computation graphs accidentally
* clear unused references
* checkpoint activations in advanced cases

The GPU is a powerful worker.

It still needs good feeding, memory discipline, and measurement.

---

# Mixed Precision

Mixed precision training uses lower-precision arithmetic for speed and memory savings while preserving enough numerical stability.

In modern PyTorch, automatic mixed precision is commonly used with `torch.autocast` and gradient scaling when appropriate.

A simplified CUDA pattern:

```python
scaler = torch.cuda.amp.GradScaler()

for x_batch, y_batch in train_loader:
    x_batch = x_batch.to(device)
    y_batch = y_batch.to(device)

    optimizer.zero_grad()

    with torch.autocast(device_type="cuda"):
        predictions = model(x_batch)
        loss = loss_fn(predictions, y_batch)

    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

Mixed precision can be valuable.

But do not introduce it before your basic training loop is correct.

First make the model learn.

Then optimize performance.

---

# torch.compile

Modern PyTorch includes `torch.compile`, which can compile PyTorch code for performance in many cases.

Example:

```python
model = torch.compile(model)
```

This can improve training or inference speed depending on model, hardware, and workload.

But compilation is not a beginner requirement.

It can introduce compile overhead, debugging complexity, and compatibility considerations.

The professional approach is:

```text
first make correctness clear
then measure
then compile if it helps
```

Do not reach for `torch.compile` to hide an inefficient data pipeline, shape bug, or unstable training setup.

Performance tools help after the system is understandable.

---

# Debugging Training

When a PyTorch model does not learn, investigate systematically.

Ask:

* Are input shapes correct?
* Are target shapes correct?
* Are target dtypes correct?
* Is the loss function appropriate?
* Are logits being passed where logits are expected?
* Are probabilities being passed where probabilities are expected?
* Are gradients nonzero?
* Is `optimizer.step()` called?
* Is `optimizer.zero_grad()` called?
* Is the learning rate reasonable?
* Is the model in train mode during training?
* Is the model in eval mode during evaluation?
* Is data normalized appropriately?
* Is the train-validation split correct?
* Is there data leakage?
* Are labels aligned with inputs?

A useful test is to overfit a tiny batch.

Take a very small subset of data and train until the model memorizes it.

If the model cannot overfit a tiny batch, something basic is wrong.

This trick is simple and powerful.

---

# Common Mistakes

The first common mistake is forgetting `optimizer.zero_grad()`.

Gradients accumulate by default.

The second common mistake is evaluating without `model.eval()`.

Dropout and batch normalization behave differently during training and evaluation.

The third common mistake is evaluating without `torch.no_grad()`.

That wastes memory and computation.

The fourth common mistake is mismatching loss function and model output.

For example, applying softmax before `CrossEntropyLoss` is usually wrong.

The fifth common mistake is using wrong target dtype or shape.

Loss functions have specific expectations.

The sixth common mistake is mixing CPU and GPU tensors.

Move model and batches to the same device.

The seventh common mistake is converting tensors to NumPy without detaching or moving to CPU.

Use `detach().cpu().numpy()` when appropriate.

The eighth common mistake is storing loss tensors instead of loss values.

If you store tensors that keep graphs alive, memory can grow.

Use `loss.item()` for logging scalar loss.

The ninth common mistake is treating loss as the only metric.

Use task-relevant metrics.

The tenth common mistake is optimizing performance before verifying correctness.

First make the model learn correctly.

Then make it faster.

---

# Professional PyTorch Checklist

When using PyTorch, practice these habits:

* Inspect tensor shapes often.
* Track dtype and device.
* Keep model and batches on the same device.
* Use `nn.Module` for models.
* Put layers in `__init__`.
* Define computation in `forward`.
* Use the correct loss for the task.
* Clear gradients before backpropagation.
* Call `loss.backward()` before `optimizer.step()`.
* Use `model.train()` during training.
* Use `model.eval()` during validation and inference.
* Use `torch.no_grad()` for evaluation.
* Log both loss and task metrics.
* Keep train, validation, and test roles separate.
* Save `state_dict` checkpoints.
* Track seeds, versions, data, and hyperparameters.
* Overfit a tiny batch when debugging learning failures.
* Measure before optimizing.
* Treat deployment as an engineering system, not just a saved model.

These habits make PyTorch work more reliable.

---

# PyTorch Versus scikit-learn

Use scikit-learn when:

* classical ML algorithms fit the problem
* tabular baselines are needed
* training should be quick and structured
* pipelines and preprocessing are central
* deep learning flexibility is unnecessary

Use PyTorch when:

* neural networks are needed
* custom architectures matter
* custom losses matter
* GPU training matters
* embeddings, transformers, or vision models matter
* research flexibility matters
* you need control over the training loop

Many projects use both.

For example:

* scikit-learn for preprocessing experiments and baselines
* PyTorch for the final neural model
* scikit-learn metrics for evaluation
* PyTorch for training and inference

The question is not which library is better.

The question is:

```text
which abstraction level matches this problem?
```

---

# Summary

PyTorch is a deep learning framework built around tensors, automatic differentiation, neural network modules, optimizers, datasets, dataloaders, devices, and explicit training loops.

Tensors are the core data structure.

They have shape, dtype, and device.

Autograd tracks computations and computes gradients through `backward`.

`nn.Module` organizes models and parameters.

Loss functions measure error.

Optimizers update parameters using gradients.

Datasets and dataloaders organize samples and batches.

Training loops require explicit steps: train mode, device movement, zero gradients, forward pass, loss, backward pass, optimizer step, and logging.

Evaluation requires eval mode and no-gradient execution.

PyTorch gives more control than scikit-learn, but that control requires stronger discipline.

The central lesson is:

```text
PyTorch turns tensor computations into trainable differentiable programs
```

Once you understand that, neural network training becomes a controlled engineering process rather than a mysterious ritual.

---

# Exercises

1. Create scalar, vector, matrix, and three-dimensional tensors.

2. Inspect tensor shape, dtype, and device.

3. Create tensors with `zeros`, `ones`, `randn`, and `arange`.

4. Perform element-wise addition, multiplication, reduction, and matrix multiplication.

5. Move a tensor to a selected device when available.

6. Create a tensor with `requires_grad=True` and compute a simple derivative with `backward`.

7. Build a small `nn.Module` with two linear layers and one activation.

8. Pass a random batch through the model and inspect output shape.

9. Choose the correct loss function for binary classification, multi-class classification, and regression.

10. Write one optimizer step manually.

11. Create a custom `Dataset` with `__len__` and `__getitem__`.

12. Wrap it in a `DataLoader` and iterate over batches.

13. Write a training loop for one epoch.

14. Write an evaluation loop using `model.eval()` and `torch.no_grad()`.

15. Save and load a model `state_dict`.

16. Overfit a tiny batch and explain why this is a useful debugging test.

17. Convert a tensor to NumPy safely with `detach().cpu().numpy()`.

18. Add dropout to a model and observe the difference between train and eval mode.

19. Log training and validation loss over several epochs.

20. Explain when PyTorch is a better fit than scikit-learn.

---

# Preview of Chapter 94

Chapter 93 studied PyTorch.

We learned how deep learning workflows are built from tensors, devices, autograd, modules, layers, losses, optimizers, datasets, dataloaders, training loops, evaluation loops, checkpoints, reproducibility, GPU usage, mixed precision, and performance tools.

Next we study TensorFlow.

TensorFlow is another major deep learning framework, often associated with Keras, production deployment tooling, graph execution, TensorFlow Serving, TensorFlow Lite, and broad platform support.

The transition is:

```text
PyTorch emphasizes flexible eager-style deep learning workflows
TensorFlow emphasizes a broad deep learning platform with strong deployment pathways
```

Chapter 94 will show how TensorFlow fits into the Python AI ecosystem and how its workflow compares with PyTorch.
