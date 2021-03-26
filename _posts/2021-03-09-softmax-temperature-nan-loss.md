---
layout: post
show_badges: true
gist_id: d8dc5c346b790aaab915be6cd10057ef
colab_id: 15XubYS4GlhkeiAjqVvo4os23Xq1dVmT6?usp=sharing
title: Use temperature in softmax function to avoid NaN loss
description: How to use temperature in softmax function with cross-entropy loss to avoid NaN
summary: How to use temperature hyperparameter in softmax to smooth the output probability distribution and avoid unwanted NaN in cross-entropy loss
tags: [deep learning, neural network, softmax, cross-entropy]
---

# Table of Contents
- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Softmax temperature](#softmax-temperature)
- [PyTorch example](#pytorch-example)

# Introduction
The `softmax` function isn't supposed to output zeros or ones, but sometimes
it happens due to floating-point precision when the input vector contains
numbers too big or too small for the exponential inside the softmax.

> Exponential growth seems slow at the beginning, but it becomes stepper in
> a very short time.

For example, $$e^{-114}\approx3.1\cdot10^{-50}$$
is essentially evaluated as zero.  
When softmax is used with cross-entropy loss function, a zero
in the former's output becomes ±$$\infin$$ as a result of the
logarithm in latter, which is theoretically correct since the adjustments
to make the network adapt are infinite, but it is of no use in practice
as the resulting loss could be [NaN](/assets/images/computer-says-nan.jpg).  

A zero or a one in the softmax output means that the model is very
confident about the prediction, therefore __the solution is to decrease that
confidence to produce a _softer_ probability distribution__.

# Softmax temperature

$$q_i = \dfrac{e^{(z_i/T)}}{\sum_j{e^{(z_j/T)}}}$$

T is the temperature parameter, usually set to 1.  
Increasing the temperature parameter will penalize bigger $$z_i$$ values
more than the smaller $$z_i$$ values, owing to the amplification effect
of the exponential, __which leads to a decrease in the confidence of the model__.

```python
T = 1
exp(-8/T) ~ 0.0003
exp(8/T)  ~ 2981
exp(3/T)  ~ 20

T = 1.2
exp(-8/T) ~ 0.01
exp(8/T)  ~ 786
exp(3/T)  ~ 3
```

In % terms, the bigger the exponent is, the more it shrinks when a
temperature `>1` is applied, which implies that the softmax function will
assign more probability mass to the smaller samples.

> Beware! A high temperature makes the NN “easily excited”,
> resulting in more mistakes.

# PyTorch example

Let's write the softmax function and the cross-entropy loss function.

```python
def softmax(input, t=1.0):
  print("input", input)
  ex = torch.exp(input/t)
  print("exp", ex)
  sum = torch.sum(ex, axis=0)
  return ex / sum

def cross_entropy(distribution):
  target = torch.tensor([0, 0, 1, 0, 0])
  print("loss", -torch.sum(target * torch.log(distribution)))
```

Define a sample containing some large absolute values and apply
the softmax function, then the cross-entropy loss.

```python
input = torch.tensor([55.8906, -114.5621, 6.3440, -30.2473, -44.1440])
cross_entropy(softmax(input))
```

    input tensor([  55.8906, -114.5621,    6.3440,  -30.2473,  -44.1440])
    exp tensor([1.8749e+24, 0.0000e+00, 5.6907e+02, 7.3074e-14, 6.7376e-20])
    loss tensor(nan)

The resulting probability distribution contains a zero, the loss value is NaN.  
Let's see what happens by setting the temperature to 10.

```python
input = torch.tensor([55.8906, -114.5621, 6.3440, -30.2473, -44.1440])
cross_entropy(softmax(input, t=10))
```

    input tensor([  55.8906, -114.5621,    6.3440,  -30.2473,  -44.1440])
    exp tensor([2.6748e+02, 1.0584e-05, 1.8859e+00, 4.8571e-02, 1.2102e-02])
    loss tensor(4.9619)

The probability is more equally distributed, the softmax function has assigned
more probability mass to the smallest sample, __from 0 to 1.0584e-05__, and less
probability mass to the largest sample, from 1.8749e+24 to 2.6748e+02.  
Finally, __the loss has changed from NaN to a valid value__.