---
layout: post
comments: true
title: "Thoughts on Software Development 1: Organization and Team Structures"
excerpt: ""
date:   2018-07-14 12:00:00
mathjax: false
image_url: "/assets/software-development-1/team.gif"
---

![]({{ site.url }}/assets/software-development-1/team.gif)

(Note: This is the first of potentially many posts where **I ramble on different Software Development topics**. Also, I may update these posts over time based on my evolving views.)

I am obsessed with productivity. This naturally leads me to ask what kind of organizations and team structures would produce more productive outcomes.
Is the flat structure better for productivity or is the hierarchical management style better for focused work? What are the patterns and anti-patterns of different teams?

* TOC
{:toc}

# Organizations

## Flat Structure

Flat structure is fairly common in small businesses - mostly in non-software businesses though.
Small software businesses usually prefer focused work over wearing too many hats. Such is the nature of the business.
Having said that, there are notable exceptions - [Valve](https://www.bbc.com/news/technology-24205497) being a prominent one.

![Flat Structure]({{ site.url }}/assets/software-development-1/flat-structures.jpg)

Flat structure usually involves a single CEO/manager or an executive team, while everyone else is at the same level without any titles and roles.
They may have loosely defined roles that can change depending on the requirement.

If you are an ambitious person who has worked in a silo for years in a very big company, this may sound like a utopia. It’s usually not. Sure, people may get a chance to take on more responsibilities easily. But, it’s usually not sustainable. Here’ why. The world is constantly changing. And by extension, the market in which your flat structure company competes is constantly changing as well. Flat structure often makes it harder for a company to successfully change their direction when the situation gets hard. There are too many voices and too much defused responsibility to steer them in a completely new direction. As soon as a flat structure company faces an upheaval battle they can’t seem to fight their way out, they turn to the Pyramid Structure. [Github is a classic example of this](https://www.bloomberg.com/news/articles/2016-09-06/why-github-finally-abandoned-its-bossless-workplace). They had to [change their direction](https://medium.com/battle-room/holacracy-and-the-mirage-of-the-boss-less-workplace-lessons-from-the-failures-at-github-medium-4355993926d4) so as to be able to rollout enterprise products while continuing to grow rapidly.

For a small company, a flat structure still makes sense. It allows the flexibility and the speed to change rapidly.
For a VC-backed consumer product startup, the lack of flexibility can break the company - especially in the phase before a product-market fit is identified. You have to be able to move quite fast in that phase.

## Pyramid Structure

Pyramid is the most common management structure for most of the companies with >= 50 employees.
The general setup involves a CEO who reports to the board. Rest of the executive team like COO, CFO, CTO report to the CEO. Each such executive would have their own reports and so on.
A big company like Microsoft or Google can have up to 10 levels of hierarchy.

![Pyramid Structure]({{ site.url }}/assets/software-development-1/pyramid.jpg)

I used to hate hierarchy. It was way too isolating for me. I like to get my hands dirty on as many things as possible.
Hierarchy seemed to be limiting my aspirations to try different things.
I also have a deep desire to understand everything (e.g. C++ compilers, US/World History, RL Algorithms, Biology, Psychology etc) in detail.

However, I have come to appreciate the boredom that comes with doing isolated work. More importantly, I have come to appreciate the need for focused work as well as the need to align individual (and team) objectives with that of the company. I also increasingly believe in Apple’s [Directly Responsible Individuals](https://medium.com/@mmamet/directly-responsible-individuals-f5009f465da4) theory - because I have seen how crippling the lack of responsibility can be.

# Teams

## Multiple Stakeholders

Some teams have more than one decision makers each having a different mandate. Let’s say a team consists of software engineers - one stakeholder has the responsibility to create new products, the other has the responsibility to maintain and improve existing workflows, while the third has the mandate to improve technical infrastructure. This is usually a recipe for disaster - it can lead to poor quality of work in all 3 areas. Like most things, this also has exceptions - we do see small teams of dedicated engineers in many startups deliver high-quality results on multiple fronts.

## Extreme Collaboration

I have been a big proponent of fostering extreme collaboration at an individual level. I have advocated for empowering individuals to be able to contribute in a number of different products at the same time. I now realize that this is a flawed concept. The need to focus on a part of the work is extremely crucial to achieving quality. True polymaths are rare. And polymaths who can deliver quality on multiple fronts simultaneously are extremely rare.

However, I still advocate for learning from different disciplines and various projects and applying those skills in that one thing you do. This is popularized by Charlie Munger in the form of [mental models](https://medium.com/@yegg/mental-models-i-find-repeatedly-useful-936f1cc405d). In his book [Poor Charlie's Almanack](https://www.poorcharliesalmanack.com/), he describes relentlessly learning from different areas and applying that knowledge to investing. After reading that book, I realized I can't possibly do an excellent job in both computer science problems and bioinformatics. At max, I can continue to stay updated on what’s going on - but, I have to choose a single target for my work.

## A Team Consists of Individuals

Individuals come in all forms and shapes. They may have different skills. They have different aspirations. They often have different and diverse backgrounds.
Recognizing this aspect of a team is important. Among all the things individuals do toward's a team goal, it's the individual's ability to take initiatives that adds the most value.
Therefore, fostering an inclusive and safe environment for everyone is important - you can only put yourself on the line if you know you would be safe even if the proposed solution/feature/product doesn't work out.

### Commandos vs Infantry vs Police

Some individuals may be predisposed towards prototyping new products, while others may like to be occupied with maintaining and improving existing systems. And another set of individuals may like to build stuff for scale. See [this wonderful post](https://blog.codinghorror.com/commandos-infantry-and-police/) by Jeff Atwood on a related topic. Recognizing these aspects of individuals would go a long way in making the team more productive.

### Hiring Signals

Some people (including Ray Dalio) advocate hiring people for various roles based on their **personality profile**. While this is not necessarily a bad idea, it can have **high noise** just like any other aspect of the hiring process. See [this post](https://erikbern.com/2018/05/02/interviewing-is-a-noisy-prediction-problem.html) by Erik Bernhardsson for more details on the inherent noise in the hiring process.

# Summary

Team structures can have massive consequences in what the team can produce, how fast they produce the results as well as the quality of the results. By carefully monitoring the minute aspects of the capabilities, team interactions, individual aspirations, it may be possible to align the team culture to achieve near-peak individual contributions - contributions that are aligned with the company goals.
