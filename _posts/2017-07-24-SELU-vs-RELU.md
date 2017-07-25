---
layout: post
comments: true
title:  "SELU vs RELU activation in simple NLP models"
excerpt: "RELU activation function has become the de facto choice in neural networks these days. Few weeks ago, some researchers proposed Scaled Exponential Linear Unit (SELU) activation function. They show a far better convergence using SELU. In this post, I am posting a simple comparison of SELU against RELU using a simple BoW model on SNLI dataset."
date:   2017-07-24 23:00:00
mathjax: false
---

### Background on SELU

Normalized outputs seem to be really helpful in stabilizing the training process. That's the main reason behind the popularity of [`BatchNormalization`](https://arxiv.org/abs/1502.03167). [`SELU`](https://arxiv.org/abs/1706.02515) is a way to output the normalized activations to the next layer.

The overall function is really simple:
<img src="/assets/selu.png">

For `mean 0` and `stdev 1` inputs, the values of α and λ come out to be `1.6732632423543772848170429916717` and `1.0507009873554804934193349852946` respectively.

```python
# PyTorch implementation
import torch.nn.functional as F

def selu(x):
    alpha = 1.6732632423543772848170429916717
    scale = 1.0507009873554804934193349852946
    return scale * F.elu(x, alpha)
```

```python
# Numpy implementation
import numpy as np

def selu(x):
    alpha = 1.6732632423543772848170429916717
    scale = 1.0507009873554804934193349852946
    return scale * ((x > 0)*x + (x <= 0) * (alpha * np.exp(x) - alpha))
```

### SNLI dataset

[SNLI dataset](https://nlp.stanford.edu/projects/snli/) is a collection of 570k english sentence pairs. The task is to classify each pair as either:
- **entailment** - "A soccer game with multiple males playing." and "Some men are playing a sport."
- **contradiction** - "A man inspects the uniform of a figure in some East Asian country." and "The man is sleeping."
- **neutral** - "A smiling costumed woman is holding an umbrella." and "A happy woman in a fairy costume holds an umbrella."

### SELU vs RELU results

Code for this excercise is available in [this repo](https://github.com/hardikp/selu_snli).

<img src="/assets/selu_vs_relu_with_batchnorm.png">

`RELU` is clearly converging much faster than `SELU`. My first was to remove the `BatchNormalization` and do the same comparison. The following graph shows the comparison after removing the BatchNorm components.

<img src="/assets/selu_vs_relu_without_batchnorm.png">

Still, `RELU` seems to be doing a much better job than `SELU` for the default configuration.

However, there could still be multiple reasons for this behavior:
- `SELU` authors recommend a specific initialization scheme for it to be effective.
- The authors also use a slightly different Dropout.

Additionally, `SELU` is a bit more computationally expensive than `RELU`. On a g2.2xlarge EC2 instance, `RELU` model took about 49 seconds to complete an epoch, while `SELU` model took 65 seconds to do the same (33% more).
