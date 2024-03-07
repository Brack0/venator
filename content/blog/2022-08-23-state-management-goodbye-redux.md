+++
title = "Goodbye Redux"

[taxonomies]
tags = ["State Management", "Opinion", "Dev"]
+++

Let's talk about application-level state management. Local states (React Hook, Vue Composable, Angular Service + RxJS Subject) have limited use by definition. With the extent of the React ecosystem in the professional world, Redux has become the obvious answer to the State Management challenge. Let's explore the fairly natural process that leads us to Redux and take a step back to reconsider our choices.

<!-- more -->

Disclaimer: If you were hoping for a plea for GraphQL, you're out of luck. Don't worry; there are plenty of other articles that discuss it. Here, we will focus on the concept of State Management, not on the impacts that an API like GraphQL could have on your architecture and State Management.

## Getting started

Let's start with a frontend build project. This project can use the technologies of your choice (Vue, React, Angular, Svelte, etc.). For the rest of the article, we'll base our discussion on Angular.

So why is Angular a good candidate, you may ask? Simply because many Angular projects start and sometimes remain without a State Management solution. The combination of RxJS and singleton services can solve most needs (SSOT: Single Source Of Truth, "reactive" data, separation of data reading and modification). Therefore, State Management comes later and not always in the right way, even though it becomes critical. For other frontends, the problem remains the same, even if some use local state management solutions such as React Hook and Vue Composable.

But let's return to our build project. The issue of State Management comes up. Indeed, the application is growing, and data management is becoming chaotic. Shared data is modified in multiple places. Some states are duplicated, and synchronization problems arise. Ultimately, the tools provided by the framework are no longer sufficient to handle all state-related issues within the application. Fortunately, our savior is here: the Redux pattern.

## Implementation

Being on an Angular project, we naturally turn to NgRx, which seems to meet our expectations. Indeed, NgRx presents itself as a solution to this problem:

> You might use NgRx when you build an application with a lot of user interactions and multiple data sources, or when managing state in services are no longer sufficient.

So we embark on implementing the Redux pattern, which involves adding many elements (store, state, actions, selectors, reducers, sometimes combineReducers, etc.). Even when using action creators and reducer generators, we quickly end up with a substantial structure (boilerplate). A cost that seems to be accepted, or even acceptable, for some. However, you also have some junior developers on the project, and you don't want to subject them to the famous JS Fatigue\* that you may have experienced.

{% advice(type="info", name="JS Fatigue") %}

Many developers feel overwhelmed by the rapid and perpetual evolution of the modern JavaScript ecosystem and feel the need to learn most emerging technologies to do their job properly. This phenomenon is, among other things, caused by the fear of not keeping up or choosing outdated and/or suboptimal solutions.

{% end %}

## Next step

To simplify usage within the application, you then proceed to implement facades. And it's natural; this design pattern is there to address these symptoms by providing a simple interface that hides the complexity of a system/module. This allows us to group all the management of a business concept (such as user session) into a single facade. In addition to simplifying usage, it also brings more meaning to the application by focusing on business concepts rather than the technical elements they rely on. All positives, right?

## What's wrong?

So what's wrong with our solution? The answer is actually quite simple when we step back from the scenario presented: the solution is disproportionate to the use cases. The power of Redux comes from extreme decoupling between elements. Adding facades, while not problematic per se, reintroduces some coupling. In conclusion, the solution is heavy to implement (the Redux pattern) but doesn't fully leverage its advantages (coupling introduced by facades). This situation is summarized by Mike Ryan (co-founder of NgRx) in a tweet:

> The Redux pattern has a high code cost to achieve indirection. Turning around and removing the indirection with facades makes me wonder why you are paying the Redux cost in the first place.

So, we can legitimately ask ourselves why we chose Redux.

## One fits all approach

Is there something better? Objectively, it's difficult to answer that question. However, it's entirely possible to answer the question, "Is there something better for our application?" And ultimately, that's the first question to ask. Which state management solution best serves our needs? Because even though Redux addresses most needs, can we absorb the development cost it incurs? Do we really need everything Redux provides? If not, it's worth looking into simpler solutions. Personally, I've identified two solutions that stand out: Undux and Akita.

## Simplify your State management

Undux presents itself as a simplified alternative to Redux and Flux. The concepts are retained, but the implementation is lighter. Reactivity is handled by RxJS. I refer you to the [Quick Start](https://undux.org/#quick-start) for examples. The "get," "set," and "on" methods replace selectors, reducers, and effects without having to write code.

However, my favorite is [Akita](https://github.com/salesforce/akita), also built with RxJS. Here, we take the recipes that work elsewhere (Flux and Redux patterns, immutability, reactive programming concepts, etc.) and build a developer-focused solution (concepts from OOP, moderate learning curve, reduced boilerplate).

Simplification also includes the choice of Vue, previously with VueX and now with Pinia. Even though Vue leaves the door open to Redux, a simpler State Management solution has been offered and even recommended in the documentation for several years.

## Choices have been made

As developers, we must make effective choices, and that's what we often do in the end. But are these choices efficient? On the other hand, a solution that perfectly meets current needs may be limited for future needs. So what do we do?

Firstly, in the vast majority of existing projects, you don't need all the features of Redux, the famous [YAGNI](https://martinfowler.com/bliki/Yagni.html). And the second response I would like to provide comes from the agile world via two fundamental principles:

> Simplicity (the art of maximizing the amount of work not done) is essential.
<!-- -->
> The best architectures, requirements, and designs emerge from self-organizing teams.

That's why understanding the needs and context of your project is crucial. With all the cards in hand and the ability to act\*, you will build a solution that best serves your product.

Have you ever been tempted by solutions other than Redux in your frontend projects? Have you taken the plunge, even on fairly substantial projects?

{% advice(type="tip", name="Inability to act") %}

If you or your team are tied down to a set of technical solutions, run away!

{% end %}
