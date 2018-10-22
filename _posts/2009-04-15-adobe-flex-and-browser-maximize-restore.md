---
layout:             post
title:              Adobe Flex and Browser Maximize/Restore
date:               2009-04-15 02:23:00 -0400
tags:               
---

So after much research and trial and error, it seems that Adobe Flex does not support detection of browser maximize and minimize actions! This is frustrating because the Flex app I'm working on now has some panels in it that resize automatically when resizing the window, but they do NOT resize when the browser is maximized or restored. It seems that the resize event isn't fired for these objects on browser maximize and restore.

The solution here is to have some JavaScript help out. In the `<body>` tag of the HTML page hosting the Flex application, add an event handler for `onresize`. You can use this event handler to call into the Flex application and let it know that the browser has resized (including maximize and restore) so that you can handle those situations.

It's very annoying, if you ask me, that this functionality isn't built in to Flex.
