---
layout: post
title: "C# Async Tips and Tricks, Part 3: Tasks and the Synchronization Context"
date: 2012-09-29 21:12:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/09/29/C-Async-Tips-and-Tricks-Part-3-Tasks-and-the-Synchronization-Context", "/post/2012/09/29/c-async-tips-and-tricks-part-3-tasks-and-the-synchronization-context"]
author: jay
---
<!-- more -->
<p><em>TL;DR: It is possible to mix C# async and basic TPL style programming, but when doing so, the synchronization context capture feature of C# async is not forwarded to TPL continuations automatically, making UI dependent (and others)&nbsp;code fail and raise exceptions. This can lead to the termination of the process when exceptions are not handled properly, particularly in WinRT/C# apps.</em></p>
<p>&nbsp;</p>
<p>I&rsquo;ve discussed in a previous article of this series,&nbsp;<a href="http://jaylee.org/post/2012/06/18/CSharp-5-0-Async-Tips-and-Tricks-Part-1.aspx">the relation between async Task/Void methods and the ambient SynchronizationContext</a>.</p>
<p>Just as a simple reminder, when executing a async method, whether it is Task, Task&lt;T&gt; or Void returning, the caller&rsquo;s <a href="http://msdn.microsoft.com/en-us/library/system.threading.synchronizationcontext.aspx">SynchronizationContext</a> is captured to ensure that all the code in an async method is executed in the same context. The main scenario for this is to easily execute UI bound code in an async method.</p>
<p>It is important to remember that async methods are based on the <a href="http://msdn.microsoft.com/en-us/library/dd460717.aspx">TPL framework</a>, and that async methods (except in <a href="http://jaylee.org/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void.aspx">infamous async void</a>) return <a href="http://msdn.microsoft.com/en-us/library/System.Threading.Tasks.Task.aspx">System.Threading.Tasks.Task</a> instances.</p>
<p>[more]</p>
<p>This means that we can write something like this :</p>
<pre class="brush: c-sharp">    private static async Task SomeMethod()
    {
        var t = Task.Delay(TimeSpan.FromSeconds(1))
                    .ContinueWith(
                        _ =&gt; Task.Delay(TimeSpan.FromSeconds(42))
                    );
        await t;
    }
</pre>
<p>This is a perfectly valid code, which could arguably be written easier like this:</p>
<pre class="brush: c-sharp">    private static async Task SomeMethod()
    {
        await Task.Delay(TimeSpan.FromSeconds(1));
        await Task.Delay(TimeSpan.FromSeconds(42));
    }
</pre>
<p>It is possible to mix both TPL and C# async based code.</p>
<p>&nbsp;</p>
<h2>The SynchronizationContext and the TPL</h2>
<p>But there&rsquo;s yet another catch.</p>
<p>If both code blocks were identical, we could write something like this:</p>
<pre class="brush: c-sharp">    private async Task&lt;string&gt; SomeUIMethod()
    {
        var t = Task.Delay(TimeSpan.FromSeconds(1))
                    .ContinueWith(
                       _ =&gt; this.Title = "Done !"
                    );

        return await t;
    }
</pre>
<p>When executed in a WPF like application, it fails with a CrossThreadAccessException or similar, where title cannot be set from the current thread.</p>
<p>There&rsquo;s a simple explanation for this, being that the TPL does not use the SynchronizationContext like a C# async does, but rather uses a <a href="http://msdn.microsoft.com/en-us/library/System.Threading.Tasks.TaskScheduler.aspx">TaskScheduler</a>.</p>
<p>With C# async methods, the underlying helper classes <a href="http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.asyncvoidmethodbuilder(v=VS.110).aspx">AsyncVoidMethodBuilder</a> and <a href="http://msdn.microsoft.com/en-us/library/hh138506.aspx">AsyncTaskMethodBuilder</a> both capture the Synchronization Context of the caller of an async method.</p>
<p>Simply put, when tasks are chained using the TPL methods like ContinueWith, <strong>the synchronization context of the caller is lost</strong>.</p>
<p>&nbsp;</p>
<h2>Capturing the synchronization context in TPL tasks</h2>
<p>A TPL task scheduler is very similar to the SynchronizationContext, and it is so similar that there&rsquo;s a helper method that allows adapting one to the other by using the TaskScheduler.FromCurrentSynchronizationContext() method.</p>
<p>It is possible to forward the SynchronizationContext to a chain of TPL tasks by specifying where the continuation's code should run, by using a special overload of ContinueWith that takes in a TaskScheduler:</p>
<pre class="brush: c-sharp">    private async Task SomeUIMethod()
    {
        var t = Task.Delay(TimeSpan.FromSeconds(1))
                    .ContinueWith(
                        _ =&gt; this.Title = "Done !",
                        // Specify where to execute the continuation
                        TaskScheduler.FromCurrentSynchronizationContext()
                    );

        return await t;
    } 
</pre>
<p>A bit tricky, isn&rsquo;t it?</p>
<p>&nbsp;</p>
<h2>Why would I bother writing non C# async based code ?</h2>
<p>You&rsquo;re right, you probably&nbsp;should not&nbsp;bother and stick with async.</p>
<p>The problem though,&nbsp;is not that you should not bother; the problem is that you can write some. It&rsquo;s very&nbsp;tempting to think that the two styles are equivalent, and that it will not matter which one is used.</p>
<p>Now, for the specific example of the thread-affinity of the UI, the issue is obvious. It crashes immediately.</p>
<p>But let&rsquo;s consider the example of a custom SynchronizationContext that needs to handle exceptions a certain way. As long as you stay in &ldquo;pure&rdquo; C# async methods, exceptions will be raised in the proper synchronization context. But if there&rsquo;s a native TPL method in the chain and an exception is raised, your process is terminated, because the exception is raised and unhandled in the ThreadPool.</p>
<p>&nbsp;</p>
<h2>The special case of <s>Metro</s> Windows Store apps</h2>
<p>The problem goes a bit further, when in the context of the Metro Style apps.</p>
<p>Since Threads do not exist anymore in&nbsp;WinRT apps, the TPL in the WinRT world is forced to go with WinRT&rsquo;s thread pool. In the case of the Task.Delay method, the promise (ContinueWith in this case) will be executed directly on the ThreadPool. If there&rsquo;s no special TaskFactory and an exception is raised, then the exception will go unhandled directly into WinRT&rsquo;s thread pool.</p>
<p>I have to admit though. This issue is a bit troubling, because it is very easy to crash the process, with any method that calls back from WinRT asynchronously. Any IAsyncOperation returning method, actually, which is a&nbsp;<strong>lot</strong> of them.</p>
<p>At the moment, I&rsquo;ve yet to find where to intercept those exceptions since neither Application.Current.UnhandledException nor TaskScheduler.UnobservedTaskException are able to intercept it. (AppDomain.CurrentDomain.UnhandledException is not available in WinRT)</p>
<p>&nbsp;</p>
<h2>What to take away</h2>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">If you decide to go the async way, try not to mix it up with the TPL, or at least not with Task.ContinueWith-like methods that may lose the current synchronization context unless you use the proper overload with a TaskScheduler. Worst case, you may end up terminating your process without realizing it.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">As a side note, I've personally written a Static Analysis rule that prevents the use of the overload of ContinueWith that does not take a TaskScheduler, to avoid falling into this trap.</p>
{% include imported_disclaimer.html %}
