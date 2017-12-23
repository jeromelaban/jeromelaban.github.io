---
layout: post
title: "Precision Timer in .NET 2.0"
date: 2005-11-18 16:25:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2005/11/18/Precision-Timer-in-NET-20", "/post/2005/11/18/precision-timer-in-net-20"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">If you&#39;ve been using the .NET Framework since the beginning, you must have had to do some early&nbsp;code profiling, or framerate computation for realtime graphics.</font>
</p>
<p>
<font face="Tahoma" size="2">The first idea that pops up&nbsp;to acheive this&nbsp;is to use the <font face="Courier New">DateTime.Now</font> property and do some subtraction of two instances. This is not a good idea since the resolution of this timer is around 10ms or so, which is clearly not enough (and your framerate counter may not go higher&nbsp;than 100 FPS or worse may not work at all).</font>
</p>
<p>
<font face="Tahoma" size="2">If you&#39;ve been in the business for long enough, and been doing some plain old &quot;native&quot; code in, say,&nbsp;C++ on Win32, you should probably used the couple <font face="Courier New"><a href="http://msdn.microsoft.com/library/default.asp?url=/library/en-us/winui/winui/windowsuserinterface/windowing/timers/timerreference/timerfunctions/queryperformancefrequency.asp">QueryPerformanceFrequency</a>/<a href="http://msdn.microsoft.com/library/default.asp?url=/library/en-us/winui/winui/windowsuserinterface/windowing/timers/timerreference/timerfunctions/queryperformancecounter.asp">QueryPerformanceCounter</a></font> to get the job done. And the same goes for .NET 1.0/1.1. Well, I don&#39;t know for you, but each time I have a project that reaches a certain critical size, I always need to use this kind of timer and I end up by writing the P/Invoke wrapper to reach these two&nbsp;methods.</font>
</p>
<p>
<font face="Tahoma" size="2">Good news is, .NET 2.0 already has this class integrated in the form of <font face="Courier New"><a href="http://msdn2.microsoft.com/en-us/library/system.diagnostics.stopwatch.aspx">System.Diagnostics.Stopwatch</a></font>, so you don&#39;t have to write it from scratch again and again because you can&#39;t find on the net&nbsp;the right &quot;free&quot; class that does enough for you.</font>
</p>
<p>
<font face="Tahoma" size="2">The BCL team has added some other nice utility classes like this one, and this saves quite some time.</font>
</p>

{% include imported_disclaimer.html %}
