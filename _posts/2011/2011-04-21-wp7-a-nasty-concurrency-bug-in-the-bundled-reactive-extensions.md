---
layout: post
title: "[WP7] A nasty concurrency bug in the bundled Reactive Extensions"
date: 2011-04-21 20:05:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/04/21/WP7-A-nasty-concurrency-bug-in-the-bundled-Reactive-Extensions.aspx", "/post/2011/04/21/wp7-a-nasty-concurrency-bug-in-the-bundled-reactive-extensions.aspx"]
author: admin
---
<!-- more -->
<p>The <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896">Reactive Extensions</a> have been included in Windows Phone 7, which comes out of the box, and that you can include the Microsoft.Phone.Reactive.dll assembly.</p>
<p>&nbsp;</p>
<p>This is a good thing, because it participates in the democratization of the Rx framework, and aside from the fact that the namespace is not exactly the same as the desktop version, Microsoft.Phone.Reactive instead of System.Concurrency and System.Linq, there were no major bugs until recently.</p>
<p>A few applications I've worked on that use the Rx framework, a very interesting unhandled exception was popping-up from time to time in my exception tracker. Unlike the <a href="http://jaylee.org/post/2011/03/27/WP7-Double-tapping-when-you-expected-only-one.aspx">double tap issue</a> I found the same way on my <a href="http://jaylee.org/rc/wp7">Remote Control application</a>, that one was a bit more tricky, see for yourself :</p>
<p><span style="font-family: courier new,courier;">Exception : System.NullReferenceException: NullReferenceException </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Reactive.CurrentThreadScheduler.Trampoline.Run()</span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Reactive.CurrentThreadScheduler.EnsureTrampoline(Action action)</span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Reactive.AnonymousObservable`1.Subscribe(IObserver`1 observer)</span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Reactive.ObservableExtensions.Subscribe[TSource](IObservable`1 source, Action`1 onNext, Action`1 onError, Action onCompleted) </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Reactive.ObservableExtensions.Subscribe[TSource](IObservable`1 source, Action`1 onNext)</span></p>
<p>This exception popped-up out of nowhere, without anything from the caller that would make that "Trampoline" method fail. No actual parameter passed to the Subscribe method was null, I made sure of that by using a precondition for calling Subscribe.</p>
<p>&nbsp;</p>
<p>Well, it turns out that <a href="http://social.msdn.microsoft.com/Forums/en-US/rx/thread/e70022a6-9b71-4d44-8c0f-c2c450eab01d">it's actually a bug</a> that is related to the use of the infamous not-supported-but-silent <a href="http://msdn.microsoft.com/en-us/library/system.threadstaticattribute(v=VS.100).aspx">ThreadStaticAttribute</a>, for which <a href="http://jaylee.org/post/2010/06/19/WP7Dev-Beware-of-the-ThreadStatic-attribute-on-Silverlight-for-Windows-Phone-7.aspx">I've had to work around</a> to make <a href="http://umbrella.codeplex.com/">umbrella</a> work properly on Windows Phone 7.</p>
<p>The lack of a Thread Local Storage creates a concurrency issue around a priority queue that is kept by the CurrentThreadScheduler to perform delayed operations. The system wide queue was accessed by multiple threads at the same time, creating random NullReferenceExceptions.</p>
<p>This means that any call to the Subscribe method may have failed if an other call to that same method was being made at the same time.</p>
<p>&nbsp;</p>
<p><strong>In short, do not use the bundled version of Rx in Windows Phone 7 (as of NoDo), but prefer using <a href="http://social.msdn.microsoft.com/Forums/en-HK/rx/thread/1b554ca0-7e23-4603-8b00-7753acf08c83">the latest release</a>&nbsp;from the <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896">DevLabs</a>, which does not have this nasty bug.</strong></p>
{% include imported_disclaimer.html %}
