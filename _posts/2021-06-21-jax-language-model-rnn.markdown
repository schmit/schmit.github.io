---
layout: post
title:  "Language modeling with Jax and RNNs"
date:   2021-06-20 21:26:24 -0700
categories: jax
tags: recurrent neural nets deep learning jax
---

[Jax](https://github.com/google/jax) is a relatively new Python library aimed as a drop in replacement for Numpy for machine learning research.
It sets itself apart due to its functional approach, which I find really enjoyable.
Recently I have been playing around with implementing a simple RNN using Flax to get beyond the basics, but without adding all the bells and whistles.

The goal is to create a character level language model using a simple RNN following the approach by Andrej Karpathy's 2015 blog post ["The Unreasonable Effectiveness of Recurrent Neural Networks"](http://karpathy.github.io/2015/05/21/rnn-effectiveness/), but focus on Jax rather than creating an unreasonably effective network.
That is, given a reasonably sized text, say a book, we want to create a model that generates content one character at a time. We will find that it is relatively easy for the simplest of RNNs to learn how to string together a few words using a small network and few training epochs.

We will assume a basic familiarity with Jax. You can find the code on [colab](https://colab.research.google.com/drive/1Qw7zilRZVnVE4PMuKEGHrKBGaaSB3NWG), and I am by no means a Jax expert, so feel free to reach out with comments and suggestions.


# Flax

There are a few libraries built on top of Jax that help with creating neural networks specifically: Haiku by DeepMind, and Flax and Trax both developed by Google AI.
Without any particularly strong preference, we will use Flax for this task.

We use the following imports
```python
import jax
import jax.ops
import jax.numpy as np
import numpy as onp  # convention: original numpy

import flax
from flax import linen as nn
from flax import optim
```

# Preparing data

When prototyping any machine learning method, it is always helpful to have a trivial example data set. This is particularly beneficial when creating a neural network, as it is very easy miss subtle bugs that cause unexpected behavior. Overfitting a model to a small dataset is a great sanity check to ensure your model is actually doing what you think it is. In this case, let's use the repetitive string `abcd...adcd...` as input.

To map characters to indices and back, let's create a convenience function which, for lack of a better name, I call a bridge:
```python
def id_bridge(iterable):
  """ provides mapping to and from ids """
  return ({elem: id for id, elem in enumerate(iterable)},
          {id: elem for id, elem in enumerate(iterable)})
```

Next, we create some more functions that are helpful for mapping from characters to model input, and from model output back to characters.
Note that `jax.numpy` is for the most part a drop-in replacement for Numpy, but its functional nature does not allow us to update arrays in place (e.g. `A[0, 1] = 3`). Instead, we can use `jax.ops.index_update` to set values at particular indices.

```python
def one_hot(i, n):
  """
  create vector of size n with 1 at index i
  """
  x = np.zeros(n)
  return jax.ops.index_update(x, i, 1)

def encode(char):
  return one_hot(char_to_id[char], len(char_to_id))

def decode(predictions, id_to_char):
  # for simplicity, pick the most likely character
  # this can be replaced by sampling weighted
  # by the probability of each character
  return id_to_char[int(np.argmax(predictions))]
```

The decode function takes the model output: predicted probabilities for each character that it will be the next character in the sequence, and returns the character the model finds most likely.

# A simple recurrent model

With that setup out of the way, we can code up a simple recurrent model.
While some recurrent cells are included in Flax, for the sake of learning we implement our own.

```python
class RNNCell(nn.Module):
  @nn.compact
  def __call__(self, state, x):
    # Wh @ h + Wx @ x + b can be efficiently computed
    # by concatenating the vectors and then having a single dense layer
    x = np.concatenate([state, x])
    new_state = np.tanh(nn.Dense(state.shape[0])(x))
    return new_state
```

`nn.compact` allows us to define the parameters in the forward pass of the model, rather than separately: Flax generates the parameters later based on a sample input.
Otherwise, this code should look rather familiar to similar definitions using other libraries, such as PyTorch.

Next, we stack three RNNCells on top of each other to create our Character RNN.
The `init_state` method is not required, but will turn out to be a rather convenient way to initialize the state when we want to generate new sequences.

```python
class ChaRNN(nn.Module):
  state_size: int
  vocab_size: int

  @nn.compact
  def __call__(self, state, i):
    x = one_hot(i, self.vocab_size)
    new_state = []

    # a rather naive way of stacking multiple RNN cells
    new_state_1 = RNNCell()(state[0], x)
    new_state_2 = RNNCell()(state[1], new_state_1)
    new_state_3 = RNNCell()(state[2], new_state_2)
    predictions = nn.softmax(nn.Dense(self.vocab_size)(new_state_3))
    return [new_state_1, new_state_2, new_state_3], predictions

  def init_state(self):
    # a convenient way to initialize the state
    return [
      np.zeros(self.state_size),
      np.zeros(self.state_size),
      np.zeros(self.state_size)
    ]
```

Let's write a function that can generate new sequences by sampling from the model.
The following function is simple but not very efficient

```python
def sample(model, params, bridge, initial='', max_length=100):
  char_to_id, id_to_char = bridge
  state = model.init_state()
  output = initial
  if len(initial) > 0:
    for char in initial[:-1]:
      state, _ = model.apply(params, state, char_to_id[char])

  next_char = initial[-1]
  for i in range(max_length):
    state, predictions = model.apply(params, state, char_to_id[next_char])
    next_char = decode(predictions, id_to_char)
    output += next_char

  return output
```

Finally, we initialize a model and test whether it can indeed generate a random sample.
Jax handles [randomness](https://jax.readthedocs.io/en/latest/notebooks/Common_Gotchas_in_JAX.html#random-numbers) differently from Numpy in that explicit keys have to be used.

```python
state_size = 8

# randomness is handled using explicit keys in Jax
key, subkey = jax.random.split(key)
model = ChaRNN(state_size, len(char_to_id))
params = model.init(subkey, model.init_state(), 0)

print(f"Model state size: {model.state_size}, vocab size: {model.vocab_size}")
# output: Model state size: 8, vocab size: 5

# run a single example through the model to test that it works
new_state, predictions = model.apply(params, model.init_state(), 0)
assert predictions.shape[0] == model.vocab_size

# calling sample on random model leads to random output
sample(model, params, (char_to_id, id_to_char), 'abc', max_length=10)
# output: 'abcadbaadbadd'
```

# Training

Now that we have verified the model code works, it is time to focus on optimizing the parameters.
Recall that we want the model to predict the next character based on the sequence of characters seen so far.
We create the following function to batch the input and creates a sequence of inputs and another sequence of targets to predict, which
is the same as the input sequence but shifted by one.

```python
def chunker(seq, size):
  """
  chunks a sequences into two subsequences
  one for inputs, another for targets, by
  shifting the input by 1
  """
  n = len(seq)
  p = 0
  while p + 1 <= n:
    # ensure the last chunk is of equal size
    yield seq[p:min(n-1, p+size)], seq[(p+1):(p+size+1)]
    p += size
```

Creating the loss function over a sequence of inputs is where Jax really shines in my opinion: we start by implementing the loss
for a single example, and then use Jax functions to vectorize, differentiate and compile.

Note we have to unroll the RNN to compute gradients, and at some point have to truncate the unrolled RNN.
In our case, we compute gradients over a batch, which will define how far the RNN will unroll.

```python
def cross_entropy_loss(predictions, label):
  # note we compute the loss for a single example.
  # we will use vmap below to vectorize
  return -np.log(predictions[label])

def rnn_loss(params, model, state, inputs, targets):
  # use lax.scan to efficiently generate a loop over the inputs
  # this function returns thefinal state, and predictions for every step
  # note: scan input array needs have shape [length, 1]
  final_state, predictions = jax.lax.scan(lambda state, x: model.apply(params, state, x),
                                          state,
                                          np.array([inputs]).T)
  loss = np.mean(jax.vmap(cross_entropy_loss)(predictions, np.array([targets]).T))
  return loss, final_state

# we want both the loss an gradient, we set has_aux because rnn_loss also return final state
# use static_argnums=1 to indicate that the model is static;
# a different model input will require recomplication
# finally, we jit the function to improve runtime
rnn_loss_grad = jax.jit(jax.value_and_grad(rnn_loss, has_aux=True),
                        static_argnums=1)
```

## Optimization

We use `flax.optim` to handle the gradient steps.
Let's define the following functions to deal with a single batch, which becomes trivial, and looping through batches to compute an epoch of updates
Note that in this case the gradients are only

Note for each epoch we start with the initial state, and we propagate states across all batches in an epoch.

```python
def batch_step(model, optimizer, state, inputs, targets):
  (loss, state), grad = rnn_loss_grad(optimizer.target, model, state, inputs, targets)
  new_optimizer = optimizer.apply_gradient(grad)
  return new_optimizer, loss, state

def epoch_step(model, optimizer, data, batch_size):
  state = model.init_state()
  total_loss = 0
  for n, (inputs, targets) in enumerate(chunker(data, batch_size)):
    optimizer, loss, state = batch_step(model, optimizer, state, inputs, targets)

    total_loss += loss
  return optimizer, total_loss / (n+1)
```

Finally, we initialize the optimizer in the following way:
```python
optimizer_def = optim.Adam(learning_rate=learning_rate,
                            weight_decay=weight_decay)

optimizer = optimizer_def.create(initial_params)
```

When we put this together in a simple training function (see the [Colab](https://colab.research.google.com/drive/1Qw7zilRZVnVE4PMuKEGHrKBGaaSB3NWG#scrollTo=bI885lfVmKab) for details) and run it on our sample input `abcd...abcd...`, we
see that the model quickly learns how to repeat the pattern:
```
Training RNN on 'abcd...abcd...'
Vocabulary size: 5
State size: 8
Adam optimizer parameters
learning_rate=0.002
weight_decay=0.000
====================
Epoch:   0 loss: 1.643 time: 1.77
Sample: abcd.bd..b..d..d..d..d..d..d..d..d..d..d..d..d..d..d..
Epoch:   5 loss: 1.528 time: 0.07
Epoch:  10 loss: 1.430 time: 0.06
Epoch:  15 loss: 1.348 time: 0.06
Epoch:  20 loss: 1.278 time: 0.06
Epoch:  25 loss: 1.207 time: 0.06
Epoch:  30 loss: 1.126 time: 0.06
Epoch:  35 loss: 1.041 time: 0.06
Epoch:  40 loss: 0.958 time: 0.06
Sample: abcd....bb............................................
Epoch:  45 loss: 0.882 time: 0.05
Epoch:  50 loss: 0.811 time: 0.06
Epoch:  55 loss: 0.744 time: 0.06
Epoch:  60 loss: 0.680 time: 0.06
Epoch:  65 loss: 0.622 time: 0.06
Epoch:  70 loss: 0.569 time: 0.08
Epoch:  75 loss: 0.522 time: 0.07
Epoch:  80 loss: 0.479 time: 0.06
Sample: abcd...abcd...abcd...abcd...abcd...abcd...abcd...abcd.
```

# Using real data

The trivial example above should give us some confidence that the code we wrote works as expected.
For a bit of fun, let's run the same code on a slightly more interesting dataset.
[Project Gutenberg](https://www.gutenberg.org/) hosts a library of free ebooks, from which I pulled a copy of The Metamorphosis by Franz Kafka.

```python
kafka = get_text('kafka.txt')

state_size = 128
vocab_size = len(set(kafka))

key, subkey = jax.random.split(key)
model = ChaRNN(state_size, vocab_size)
params = model.init(subkey, model.init_state(), 0)

print(f"Model state size: {model.state_size}, vocab size: {model.vocab_size}")

result, losses, bridge = train(kafka, model, params, 400,
                               batch_size=256,
                               max_epoch_size=10000,
                               weight_decay=1e-7,
                               sample_every=25, sample_prompt="Gregor")
```

```
Model state size: 128, vocab size: 64
Training RNN on 'I


One mo...'
Vocabulary size: 64
State size: 128
Adam optimizer parameters
learning_rate=0.002
weight_decay=0.000
====================
Epoch:   0 loss: 3.134 time: 7.32
Sample: Gregor to  to  oe to  oe  oe to  te  oe to  te  oe  oe t
Epoch:   5 loss: 2.329 time: 3.34
Epoch:  10 loss: 2.042 time: 3.34
Epoch:  15 loss: 2.031 time: 3.24
Epoch:  20 loss: 1.756 time: 3.27
Epoch:  25 loss: 1.663 time: 3.25
Sample: Gregor and her her the was not her the was not her the w
Epoch:  30 loss: 1.687 time: 3.22
Epoch:  35 loss: 1.641 time: 3.23
Epoch:  40 loss: 1.527 time: 3.27
Epoch:  45 loss: 1.590 time: 3.24
Epoch:  50 loss: 1.614 time: 3.25
Sample: Gregor was of the father was of the father was of the fa
Epoch:  55 loss: 1.488 time: 3.25
Epoch:  60 loss: 1.411 time: 3.25
Epoch:  65 loss: 1.495 time: 3.35
Epoch:  70 loss: 1.589 time: 3.21
Epoch:  75 loss: 1.467 time: 3.30
Sample: Gregor’s bone the could have the could have the could ha
Epoch:  80 loss: 1.459 time: 3.24
Epoch:  85 loss: 1.469 time: 3.31
Epoch:  90 loss: 1.513 time: 3.37
Epoch:  95 loss: 1.400 time: 3.31
Epoch: 100 loss: 1.460 time: 3.27
Sample: Gregor’s sister was nothing the door and state and state
Epoch: 105 loss: 1.369 time: 3.27
Epoch: 110 loss: 1.429 time: 3.29
Epoch: 115 loss: 1.389 time: 3.29
Epoch: 120 loss: 1.366 time: 3.30
Epoch: 125 loss: 1.459 time: 3.23
Sample: Gregor’s mother would be the table to the table to the t
Epoch: 130 loss: 1.330 time: 3.24
Epoch: 135 loss: 1.387 time: 3.30
Epoch: 140 loss: 1.335 time: 3.29
Epoch: 145 loss: 1.325 time: 3.25
Epoch: 150 loss: 1.350 time: 3.23
Sample: Gregor’s father to his father to his father to his fathe
```

After running for 400 epochs, and seeing the training loss flatten, we can sample a longer snippet to see where the model has landed:

```python
sample(model, result.target, bridge, initial='Gregor', max_length=500)
```
outputs
```
Gregor’s father and he had been to his father and he had been
to his father and he had been to his father and he had been
to his father and he had been to his father and he had been
to his father and he had been to his father and he had been
to his father and he had been to his father and he had been
...
```

While the model is able to string together words quite quickly, the deterministic sampling converges to equilibria of repeating patterns.

# Wrapping up

Of course, from here on out we can improve on all aspects of this modeling exercise, but for now, I hope this has been useful in getting a better grasp of Jax.
Personally, I find Jax a pleasure to work with: the functional style makes it easy to write transparent code.
Furthermore, if you have experience with Numpy, the syntax obviously feels familiar.
It's true that the handling of randomness requires a bit of getting used to,
and libraries such as Flax do some magic under the hood to help make the functional approach practical, but both are for the greater good.

_If you have questions, comments or find an error, please reach out via Twitter or email!_

