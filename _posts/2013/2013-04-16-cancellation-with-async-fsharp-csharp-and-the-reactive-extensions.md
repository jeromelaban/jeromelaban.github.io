---
layout: post
title: "Cancellation with Async F#, C# and the Reactive Extensions"
date: 2013-04-16 20:25:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2013/04/16/Cancellation-with-Async-Fsharp-Csharp-and-the-Reactive-Extensions", "/post/2013/04/16/cancellation-with-async-fsharp-csharp-and-the-reactive-extensions"]
author: jay
---
<p><em>TL;DR: C# 5.0 async/await&nbsp;does not include the implicit support for cancellation, and needs to pass CancellationToken instances to every async method. F# and the Reactive Extensions offer solutions to this problem, with both implicit and explicit support for cancellation.</em></p>
<p>&nbsp;</p>
<p>My development style has slowly shifted to a more functional approach, during the past year. I&rsquo;ve been peeking a F# for a while and that shift to a more functional mindset&nbsp;in C# lends me toward understanding a lot better the concepts behind core features of F#, and more specifically the async &ldquo;support&rdquo; in F#.</p>
<p>It&rsquo;s known that F# inspired a lot the implementation of C# async, but having looked at the way it&rsquo;s been implemented in F# gives me some more points against the &ldquo;unfinished&rdquo; implementation in C#.</p>
<p>&nbsp;</p>
<p>Recently, now that people are effectively using async, in real-world scenarios, problems are starting to bubble up, and some to <a href="https://twitter.com/josefajardo/status/303998917027192832">giggle</a>. <a href="http://www.jaylee.org/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void.aspx">Async void, async void lambdas</a>, the fact that continuations run mostly on the UI thread when not taken care of properly, <a href="http://www.jaylee.org/post/2012/09/29/C-Async-Tips-and-Tricks-Part-3-Tasks-and-the-Synchronization-Context.aspx">obscure exception handling scenarios</a>, <a href="http://www.jaylee.org/post/2012/06/18/CSharp-5-0-Async-Tips-and-Tricks-Part-1.aspx">the &ldquo;magic&rdquo; relation to the SynchronizationContext</a>, that it does not address parallelism, and one that&rsquo;s been pretty&nbsp;low-key, <strong>cancellation</strong>.</p>
<!-- more -->
<p>Cancellation is pretty important, particularly, when the operations can take a pretty long time. If the user awaits an async result, he may have the right to cancel the operation, either because it took too long, or because the rest of the processing is not needed, as&nbsp;the user&nbsp;has gone away. This avoids wasting precious CPU cycles, and sometimes network or I/O related&nbsp;resources for computations for which the result will not be used.</p>
<p>&nbsp;</p>
<h2>Cancelling a C# async method</h2>
<p>Now, consider the following code sample, that tries to cancel the execution of an async method:</p>
<pre class="brush: c-sharp">public static void Run()
{
    var cts = new CancellationTokenSource();
    Task.Run(async () =&gt; await Test(), cts.Token);
    Console.ReadLine();
    cts.Cancel();
    Console.WriteLine("Cancel...");
    Console.ReadLine();
}

private static async Task Test()
{
    while (true)
    {
        await Task.Delay(1000);
        Console.WriteLine("Test...");
    }
}
</pre>
<p>Which produces the following output :</p>
<pre class="brush: c-sharp">Test...
Test...
Test...

Cancel...
Test...
Test...
Test...
Test...
</pre>
<p>The problem here is pretty simple: the cancellation token only cancels the task created by Task.Run(). This can be problematic, particularly if the sub-tasks are computationally intensive, or are making external calls (e.g. an http web request) that effectively need to be cancelled.</p>
<p>With&nbsp;the TPL, the notion of cancellation is present through the concept of <a href="http://msdn.microsoft.com/en-us/library/system.threading.cancellationtoken.aspx">CancellationToken</a>. A cancellation token is basically a glorified thread-safe boolean value that tells that it&rsquo;s creator, a <a href="http://msdn.microsoft.com/en-us/library/system.threading.cancellationtokensource.aspx">CancellationTokenSource</a>, has been cancelled.</p>
<p>So, there are two things to do to support cancellation in an async method.</p>
<p>First, by passing a CancellationToken explicitly and poll the token frequently&nbsp;:</p>
<pre class="brush: c-sharp">public static void Run()
{
    var cts = new CancellationTokenSource();
    Task.Run(async () =&gt; await Test(cts.Token), cts.Token);
    Console.ReadLine();
    cts.Cancel();
    Console.WriteLine("Cancel...");
    Console.ReadLine();
}

private static async Task Test(CancellationToken token)
{
    while (true)
    {
        await Task.Delay(1000);
        Console.WriteLine("Test...");

        if (token.IsCancellationRequested)
        {
            break;
        }
    }
    Console.WriteLine("Test cancelled");
}
</pre>
<p>Second, via a delegate passed through <a href="http://msdn.microsoft.com/en-us/library/dd321635.aspx">CancellationToken.Register</a>:</p>
<pre class="brush: c-sharp">public static void Run()
{
    var cts = new CancellationTokenSource();
    Task.Run(async () =&gt; await Test(cts.Token), cts.Token);
    Console.ReadLine();
    cts.Cancel();
    Console.WriteLine("Cancel...");
    Console.ReadLine();
}

private static async Task Test(CancellationToken token)
{
    var wr = HttpWebRequest.Create("http://1.2.3.4");

    token.Register(() =&gt; { 
        Console.WriteLine("Query cancelled");
        wr.Abort(); 
    });

    var r = await Task&lt;WebResponse&gt;.Factory.FromAsync(wr.BeginGetResponse, wr.EndGetResponse, null);

    if (token.IsCancellationRequested)
    {
        return;
    }

    Console.WriteLine("Got a result");
}</pre>
<p>There are multiple problems with this approach, one being that the cancellation token&nbsp;parameter needs to appear explicitly in every single method of the call tree. If you have multiple layers of code and abstractions, cancellation becomes pretty much prominent.</p>
<p>Second is, to be thorough, to check the cancellation token as often as possible to avoid continuing doing work if it&rsquo;s not necessary, even if you've detected that the task has been cancelled.</p>
<p>&nbsp;</p>
<h2>Cancelling an F# async function</h2>
<p>With F#, <a href="http://msdn.microsoft.com/en-us/library/dd233250.aspx">async</a> is actually a &ldquo;side-effect&rdquo; of the more powerful feature called the <a href="http://msdn.microsoft.com/en-us/library/dd233182.aspx">Computation Expressions</a>. Naively, I see this a super-feature that allows the creation of some of the syntactic sugar based-features of C#, async and the iterators.</p>
<p>Async support takes the form of an <a href="http://msdn.microsoft.com/en-us/library/dd233250.aspx">Async type</a>, that allows the creation of an &ldquo;async&rdquo; computation expression that, in turn, allows for the manipulation of Async&lt;T&gt; returning expressions. If this sounds familiar, you&rsquo;re spot-on, this is roughly the same as Task&lt;T&gt; and C# async/await.</p>
<p>Now, the interesting thing about these Async&lt;T&gt; expressions is that when created, Async&lt;T&gt; functions respect the cancellation token used when starting the async expression :</p>
<pre class="brush: text">open System
open System.Threading
let main argv = 

    let myAsync = 
        async {
            while true do 
                do! Async.Sleep(1000)
                Console.WriteLine(DateTime.Now)
        }

    let tokenSource1 = new System.Threading.CancellationTokenSource()
    let val1 = Async.Start(myAsync, cancellationToken=tokenSource1.Token)    
    Console.ReadLine() |&gt; ignore
    tokenSource1.Cancel()
    Console.ReadLine() |&gt; ignore
    0</pre>
<p>When cancelled, the async expression stops processing its content, without any explicit support of the token.</p>
<p>But the most interesting part in this is that this token flows through to the other async expressions !</p>
<p>To better demonstrate this, there is the ability to register a function that will be called when the async expression is called :</p>
<pre class="brush: text">let main argv = 

    let r2 = 
        async {
            use! c = Async.OnCancel(fun () -&gt; Console.WriteLine("Cancelled 1"))

            // In case there is a non async computation that needs 
            // to check on cancellation
            let! token = Async.CancellationToken
            
            while true do 
                do! Async.Sleep(1000)
                Console.WriteLine(DateTime.Now)

            return 42
        }

    let r = 
        async {
            use! c = Async.OnCancel(fun () -&gt; Console.WriteLine("Cancelled 2"))

            let! test = r2

            while true do 
                do! Async.Sleep(100)

            return 42
        }

    let tokenSource1 = new System.Threading.CancellationTokenSource()
    let res3 = Async.StartAsTask(r, cancellationToken=tokenSource1.Token)    
    Console.ReadLine() |&gt; ignore
    tokenSource1.Cancel()
    Console.ReadLine() |&gt; ignore
    0</pre>
<p>This way, there is no explicit need for passing around a cancellation token, and if it is needed, then it can be used explicitly. There is also the ability to asynchronously get the ambient <a href="http://msdn.microsoft.com/en-us/library/ee353544.aspx">CancellationToken</a>&nbsp;(which is also async !), in case there is a CPU bound computation that needs to be cancelled. Or you can just call any ! (bang) operator that will automatically check on the cancellation.</p>
<p>Pretty powerful !</p>
<h2>Cancelling an Rx query</h2>
<p>On the same topic of cancellation, the <a href="http://msdn.microsoft.com/en-us/data/gg577609">Reactive Extensions</a> also have the notion of cancellation embedded into the flow.</p>
<pre class="brush: c-sharp">public static void Run()
{
    var s = Observable.Create(o =&gt;
        {
            var wr = HttpWebRequest.Create("http://1.2.3.4");

            var subscriptions = new CompositeDisposable();

            subscriptions.Add(Disposable.Create(() =&gt; wr.Abort()));

            var s = Observable
                .FromAsyncPattern(wr.BeginGetResponse, wr.EndGetResponse)()
                .Subscribe(o);

            subscriptions.Add(s);

            return subscriptions;
        }
    )
    .Subscribe(_ =&gt; Console.WriteLine("Got result"));

    Console.ReadLine();
    s.Dispose();
    Console.ReadLine();
}
</pre>
<p>Every observable can be subscribed to, and returns a disposable instance. Whenever the final subscription gets disposed, the whole query gets disposed as well, along with the ability to intercept that disposition with the Observable.Create operator.</p>
<p>&nbsp;</p>
<h2>Final words&hellip;</h2>
<p>Cancellation is an important topic that has been &ldquo;forgotten&rdquo; in C# async. This is a very important topic, and this one adds up to all the others about the implementation in C# 5.0, for which I hope this will get fixed in a future release in the language.</p>
<p>For the moment, both Rx and F# approach async in a more thorough manner, and moreover, attack head-on the concurrency side of asynchrony which is ignored by C# 5.0...</p>
<p>If you're interested in a more detailed comparison of F# and C# on the subject,&nbsp;read <a href="http://tomasp.net/blog/csharp-fsharp-async-intro.aspx">Tomas Petricek blog post series</a>.</p>
{% include imported_disclaimer.html %}
