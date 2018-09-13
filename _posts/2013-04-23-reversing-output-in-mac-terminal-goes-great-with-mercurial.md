---
layout:             post
title:              Reversing output in Mac terminal goes great with Mercurial log
date:               2013-04-23 14:41:00 -0400
tags:               
---

I hate Mercurial.

Okay, maybe that's a little strong. But I'm definitely not a fan. I prefer git. A lot. But sometimes, I don't have the option. I won't get into *all* the details about why I don't like Mercurial, but I will tell you about one.

For this project I'm working on, I need to report the changeset ID that fixes each bug I've fixed. So each time I commit, I want to print the last item in the log so I can copy that ID. Well, I could just do this:

```
$ hg log --limit 1
```

But that's a lot to type. So I made a script with the short name 'hl' that does the same thing. Sweet! But I have some use cases where I want the last few items in the log. So I expanded the limit. But now, that most recent entry scrolls off the top of the terminal window and I have to scroll to find it! Bah! Is there no way to show the last few items in the Mercurial log in reverse order!?

After doing some research on the internet (they can't put anything on the internet that isn't true), I found this nifty little gem:

```
$ tail -r filename.txt
```

Anything you tell it to print, it prints the lines in reverse order. Crazy! Here's a text file I called 'some_file.txt':

```
These lines are
in a specific order.
If you put
them out of order,
these sentences won't
make much sense.
```

If I want to print it backwards, I do it like so:

```
$ tail -r some_file.txt
make much sense.
these sentences won't
them out of order,
If you put
in a specific order.
These lines are
```

Almost sounds like Yoda. How about that.

So that's great for a file. How do I get the Mercurial log in backwards order? Do I have to save it to a file then use `tail -r` on that? No! I just stick it in a pipe!

```
$ hg log --limit 10 | tail -r
```

Smokin'. But you of course already know what the pipe does, right? I'll just assume you do.

Great! Now you learned two things today:
- Mercurial is no fun
- `tail -r` reverses lines of output
