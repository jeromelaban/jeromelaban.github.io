---
layout: post
title: "[WP7] HttpWebRequest and the Flickr app 'Black Screen' issue"
date: 2011-04-22 14:54:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/04/22/WP7-HttpWebRequest-and-the-Flickr-app-Black-Screen-issue", "/post/2011/04/22/wp7-httpwebrequest-and-the-flickr-app-black-screen-issue"]
author: jay
---
<!-- more -->
<p><em>TL;DR: While trying to fix the "Black Screen" issue of the Windows Phone 7 flickr app 1.3, I found out that HttpWebRequest is internally making a synchronous call to the UI Thread, making a network call negatively impact the UI. The entire building of an asynchronous web query is performed on the UI thread, and you can't do anything about it.</em></p>
<p><em>Edit: This post was formerly named "About the UI Thread performance and HttpWebRequest", but was in fact about Yahoo's Flickr application and was enhanced accordingly.</em></p>
<p>When programming on Windows Phone 7, you'll hear often that to improve the perceived performance, you'll need to get off of the UI Thread (i.e. the <a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatcher.aspx">dispatcher</a>) to perform non UI related operations. By good perceived performance, I mean having the UI respond immediately, not stall when some background processing is done.</p>
<p>To acheive this,&nbsp;you'll need to use the common asynchrony techniques like queueing in the <a href="http://msdn.microsoft.com/en-us/library/kbf0f1ct.aspx">ThreadPool</a>, create a new thread, or use the <a href="http://msdn.microsoft.com/en-us/library/ms228963.aspx">Begin/End pattern</a>.</p>
<p>All of this is very true, and one very good example&nbsp;of bad UI Thread use&nbsp;is the processing of the body of a web request, particularly when <a href="http://jaylee.org/post/2010/06/22/WP7Dev-Using-the-WebClient-with-Reactive-Extensions-for-Effective-Asynchronous-Downloads.aspx">using the WebClient where the raised events</a> are in the context of the dispatcher. From a beginner's perspective, not having to care about changing contexts when developing a simple app that updates the UI, provides a particularly good and simple experience.</p>
<p>But that has the&nbsp;annoying&nbsp;effect of degrading the perceived performance of the application, because many parts of the&nbsp;application&nbsp;tend to run on the UI thread.</p>
<p>&nbsp;</p>
<h2>HttpWebRequest to the rescue ?</h2>
<p>You'll find that the <a href="http://msdn.microsoft.com/en-us/library/system.net.httpwebrequest.aspx">HttpWebRequest</a> is a better choice in that regard. It uses the Begin/End pattern and&nbsp;the execution of the AsyncCallback is performed in the context of ThreadPool. This performs the execution of the code in that callback in a way that does not impact the perceived performance of the application.</p>
<p>Using the <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896">Reactive Extensions</a>, this can be written like this :</p>
<p>[code:c#]</p>
<p>var request = WebRequest.Create("http://www.google.com");</p>
<p>var&nbsp;queryBuilder = Observable.FromAsyncPattern(<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (h, o) =&gt; request.BeginGetResponse(h, o),<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ar =&gt; request.EndGetResponse(ar));</p>
<p>queryBuilder()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; /* Perform the expensive work in the context of the AsyncCall back */<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; /* from the WebRequest. This will be the ThreadPool. */<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .Select(response =&gt; DoSomeExpensiveWork(response))</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; /* Go back to the UI Thread to execute the OnNext method on the subscriber */<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .ObserveOnDispatcher()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .Subscribe(result =&gt; DisplayResult(result));<br />[/code]</p>
<p>That way, you'll get most of your code to execute out of the UI thread, where that does not impact the perceived performance of the application.</p>
<p>&nbsp;</p>
<h2>Why would it not be to the rescue then ?</h2>
<p>Actually, it will always be (as of Windows Phone NoDo), but there's a catch. And that's a big deal, from a performance perspective.</p>
<p>Consider this code&nbsp;:</p>
<p>[code:c#]</p>
<p>&nbsp;public App()<br />&nbsp;{<br />&nbsp;&nbsp;/* some application initialization code */</p>
<p><br />&nbsp;&nbsp;ManualResetEvent ev = new ManualResetEvent(false);</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp; ThreadPool.QueueUserWorkItem(<br />&nbsp;&nbsp;d =&gt;<br />&nbsp;&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var r = WebRequest.Create("http://www.google.com");<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; r.BeginGetResponse((r2) =&gt; { }, null);</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ev.Set();<br />&nbsp;&nbsp;}<br />&nbsp;&nbsp;&nbsp;&nbsp; );</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp; ev.WaitOne();<br />&nbsp;}</p>
<p>[/code]</p>
<p>This code is basically beginning a request on the thread pool, while blocking the UI thread in the App.xaml.cs file. This makes the construction (but not the actual call on the network)&nbsp;of the WebRequest synchronous, and makes the application wait for the request to begin before showing any page to the user.</p>
<p>While this code is definitely not a best practice,&nbsp;there was a code path in the Flickr 1.3 application&nbsp;that was doing something remotely similar, in a more convoluted way. And if you try it for yourself, <strong>you'll find that the application hangs in a deadlock&nbsp;during the startup of the application</strong>, meaning that our event is never set.</p>
<p>&nbsp;</p>
<h2>What's happening ?</h2>
<p>If you dig a bit, you'll find that the stack trace for&nbsp;a&nbsp;thread in the thread pool is the following :</p>
<p><span style="font-family: courier new,courier;">&nbsp;&nbsp;mscorlib.dll!System.PInvoke.PAL.Threading_Event_Wait()&nbsp;</span><br /><span style="font-family: courier new,courier;">&nbsp;&nbsp;mscorlib.dll!System.Threading.EventWaitHandle.WaitOne()&nbsp;</span><br /><span style="font-family: courier new,courier;">&nbsp;&nbsp;System.Windows.dll!System.Windows.Threading.Dispatcher.FastInvoke(...)&nbsp;</span><br /><span style="font-family: courier new,courier;">&nbsp;&nbsp;System.Windows.dll!System.Net.Browser.AsyncHelper.BeginOnUI(...)</span><br /><span style="font-family: courier new,courier;">&nbsp;&nbsp;System.Windows.dll!System.Net.Browser.ClientHttpWebRequest.BeginGetResponse(...)&nbsp;</span><br /><span style="font-family: courier new,courier;">&nbsp; WindowsPhoneApplication2.dll!WindowsPhoneApplication2.App..ctor.AnonymousMethod__0(...)</span></p>
<p><strong>The BeginGetResponse method is trying to execute something on the UI thread. </strong>And in our example, since the UI thread is blocked by the manual reset event, the application hangs in a deadlock between a resource in the dispatcher and&nbsp;our manual reset event.</p>
<p>This is also&nbsp;the case for the EndGetResponse method.</p>
<p>But&nbsp;if you dig even deeper, you'll find in the version of the System.Windows.dll assembly in the WP7 emulator&nbsp;(the one in the SDK is a stub for all public types), that the BeginGetResponse method is doing all the work of&nbsp;actually building the web query on the UI thread !</p>
<p>That is&nbsp;particularly disturbing. I'm still wondering why that&nbsp;network-only&nbsp;code would need to be&nbsp;executed to UI Thread.</p>
<p>&nbsp;</p>
<h2>What's the impact then ?</h2>
<p>The impact is fairly simple : The more web requests you make, the less your UI will be responsive, both for processing the beginning and the end of a web request. Each call to the methods BeginGetResponse and EndGetResponse implicitly goes to the UI thread.</p>
<p>In the case of Remote Control applications like mine that are trying&nbsp;to have&nbsp;remote mouse control, all are affected by the same lagging behavior of the mouse. That's partially because the UI thread&nbsp;is particularly busy&nbsp;processing <a href="http://msdn.microsoft.com/en-us/library/system.windows.uielement.onmanipulationdelta.aspx">Manipulation events</a>, this explains a lot about the performance issues of the web requests performed at the same time, even by using HttpWebRequest instead of WebClient. This also explains why until the user stops touching the screen, the web requests will be strongly slowed down.</p>
<p>&nbsp;</p>
<h2>The Flickr 1.3 "Black Screen" issue</h2>
<p>In the Flickr application for which I've been able to work on, a lot of people were reporting a "black screen" issue, where the application stopped working after a few days.</p>
<p>The application was actually&nbsp;trying to update a resource from the application startup in an asynchronous fashion using the HttpWebRequest. Because of a race condition with an other lock in the application and UI Thread that was&nbsp;waiting in the app's initialization, this resulted in an infinite "Black Screen" that could only be bypassed by reinstalling the application.</p>
<p>Interestingly enough, at this point in the application's initialization, in the App's class constructor, the application is not killed after 10 seconds if it is not showing a page to the user. However, if the application stalls in the constructor of the first page, the application is automatically killed by the OS after something like 10 seconds.</p>
<p>Regarding the use of the UI Thread inside the HttpWebRequest code, applications that are network intensive to get a lot of small web resources like images, this is has a negative&nbsp;impact on the performance. The UI thread is constantly interrupted to process network resources query and responses.</p>
<p>&nbsp;</p>
<h2>Can I do something about it ?</h2>
<p>During the analysis of the emulator version of the System.Windows.dll assembly, I noticed that the BeginGetResponse is checking whether the current context is the UI Thread, and does not push the execution on the dispacther.</p>
<p>This means that if you can group the calls to BeginGetResponse calls in the UI thread, you'll spend less time switching between contexts. That's not the panacea, but at the very least you can gain on this side.</p>
<p>&nbsp;</p>
<h2>What about future versions of&nbsp;Windows Phone&nbsp;?</h2>
<p>On the good news side, Scott Gu annouced at the Mix 11 that the manipulation events will be moved out the the UI thread, making the UI "buttery smooth" to take his words. This will a lot of applications benefit from this change.</p>
<p>Anyway, let's wait for Mango, I'm guessing that will this will change is a very positive way, and allow us to have high performance apps on the Windows Phone platform.</p>
{% include imported_disclaimer.html %}
