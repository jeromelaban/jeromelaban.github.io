---
layout: post
title: "To be fair when comparing Rx to C# 5.0 Async..."
date: 2011-08-03 20:45:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2011/08/03/To-be-fair-when-comparing-Rx-to-csharp-50-Async", "/post/2011/08/03/to-be-fair-when-comparing-rx-to-csharp-50-async"]
author: admin
---
<!-- more -->
<p>After reading the <a href="http://blog.cwa.me.uk/2011/07/22/the-morning-brew-900/?utm_source=feedburner&amp;utm_medium=feed&amp;utm_campaign=Feed%3A+ReflectivePerspective+%28Reflective+Perspective+-+Chris+Alcock%29&amp;utm_content=Google+Reader">900th morning brew</a>, one article by <a href="http://mtaulty.com/CommunityServer/blogs/mike_taultys_blog/archive/2011/07/21/rx-tpl-async-ctp-oh-my.aspx">mike taulty about comparing Rx, TPL and async</a> caught my attention.</p>
<p>Mike tries to explain the history, differences and similarities between all these frameworks, and kudos, that's not an easy thing to do.</p>
<p>Asynchrony, (and I'm not talking parallelism), is a complex topic that fools even the best of us.</p>
<p>&nbsp;</p>
<h2>Comparing Rx and Async</h2>
<p>Rx and Async and much more similar in that regard, because going off to an other thread is not mandatory, and most operators use either the CurrentThread (timebase priority queue) scheduler or the Immediate (passive wait) scheduler.</p>
<p>This means that the code you are writing is doing some cooperative multi-threading, or <a href="http://msdn.microsoft.com/en-us/library/ms682661(v=vs.85).aspx">fibers-like</a> processing. Everything happens on the same thread, except that work is optionally&nbsp;being queued up, and that thread works as long as there's work left to do, then waits for outstanding operations. These oustanding operations can be I/O completion port bound, like Stream.BeginRead/EndRead.</p>
<p>But back to the comparison, mike is trying to do buffer reading of content from a web response stream, and doing so using the Async CTP's ReadAsync and WriteAsync eases a lot the writing of that kind of code. (Also, the Rx example does not work correctly, but I'll talk about that later in this post.)</p>
<p>These two&nbsp;functions are not tied to the complexity of the BeginRead/EndRead, and behave very much like an IObservable would. Call ReadAsync, you get a Task and wait on it.</p>
<p>Let's jump to the end result&nbsp;and we can get this with Rx based composed operators, using&nbsp;methods symilar to the ReadAsync and GetResponseAsync&nbsp;:</p>
<pre class="brush: c-sharp">string url = "http://www.microsoft.com";

var webRequest = WebRequest.Create(url);

webRequest.ToResponse()
          .SelectMany(wr =&gt; wr.GetResponseStream().ToBytes())
          .ForEach(
            b =&gt; Console.WriteLine(Encoding.Default.GetString(b))
          );
</pre>
<p>That way, it is a lot easier to read. ToResponse maps to GetResponseAsync and ToBytes maps to ReadAsync.</p>
<p>I'll concede the complexity of&nbsp;the SelectMany operator that is related to the fact that IObservable deals with sequences (the duality with IEnumerable). What we would need is more like a IObservableValue that returns only one value. At that point, an appropriate operator would be something like SelectOne, but that's an other topic I'll discuss soon.</p>
<p>&nbsp;</p>
<h2>The ToResponse operator</h2>
<p>This one is easy, and is pretty much an encapsulation of the code provided in Mike's Rx example :</p>
<pre class="brush: c-sharp">public static IObservable ToResponse(this WebRequest request)
{
    var asyncGetResponse = Observable.FromAsyncPattern(
                            request.BeginGetResponse, request.EndGetResponse);

    return Observable.Defer(asyncGetResponse);
}</pre>
<p>The use of defer allows the execution of the actual call to GetResponse when someone is subscribing to the deffered observable.</p>
<p>&nbsp;</p>
<h2>The ToBytes operator</h2>
<p>That one is a bit tricker :</p>
<pre class="brush: c-sharp">public static IObservable ToBytes(this Stream stream)
{
    return 
        Observable.Create(
            observer =&gt;
            {
                byte[] buffer = new byte[24];

                var obsReadFactory = Observable.Defer(() =&gt; stream.AsReader()(buffer, 0, buffer.Length));

                return Observable
                         .Repeat(obsReadFactory)
                         .Select(i =&gt; buffer.Take(i).ToArray())

                         // Subscribe on the thread pool, otherwise the repeat operator will operate during the 
                         // call to subscribe, preventing the whole expression to complete properly
                         .SubscribeOn(Scheduler.ThreadPool)

                         .Subscribe(
                             _ =&gt;
                             {
                                 if (_.Length &gt; 0)
                                 {
                                     observer.OnNext(_);
                                 }
                                 else
                                 {
                                     observer.OnCompleted();
                                 }
                             },
                             observer.OnError,
                             observer.OnCompleted
                         );
            }
        );
}</pre>
<p>and&nbsp;needs a bit of explaining.</p>
<p>The ToBytes extension is creating a Observable that will be able to control finely the OnNext/OnCompleted events, especially because of the loopy nature of the BeginRead API. Loopy means that in a synchronous mode, you needs to call Read through a loop until you get all you need. The BeginRead/EndRead still expose this loopy nature, but in an asynchronous way.</p>
<p>With Rx, that loop can be introduced with the use of Repeat, the same way a while(true) would do.</p>
<p>The Select operator is pretty straightforward, even if it may not be as fast as a Buffer.BlockCopy, it's pretty conscise.</p>
<p>The SubscribeOn is the tricky part of this method, and is very important for the OnCompleted events to get through. If this operator is not present, the call to the ToBytes method blocks in the Subscribe of the SelectMany operator in the final example. This means that events like OnCompleted get buffered and not interpreted, and the repeat operator will continue indefinitely&nbsp;to turns into loops getting nothing, because noone's unsubscribing. This would be CPU consuming, and memory consuming because the observable expression could not get dispose.</p>
<p>Then in the subscribe, we notify the observer that either there's a new buffer, or that we're done because the EndRead method returned 0 (or an exception).</p>
<p>&nbsp;</p>
<h2>Continuing the comparison</h2>
<p>All this to say that the .NET 5.0 (or whatever it will be called) has a Task friendly&nbsp;BCL that makes it easy to&nbsp;write&nbsp;asynchronous code.</p>
<p>I'm definitely not saying that Rx is as easy as Async will be, but, with a minimum of understanding and abstraction, it can be as powerful and even more powerful because of its ability to compose observables.</p>
<p>Now, for the issues in Mike's Rx sample :</p>
<ul>
<li>The TakeWhile operator only completes when the source observable completes, and the source is a Repeat observable, which never ends, meaning that the whole subscription will never get disposed</li>
<li>The Observable.Repeat operator runs on the&nbsp;Immediate scheduler, meaning that the OnNext method will be called&nbsp;in the&nbsp;thread context of the&nbsp;original Subscribe, and the expression will not get disposed.</li>
</ul>
<p>The sample actually shows something, but the CPU will stay at 100% until the process ends.</p>
<p>&nbsp;</p>
<h2>A word on Asynchrony</h2>
<p>Asynchrony is a complex topic, very easy to get wrong, even with the best intentions. Parallelism is even more complex (and <strong>don't </strong>get me started on the lock() keyword).</p>
<p>I'm expecting that People are going to have a hard time grasping it, and I'm worried that Async will make it too easy to make parallelism (not asynchrony this time)&nbsp;mistakes, because it is will be easy to introduce mutating states in the loop, hence hard to reproduce transitory states&nbsp;bugs. Calling "new Thread()" was scary, and for good reasons, but using await will not, but with mostly the same bad&nbsp;consequences.</p>
<p>I'd rather have better support for immutable structures method or class&nbsp;purity&nbsp;and some more functional concepts baked into C#, where the language makes it harder to make mistakes, than trying to&nbsp;bend (or abstract)&nbsp;asynchrony to make it work with the current state of C#.</p>
<p>Then again, I'm not saying Rx is better, it's trying to work around the fact that the BCL and C# 3.0 don't have asynchrony baked in, so the complexity argument still stands.</p>
<p>&nbsp;</p>
<p>On the other side, the more developers use Async, the more developers will need async savvy&nbsp;consultants as firemen :)</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
{% include imported_disclaimer.html %}
