---
layout: simple_post
title: Probability Primer
permalink: /probability/
mathjax: true
---

## Random Variables and Probability Distributions

Random Variables (RV) and Probability Mass Function (PMF):
* Domain of $$P$$ is the set of all possible events: $$\Omega$$
* Probabilities range from 0 to 1. $$\forall{x}\in\mathbf{x}, 0 \le P(x) \le 1$$
* Probabilities sum up to 1. $$\sum_{x \in \mathbf{x}}P(x) = 1$$
* Example, a **uniform distribution** with $$k$$ different states. $$P(\mathbf{x}=x_i) = \frac{1}{k}$$. In a coin toss, $$k$$ is 2.

Continuous Random Variables and Probability Density Functions (PDF):
* Domain of $$p$$ is the set of all possible events: $$\Omega$$
* Function $$p$$ is not a probability here. $$\forall{x}\in\mathbf{x}, p(x) \ge 0$$
* $$p(x)\delta{x}$$ represents the probability of landing inside $$\delta{x}$$ sized region. $$\int p(x)dx = 1$$ 
* Example: If a random variable $$\mathbf{x}$$ is represented by the uniform distribution that takes values from real value $$a$$ to real value $$b$$, it can be represented as $$\mathbf{x} \sim U(a, b)$$.

Marginal Probability

$$p(x) = \int p(x, y)dy$$

Considitional Probability

$$P(\mathbf{y} = y | \mathbf{x} = x) = \frac{P(\mathbf{y} = y, \mathbf{x} = x)}{P(\mathbf{x} = x)}$$

Chain Rule of Probabilities

$$P(a,b,c) = P(a|b,c)P(b|c)P(c)$$

Independence and Conditional Independence

Expectation, Variance and Covariance

Common Distributions
* Burnoulli
* Normal

Bayes' Rule

$$P(x|y) = \frac{P(x)P(y|x)}{P(y)}$$

## Resources
* [Probability Cheatsheet](http://www.wzchen.com/probability-cheatsheet)
* [Probability and Information Theory](http://www.deeplearningbook.org/contents/prob.html) chapter in the [Deep Learning](http://www.deeplearningbook.org/) book.
* [Introduction to Probability and Statistics](https://ocw.mit.edu/courses/mathematics/18-05-introduction-to-probability-and-statistics-spring-2014/index.htm) course from MIT OpenCourseWare.
