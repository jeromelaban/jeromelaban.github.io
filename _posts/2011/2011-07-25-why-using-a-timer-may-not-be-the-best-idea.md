---
layout: post
title: "Why using a timer may not be the best idea"
date: 2011-07-25 00:00:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2011/07/25/Why-using-a-timer-may-not-be-the-best-idea", "/post/2011/07/25/why-using-a-timer-may-not-be-the-best-idea"]
author: jay
---
<!-- more -->
<p><strong><em>TLDR: The <a href="http://msdn.microsoft.com/en-us/data/gg577609">Reactive Extensions</a> provide a TestScheduler class that abstracts time and allows for a precise control of a virtual time base. Using the Rx Schedulers mecanism instead of real timers enables the creation of fast running unit tests that validate time related algorithms without the inconvients of the actual time.</em></strong></p>
<p>&nbsp;</p>
<p>Granted, the title is a bit provocative, but nonetheless it's still a bad idea to use timer classes like System.Threading.Timer.</p>
<p>Why is that ? Because classes&nbsp;like this one&nbsp;are based on the actual time, and that makes it a problem because it is non-deterministic. This means that each time you want to test a piece of code that depends on time, you'll be having a somehow different result, and particularly&nbsp;if you're using very long delays, like a few hours, you probably will not want to wait that long to make sure your code works as expected.</p>
<p>What you want actually, to avoid the side effect of "real"&nbsp;time passing by, is virtual time.</p>
<p>&nbsp;</p>
<h2>Abstracting Time with the Reactive Extensions</h2>
<p>The reactive extensions are pretty good at abstracting time, with the IScheduler interface and TestScheduler class.</p>
<p>Most operators in the <a href="http://msdn.microsoft.com/en-us/data/gg577609">Rx&nbsp;framework</a> have an optional scheduler they can use, like the Dispatcher or the ThreadPool schedulers. These schedulers are used to change the context of execution of the OnNext, OnCompleted and OnError events.</p>
<p>But for the case of time, the point is to&nbsp;freeze the time and make it "advance" to the point in time we need, and most importantly when we need it.</p>
<p>Let's say that we have a view model&nbsp;with a&nbsp;command that performs a lengthy&nbsp;action on the network, and that we need that action to timeout after a few seconds.</p>
<p>Consider this service contract :</p>
<pre class="brush: c-sharp">public interface IRemoteService
{
    /// 
    /// Gets the data from the server,
    /// returns an observable call to the server 
    /// that will provide either no values, one
    /// value or an error.
    /// 
    IObservable&lt;string&gt; GetData(string url);
}</pre>
<p>This implementation is exposing&nbsp;an observable&nbsp;based API, where the consumer of this contract must take into account the fact that&nbsp;getting data must be performed asynchronously, because it may not provide any value for a long time or even not return anything at all.</p>
<p>Next,&nbsp;a method of a view model that is using it :</p>
<pre class="brush: c-sharp">public void GetServerDataCommand()
{
    IsQueryRunning = true;
    ShowError = false;

    Service.GetData(_url)
           .Timeout(TimeSpan.FromSeconds(15))
           .Finally(() =&gt; IsQueryRunning = false)
           .Subscribe(
               s =&gt; OnQueryCompleted(s),
               e =&gt; ShowError = true
           );
}
</pre>
<p>And we can test it using Moq like this :</p>
<pre class="brush: c-sharp">var remoteService = new Mock&lt;IRemoteService&gt;();

// Returns far in the future
remoteService.Setup(s =&gt; s.GetData(It.IsAny()))
             .Returns(
                Observable.Return("the result")
                          .Delay(TimeSpan.FromDays(1))
             );

var model = new ViewModel(remoteService.Object);

// Call the fake server
model.GetServerDataCommand();
Assert.IsTrue(model.IsQueryRunning);
Assert.IsFalse(model.ShowError);

// Sleep for a while, before the timeout occurs
Thread.Sleep(TimeSpan.FromSeconds(5));
Assert.IsTrue(model.IsQueryRunning);
Assert.IsFalse(model.ShowError);

// Sleep beyond the timeout
Thread.Sleep(TimeSpan.FromSeconds(11));

// Do we have an error ?
Assert.IsFalse(model.IsQueryRunning);
Assert.IsTrue(model.ShowError);
</pre>
<p>The problem with this test is that it depends on actual time, meaning that it takes at least 16 seconds to complete. This is not acceptable in a automated tests scenario, where you want your tests to run as fast as possible.</p>
<p>&nbsp;</p>
<h2>Adding the IScheduler in the loop</h2>
<p>We can introduce the use of an injected IScheduler instance into the view model, like this :</p>
<pre class="brush: c-sharp">.Timeout(TimeSpan.FromSeconds(15), _scheduler)</pre>
<p>Meaning that the both Start and Timeout will get executed on the scheduler we provide, for which the time is controlled.</p>
<p>We can update the test like this :</p>
<pre class="brush: c-sharp">var remoteService = new Mock&lt;IRemoteService&gt;();
var scheduler = new TestScheduler();

// Never returns
remoteService.Setup(s =&gt; s.GetData(It.IsAny&lt;string&gt;()))
             .Returns(
                Observable.Return("the result", scheduler)
                          .Delay(TimeSpan.FromDays(1), scheduler)
            );

var model = new ViewModel(remoteService.Object, scheduler);

// Call the fake server
model.OnGetServerData2();
Assert.IsTrue(model.IsQueryRunning);
Assert.IsFalse(model.ShowError);

// Go before the failure point
scheduler.AdvanceTo(TimeSpan.FromSeconds(5).Ticks);
Assert.IsTrue(model.IsQueryRunning);
Assert.IsFalse(model.ShowError);

// Go beyond the failure point
scheduler.AdvanceTo(TimeSpan.FromSeconds(16).Ticks);

// Do we have an error ?
Assert.IsFalse(model.IsQueryRunning);
Assert.IsTrue(model.ShowError);</pre>
<p>When the scheduler is created, it holds of all the scheduled operations until AdvanceTo is called. Then the scheduled actions are executed according to the virtual current time.</p>
<p>That way, your tests run at full speed, and you can test properly your time depedent code.</p>
{% include imported_disclaimer.html %}
