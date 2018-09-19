---
layout:             post
title:              What happened to my orientations? A story of iOS6
date:               2012-10-22 14:34:00 -0400
tags:               
---

So what did happen to my iPad app's orientations? I updated to iOS 6 and all of a sudden, my app stopped rotating.

Well, here's what happened. Apple changed how orientation support works on us. The old method of `shouldAutorotateToInterfaceOrientation:` is now deprecated. Instead we have the following.

When the user rotates the device, the system calls [`supportedInterfaceOrientations`](http://developer.apple.com/library/ios/documentation/uikit/reference/UIViewController_Class/Reference/Reference.html#//apple_ref/occ/instm/UIViewController/supportedInterfaceOrientations) on the root view controller or the topmost view controller. This method should return a bitmask of [`UIInterfaceOrientationMask`](http://developer.apple.com/library/ios/documentation/uikit/reference/UIViewController_Class/Reference/Reference.html#//apple_ref/occ/instp/UIViewController/definesPresentationContext) to identify which orientations are supported.

The system also calls `shouldAutoRotate` before rotating. Simply put, if you want to autorotate, implement this and return true.

One thing to note is that the system checks your target's settings for supported orientations first. Make sure all of those are marked as supported that should be.

### Autorotation with UINavigationController
If you are using a `UINavigationController` and you have different views that support different orientations, you could have your navigation controller ask the current view. Subclass `UINavigationController` and use that for your navigation controller. Then add this to your subclass:

```
- (NSUInteger)supportedInterfaceOrientations
{
    return [self.topViewController supportedInterfaceOrientations];
}

- (BOOL)shouldAutorotate
{
    return [self.topViewController supportedInterfaceOrientations];
}
```

And this to each of your UIViewControllers, specifying the appropriate bitmask:

```
- (NSUInteger)supportedInterfaceOrientations
{
    return UIInterfaceOrientationMaskPortrait | UIInterfaceOrientationMaskLandscapeLeft;
}

- (BOOL)shouldAutorotate
{
    return YES;
}
```

You can use a similar solution for other navigation controllers, such as `UITabBarController`.

### Listening to autorotation events
One nifty new feature of this system is that it's much easier to listen for autorotation events. Simply implement the following and place any code you need in order to react to a rotation.

```
- (void)willRotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation duration:(NSTimeInterval)duration
{
    // Code to react to rotation here
}
```

Go ahead and leave in `shouldAutorotateToInterfaceOrientation:` for now in order to support iOS 5 and earlier. I would advise changing those methods simply to use your new iOS 6 code to avoid the maintenance nightmares of having code that does the same things in two places.

```
-(BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation
{
    NSUInteger supported = [self supportedInterfaceOrientations];
    return (toInterfaceOrientation == UIInterfaceOrientationPortrait &amp;&amp; (supported &amp; UIInterfaceOrientationMaskPortrait))
        || (toInterfaceOrientation == UIInterfaceOrientationPortraitUpsideDown &amp;&amp; (supported &amp; UIInterfaceOrientationMaskPortraitUpsideDown))
        || (toInterfaceOrientation == UIInterfaceOrientationLandscapeLeft &amp;&amp; (supported &amp; UIInterfaceOrientationMaskLandscapeLeft))
        || (toInterfaceOrientation == UIInterfaceOrientationLandscapeRight &amp;&amp; (supported &amp; UIInterfaceOrientationMaskLandscapeRight));
}
```
