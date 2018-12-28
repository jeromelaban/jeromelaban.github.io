---
layout: post
title: "[WP7Dev] Diagnosing StackOverflowExceptions, or the lack thereof"
date: 2011-09-21 19:04:00 -0400
comments: true
category: Archive
tags: []
redirect_from: ["/post/2011/09/21/WP7Dev-Diagnosing-StackOverflowExceptions-or-the-lack-thereof.aspx", "/post/2011/09/21/wp7dev-diagnosing-stackoverflowexceptions-or-the-lack-thereof.aspx"]
author: jay
---
<!-- more -->
<p>Developing for Windows Phone is a lot of fun, but there are <a href="http://jaylee.org/post/2011/07/29/wp7dev-Error-code-0xc00cee65.aspx">some times</a>, where it's a bit frustrating.</p>
<p>From time to time, you'll find that your app exiting for no apparent reason. There are no exceptions raised, either in the emulator or a real device.</p>
<p>For most of the exceptions, like navigation or XAML parsing exceptions, you can use the Debug / Exceptions dialog (given that you <a href="http://jaylee.org/post/2010/07/05/VS2010-On-the-Impacts-of-Debugging-with-Just-My-Code.aspx">use the "Just my code" feature </a>wisely) and break the execution at the proper location.</p>
<p>But there is a kind of exception that will not be caught by the exception handling : The StackOverflowException.</p>
<p>If you run this nice piece of code :</p>
<pre class="brush: c-sharp">public int Test(int i, int j, int k)
{
    return Test(i + 1, i + 1, i + 1);
}
</pre>
<p>Well guess what, your application will exit without any&nbsp;notice, and it won't seem like an unhandled exception.</p>
<p>This particular example is simple, but if you got a large codebase, or that the StackOverflow is in a separate thread, finding the culprit can be pretty difficult in the dark...</p>
<p>&nbsp;</p>
<h1>Debugging a StackOverflowException</h1>
<p>&nbsp;</p>
<p>So how is it possible to debug that kind of exception&nbsp;? Well, there are a few options like guessing with the use of Debug.WriteLine or putting some breakpoints at various places to find which part of the code causes the stack overflow.</p>
<p>But the Windows Phone 7.1 SDK comes with a nice tool : <a href="http://msdn.microsoft.com/en-us/library/hh202934(v=vs.92).aspx">The Performance&nbsp;Profiler</a> !</p>
<p>The tool has features to analyze the performance of the application by doing stack sampling, visual tree analysis, GPU and Compositor thread analysis, Thread context breakdown, and many more.</p>
<p>But for this particular issue, getting a sense of what's been executed can be pretty useful.</p>
<p>So if we run the previous piece of code using the profiler, this is what we get :</p>
<p><img src="/pics/wp7_profiler_02.PNG" alt="" /></p>
<p>Pretty interesting read. It becomes clear that the "Test" is the problem.</p>
<p>You may also want to run the profiler on the a real device, as a full blown I7 is filling the stack in less than 10ms... And don't let it run too long as well, as the profiler will not like it very much, and you might crash visual studio&nbsp;:)</p>
{% include imported_disclaimer.html %}
