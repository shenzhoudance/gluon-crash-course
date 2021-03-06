# Automatic differentiation with `autograd`

We train models to get better and better as a function of experience. Usually, getting better means minimizing a loss function. To achieve this goal, we often iteratively compute the gradient of the loss with respect to weights and then update the weights accordingly. While the gradient calculations are straightforward through chain rule, for complex models, working it out by hand can be a pain.

Before diving deep into the model training, let's go through how MXNet’s autograd package expedites this work by automatically calculating derivatives. 

Let's first import the `autograd`

```{.python .input}
from mxnet import nd
from mxnet import autograd
```

As a toy example, let’s say that we are interested in differentiating a function $f = 2 x^2$ with respect to parameter $x$. We can start by assigning an initial value of $x$.

```{.python .input  n=3}
x = nd.array([[1, 2], [3, 4]])
x
```

Once we compute the gradient of $f$ with respect to $x$, we’ll need a place to store it. In MXNet, we can tell an NDArray that we plan to store a gradient by invoking its `attach_grad` method.

```{.python .input  n=6}
x.attach_grad()
```

Now we’re going to define the function $f$. To let MXNet store $f$ so that we can compute gradients later, we need to put the definition inside a `autograd.record()` scope.

```{.python .input  n=7}
with autograd.record():
    y = x * 2
    z = y * x
```

Let’s backprop by calling `z.backward()`. When $z$ has more than one entry, `z.backward()` is equivalent to `z.sum().backward()`.

```{.python .input  n=8}
z.backward()
```

Now, let’s see if this is the expected output. Remember that `y = x * 2`, and `z = x * y`, so $z=2x^2$ and $\frac{dz}{dx} = 4x$, which should be `[[4, 8],[12, 16]]`. Let check the automatically computed results

```{.python .input  n=9}
x.grad
```

## Use Python control flows

Sometimes we want to write dynamic programs, namely the execution depends on some real time values. MXNet will record the execution trace and compute the gradient as well.

Consider the following function `f`, it doubles the inputs until it's norm reaches to 1000. And then select one element depends on the sum of its elements.

```{.python .input}
def f(a):
    b = a * 2
    while b.norm().asscalar() < 1000:
        b = b * 2
    if b.sum().asscalar() >= 0:
        c = b[0]
    else:
        c = b[1]
    return c
```

We record the trace and feed in a random value

```{.python .input}
a = nd.random.uniform(shape=2)
a.attach_grad()
with autograd.record():
    c = f(a)
c.backward()
```

We know that `b` is a linear function of `a`, and `c` is chosen from `b`. Then the gradient with respect to `a` be will either `[c/a[0], 0]` or `[0, c/a[1]`, depends which element from `b` we picked. Let's find the results:

```{.python .input}
[a.grad, c/a]
```
