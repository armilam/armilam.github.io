---
layout:             post
title:              Simple XML parsing in iOS
date:               2012-09-19 17:01:00 -0400
tags:               
---

I recently had need to parse XML in an iOS app. A while back I learned how to do it with [NSXMLParser](https://developer.apple.com/library/ios/#documentation/Cocoa/Reference/Foundation/Classes/NSXMLParser_Class/Reference/Reference.html), but that's more complicated than I wanted. It is event-based, which means it's done asynchronously. That's great and all for more complicated applications, I'm sure, but I had some very simple XML that should take milliseconds to parse and I didn't want to mess with async on that.

So I found [TBXML](https://github.com/71squared/TBXML). It's simple. It's perfect. You get a reference to the root element and find children based on order or on element name. From any given element, you can look at that element's children or its siblings. And, of course, you can get element attributes and element text.

Try it out, I recommend it.
