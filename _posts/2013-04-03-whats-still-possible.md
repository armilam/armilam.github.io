---
layout:             post
title:              What's Still Possible
date:               2013-04-03 12:50:00 -0400
tags:               
---

This is a lesson for the newer programmers out there. And experienced programmers, too; I learn this lesson over and over again.

No matter how much you've patched up your program, no matter how many holes you've closed in your code, no matter how perfect it seems, it's **still possible** to break your precious application. The lesson here is to know what's acceptable and how to handle those cases you can't control.

For example, got "perfect" code that never crashes? What if the computer is suddenly unplugged _while it's in use_? Like I said, it's always possible to break your application in some way. My advice: let it fail gracefully.

If your app is a word processor, for example, save the document's state constantly (in a separate, hidden file - sometimes people like to make unsaved changes). Heck, that goes for any kind of editor.

Now, it's important to know what exactly should happen. If you're writing a banking application and something happens, don't leave the app in some unusable state. I learned this lesson from an iPhone banking app I wrote. I used nifty alert boxes to inform the user that data was happening (in user-friendlier words). These alerts prevented the user from doing anything so they couldn't break anything (concurrency-wise (how many parentheses can I use (and will I balance them correctly? (sorry, I'm reading a book on Lisp)) in this post?)). Problem was, if something unexpected were to happen, these alerts would sometimes stay up and the user would have to manually kill the app to make it usable again. Bleh, I would delete that app immediately. Fortunately I made it better.

I'm not going to sit here and pretend that I can tell you all the cases to look for. I'd be rich if I could solve the software bug problem once and for all. Just take this one thing away: you can't possibly think of all the ways your application can fail. The best you can do is try to handle unexpected failures gracefully.

_This post is part of a weekly [Blog Battle](http://swanson.github.com/blog/2011/10/13/sep-blog-off.html)-turned-tournament started by some of my coworkers. Each week, bloggers are matched up based on the tourney bracket and a random topic/title is chosen for each match. The bloggers get to interpret the title however they wish. My [SEPeers](http://www.sep.com/) get to vote for the winner and the champion is, well... the champion!_
