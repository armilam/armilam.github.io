---
layout:             post
title:              IIS Windows Authentication Problem on the Domain
date:               2009-05-15 12:13:00 -0400
tags:               
---

I was involved in setting up an IIS web site that uses Windows Authentication for users on the domain. When testing it out before full deployment, it was discovered that Firefox would send Windows credentials to the server as expected, but Internet Explorer just kept asking the user to enter username and password.

It turns out that happened because the users were accessing the web page using the server's IP address since the server didn't have a domain entry setup yet. IE didn't recognize that the server was on the same domain as the client and therefore thought that it shouldn't send Windows credentials.

So let it be known that IE doesn't send Windows credentials unless the server is known to be on the same domain as the client.
