---
layout: post
title:  What Is a Software Architect?
date:   2018-10-17 21:20:00 -0400
categories: 
---

I have been thinking a lot about what it means to be a Software Architect, primarily because I am working to define that role at Lessonly. What does an architect do that is different from other engineers on the team? What do they *not* do? What skills do they need to have and how do they use them?

## It's a way of thinking

Architecture, to oversimplify it, is a way of thinking. Architects are able to think in systems. They think about how systems are defined, how they interact with each other, how they are structured and organized. Non-architects sweat the details within the systems while architects think a level above that. Both of these mindsets are absolutely essential on a software team.

## Interactions between systems

Any software application comprises several parts, whether they are disparate services, modular components, or layers within a single application. Following the principles of [SOLID](https://en.wikipedia.org/wiki/SOLID), the parts should not rely on each other directly; they should only rely on each other's contracts. That is, each component should define how it communicates with other components, and the components should communicate with each other only along their defined APIs.

An architect might think about those interfaces, how they are defined, how they might affect the components in the future, how they might be used, and how the underlying systems might need to change in the future. They understand the current needs of each system and consider their possible futures so that they can design the APIs to meet those needs.

## Considering stakeholders' needs

An architect considers the needs not only of software systems, but also of the business and of those who interact with the software. For example, a SaaS startup is likely to need to optimize for the ability to develop and deploy software quickly, while a more mature SaaS company probably needs to think more about security, data privacy, and performance to keep their larger customers feeling safe.

The startup is more likely to keep its systems consolidated so that developers can write entire features quickly. The mature company likely needs to keep components separated so that they can secure and optimize each of them independently.

## Defining standards

The kinds of decisions architects make tend to be the kind that define how the software is built. They create patterns and standards such that if anything needed to change, it would be expensive to do so because those patterns and standards affect *everything*. So as the architect considers the shape of the software they are designing, they must think about how to build it in a way that considers how parts of it could change without affecting other pieces in order to minimize the impact of the change.

Still, since some of the work done by architects is to design the interfaces between systems, and the patterns that permeate through infrastructure and applications, it's often difficult to keep the impact of change minimal. This is why it's so important for architects to be able to consider present *and* future needs of all the stakeholders involved.

## Bringing it all together

Software Architecture is systems-level thinking. Architects define how the software is built. And they must consider the present and possible future needs of all stakeholders.
