---
layout:             post
title:              Email From Your iPhone App
date:               2009-08-05 13:44:00 -0400
tags:               
---

Programming your iPhone application to set up an email for the user to send is easy, as long as you don't mind it quitting your application. The following code will open the Mail application with a new email ready to be sent with the given email address, subject, and email body.

```objc
NSString *emailLink = [NSString stringWithFormat:@"mailto:address@example.com?subject=My Subject&body=%@", [@"This is the body of the email message" stringByAddingPercentEscapesUsingEncoding:NSASCIIStringEncoding]];
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:emailLink]];
```

This works the same way as clicking a 'mailto:' link on a web page.
