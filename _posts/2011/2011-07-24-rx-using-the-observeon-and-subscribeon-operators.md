---
layout: post
title: "[Rx] Using the ObserveOn and SubscribeOn operators"
date: 2011-07-24 20:04:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2011/07/24/Rx-Using-the-ObserveOn-and-SubscribeOn-operators", "/post/2011/07/24/rx-using-the-observeon-and-subscribeon-operators"]
author: jay
---
<!-- more -->
<p><strong><em>TLDR: This post talks about how the </em><a href="http://msdn.microsoft.com/en-us/data/gg577609"><em>Reactive Extensions</em></a><em> ObserveOn operator changes the execution context (the thread) of the IObservable OnNext/OnComplete/OnError methods, whereas the SubscribeOn operator changes the execution context of the implementation of the Subscribe method in the chain of observers. Both methods can be useful to improve the performance of an application's UI by putting work on background threads.</em></strong></p>
<p>&nbsp;</p>
<p>When developing asynchronous code, or consuming asynchronous APIs, you find yourself forced to use specific methods on a specific thread.</p>
<p>The most prominent examples being WPF/Silverlight and Winforms, where you cannot use UI bound types outside of the UI Thread. In the context of WPF, you'll find yourself forced to use the <a href="http://msdn.microsoft.com/en-us/library/system.windows.threading.dispatcher.invoke.aspx">Dispatcher.Invoke</a> method to manipulate the UI in the proper context.</p>
<p>However, you don't want to execute everything on the UI Thread, because the UI performance relies on it. Doing too much on the UI thread can lead to a very bad percieved performance, and angry users...</p>
<p>&nbsp;</p>
<h2>Rx framework's ObserveOn operator</h2>
<p>I've <a href="http://jaylee.org/post/2011/04/22/WP7-HttpWebRequest-and-the-Flickr-app-Black-Screen-issue.aspx">discussed</a> a <a href="http://jaylee.org/post/2010/07/26/Revisited-with-the-Reactive-Extensions-DataBinding-and-Updates-from-multiple-Threads.aspx">few times</a> the <a href="http://jaylee.org/post/2010/06/22/WP7Dev-Using-the-WebClient-with-Reactive-Extensions-for-Effective-Asynchronous-Downloads.aspx">use of ObserveOn</a> in&nbsp;the context of WP7, where it is critical to leave the UI thread alone and avoid choppy animations, for instance.</p>
<p>The ObserveOn operator changes the context of execution (scheduler)&nbsp;of a chain of operators until an other operator changes it.</p>
<p>To be able to demonstrate this, let's write our own scheduler :</p>
<pre class="brush: c-sharp">public class MyScheduler : IScheduler
{
    // Warning: ThreadStatic is not supported on Windows Phone 7.0 and 7.1
    // This code will not work properly on this platform.
    [ThreadStatic]
    public static int? SchedulerId;

    private int _schedulerId;
    private IScheduler _source;
        
    public MyScheduler(IScheduler source, int schedulerId)
    {
        _source = source;
        _schedulerId = schedulerId;
    }

    public DateTimeOffset Now { get { return _source.Now; } }

    public IDisposable Schedule&lt;TState&gt;(
              TState state, 
              Func&lt;IScheduler, TState, IDisposable&gt; action
           )
    {
        return _source.Schedule(state, WrapAction(action));
    }

    private Func&lt;IScheduler, TState, IDisposable&gt; WrapAction&lt;TState&gt;(
              Func&lt;IScheduler, TState, IDisposable&gt; action)
    {
        return (scheduler, state) =&gt; {

            // Set the TLS with the proper ID
            SchedulerId = _schedulerId;

            return action(_source, state);
        };
    }
}

</pre>
<p>This scheduler's purpose is to intercept calls to the ISchedule methods (You'll fill the missing Schedule methods by yourself) and flag them with a custom thread ID. That way, we'll know which scheduler is executing our code.</p>
<p>Note that&nbsp;this code will not work properly on Windows Phone 7, since <a href="http://jaylee.org/post/2010/06/19/WP7Dev-Beware-of-the-ThreadStatic-attribute-on-Silverlight-for-Windows-Phone-7.aspx">ThreadStaticAttribute is not supported</a>. And it's still not supported on 7.1... Seems like not enough people are using ThreadStatic to make its way to the WP7 CLR...</p>
<p>Anyway, now if we write the following Rx expression :</p>
<pre class="brush: c-sharp">Observable.Timer(TimeSpan.FromSeconds(1), new MyScheduler(Scheduler.ThreadPool, 42))
          .Do(_ =&gt; Console.WriteLine(MyScheduler.SchedulerId))
          .First();
</pre>
<p>We force the timer to raise OnNext on the ThreadPool through our scheduler, and we'll get the following :</p>
<pre class="brush: c-sharp">42</pre>
<p>Which means that the lambda passed as a parameter to the Do operator got executed in the context of the Scheduler used when declaring the Timer operator.</p>
<p>If we go a bit farther :</p>
<pre class="brush: c-sharp">Observable.Timer(TimeSpan.FromSeconds(1), new MyScheduler(Scheduler.ThreadPool, 42))
          .Do(_ =&gt; Console.WriteLine("Do(1): " + MyScheduler.SchedulerId))
          .ObserveOn(new MyScheduler(Scheduler.ThreadPool, 43))
          .Do(_ =&gt; Console.WriteLine("Do(2): " + MyScheduler.SchedulerId))
          .ObserveOn(new MyScheduler(Scheduler.ThreadPool, 44))
          .Do(_ =&gt; Console.WriteLine("Do(3): " + MyScheduler.SchedulerId))
          .Do(_ =&gt; Console.WriteLine("Do(4): " + MyScheduler.SchedulerId))
          .First();
</pre>
<p>We'll get the following :</p>
<pre class="brush: c-sharp">Do(1): 42
Do(2): 43
Do(3): 44
Do(4): 44</pre>
<p>Each time a scheduler was specified, the following operators OnNext&nbsp;delegates&nbsp;were executed on that scheduler.</p>
<p>In this case, we're using the Do operator which does not take a scheduler as a parameter. There some operators though, like Delay, that&nbsp;implicitly use a scheduler that changes the context.</p>
<p>Using this operator is particularly useful when the OnNext delegate is performing a context sensitive operation, like manipulating the UI, or when the source scheduler is the UI and the OnNext delegate is not related to the UI and can be executed on an other thread.</p>
<p>You'll find that operator handy with the WebClient or GeoCoordinateWatcher classes, which both execute their handlers on the UI thread. Watchout for Windows Phone 7.1 (mango) though, this may have changed a bit.</p>
<p>&nbsp;</p>
<h2>An&nbsp;Rx Expression's life cycle</h2>
<p>Using an Rx expression is performed in a least&nbsp;5 stages :</p>
<ul>
<li>The construction of the expression,</li>
<li>The subscription to the expression,</li>
<li>The optional&nbsp;execution of the OnNext delegates passed as parameters (whether it be observers or explicit OnNext delegates),</li>
<li>The observer chain gets disposed either explicitly or implicitly,</li>
<li>The observers can optionally get collected by the GC.</li>
</ul>
<p>The third part's execution context is covered by ObserveOn. But for the first two, this is different.</p>
<p>The expression is constructed like this :&nbsp;</p>
<pre class="brush: c-sharp">var o = Observable.Timer(TimeSpan.FromSeconds(1));</pre>
<p>Almost nothing's been executed here, just the creation of the observers for the entire expression, in a similar way IEnumerable expressions work. Until you call the <a href="http://msdn.microsoft.com/en-us/library/system.collections.ienumerator.movenext.aspx">IEnumerator.MoveNext</a>, nothing is performed. In Rx expressions, until the Subscribe method is called, nothing is happening.</p>
<p>Then you can subscribe to the expression :</p>
<pre class="brush: c-sharp">var d = o.Subscribe(_ =&gt; Console.WriteLine(_));</pre>
<p>At this point, the whole chain of operators get their Subscribe method called, meaning they can start sending OnNext/OnError/OnComplete messages.</p>
<p>&nbsp;</p>
<h2>The case of Observable.Return and SubscribeOn</h2>
<p>Then you meet that kind of expressions :</p>
<pre class="brush: c-sharp">Observable
   .Return(42L)
   // Merge both enumerables into one, whichever the order of appearance
   .Merge(
      Observable.Timer(
         TimeSpan.FromSeconds(1), 
         new MyScheduler(Scheduler.ThreadPool, 42)
      )
   )
   .Subscribe(_ =&gt; Console.WriteLine("Do(1): " + MyScheduler.SchedulerId));

Console.WriteLine("Subscribed !");</pre>
<p>This expression will merge the two observables into one that will provide two values, one from Return and one from the timer.</p>
<p>And this is the output :</p>
<pre class="brush: c-sharp">Do(1):
Subscribed !
Do(1): 42</pre>
<p>The Observable.Return OnNext was executed during the call to Subscribe, and has that thread has no SchedulerId, meaning that a whole lot of code has been executed in the context of the caller of Subscribe. You can imagine that if that expression is complex, and that the caller is the UI Thread, that can become a performance issue.</p>
<p>This is where the SubscribeOn operator becomes handy :</p>
<pre class="brush: c-sharp">Observable
   .Return(42L)
   // Merge both enumerables into one, whichever the order of appearance
   .Merge(
      Observable.Timer(
         TimeSpan.FromSeconds(1), 
         new MyScheduler(Scheduler.ThreadPool, 42)
      )
   )
   .SubscribeOn(new MyScheduler(Scheduler.ThreadPool, 43))
   .Subscribe(_ =&gt; Console.WriteLine("Do(1): " + MyScheduler.SchedulerId));

Console.WriteLine("Subscribed !");</pre>
<p>You then get this :</p>
<pre class="brush: c-sharp">Subscribed !
Do(1): 43
Do(1): 42</pre>
<p>The first OnNext is now executed under of a different scheduler, making subscribe a whole lot faster from the caller's point of view.</p>
<p>&nbsp;</p>
<h2>Why not always Subscribe on an other thread ?</h2>
<p>That might come in handy, but you may not want that as an opt-out because of this scenario :</p>
<pre class="brush: c-sharp">Observable.FromEventPattern(textBox, "TextChanged")
          .SubscribeOn(new MyScheduler(Scheduler.ThreadPool, 43))
          .Subscribe(_ =&gt; { });

Console.WriteLine("Subscribed !");</pre>
<p>You'd get an "Invalid cross-thread access." System.UnauthorizedAccessException, because yo would try to add an event handler to a UI element from a different thread.&nbsp;</p>
<p>Interestingly though, this code does not work on WP7 but does on WPF 4.</p>
<p>An other scenario may be one where delaying the subscription may loose messages, so you need to make sure you're completely subscribed before raising events.</p>
<p>&nbsp;</p>
<p>So there you have it :)&nbsp;I hope this helps you understand a bit better those two operators.</p>
{% include imported_disclaimer.html %}
