---
layout: post
title:  My Hacker Muscle Has Atrophied
date:   2019-03-12 22:15:00 -0400
categories:
---

I have found myself building a new web app in my spare time. I have a need which I am trying to solve - that's how these things start, right? It's been a while since I started something in my own time that I was excited about, so it feels good to be building something outside of work.

### Discovery

I started by identifying what I need a tool to do to solve my problem. I asked myself: what am I trying to do that is difficult right now? What would make that easier?

This line of thinking ended up with me creating an affinity map of the things I would like my tool to do. It's pretty extensive. It includes solving my basic problem and some really neat tangential features. I identified the area that is most important to me and I thought through a data model that would allow me to store the data I need to build a tool.

### Engineering

I began building a Rails app (it's what I know best at the moment). I knew what I needed to start with: a simple Rails app with a Postgres database and [Clearance](https://github.com/thoughtbot/clearance) for easy user management. And I did it with Rails 6.0.0.beta2, because why not live on the edge? Then I began building my tool.

The code is beautiful, really. The controller actions are short. The business logic lives in service objects. The code is DRY and accommodates multiple similarly-behaving integrations. But I still don't yet have something I can test in the real world.

As I'm writing my code, I'm realizing something. I'm not hacking. Instead, I'm thinking through the architecture. I'm considering how I want everything to work together in the future. **I am slowing myself down.**

### Being Lean

One of the more important ideas from Lean is that you should try something quick and see how it does before building something elaborate. If you build the elaborate thing first and end up realizing that what you built doesn't actually solve the problem you're trying to solve, then you're stuck with having spent a lot of time on something that doesn't work _and_ you're stuck with the large task of changing that elaborate thing. When you build something small first, you can test how that does and find out what needs to improve without having wasted any effort.

I've been in an architecture role at my job for a while now. We have a relatively mature ecosystem where everything needs to work both for our customers and for the engineers. We need a well thought-out way of doing things to make sure that our application's complexities are easy enough to follow, are performant, make sense to our end users, and that they just work. I have been in this mindset for so long that I'm finding it difficult to go back to hacking.

### Evolution

At my previous job, quickly hacking something together was frequently a useful skill. I could whip up a project and have something to show for it in a few hours' time. I could get some feedback and iterate before the day was over. This was my biggest strength.

That skill was important at my current job at Lessonly when I first started. When I joined, we were still an early stage startup, so we were trying things left and right to see what would stick and what would sell. When I first started, the transition was easy because I was doing the same kind of work - just in a single project instead of many.

As time went on, we hired more engineers and my role began to shift. I became a manager for a while. We as a team realized we needed to standardize how we work. The application became more complex, so we needed to build an architecture that made sense. And then I shifted into a role overseeing that architecture.

Now I spend my days mitigating risk through how we build software. It's my job to make sure that everything is built in a way that makes sense now and is going to continue making sense for the foreseeable future. I have to predict the problems we will have and make sure we are able to prevent them. This is my new _modus operandi_.

### Re-Learning to Hack

I'm glad I realized what is going on. The most effective thing I can do right now is to get something working so I can try it out. Otherwise, I'm building toward a dream that I hope is as awesome in reality as I'm imagining.

It's better to start in the wild west. Sure, I can write beautiful code, but what use is it if I'm not positive it's going to live for more than one iteration? The best thing I can do is hack together something that works and try it out. And if I like it, I can build on it. And refactor it to my heart's content.

I hope I build something neat that I can share with you all some day soon.
