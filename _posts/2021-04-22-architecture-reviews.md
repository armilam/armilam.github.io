---
layout: post
title: Architecture reviews
date: 2021-04-22 01:27:00 -0400
tags:
---

Someone recently asked to hear about others' experiences with architecture design reviews, teams, and processes. I shared one experience from a past job and I think it's something worth sharing here, too.

---

#### &nbsp;

I was part of implementing an architecture process at one of my past jobs. I’m not sure the way we did it was entirely effective. We built it up over time as perceived needs arose.

At first, we were finding that some database migrations were causing the app to lock up at deploy time because not everyone was aware that, say, adding a NOT NULL column to a large table can do that. So we implemented a review process for those where someone experienced with PG would sign off on migrations.

Later we noticed that sometimes people would add dependencies redundant to other dependencies that they weren’t aware we already had, so we tacked that on to the review process.

As our system became more complex, we decided we needed some people who had an awareness of all the parts and how they fit together. So any new features over some threshold of complexity had to have an architecture document and review by the ADT. That complexity threshold wasn't well defined - it was more of a list of things to look out for, which did not help with clarity in the process.

A problem with the way we did this is that once a team came up with a design and asked for a review, they’d already put a lot of work into it and it was hard to have them change course when they needed to. It wasted time. And the team wasn’t always clear about what needed review.

I wish we’d done a more collaborative approach where an architect would join the team for their design process rather than have the team come to the architects for review. It would have saved time, feelings, and helped the team learn more effectively about how to make good architectural decisions.

Anyway, the process was okay. But I think that bringing everyone along the process would have been better than inserting a step into their process.
