---
layout: post
title: "Controlling actual Thread Priority in WinRT and Windows Phone"
date: 2014-02-18 19:58:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8", "Windows Phone Dev"]
redirect_from: ["/post/2014/02/18/Controlling-actual-Thread-Priority-in-WinRT-and-Windows-Phone", "/post/2014/02/18/controlling-actual-thread-priority-in-winrt-and-windows-phone"]
author: jay
---
<p><em>tl;dr: Setting the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.system.threading.workitempriority">WorkItemPriority</a> in <a href="http://msdn.microsoft.com/en-us/library/windows/apps/br230594">ThreadPool.RunAsync</a> actually changes the thread priority the code runs on, not just the position in the pending work queue.</em></p>
<p><em>It&rsquo;s been a while since I&rsquo;ve blogged, but this entry has been a finding that had long eluded me and this was a good chance to blog again. If you&rsquo;re still reading, thanks :)</em></p>
<p>In the Plain Old (or might I say, complete) .NET framework, there was a pretty useful property named <a href="http://msdn.microsoft.com/en-us/library/system.threading.thread.priority(v=vs.110).aspx">Thread.Priority</a>, which gave a lot of control to app developers. This allowed a very control of what would run, where, and how.</p>
<p>Using this API, you could have CPU bound (hence blocking) code that could run at a very low priority, without the need to be yielded somehow, like it&rsquo;s suggested now with async and <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.yield(v=vs.110).aspx">Task.Yield()</a>.</p>
<p>I was under the impression, since Windows Phone 8.0 and WinRT 8.0 have been introduced, that there was no available way to control the actual thread priority, since either the property does not exist, or even Thread does not exist anymore.</p>
<p>The suggested counterpart, Task, does not provide such a feature, leaving developers no choice but chunking the work, by using <a href="http://blogs.msdn.com/b/pfxteam/archive/2010/04/09/9990424.aspx">clever tricks or work item priority scheduling</a>.</p>
<!-- more -->
<h2>A hidden gem</h2>
<p>In WinRT 8.0+ and Windows Phone 8.0, there is one API, <a href="http://msdn.microsoft.com/en-us/library/windows/apps/br230594">ThreadPool.RunAsync</a>, which is very similar to .NET&rsquo;s own thread pool, but provides an additional parameter, a <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.system.threading.workitempriority">WorkItemPriority</a>.</p>
<p>Intuitively, and knowing&nbsp;historically&nbsp;how thread pools work, and that they are supposed to work with relatively small units of work, I thought &ldquo;Ok, yet another API that will prioritize a work queue&hellip;&rdquo;.</p>
<p>I could not be more wrong!</p>
<p>This API not only schedules in the queue using the work item priority, but also executes the work item with the specified thread priority !</p>
<p>To demonstrate the feature, here&rsquo;s a simple code sample :</p>

```csharp
private void Profile()
{
   ThreadPool
      .RunAsync(_ => Run(s => textbox1.Text = s), WorkItemPriority.Low);
   ThreadPool
      .RunAsync(_ => Run(s => textbox2.Text = s), WorkItemPriority.Normal);
   ThreadPool
      .RunAsync(_ => Run(s => textbox3.Text = s), WorkItemPriority.High);
}

private void Run(Actiondisplay)
{
	for (int i = 0; i &lt; int.MaxValue; i++)
	{
		i.ToString(); // Slow things down a litthe bit...
				
		if((i % 100000) == 0)
		{
			Dispatcher.BeginInvoke(() => display(i.ToString()));
		}
	}
}
```
<p>The result is that after a few seconds, the higher the priority, the higher the number displayed on the screen.</p>
<p>Needless to say, this changes quite a bit of things, where it is definitely possible to having background threads that impact a lot less the dispatcher, even if they are 100% loaded. Good for the UI, good for the user.</p>
<p>That&rsquo;s it for now, happy threading !</p>
{% include imported_disclaimer.html %}
