---
layout: post
title: "Google Transit and Montreal's STM"
date: 2009-03-15 21:53:00 -0400
comments: true
category: Archive
tags: [".NET", "Bus Stop Locator"]
redirect_from: ["/post/2009/03/15/Google-Transit-and-Montreals-STM.aspx", "/post/2009/03/15/google-transit-and-montreals-stm.aspx"]
author: jay
---
<!-- more -->
<p>
A while ago, the Montr&eacute;al&#39;s STM transit system <a href="http://blog.fagstein.com/2008/10/24/google-transit-includes-stm-buses-metro/">announced</a> that they were now supported by Google Transit.
</p>
<p>
While it is possible to trace proper routes, Google&#39;s having the same problem as I do, which is that the STM is updating schedules per trimester. And since it&#39;s the STM that is providing the data and that it&#39;s not been updated since the 1st of January 2009, schedules have been incorrect ever since.
</p>
<p>
To be perfectly fair, I did not update the schedules in <a href="http://www.jaylee.org/stm">my application</a> since that time too by lack of time to create a proper update procedure, but I&#39;m not paid for that either... 
</p>
<p>
Now that I&#39;ve given it some thoughts, I&#39;m now streamlining the schedule updates stops after stops as long as they are out of date. Previously, I updated the database all at once, but this does not scale... Now the updates are progressive, which is far more manageable for me.
</p>
<p>
Anyway, now there may be a simple message saying that the displayed schedule is outdated, which is better than trusting the time and blaming the STM for no reason :) 
</p>

{% include imported_disclaimer.html %}
