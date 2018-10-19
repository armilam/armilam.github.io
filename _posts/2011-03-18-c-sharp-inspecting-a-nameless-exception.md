---
layout:             post
title:              "C#: Inspecting a nameless exception"
date:               2011-03-18 17:06:00 -0400
tags:               
---

Finding the answer to this eased a great pain for me today.

C# allows you to catch an exception like this:

```cs
try
{
    if (Installation_State == InstallationState.Installed)
    {
        // Some code that may throw an exception
    }
}
catch (Exception)
{
    Installation_State = InstallationState.Not_Installed;
}
```

When I'm debugging in Visual Studio, how do I inspect that exception? It does not have a variable name! I knew there had to be an easy way, and a coworker led me to it. In the debugger Watch list, watch the variable `$exception`.

<div style="clear: both; text-align: center; margin: 2em;">
  <img src="/assets/2011-03-18-c-sharp-inspecting-a-nameless-exception/watchlist.png" alt="$exception in the Watchlist" title="$exception in the Watchlist" />
</div>

Alternatively, you can mouse over the "catch" to show the little red error icon. Clicking on that will give you the same information.

<div style="clear: both; text-align: center; margin: 2em;">
  <img src="/assets/2011-03-18-c-sharp-inspecting-a-nameless-exception/catchinfo.png" alt="Catch info" title="Mouse over 'catch' to get the exception information" />
</div>
