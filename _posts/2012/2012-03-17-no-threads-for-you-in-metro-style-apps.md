---
layout: post
title: "No Threads for you ! (in metro style apps)"
date: 2012-03-17 13:06:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/03/17/No-Threads-for-you-in-metro-style-apps", "/post/2012/03/17/no-threads-for-you-in-metro-style-apps"]
author: jay
---
<!-- more -->
<p><a href="http://blogs.developpeur.org/jay/archive/2012/03/25/pas-de-threads-pour-vous-dans-les-applications-style-metro.aspx"><em>Cet article est disponible en francais.</em></a></p>
<p>As would <a href="http://en.wikipedia.org/wiki/The_Soup_Nazi" target="_blank">say this guy</a>, since you&rsquo;ve most probably been using threads the wrong way (as Microsoft seems to think), you won&rsquo;t be able to use the <a href="http://msdn.microsoft.com/en-us/library/system.threading.thread.aspx" target="_blank">Thread class</a> anymore in Metro Style applications. The class is simply not available anymore, and neither are <a href="http://msdn.microsoft.com/en-us/library/system.threading.timer.aspx" target="_blank">Timer</a> or <a href="http://msdn.microsoft.com/en-us/library/system.threading.threadpool.aspx" target="_blank">ThreadPool</a>.</p>
<p>That may come a shock to you, but this actually makes a lot of sense. But don&rsquo;t worry, the concept of parallel execution is still there, but it takes the form of Tasks.</p>
<p>&nbsp;</p>
<h2>Why using Threads is not good for you</h2>
<p>Threads are very powerful but there are a lot of terrible gotchas that come with it :</p>
<ul>
<li>Unhandled exceptions in threads handlers, either raised from a Timer, a Thread or ThreadPool thread, <a href="http://msdn.microsoft.com/en-us/library/ms228965.aspx">lead to the termination of the process</a></li>
<li>Using Abort is <a href="https://msmvps.com/blogs/peterritchie/archive/2007/08/22/thead-abort-is-a-sign-of-a-poorly-designed-program.aspx" target="_blank">quite bad for the process</a>, and should be avoided</li>
<li>People tend to use <a href="http://msdn.microsoft.com/en-us/library/d00bd51t.aspx" target="_blank">Thread.Sleep</a> to arbitrarily wait for some constant time that will most probably be incorrect, and that will waste CPU resources to manage a thread that does not do anything while it waits,</li>
<li>People tend to come up with complex designs to chain operations on threads, which most of the time fail miserably.</li>
</ul>
<p>There are some more, but these a main scenarios where using Threads fall short.</p>
<p>[more]</p>
<p>I&rsquo;ve been advocating to stay away from Threads, at least not directly, for all these reasons (and more, but that&rsquo;s out of scope here).</p>
<p>&nbsp;</p>
<h2>Using Task, exclusively</h2>
<p>Since Microsoft went back to rethink some patterns that were introduced in the original BCL and CLR, they probably thought it was time to time to remove the Thread class in favor of the <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.aspx" target="_blank">Task class</a>, which does a far better job, and handles all the cases I listed above :</p>
<ul>
<li>Tasks now <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.exception.aspx" target="_blank">handle exceptions properly</a>, forwarding the exception to the code that receives the result of that task,</li>
<li>It&rsquo;s now possible to use the <a href="http://msdn.microsoft.com/en-us/library/system.threading.cancellationtokensource(v=vs.110).aspx" target="_blank">CancellationTokenSource</a> class to handle cancellation in a very nice way,</li>
<li>Even though <a href="http://msmvps.com/blogs/peterritchie/archive/2007/04/26/thread-sleep-is-a-sign-of-a-poorly-designed-program.aspx" target="_blank">it&rsquo;s a sign of poorly designed program</a>, <a href="http://msdn.microsoft.com/en-us/library/hh194845(v=vs.110).aspx" target="_blank">Task.Delay</a> is the new replacement for Thread.Sleep.</li>
<li>There are methods like <a href="http://msdn.microsoft.com/en-us/library/dd270672(v=vs.110).aspx" target="_blank">Task.WaitAny</a>,<a href="http://msdn.microsoft.com/en-us/library/hh160374(v=vs.110).aspx" target="_blank">Task.WhenAll</a> or <a href="http://msdn.microsoft.com/en-us/library/dd270696(v=vs.110).aspx" target="_blank">Task.ContinueWith</a> that allow for very safe task chaining operations.</li>
</ul>
<p>All these operations blend very nicely into the new async feature, for which it is <a href="http://msdn.microsoft.com/en-us/library/hh191443(VS.110).aspx" target="_blank">very easy to wait on a Task</a>.</p>
<p>&nbsp;</p>
<h2>ThreadPool and Timer moved to WinRT</h2>
<p>The Thread Pool is still there actually, and so is the timer in the form of the class <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.system.threading.threadpooltimer.aspx">TheadPoolTimer</a>, but they&rsquo;ve both moved to the WinRT side.</p>
<p><a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.system.threading.threadpool.aspx">ThreadPool</a> is async awaitable, and supports priority, much like Task. As of now, I do not see a very good compelling reason to use it, since Task has a far greater feature set.</p>
<p>ThreadPoolTimer can still be interesting, though exceptions thrown in this context seem to be handled silently by WinRT, and I&rsquo;ve yet to find where that goes. But I&rsquo;d recommend not using it, in favor of the <a href="http://msdn.microsoft.com/en-us/data/gg577609" target="_blank">Reactive Extensions'</a>&nbsp;<a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.timer(v=vs.103).aspx" target="_blank">Observable.Timer</a> which far more useful than this simple timer.</p>
<p>&nbsp;</p>
<h2>Thread left-overs</h2>
<p>There&rsquo;s actually one place where the &ldquo;Threads&rdquo; still surface in the BCL.</p>
<p>If we take this C# code :</p>
<pre class="brush: c-sharp">public IEnumerable IteratorSample() 
{ 
    yield return 1; 
}</pre>
<p>One feature of the compiler generated iterators is that they are explicitly not thread safe, and must be used on the thread they were generated on. The iterator is capturing the original thread, and is checking that subsequent calls stay on that thread.</p>
<p>So if we look at the what is internally expanded to an internal iterator class, using the C# compiler on .NET 4.0 and earlier :</p>
<pre class="brush: c-sharp">[DebuggerHidden] 
public d__0(int &lt;&gt;1__state) 
{ 
   this.&lt;&gt;1__state = &lt;&gt;1__state; 
   this.&lt;&gt;l__initialThreadId = Thread.CurrentThread.ManagedThreadId; 
}</pre>
<p>Whereas, using the .NET 4.5 C# compiler, this will be generated :</p>
<pre class="brush: c-sharp">[DebuggerHidden] 
public d__0(int &lt;&gt;1__state) 
{ 
   this.&lt;&gt;1__state = &lt;&gt;1__state; 
   this.&lt;&gt;l__initialThreadId = Environment.CurrentManagedThreadId; 
}</pre>
<p>The .NET 4.5 generated code is making use of the new <a href="http://msdn.microsoft.com/en-us/library/system.environment.currentmanagedthreadid(v=vs.110).aspx" target="_blank">Environment.CurrentManagedThreadId</a> property, because that specific iterator feature needs to have access to the actual thread ID, even though the Thread class does not exist anymore.</p>
<p>Interesting, isn&rsquo;t it ?</p>
<p>This has a very unfortunate effect, though. C# compiled by a compiler below .NET 4.5 is not binary compatible with the Metro Style apps BCL, and will not run without being recompiled. But that&rsquo;s not a big deal, because most the .NET surface APIs have either changed (to be async only), moved (<a href="http://msdn.microsoft.com/en-us/library/system.reflection.introspectionextensions.gettypeinfo(v=vs.110).aspx" target="_blank">like the Reflection API</a>) or simply removed (like the System.IO namespace that went into WinRT), so you would have to <a href="http://msdn.microsoft.com/en-us/library/windows/apps/br230302(v=vs.110).aspx#convert" target="_blank">adapt your code anyway</a>.</p>
<p>I&rsquo;m guessing that the C# team had to fight for this feature to maintain compatibility and have the same behavior as in previous versions of C#. And I&rsquo;m glad this property stayed, because I&rsquo;ve been using it to log the ThreadID in my logging framework.</p>
<p>&nbsp;</p>
<p>Happy WinRT'ing !</p>
{% include imported_disclaimer.html %}
