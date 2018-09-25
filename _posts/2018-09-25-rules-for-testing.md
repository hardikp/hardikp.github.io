---
layout: post
comments: true
title: "My Rules of Thumb for Unit/Integration Tests"
excerpt: ""
date:   2018-09-25 12:00:00
mathjax: false
---

Testing is not all that black and white.

IMO, there are times when unit tests help, there are times when integration tests help and there are times when it’s best not to write any tests at all.

Based on my experience working with huge codebases as well as starting codebases from scratch and working with teams as well as working on a project on my own, I have found that the optimal test/testing requirement varies a lot. However, in times of doubt, it’s probably **a good idea to err on the side of writing tests**. (Disclaimer: I hate writing tests.)

These are my rules of thumb for unit/integration tests (they are still evolving):

* Do not write tests if the interfaces are evolving/changing fast. There will be a lot more hesitation to make needed interface changes if you write tests very early on in that process.
* You can choose not to write tests if you’re the only one responsible for maintaining it and there isn’t any big consequence if it breaks.
* You should write tests if multiple people with different experiences are involved.
* You have to write tests if the cost of breaking something downstream is very high.
* Do not overdo unit/integration testing — recognize why a specific test is needed before writing one. In other words, understand the potential cost of not writing a test.
* Integration vs unit: If you have to choose one, choose the integration tests. Because ultimately, it’s the final system that matters the most.
* Invest in a good CI infrastructure.

It also matters how a test is written. I have seen some really good tests (i.e. readable, serves the purpose and concise) tests as well as very bad tests. Good tests can also serve as an unofficial guide to using the interface. Bad tests, on the other hand, confuse people about the purpose of the interface and unnecessarily create an obstacle to further changes.
