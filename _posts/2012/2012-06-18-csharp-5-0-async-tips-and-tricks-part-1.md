---
layout: post
title: "C# 5.0 Async Tips and Tricks, Part 1"
date: 2012-06-18 21:22:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/06/18/CSharp-5-0-Async-Tips-and-Tricks-Part-1.aspx", "/post/2012/06/18/csharp-5-0-async-tips-and-tricks-part-1.aspx"]
author: jay
---
<!-- more -->
<p><em><span lang="EN"><a href="http://blogs.developpeur.org/jay/archive/2012/06/23/csharp-5-0-async-trucs-et-astuces-partie-1.aspx">Cet article est aussi disponible en francais.</a></span></em></p>
<p><em><span lang="EN">TL;DR: This article is a discussion about how C#5.0 async captures the executing context when running an async method, which allows to easily stay on the UI Thread to access UI elements, but can be problematic when running CPU-bound work. Off of the UI thread, an async method may jump from thread to thread, breaking thread context dependent code along with it.</span> </em></p>
<p><em>You can also read : <a href="http://www.jaylee.org/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void.aspx">Part 2, Async void</a>&nbsp;</em></p>
<p><span lang="EN">C# 5.0 has introduced, language-wise,<em> just</em> two of new features (<a href="http://msdn.microsoft.com/en-us/library/hh156513(v=vs.110).aspx">async</a> and <a href="http://msdn.microsoft.com/en-us/library/hh534540(v=vs.110).aspx">caller information</a>) and a gotcha fix (<a href="http://stackoverflow.com/a/8899347">a lambda&nbsp;lifted foreach loop variable scope change</a>).</span></p>
<p><span lang="EN">The biggest addition though, in terms of compiler magic, is the addition async. <a href="http://msdn.microsoft.com/en-us/library/hh191443(v=vs.110).aspx">It&rsquo;s</a> <a href="http://blogs.msdn.com/b/csharpfaq/archive/2010/10/28/async.aspx">been</a> <a href="http://blogs.msdn.com/b/ericlippert/archive/2010/10/28/asynchrony-in-c-5-part-one.aspx">covered</a> <a href="http://blogs.msdn.com/b/ericlippert/archive/2010/10/29/asynchronous-programming-in-c-5-0-part-two-whence-await.aspx">all</a> <a href="http://msmvps.com/blogs/jon_skeet/archive/2010/10/29/initial-thoughts-on-c-5-s-async-support.aspx">over</a> <a href="http://msmvps.com/blogs/jon_skeet/archive/tags/Eduasync/default.aspx">the</a>&nbsp;<a href="http://blogs.msdn.com/b/pfxteam/archive/2012/02/29/10274035.aspx">place</a>, and I will just comment the addition feature by itself, and the impacts it has when using it. </span></p>
<p><span lang="EN">This article is the first of a small series about some C# async tricks and gotchas that I will try to cover to give a bit of insight, and hopefully help developers avoid them.</span></p>
<p>[more]</p>
<h2><span lang="EN">C# 5.0 Async : The Objectives</span></h2>
<p>Async programming has been a key concern over the past few years, originally to be able to have more fluid user interfaces. GUI on Windows are all using the concept of a UI Thread, that takes the form of a message pump that should always be processing messages. If that pump is stalled, because of some button click that is executing some blocking code, the UI freezes.</p>
<p>This is a very bad user experience, and Microsoft as well as others, have been either suggesting (or forcing, like in Silverlight or WinRT) developers to use asynchronous API calls.</p>
<p><span lang="EN">When going asynchronous, using existing techniques like event completion messages or the Begin/End pattern, the biggest challenge is the keeping of state between each of these asynchronous calls. It requires the developer to make its own state machine to handle the passing of state between each stage of the asynchronous logical &ldquo;operation&rdquo;. This is quite a complex plumbing code, and having to write this every time is both time consuming and error prone.</span></p>
<p><span lang="EN">From the developer&rsquo;s point of view, C#&rsquo;s async is to trying to make asynchronous code look synchronous. </span><span lang="EN">Asynchrony is a big problem in C#, and the C# team has introduced the feature to try to ease the life of developers when they have to deal with it.</span></p>
<p><span lang="EN">Another objective of C#&rsquo;s async is to be able to perform thread yielding, by explicitly releasing the control flow, therefore allowing a thread to be reused more effectively, by not actively waiting on some external processing. In some sense, it resembles to the concept of Win32 Fibers or more generally to concept of cooperative multitasking.</span></p>
<p><span lang="EN">From my understanding, the ultimate goal is that over time, developers will favor writing asynchronous methods that will free up the UI thread when there is an asynchronous call involved.&nbsp;</span><span lang="EN">&nbsp;</span></p>
<h2><span lang="EN">C# 5.0 Async, Magic Level 1</span></h2>
<p><span lang="EN">When you start using it, it feels like it is doing a bit of &ldquo;magic&rdquo;, at first. Just put some async and await here and there, and it just works.</span></p>
<p><span lang="EN">The first level of Magic happens in the state machine that is generated by the compiler the same way a developer would, and for the developer, this is <em>almost</em> transparent. An async method is then a series of chained delegates that are executed after each use of the await keyword. </span></p>
<p><span lang="EN">Just taking a look at an <a href="http://wiki.sharpdevelop.net/ILSpy.ashx">ILSpy</a>&nbsp;output shows all the guts of an async method, where a simple method is chunked into little pieces, driven by the presence of the await keyword. </span></p>
<p><span lang="EN">You&rsquo;ll find that all of your async method has been placed in a generated &ldquo;Display Class&rdquo;, and more specifically in a method called MoveNext.</span></p>
<p><span lang="EN">While this code generation is more anecdotic than useful, it helps to understand the overall behavior of an async method.</span></p>
<p class="MsoNormal" style="margin: 0in 0in 10pt;">&nbsp;</p>
<h2><span lang="EN">The SyncronizationContext, Magic Level 2</span></h2>
<p><span lang="EN">If you&rsquo;ve written some asynchronous code (without async), you must know about methods like Dispatcher.BeginInvoke or Form.BeginInvoke, that queue messages on the UI Thread. So, unless the framework does that for you (like with WebClient in Silverlight) you have to use these methods to update your UI when you asynchronous work has completed.</span></p>
<p><span lang="EN">You must then wonder&hellip; How does an async method allow the update of the UI? That&rsquo;s the Magic Level 2, known as the <a href="http://msdn.microsoft.com/en-us/library/system.threading.synchronizationcontext.aspx">SynchronizationContext</a>. </span></p>
<p><span lang="EN">The synchronization context is basically an abstraction of a message queue, where you can <a href="http://msdn.microsoft.com/en-us/library/system.threading.synchronizationcontext.send.aspx">synchronously</a> or <a href="http://msdn.microsoft.com/en-us/library/system.threading.synchronizationcontext.post.aspx">asynchronously</a> queue up work, and get notified when it&rsquo;s done. The Silverlight and WPF <a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatcher.aspx">Dispatcher</a> have one that is set automatically when executing UI related code. This simplification allows for an easy access to UI elements from an async method.</span></p>
<p><span lang="EN">When creating an async method, the compiler relies on the <a href="http://msdn.microsoft.com/en-us/library/hh138506(v=VS.110).aspx">AsyncTaskMethodBuilder</a>, a BCL class that is used a coordinator to handle the magic behind the fact that when an async method is created, it captures the current synchronization context of a callee. It reuses that SynchronizationContext to queue up the execution of the code in the async method after the first await, so that it stays in the same context. </span></p>
<p><span lang="EN">While this is internally a bit tricky, this is very effective when working on the UI Thread in an async method. There is <em>almost</em> no need to worry about going back on the UI Thread to update it, because the <a href="http://msdn.microsoft.com/en-us/library/system.threading.synchronizationcontext.aspx">SynchronizationContext</a>&nbsp;brought the execution flow back to it. </span></p>
<p>&nbsp;</p>
<h2><span lang="EN">Asynchrony vs. Concurrency</span></h2>
<p><span lang="EN">Now that you&rsquo;re executing your code in your async method, you may be tempted to think that no matter what you&rsquo;ll do in your code, you won&rsquo;t freeze the UI because you&rsquo;re in an async method.</span></p>
<p><span lang="EN">You&rsquo;re going to be tempted to make a web call, then deserialize </span><span lang="EN">its content</span> <span lang="EN">in </span><span lang="EN">your async method. You&rsquo;ll soon find out&nbsp;that it freezes up your UI when the JSON/XML is deserialized.</span></p>
<p><span lang="EN">That is </span><span lang="EN">one</span> <span lang="EN">thing to remember about asynchrony. It&rsquo;s not concurrency.</span></p>
<p><span lang="EN">The tricky part where the SynchronizationContext brings you back on the UI Thread, brings you back exactly on it. Deserializing JSON is a CPU bound operation, which will block the thread it is working on, bringing you back to square one: your UI stutters.</span></p>
<p><span lang="EN">At this point, you want concurrency, which is where async will not help you much. You need to execute your work on another thread, either via a Thread or a Task, or a Thread Pool item, &hellip; You&rsquo;ll need to handle resources locking properly, graceful termination, all those nitty gritty details.</span></p>
<p><span lang="EN">As soon as you leave the context of your UI bound async method, you need to handle communication with the UI by yourself, using <a href="http://msdn.microsoft.com/en-us/library/cc190824.aspx">Dispatcher.BeginInvoke</a> and alike. </span></p>
<p><span lang="EN">I personally think that it will&nbsp;be (and already is)&nbsp;one of the biggest misunderstandings of the use of async, where developers are going to think they&rsquo;re doing parallel work using async, for which it is not the best fit.</span></p>
<p>&nbsp;</p>
<h2><span lang="EN">Async Methods Life Cycle</span></h2>
<p class="MsoNormal" style="margin: 0in 0in 10pt;"><span lang="EN">Let&rsquo;s consider this code :</span></p>
<pre class="brush: c-sharp">private async void Button_Click_1(object sender, RoutedEventArgs e)
{
  await Test();
  
  var t = Task.Run(async () =&gt; await Test());
  
  await t;
}

private async Task Test()
{
  Debug.WriteLine("SyncContext : {0}", SynchronizationContext.Current);
  
  Debug.WriteLine("1. {0}", Thread.CurrentThread.ManagedThreadId);
  
  await Task.Delay(1000);
  
  Debug.WriteLine("2. {0}", Thread.CurrentThread.ManagedThreadId);
  
  await Task.Delay(1000);
  
  Debug.WriteLine("3. {0}", Thread.CurrentThread.ManagedThreadId);
  
  await Task.Delay(1000);
  
  Debug.WriteLine("4. {0}", Thread.CurrentThread.ManagedThreadId);
}
</pre>
<p>Which, when executed produces this output :</p>
<p class="MsoNormal" style="margin: 0in 0in 0pt; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9.5pt; font-family: Consolas; color: black;">&nbsp;</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">SyncContext : System.Windows.Threading.DispatcherSynchronizationContext</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">1. 10</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">2. 10</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">3. 10</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">4. 10</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">SyncContext : </span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">1. 6</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">2. 12</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">3. 12</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; background: white; color: #525354; mso-highlight: white;">4. 6</span></p>
<p class="MsoNormal" style="margin: 0in 0in 0pt 0.5in; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: 9pt; font-family: Consolas; color: #525354;"><span style="mso-tab-count: 1;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span></span></p>
<p>You&rsquo;ll notice here is that the first execution of the method stays on the same Thread, the Dispatcher thread, using the <a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatchersynchronizationcontext.aspx">DispatcherSynchronizationContext</a>. This is why it is possible to execute UI bound code in this method.</p>
<p>In the case of WPF/Silverlight/Metro, the <a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatchersynchronizationcontext.aspx">DispatcherSynchronizationContext </a>is automatically set on a <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms686749(v=vs.85).aspx">TLS</a>&nbsp;(the T<a href="http://msdn.microsoft.com/en-us/library/system.threadstaticattribute.aspx">hread Static attribute</a>) variable, where every single UI element callback will have access to it, and by extension, every async method executed in the same context.</p>
<p>But then, when the same method is executed inside of a manually started Task, the behavior is completely different. The code starts on the Thread 6, continues on the Thread 12, then back to 6.</p>
<p>The reason for this is the absence of SynchronizationContext when the async method was started. The default behavior of an async method, which is backed by <span lang="EN"><a href="http://msdn.microsoft.com/en-us/library/hh138506(v=VS.110).aspx">AsyncTaskMethodBuilder</a>, is to queue up work on the ThreadPool. By nature, it is not possible to choose which thread a queued work item is going to be executed on, hence the thread switching. </span></p>
<p>Do not assume that UI work can always be done in an async method, and that is it hazardous for code to rely on it.</p>
<p class="MsoNormal" style="margin: 0in 0in 10pt;">&nbsp;</p>
<h2>&nbsp;</h2>
<h2>Migrating Threading Context Dependent Code</h2>
<p>Let&rsquo;s consider this legacy code, which runs fine on WinForms 2.0 :</p>
<pre class="brush: c-sharp">  private void Button_Click_1(object sender, RoutedEventArgs e)
  {
     _lock.AcquireWriterLock(1000);

     // A context sensitive operation
     Thread.Sleep(3000);

     _lock.ReleaseWriterLock();
  }
 </pre>
<p>This is not the kind of code you&rsquo;d want to see behind a UI event handler, but the use of a <a href="http://msdn.microsoft.com/en-us/library/system.threading.readerwriterlock.aspx">ReaderWriterLock</a> could definitely be found in some background service, so bear with me.</p>
<p>Let&rsquo;s say that you need to have you code migrated to WinRT, and that instead of the <a href="http://msdn.microsoft.com/en-us/library/d00bd51t.aspx">Thread.Sleep</a>, you need to call an async method to <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.storage.storagefile.openreadasync.aspx">do some file access</a>:</p>
<p class="MsoNormal" style="margin: 0in 0in 0pt; line-height: normal; mso-layout-grid-align: none;"><span style="font-size: small; font-family: Calibri;">&nbsp;</span></p>
<pre class="brush: c-sharp">  private async void Button_Click_2(object sender, RoutedEventArgs e)
  {
      _lock.AcquireWriterLock(1000);
    
      // Do some context sensitive stuff...
      await Task.Delay(1000);

      _lock.ReleaseWriterLock();
  }
</pre>
<p>Everything will run properly.</p>
<p>But you'll see that your UI freezes, so you want to put your code in a background Task:</p>
<pre class="brush: c-sharp">  private void Button_Click_3(object sender, RoutedEventArgs e)
  {
      Task.Run(
        async () =&gt;
        {
          try
          {
            _lock.AcquireWriterLock(1000);

            // Do some context sensitive stuff...
            await Task.Delay(1000);

            _lock.ReleaseWriterLock();
          }
          catch (Exception ex)
          {
            Debug.WriteLine(ex);
          }
        }
      );
  }

</pre>
<p>(Note: We'll come back on async lambdas, so move along :) )</p>
<p>That code will almost certainly break, probably nine times out of ten.</p>
<p>The reason for this is that the async code is run in a background thread, and that the code after the await will almost certainly not run on the same Thread as the code before the await.</p>
<p>But you may be lucky and run on the same thread. Your code may work. Sometimes.</p>
<p>That&rsquo;s the reason why the lock keyword cannot be used in an async method, because the compiler will not ensure that the method will always be executed on the same thread, as it depends on the caller of the method.</p>
<p>&nbsp;</p>
<h2>What to take away</h2>
<ul>
<li>Take a look at the generated code behind your async method, it&rsquo;s very instructive.</li>
<li>An async method is not guaranteed to run on the UI thread, it is using the ambient SynchronizationContext.</li>
<li>Beware of thread dependent code executed in your async method, as the current thread may change throughout your method, depending on the ambient SynchronizationContext.</li>
</ul>
<p>&nbsp;</p>
<p>Stay tuned for the next part of this small series !</p>
{% include imported_disclaimer.html %}
