---
layout: post
title:  "Bringing vision into legacy code"
date:   2020-12-26
categories: "leading-change"
author: Christoph Portmann
status: "draft"
---

This post is about how to inject and persist architectural vision in a code base, based on recent experience. 
 
1. **High-level approach** to restructure and refactor of a messy code base
2. **Persist architectural vision** outlives people and team changes and moves in a single direction.

## The tale of my own legacy
All code is legacy, we get it. But some code comes with a long history and code always reflects the opinion and knowledge of all the people that have worked on it. Recently I had the honor to work on a rather small, but messy code base that has been passed on between teams multiple times over the course of 3 years. In fact, I had worked on it myself for around 5 months last year and I remember very well how we were sitting in a small meeting room with stifling air, talking over the code smells and tried to come up with ideas on how to make it shine.
Due to organisational reasons, it was soon decided that this code base should be transferred to another team, in fact, to another continent. In a remote 'hand-over' session, we transferred our gained knowledge as best we could and explained our half-baked ideas on what would be the next steps.

One year later this code became a bottleneck for the project and the assumption is that no more than two pairs are able to work on it in parallel. As an experiment we put together a team of two pairs, one from each team, with the mission to make work on this code base more parallelizable - of course with the premise that we keep delivering new features in this code base. 
As part of this new pop-up team the code base came back to me after over a year. The good news: The other team has made vast improvements and did an excellent job with the time given. But big parts were untouched and moreover our half-baked solutions from one year ago are now just additional complexity and became part of the problem.
It's great to work in a small team on a single code base, even if the code is messy. The scope is very narrow, which leaves our brains the energy to understand the technical details in-depth and make the right decisions on what, how and when to refactor. 

## Restructuring Approach
It's of course not economical to aim for cleaning up everything. The goal is to create a well structured base, in which we can efficiently implement changes (features) and evolve further according to the business needs. Small messy parts that are rarely touched, can be moved into isolation until the right moment to change it, which is usually when it needs to be touched or when someone takes particular interest in that part. 
We approached the refactoring in four steps: 
1. **Get a shared understanding of the business context**   
Since we have multiple different minds on this code, from historic knowledgeable (me) to recent context (pair from other team) to no context (my pair), a simplified event storming session helped us to get to know what we are dealing with business-wise. This is very important, because during the refactorings we often have to distinguish between essential and accidental complexity in order to eliminate the latter. Further it helped us to talk about the code by having a common understanding of terms.
2. **Name the mess**   
We collect the low-hanging fruits. This means reading the code, split and move files to reach higher cohesion and rename things to reflect their semantic - the terms from the event storming session come in handy here. At this point we don't really touch the code itself, but simply rename and move files, classes and (top-level) functions. Using Intellij, these are quite risk-free refactorings with immediate benefit. For example a package on top-level is called 'api', inside are 2 files that implement two specific features, which we move to their corresponding feature package and delete the api package.
Another example is a bunch of tools, which are not directly related to business cases (e.g. swagger setup) and which we collect in a tools package or move other things to a config package. 'tools' might not be the best name, but the important thing here is to split the mess into overseeable parts that can be isolated and further worked on. 
One of our measurements of success is, that a new developer should be able to intuitively navigate to the code related to a feature they are working on
3. **Identify and prioritise the hot spots**   
While a lot of things are tightly coupled, we could identify a few hot spots that were central to the application logic and particularly complex. Identifying these hotspots was mostly done exploratory. Tools like a dependency graph help to visualize and explore the mess in this phase. Some hot spots, especially in the frontend became very obvious during the feature work. Such discoveries are actually the most valuable ones, because solving them first will have an immediate impact on our feature delivery speed, which makes them cheap improvements.
4. **Restructure**
This is the most fun phase for the devs! We refactor the code to make it more understandable, maintainable and evolvable. It's advisable to limit and coordinate the bigger refactorings, as some might go deep into the fundamental structure of the application. In an ideal case we do this as part of the feature work, but since we want to significantly improve the code base in limited time, we also run refactorings as tasks on their own, so that a pair can concentrate on finding the best way. While refactoring per definition excludes changing behaviour, don't be afraid to question existing logic. Again the question is, if this complexity is accidental or essential. One of the greatest feelings is when you spend half a day to understand what a particularly complex part is doing, just to come to the conclusion, that we don't need it at all.


## Continued Vision
It's natural that the entropy of a code base increases with the features and changes added to it. Therefore refactoring should be part of everyday development work. But changes and refactorings need to go into one direction, a common vision on how the code base should look like. A good analogy I heard for this is that "a code base should look like it's written by a single person". In teams this is supported by the direct communication between the members which happens especially while pairing and in meetings like a Tech Huddle. But in fact it can be shaped by any interaction and casual conversation team members have. With bigger teams this becomes increasingly difficult and when a code base is even passed on between teams, the communication of the vision barely happens. So considering that this team has an expiry date, a big concern is how we can transfer the vision and knowledge that we are developing in this short but intense time to whoever will own this codebase. The vision should be translated into the code base. Ideally it is readable and obvious, when a developer wants to do a change, they should feel "this is how I should do it". But even if the code base still is a bit messy, we want to ensure the direction it goes. We decided on a few tools, which have yet to show their effectiveness in the future.

1. **[ArchUnit](https://github.com/TNG/ArchUnit)**   
ArchUnit is a testing framework for the architecture of a JVM based application. Our code base is in Kotlin and the framework does a wonderful job. It allows us to express our intentions in small tests that then asserts that the rule is not violated. Here is a simple example of an ArchUnit test. ArchUnit is a really amazing tool and I'm yet to find out if similar tools exist for other technologies.

[TODO: Inject example]

2. **Manifesto / Code of Conduct**   
We created a Code of Conduct in the root folder of the code base, requesting any external contributor to read it before writing code. The goal of the code of conduct is to raise awareness, not explain in detail. It describes in short the business scope, the architectural goal of the code base, current state, as well as Do's and Dont's. The document should be short enough for anybody to read it in 5 minutes, not an extensive documentation. The points are at least mid-term, so that they don't need to be updated every week. Some parts might be a bit more short-lived, such as "Don't write more Enzyme tests, we want to migrate to React Testing Library tests" or "Do not add more properties to the Preview class, we want to move to more specialised interfaces". Where possible, move such things to ArchUnit tests, where they are much more practical.

3. **Encourage continuous migration with pre-commit hooks**   
On a similar line as the ArchUnit tests, we've introduced a git pre-commit hook (using husky), which runs through the touched files and verifies that certain things are not present anymore. This supports a continuous refactoring approach to things like renaming or change of tool. For example whenever you have touched a test, all junit4 tests should be junit5 now and the name 'OutputView' should not appear anymore in any variable name, but be replaced with 'View'. Scanning through the file, the script searches for appearances of unwanted content and prints the violation to the console. Each violation has an explaining comment which is printed along-side. Violation: File Application.test.kt contains org.junit.test. Please migrate Junit4 tests to Junit5.


## TL;DR
We looked at an approach to restructure a messy code base in four steps:
1. *Get a shared understanding of the business context*: bring everything on the table, share knowledge and agree on terms 
2. *Name the mess*: Get low hanging fruits by moving and renaming things according to their semantic.
3. *Identify and prioritise the hot spots*: Analyse the code base to identify where complexity lies, prioritise it according to impact on the upcoming work.
4. *Restructure*: Limit and coordinate the more complex refactorings to stay focused. Don't be afraid to question logic.

We looked at a three tools that support the code base in moving into one direction, even if the people around it change:
1. *ArchUnit*: for expressing intention in form of tests.
2. *Manifesto / Code of Conduct*: For sharing the core ideas and direction with any person unfamiliar with it, explicit and in a short time.
3. *Encourage continuous migration with pre-commit hooks*: For verifying that no unwanted content is present in any touched file, this pushes developers to apply the Boy Scout rule.

Send me some feedback on this post via Social Media.
