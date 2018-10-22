---
layout:             post
title:              Emulating the iPhone in an iPhone Web App
date:               2009-07-16 13:49:00 -0400
tags:               
---

I've been working with iUI, a CSS- and JavaScript-based framework for building websites that appear just like iPhone applications on the iPhone. You can use the same TableViews, TabBars, swiping actions, and even landscape/portrait mode. With this framework, it is possible to create a web application that looks very similar to an iPhone app that uses the iPhone's built-in controls and features. 

It turns out that this is a bad idea. 

While it is possible to do, just about every aspect of it is ugly. Because of Mobile Safari's incomplete support of CSS fixed, implementing several screen items involves using JavaScript to keep them in place. Positioning of other elements is a nightmare. Testing in anything but Mobile Safari will give you drastically different look and functionality. And the end result is somewhat unimpressive. 

I recommend that a web application targeted for the iPhone be thought of as a web application, not an iPhone app. Extensive use of CSS can make an iPhone web app very pretty, but do avoid fixed elements. You can use elements that look similar to iPhone controls, but the design of a good web application has clear differences from the design of a native application. 

What you can do with iPhone's Mobile Safari is pretty cool. To make sure that Mobile Safari displays the web pages without zooming, the following meta tag works. It defines the page's width at 320 pixels and prevents it from being scaled to any scale other than 100%.

```html
<meta name="viewport" content="width=320; initial-scale=1.0; maximum-scale=1.0; user-scalable=0;"/>
```

The ability to support landscape and portrait modes is a very nice feature.  For CSS specific to portrait mode, use

```css
body[orient="portrait"]
```

and for landscape mode use

```css
body[orient="landscape"]
```

Web applications targeted for the iPhone really can be very versatile. Look up CSS examples and tutorials for iPhone web applications to find what all Mobile Safari is capable of. Just don't try to make it look like an iPhone app.
