---
layout: post
title: "Asynchronous Programming with the Reactive Extensions (while waiting for async/await)"
date: 2011-11-25 20:43:00 -0500
comments: true
category: Archive
tags: []
redirect_from: ["/post/2011/11/25/Asynchronous-Programming-with-the-Reactive-Extensions-(while-waiting-for-asyncawait)", "/post/2011/11/25/asynchronous-programming-with-the-reactive-extensions-(while-waiting-for-asyncawait)"]
author: jay
---
<!-- more -->
<p><em>This article was originally published on the <a href="http://blogs.msdn.com/b/mvpawardprogram/archive/2011/11/30/asynchronous-programming-with-the-reactive-extensions-while-waiting-for-async-await-programmation-asynchrone-avec-les-reactive-extensions-en-attendant-async-await.aspx">MVP Award Blog</a> in December 2011.</em></p>
<p>Nowadays, with applications that use more and more services that are in the cloud, or simply perform actions that take a user noticeable time to execute, it has become vital to program in an asynchronous way.</p>
<p>But we, as developers, feel at home when thinking sequentially. We like to send a request or execute a method, wait for the response, and then process it.</p>
<p>Unfortunately for us, an application just cannot wait synchronously for a call to end anymore. Reasons can be that the user expects the application to continue responding, or because the application joins the results of multiple operations, and it is necessary to perform all these operations simultaneously for good performance.</p>
<p>Frameworks that are heavily UI dependent (like Silverlight or Silverlight for Windows Phone) are trying the force the developer's hand into programming asynchronously by removing all synchronous APIs. This leaves the developer alone with either the <a href="http://msdn.microsoft.com/en-us/library/ms228963.aspx">Begin/End pattern</a>, or the plain old C# events. Both patterns are not flexible, not easily composable, often lead to memory leaks, and are just plain difficult to use or worse, to read.</p>
<h4>C# 5.0 async/await</h4>
<p>Taking a quick look at the not so distant future, Microsoft has taken the bold approach to augment its new .NET 4.5 to include asynchronous APIs and in the case of the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh464942(v=vs.85).aspx">Windows Runtime</a> (WinRT), restrict some APIs to be asynchronous only. These are based on the <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.aspx">Task</a>&nbsp;class, and are backed by languages to ease asynchronous programming.</p>
<p>In the upcoming C# 5.0 implementation, the <a href="http://blogs.msdn.com/b/ericlippert/archive/2010/10/29/asynchronous-programming-in-c-5-0-part-two-whence-await.aspx">async/await pattern</a>&nbsp;is trying to handle this asynchrony problem by making asynchronous code look synchronous. It makes asynchronous programming more "familiar" to developers.</p>
<p>If we take this example:</p>
<pre class="brush: csharp">    static void Main(string[] args)
    {
        // Some initialization of the DB...
        Task&lt;int&gt; t = GetContentFromDatabase();

        // Execute some other code when the task is done
        t.ContinueWith(r =&gt; Console.WriteLine(r.Result));

        Console.ReadLine();
    }

    public static async Task&lt;int&gt; GetContentFromDatabase()
    {
        int source = 22;

        // Run starts the execution on another thread
        var result = (int) await Task.Run(
            () =&gt; { 
                // Simulate DB access
                Thread.Sleep(1000);
                return 10; 
            }
        );

        return source + result * 2;
    }

</pre>
<p>The code in <em>GetContentFromDatabase</em>looks synchronous, but under the hood, it's actually split in half (or more) where the await keyword is used.</p>
<p>The compiler is applying a technique used many times in the C# language, known as syntactic sugar. The code is expanded to a form that is less readable, but is more of a plumbing code that is painful to write &ndash; and get right &ndash; each time. The <a href="http://msdn.microsoft.com/en-us/library/yh598w02.aspx">using statement</a>, <a href="http://msdn.microsoft.com/en-us/library/dscyy5s0.aspx">iterators</a>&nbsp;and more recently LINQ are very good examples of that syntactic sugar.</p>
<p>Using a plain old thread pool call, the code actually looks a lot more like this, once the compiler is done:</p>
<pre class="brush: csharp">    public static void Main()
    {
        MySyncMethod(result =&gt; Console.WriteLine(result));
        Console.ReadLine();
    }

    public static void GetContentFromDatabase (Action&lt;int&gt; continueWith)
    {
        // The first half of the async method (with QueueUserWorkItem)
        int source = 22;

        // The second half of the async method
        Action&lt;int&gt; onResult = result =&gt; {
            continueWith(source + result * 2);
        };

        ThreadPool.QueueUserWorkItem(
            _ =&gt; {
                // Simulate DB access
                Thread.Sleep(1000);

                onResult(10);
            }
        );
    }
</pre>
<p>This sample somewhat more complex, and does not properly handle exceptions. But you probably get the idea.</p>
<h4>Asynchronous Development now</h4>
<p>Nonetheless, you may not want or will be able to use C# 5.0 soon enough. A lot of people are still using .NET 3.5 or even .NET 2.0, and new features like <em>async</em> will take a while to be deployed in the field. Even when the framework has been offering it for a long time, the <a href="http://msdn.microsoft.com/en-us/library/bb397926.aspx">awesome LINQ</a>&nbsp;(a C# 3.0 feature) is still being adopted and is not that widely used.</p>
<p>The <a href="http://msdn.microsoft.com/en-us/data/gg577609">Reactive Extensions</a>&nbsp;(Rx for friends) offer a framework that is available from .NET 3.5 and functionality similar to C# 5.0, but provide a different approach to asynchronous programming, more functional. More functional is meaning fewer variables to maintain states, and a more declarative approach to programming.</p>
<p>But don't be scared. Functional does not mean abstract concepts that are not useful for the mainstream developer. It just means (<em>very</em>roughly) that you're going to be more inclined to separate your concerns using functions instead of classes.</p>
<p>But let's dive into some code that is similar to the two previous examples:</p>
<pre class="brush: csharp">    static void Main(string[] args)
    {
        IObservable&lt;int&gt; query = GetContentFromDatabase();

        // Subscribe to the result and display it
        query.Subscribe(r =&gt; Console.WriteLine(r));

        Console.ReadLine();
    }

    public static IObservable&lt;int&gt; GetContentFromDatabase()
    {
        int source = 22;

        // Start the work on another thread (using the ThreadPool)
        return Observable.Start&lt;int&gt;(
                   () =&gt; {
                      Thread.Sleep(1000);
                      return 10;
                   }
               )

               // Project the result when we get it
               .Select(result =&gt; source + result * 2);
    }

</pre>
<p>From the caller's perspective (the main), the GetContentFromDatabase method behaves almost the same way a .NET 4.5 Task would, and the Subscribe pretty much replaces the <a href="http://msdn.microsoft.com/en-us/library/dd270696.aspx">ContinueWith</a>&nbsp;method.</p>
<p>But this simplistic approach works well for an example. At this point, you could still choose to use the basic <em>ThreadPool</em>example shown earlier in this article.</p>
<h4>A word on IObservable</h4>
<p>An <a href="http://msdn.microsoft.com/en-us/library/dd990377.aspx">IObservable</a> is generally considered as a stream of data that can push to its subscribers zero or more values, and either an error or completion message. This &ldquo;Push&rdquo; based model that allows the observation of a data source without blocking a thread. This is opposed to the Pull model provided by <a href="http://msdn.microsoft.com/en-us/library/9eekhta0.aspx">IEnumerable</a>, which performs a blocking observation of a data source. A <a href="http://channel9.msdn.com/Shows/Going+Deep/Expert-to-Expert-Brian-Beckman-and-Erik-Meijer-Inside-the-NET-Reactive-Framework-Rx">very good video with Erik Meijer</a>&nbsp;explains these concepts on Channel 9.</p>
<p>To match the .NET 4.5 Task model, an IObservable needs to provide at most one value, or an error, which is what the <a href="http://msdn.microsoft.com/en-us/library/hh211971(v=VS.103).aspx">Observable.Start</a>&nbsp;method is doing.</p>
<h4>A more realistic example</h4>
<p>Most of the time, scenarios include calls to multiple asynchronous methods. And if they're not called at the same time and joined, they're called one after the other.</p>
<p>Here is an example that does task chaining:</p>
<pre class="brush: csharp">    public static void Main()
    {
        // Use the observable we've defined before
        var query = GetContentFromDatabase();

              // Once we get the token from the database, transform it first
        query.Select(r =&gt; "Token_" + r)

             // When we have the token, we can initiate the call to the web service
             .SelectMany(token =&gt; GetFromWebService(token))

             // Once we have the result from the web service, print it.
             .Subscribe(_ =&gt; Console.WriteLine(_));
    }

    public static IObservable&lt;string&gt; GetFromWebService(string token)
    {
        return Observable.Start(
            () =&gt; new WebClient().DownloadString("http://example.com/" + token)
        )
        .Select(s =&gt; Decrypt(s));
    }

</pre>
<p>The <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.selectmany(v=VS.103).aspx"><em>SelectMany</em></a> operator is a bit strange when it comes to the semantics of an IObservable that behaves like a Task. It can then be thought of a <a href="http://msdn.microsoft.com/en-us/library/dd270696.aspx"><em>ContinueWith</em></a>&nbsp;operator. The GetContentFromDatabase only pushes one value, meaning that the provided lambda for the SelectMany is only called once.</p>
<h4>Taking the Async route</h4>
<p><a href="http://jaylee.org/post/2011/10/17/WinRT-and-the-sugar-of-NET-event-handlers.aspx">A peek at WinRT</a> and the <a href="http://channel9.msdn.com/events/BUILD/BUILD2011">Build conference</a>&nbsp;showed a very interesting rule used by Microsoft when moving to asynchronous API throughout the framework. If an API call nominally takes more than 50ms to execute, then it's an asynchronous API call.</p>
<p>This rule is easily applicable to existing .NET 3.5 and later frameworks by exposing IObservable instances that provide at most one value, as a way to simulate a .NET 4.5 Task.</p>
<p>Architecturally speaking, this is a way to enforce that the consumers of a service layer API will be less tempted to synchronously call methods and negatively impact the perceived or actual performance of an application.</p>
<p>For instance, a "favorites" service implemented in an application could look like this, using Rx:</p>
<pre class="brush: csharp">    public interface IFavoritesService
    {
        IObservable&lt;Unit&gt; AddFavorite(string name, string value);
        IObservable&lt;bool&gt; RemoveFavorite(string name);
        IObservable&lt;string[]&gt; GetAllFavorites();
    }

</pre>
<p>All the operations, including ones that alter content, are executed asynchronously. It is always tempting to think a select operation will take time, but we easily forget that an <em>Add</em>operation could easily take the same amount of time.</p>
<p>A word on unit: The name comes from functional languages, and represents the void keyword, literally. A deep .NET CLR limitation prevents the use of System.Void as a generic type parameter, and to be able to provide a void return value, Unit has been introduced.</p>
<h4>Wrap up</h4>
<p>Much more can be achieved with Rx but for starters, using it as a way to perform asynchronous single method call seems to be a good way to learn it.</p>
<p>Also, a note to Rx experts, shortcuts have been taken to explain this in the most simple form, and sure there are many tips and tricks to know to use Rx effectively, particularly when it is used all across the board. The omission of the Completed event is one of them.</p>
<p>Finally, explaining the richness of the Reactive Extensions is a tricky task. Even the smart guys of the Rx team have a hard time doing so... I hope this quick start will help you dive into it!</p>
{% include imported_disclaimer.html %}
