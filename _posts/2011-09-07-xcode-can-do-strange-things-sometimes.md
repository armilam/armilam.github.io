---
layout:             post
title:              Xcode can do strange things sometimes
date:               2011-09-07 14:28:00 -0400
tags:               
---

I am really not sure what caused this, but I can tell you what was wrong.

I'm building a navigation-based iPhone app at work. I'm making custom UITableViewCells for a UITableView. When this particular table view was getting created, I set the text of a couple of labels. Both labels were connected properly in Interface Builder. The first label got set properly.

But trying to set the second label gave me EXC_BAD_ACCESS. Turns out the address to the second label was 0x3. I still have no idea how this happened. But I know how I fixed it.

I just went into Interface Builder, disconnected the label, reconnected it to the IBOutlet, cleaned the project, and tried again. It worked. Strange errors, simple solutions.
