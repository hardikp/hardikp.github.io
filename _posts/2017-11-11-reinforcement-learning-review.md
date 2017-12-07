---
layout: post
comments: true
title: "A Brief Review of Reinforcement Learning"
excerpt: "Reinforcement Learning is a mathematical framework for experience-driven autonomous learning. An RL agent interacts with its environment and, upon observing the consequences of its actions, can learn to alter its own behaviour in response to the rewards received. The goal of the agent is to learn a policy ππ that maximizes the expected return (cumulative, discounted reward)."
date:   2017-11-11 22:00:00
mathjax: true
image_url: "/assets/rl-intro/agent-env.png"
---
Reinforcement Learning is a mathematical framework for experience-driven autonomous learning. An **RL agent** interacts with its **environment** and, upon observing the consequences of its **actions**, can learn to alter its own behaviour in response to the **rewards** received. The goal of the agent is to learn a **policy** $$\pi$$ that maximizes the expected **return** (cumulative, discounted reward).

* TOC
{:toc}

<img src="/assets/rl-intro/agent-env.png">

## Background

At each time step $$t$$, the agent receives a state $$s_t$$ from the state space $$S$$ and selects an action $$a_t$$ from the action space $$A$$, following a policy $$\pi(a_t \vert s_t)$$. Policy $$\pi(a_t \vert s_t)$$ is the agent's behavior. It is a mapping from a state $$s_t$$ to an action $$a_t$$. Upon taking the **action $$a_t$$**, the agent receives a scalar **reward $$r_t$$**, and transitions to the **next state $$s_{t+1}$$**.

Transition to the next state $$s_{t+1}$$ as well as the reward $$r_t$$ are governed by the environment dynamics (or a model). **State transition probability $$P(s_{t+1} \vert s_t,a_t)$$** is used to describe the environment dynamics.

The **return $$R_t = \sum_{k=0}^{\infty}\gamma^kr_{t+k}$$** is the discounted, accumulated reward with the discount factor $$\gamma \in (0, 1]$$. The agent aims to maximize the expectation of such long term return from each state.

{% comment %}
<img src="https://chainer.org/assets/chainerrl_breakout.gif">
{% endcomment %}

A value function is a prediction of the expected, accumulative, discounted, future reward. In other words, a value function is the expected **return**. A value function measures how good each state, or state-action pair, is. The state value $$v_{\pi}(s)$$ is the expected return for following policy $$\pi$$ from state $$s$$.

$$v_{\pi}(s) = E[R_t|s_t = s]$$

$$v_{\pi}(s)$$ decomposes into the Bellman equation:

$$v_{\pi}(s) = \sum_a \pi(a|s) \sum_{s',r}p(s',r|s,a)[r + \gamma v_{\pi}(s')]$$

We are assuming a stochastic policy $$\pi(a \vert s)$$ in the above equation. Therefore, we are summing over all actions. $$p(s',r \vert s,a)$$ is the state transition probabily for the given environment. And $$r + \gamma v_{\pi}(s')$$ is the return for taking action $$a$$ from state $$s$$.

An optimal state value $$v_*(s)$$ is the maximum state value achievable by any policy for state $$s$$.

$$v_*(s) = max_{\pi}v_{\pi}(s) = max_a q_{\pi^*}(s,a)$$

$$v_*(s) = max_a \sum_{s',r}p(s',r \vert s,a)[r + \gamma v_*(s')]$$

The action value $$q_{\pi}(s,a)$$ is the expected return for selecting action $$a$$ in state $$s$$ and then following policy $$\pi$$.

$$q_{\pi}(s,a) = E[R_t \vert s_t = s, a_t = a]$$

$$q_{\pi}(s,a) = \sum_{s',r}p(s',r \vert s,a)[r + \gamma \sum_{a'}\pi(a' \vert s')q_{\pi}(s',a')]$$

The optimal action value function $$q_*(s,a)$$ is the maximum action value achievable by any policy for state $$s$$ and action $$a$$.

$$q_*(s,a) = max_{\pi}q_{\pi}(s,a)$$

$$q_*(s,a) = \sum_{s',r}p(s',r \vert s,a)[r + \gamma max_{a'}q_*(s',a')]$$

<hr>

## Value Function Algorithms

The goal of these algorithms is to come up with an Action Value Function $$q_*(s,a)$$ or a State Value Function $$v_*(s)$$. An action value function $$q_*(s,a)$$ can be converted into an **$$\epsilon$$-greedy policy**.

A state value function $$v_*(s)$$ can be converted into the policy by greedily selecting the actions that result in better state value functions for $$s_{t+1}$$.

### TD Learning

TD Learning learns the value function $$V(s)$$ from experiences using TD errors. TD learning is prediction problem. The update rule for TD learning is $$V(s)$$ <- $$V(s) + \alpha[r + \gamma V(s') - V(s)]$$. $$\alpha$$ is the **learning rate** and $$r + \gamma V(s') - V(s)$$ is the **TD error**. More accurately, this is called TD(0) learning, where 0 indicates one-step return.

**Input**: The policy $$\pi$$ to be evaluated<br>
**Output**: value function $$V$$
```
Initialize V arbitrarily
for each episode do
    initialize state s
    for each step of episode, state s is not terminal do
        a <- action given by policy pi for s
        take action a, observe r, s'
        V(s) <- V(s) + alpha * [r + gamma * V(s') - V(s)]
```

<hr>

### SARSA
SARSA (state, action, reward, state, action) is an **on-policy** algorithm to find the optimal policy. The same policy is being used for both state space exploration and the value function update.

**Output**: action value function $$Q$$
```
Initialize Q arbitrarily
for each episode do
    initialize state s
    for each step of episode, state s is not terminal do
        a <- action for s derived by Q, e.g. ϵ-greedy
        take action a, observe r, s'
        a' <- action for s' derived by Q, e.g. ϵ-greedy
        Q(s,a) <- Q(s,a) + alpha * [r + gamma * Q(s',a') - Q(s,a)]
```

<hr>

### Q-Learning
Q-Learning is an off-policy algorithm to find the optimal policy. It's an off-policy method because the update rule greedily uses the action that maximizes the Q value, which differs from the policy used for evaluation (ϵ-greedy).

**Output**: action value function $$Q$$
```
Initialize Q arbitrarily
for each episode do
    initialize state s
    for each step of episode, state s is not terminal do
        a <- action for s derived by Q, e.g. ϵ-greedy
        take action a, observe r, s'
        a' <- argmax Q(s',b)
        Q(s,a) <- Q(s,a) + alpha * [r + gamma * Q(s',a') - Q(s,a)]
```

<hr>

{% comment %}
### DQN

There has been a lot recent work in the field of Deep RL. **DQN** has contributed the most in generating interest in the Deep RL field. However, most of the Deep RL work comes down to converting it into a supervised problem and solving the supervised problem.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Ironically, the major advances in RL over the past few years all boil down to making RL look less like RL and more like supervised learning.</p>&mdash; Denny Britz (@dennybritz) <a href="https://twitter.com/dennybritz/status/925028640001105920?ref_src=twsrc%5Etfw">October 30, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

**DQN** uses a Convolutional neural network (16 8 $$\times$$ 8 filters with 4 stride followed by 32 4 $$\times$$ 4 filters with 2 stride) to approximate the Q function. It relies on 2 important techniques to stabilize the training:
1. Use of experience replay. **Experience Replay** is a cyclic memory buffer. Minibatches are randomly sampled from this buffer for training. Consecutive states are generally very correlated which causes problems during the training. Randomly sampled minibatches solves this problem.
2. Using separate Update and Target networks. **Target network** is updated less frequently by copying the weights from the Update network. This allows the network to not get purturbed too much.

**Algorithm**

**Input**: The pixels and the game score<br>
**Output**: Q action value function<br>
Initialize replay memory $$D$$<br>
Initialize action-value function $$Q$$ with random weight $$\theta$$<br>
Initialize target action-value function $$\hat{Q}$$ with weights $$\theta^- = \theta$$<br>
for episode = 1 to M do<br>
    &nbsp;&nbsp;&nbsp;&nbsp;Initialize sequence $$s_1 = \{x_1\}$$ and preprocessed sequence $$\phi_1 = \phi(s_1)$$<br>
    &nbsp;&nbsp;&nbsp;&nbsp;for t = 1 to T do<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// Following $$\epsilon$$-greedy policy<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;with probability $$\epsilon$$:<br>
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$a_t$$ = random action<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:<br>
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$a_t = argmax_a Q(\phi(s_t),a;\theta)$$<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Execute action $$a_i$$ in emulator and observe reward $$r_t$$ and image $$x_{t+1}$$<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Set $$s_{t+1} = s_t, a_t, x{t+1}$$ and preprocess $$\phi_{t+1} = \phi(s_{t+1})$$<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Store transition $$(\phi_t, a_t, r_t, \phi_{t+1})$$ in D<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// Experience Replay<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sample random minibatch of transitions $$(\phi_j, a_j, r_j, \phi_{j+1})$$ from D<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if $$s_{j+1}$$ is a terminal state:<br>
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$y_j = r_j$$<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:<br>
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$y_j = r_j + \gamma * max_{a'} \hat{Q} (\phi_{j+1}, a';\theta^-)$$<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Perform a gradient descent step on $$(y_j-Q(\phi_j,a_j;\theta))^2$$<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// Periodic Update of Target network<br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Every C steps reset $$\hat{Q} = Q$$, i.e. set $$\theta^- = \theta$$<br>
    &nbsp;&nbsp;&nbsp;&nbsp;end<br>
end

**Notes**

* $$\phi$$ preprocessing function:
    * 210 $$\times$$ 160 pixel color image is converted to grayscale and downsampled to 110 $$\times$$ 84 image.
    * 84 $$\times$$ 84 region is cropped from the 110 $$\times$$ 84 image.
    * 4 consecutive frames form a single state. i.e. $$s_t$$ shape is 84 $$\times$$ 84 $$\times$$ 4.
* $$\epsilon$$ of the $$\epsilon$$-greedy strategy is linearly reduced from 1 to 0.1 over the first million frames and then fixed at 0.1 for subsequent frames.
* Action is decided on every $$k = 4$$ frames and repeated for frames in between.

<hr>
{% endcomment %}

## Policy Gradient Algorithms

Policy gradient algorithms directly optimize the function (policy) we care about. Value function based algorithms, on the other hand, come to the final solution indirectly.

Policy gradient algorithms are more compatible as online (on-policy) methods, while the value algorithms are better suited as off-policy. Off-policy refers different policies for exploration (i.e. how you collect your data) and update (i.e. value function update) - off-policy methods can potentially do a better job of exploring the state space.

While value function based algorithms are **more sample-efficient** since they can use the data more efficiently through better state exploration, policy gradient algorithms are **more stable**.

This is an excellent [video](https://www.youtube.com/watch?v=S_gwYj1Q-44) where [Pieter Abbeel](https://people.eecs.berkeley.edu/~pabbeel/) explains the theoretical background behind Policy Gradient Methods.

The goal is to maximize the utility function $$U(\theta) = \sum_{\tau}P(\tau;\theta)R(\tau)$$. Utility function is the return function of a trajectory $$\tau$$ weighted by the probability of its occurance. The final gradient turns out to be:

$$\nabla_{\theta}U(\theta) \approx \frac{1}{m} \sum_{i=1}^{m}\nabla_{\theta}log \pi_{\theta}(u_t^i|s_t^i) \cdot R(\tau^i)$$
<hr>

### REINFORCE
Vanilla REINFORCE algorithm is the simplest policy gradient algorithm out there. However, REINFORCE with baseline is a more popular version. Adding a baseline reduces the variance during training. The following psuedocode uses the value function as a baseline.

**Input**: policy $$\pi(a \vert s,\theta), \hat{v}(s,w)$$<br>
**Parameters**: step sizes, $$\alpha > 0, \beta > 0$$<br>
**Output**: policy $$\pi(a \vert s,\theta)$$
```
for true do
    generate an episode s[0], a[0], r[1],...,s[T-1], a[T-1], r[T], following π(.|.,θ)
    for each step t of episode 0,...,T-1 do
        Gt <- return from step t
        delta <- Gt - v(s,w)
        w <- w + beta * delta * gradient
        θ <- θ + alpha * gamma * delta * gradient
```

The following GIF shows the [REINFORCE](https://github.com/pytorch/examples/blob/master/reinforcement_learning/reinforce.py) policy learned on the CartPole-v0 env of OpenAI gym.

<img src="/assets/rl-intro/env.gif">
{% comment %}
<hr>

### Actor-critic
**Input**: policy $$\pi(a \vert s,\theta), \hat{v}(s,w)$$<br>
**Parameters**: step sizes, $$\alpha > 0, \beta > 0$$<br>
**Output**: policy $$\pi(a \vert s,\theta)$$
```
initialize policy parameters θ and state-value weights w
for true do
    initialize s, the first state of the episode
    I <- 1
    for s is not terminal do
        a ~ π(.|s,θ)
        take action a, observe s', r
        delta <- r + gamma * v(s',w) - v(s,w) (if s' is terminal, v(s',w) = 0)
        w <- w + beta * delta * gradient
        θ <- θ + alpha * I * delta * gradient
        I <- gamma * I
        s <- s'
```
{% endcomment %}
