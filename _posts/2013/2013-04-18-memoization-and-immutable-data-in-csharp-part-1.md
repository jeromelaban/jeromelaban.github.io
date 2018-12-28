---
layout: post
title: "Immutable Data and Memoization in C#, Part 1"
date: 2013-04-18 20:49:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2013/04/18/Memoization-and-Immutable-data-in-CSharp-Part-1.aspx", "/post/2013/04/18/memoization-and-immutable-data-in-csharp-part-1.aspx"]
author: jay
---
<p><em>TL;DR: Immutable data and memoization are functional programming concepts that can be applied to C# programming. These patterns have their strengths and weaknesses,&nbsp;discussed a bit&nbsp;in this article.&nbsp;</em></p>
<p>&nbsp;</p>
<p>I&rsquo;ve grown to not being a great fan a data mutability.</p>
<p>Data mutability can introduce a lot of side effects in the code, and it can be pretty complex to go back in time to know what a specific state was before the code failed. This gets worse when multiple threads are involved, and that tends to happen a lot more these days, now that even phones have multiple cores.</p>
<p>Sure, we can use <a href="http://msdn.microsoft.com/en-us/magazine/ee336126.aspx">IntelliTrace </a>to ease that kind of debugging, but that&rsquo;s pretty much limited to issues you already know about. That means you&rsquo;re reacting to issues that already happened, you&rsquo;re not proactively preventing those issues from happening.</p>
<p>So, to address this more reliably, there&rsquo;s the concept of <a href="https://en.wikipedia.org/wiki/Immutable_object">immutability</a>. When a set of data is built, it cannot change anymore. This means that you can pass it around, do computation with it, use it any place you want, there&rsquo;s not going to be any subtle concurrency issues because the data changed under your feet.</p>
<!-- more -->
<h2>Immutability and Performance</h2>
<p>The thing is, taken as is, immutability is not&nbsp;great friend of performance at first. When a computation is performed, the result is just passed to the next function that needs it, but if there is another function that depends on that same function's output, for the same input, then the computation is performed again.</p>
<p>There&rsquo;s already a solution for this problem, known as <a href="https://en.wikipedia.org/wiki/Memoization">Memoization</a>.</p>
<p>The basic idea is that, if the input data is immutable, and that the computation has no side effect, then for a given set of inputs, the output will always be the same. This in turn, means that the result can be cached for later reuse. This is called <a href="http://en.wikipedia.org/wiki/Referential_transparency_(computer_science)">Referential Transparency</a>.</p>
<p>Here&rsquo;s a simple example of memoization :</p>
<pre class="brush: c-sharp">public static Func&lt;TArg, TResult&gt; 
   AsMemoized&lt;TArg, TResult&gt;(this Func&lt;TArg, TResult&gt; func)
{
    var values = new Dictionary&lt;TArg, TResult&gt;();

    return (v) =&gt;
    {
        TResult value;

        if (!values.TryGetValue(v, out value))
        {
            value = values[v] = func(v);
        }

        return value;
    };
}
    </pre>
<p>And here a simple use for this, where you know the data won&rsquo;t change :</p>
<pre class="brush: c-sharp">static void Main(string[] args)
{
    // Get the JIT out of the way
    GetAssignableTypes(typeof(IDisposable));

    Func&lt;Type, Type[]&gt; getAssignableTypes = GetAssignableTypes;
    getAssignableTypes = getAssignableTypes.AsMemoized();

    for (int i = 0; i &lt; 10; i++)
    {
        var sw1 = Stopwatch.StartNew();
        var r1 = getAssignableTypes(typeof(IDisposable)).Length;
        Console.WriteLine(sw1.Elapsed);
    }
}

public static Type[] GetAssignableTypes(Type source)
{
    var r =
        from asm in AppDomain.CurrentDomain.GetAssemblies()
        from type in asm.GetTypes()
        where type.IsAssignableFrom(source)
        select type;

    return r.ToArray();
}
</pre>
<p>The first call takes about 90ms on my machine, at the rest about 0.7&micro;s to execute.</p>
<p>As most of the time, this is tradeoff between memory and CPU. In this case, the result of the computation is store in a dictionary that is capture in the closure created in the AsMemoized method.</p>
<p>So, as long as the delegate produced by AsMemoized lives, so will the cached data.</p>
<p>&nbsp;</p>
<h2>Memoization storage and Immutability</h2>
<p>But there are multiple issues with this pattern of memoization, meaning that the memoized delegate needs to be stored somewhere.</p>
<p>The first issue is that the memoized delegate can be stored along with an instance that has a greater lifetime. This can work if the overall memoized dataset is not too important, because with a na&iuml;ve implementation as the one above, the data will not be discarded as long as the outer instance lives. The memoization can be reset, by re-creating the delegate, but it is&nbsp;very coarse.</p>
<p>The second one is about placing the memoized delegate along with the data that was used to create it. The memoized content will live as long as the data is alive, but there is a clear separation of concerns issue. Every time there is a new operation that needs to be performed on this data, then a new delegate has to be created to take care of this.</p>
<p>And finally, if there is a need for composite computation, which takes two or more instances, at this point, storing the delegate can be a memory-management challenge.</p>
<p>&nbsp;</p>
<h2>Next time</h2>
<p><a href="http://www.jaylee.org/post/2013/04/22/Immutable-Data-and-Memoization-in-CSharp-Part-2.aspx">Next time</a>, we'll see how memoization can be a little more friendly with the memory.</p>
<p>&nbsp;</p>
{% include imported_disclaimer.html %}
