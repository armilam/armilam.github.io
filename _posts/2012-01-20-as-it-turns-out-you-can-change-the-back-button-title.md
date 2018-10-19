---
layout:             post
title:              As it turns out, you CAN change the back button title in a UINavigationController
date:               2012-01-20 17:17:00 -0400
tags:               
---

Long have I wished to change the back button title in a UINavigationController. Long have I failed.

I found forum posts out there saying it's not possible, and I became discouraged.

I found forum posts out there saying you need to set the 
`self.navigationItem.backBarButtonItem.title` property. When that didn't do anything, I became discouraged.

Others said to change the view's title when changing the view, but that change is ugly and visible to the user. I'm not willing to do that, and I became discouraged.

I later found posts saying you need to set 
`self.navigationItem.backBarButtonItem.title` on the UIViewController the back button points back to. Genius, I thought. The back button title gets set independent of the current view controller, which may not know where it came from. But when that didn't work, I became discouraged.

Still others said that you need to create a new UIBarButtonItem and set 
`self.navigationItem.backBarButtonItem` to that new button. That worked, but in order to make it LOOK like a back button, you need to use an undocumented style value. Apple would not like that. Plus you have to write your own handler for that button.

I knew that had to be a better way...

Finally I found it. Interface Builder does it for you. And holy crap, is it easy. But hidden.

## The Solution

In Interface Builder, open up the .xib for the view whose back button title you need to set. That is, the view back to which the back button will point.

Add a UINavigationItem object to the .xib. That object has three settable properties: Title (obvious), Prompt (haven't tried that one yet), and Back Button (genius). Set the Back Button to the title you need, and done!

Well, not quite. You also need to connect that UINavigationItem back to your UIViewController. So create a 
`@property (nonatomic, retain) IBOutlet UINavigationItem* navigationItem;` in your header and be sure to synthesize and release it appropriately in the implementation of your view controller. And now you're done!

This is a bit of information I wish would have been easier to find. Now I get to go back through my app, find all the places where this would be helpful, and make the changes.
