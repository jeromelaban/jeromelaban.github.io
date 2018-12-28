---
layout: post
title: "Immutable Data and Memoization in C#, Part 2"
date: 2013-04-22 21:10:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2013/04/22/Immutable-Data-and-Memoization-in-CSharp-Part-2.aspx", "/post/2013/04/22/immutable-data-and-memoization-in-csharp-part-2.aspx"]
author: jay
---
<p><em>tl;dr: Memoization can be associated with the <a href="http://msdn.microsoft.com/en-us/library/dd287757.aspx">ConditionalWeakTable</a>&nbsp;class, which allows the addition of memoized computation results to immutable types. This makes the memoized results live as long as the instances that was used to create it.</em></p>
<p>&nbsp;</p>
<p><a href="http://www.jaylee.org/post/2013/04/18/Memoization-and-Immutable-data-in-CSharp-Part-1.aspx">In the first part of this article</a>, we discussed a bit the Memoization pattern in C#. In this second part, we will discuss how to alleviate the memory management issue for memoized computation results.</p>
<!-- more -->
<h2>ConditionalWeakDictionary to the rescue</h2>
<p>In .NET 4.0 &ndash; quite a while ago now&nbsp;&ndash; a new class was added, the <a href="http://msdn.microsoft.com/en-us/library/dd287757.aspx">ConditionalWeakTable</a>,&nbsp;to help in the creation of dynamic languages based on the <a href="http://msdn.microsoft.com/en-us/library/dd233052.aspx">DLR</a>, where there was a need to be able to attach data to existing type instances, much like a <a href="http://blogs.msdn.com/b/ericlippert/archive/2009/10/05/why-no-extension-properties.aspx">fictional</a>&nbsp;<a href="http://stackoverflow.com/questions/619033/does-c-sharp-have-extension-properties">extension property</a> could be. This topic is not covered much, and since it has to do with the GC, it is often misunderstood.</p>
<p>The idea is pretty simple: It is a dictionary that takes an type instance as a key, and a value associated to it. The key is stored as a weak reference, meaning that the data is held as a hard-reference as long as the key lives. When the key is collected by the GC, the hard-link is removed, making the data available for collection if it&rsquo;s not referenced anywhere else.</p>
<p>Here&rsquo;s how to use it:</p>

<pre class="brush: c-sharp">ConditionalWeakTable&lt;object, object&gt; table 
    = new ConditionalWeakTable&lt;object, object&gt;();

var o1 = new object();
var o2 = new object();
var r2 = new WeakReference(o2);

table.Add(o1, o2);

//Release the reference so it's only kept by the table
o2 = null;

GC.Collect(2);
Console.WriteLine("{0}/{1}", r2.IsAlive, table.GetOrCreateValue(o1));

// Release the key instance, hence releasing o2.
o1 = null;

GC.Collect(2);
Console.WriteLine("{0}", r2.IsAlive);

//
// Output:
//
//      True/System.Object
//      False
</pre>
<p>Implementing this class without having a privileged access to the GC is <em>pretty</em> tricky. You quickly end up having to perform a weak reference dictionary that you&rsquo;ll need to cleanup by hand and you&rsquo;ll need to handle circular references properly. This can end up messy.</p>
<p>&nbsp;</p>
<h2>Composing the Memoization with the ConditionalWeakTable</h2>
<p>Knowing this, it is pretty easy to see that it may be possible to create a version of the <em>AsMemoized()</em> helper that stores its computation results somewhere else, in a memory-friendly fashion.</p>
<p>The result of a computation can be associated with the data that was used to create it, and kept alive as long as the first source lives.</p>
<p>Here&rsquo;s how this can be implemented :</p>
<pre class="brush: c-sharp">using Storage = System.Collections.Concurrent.ConcurrentDictionary&lt;object, object&gt;;

public static partial class FuncExtensions
{
    private static ConditionalWeakTable&lt;object, Storage&gt; _weakResults =
        new ConditionalWeakTable&lt;object, Storage&gt;();

    public static Func&lt;TParam, TResult&gt; 
        AsWeakMemoized&lt;TSource, TResult, TParam&gt;(
            this Func&lt;TSource, TParam, TResult&gt; selector,
            TSource source
        )
    {
        return param =&gt;
        {
            // Get the dictionary that associates delegates 
            // to a parameter, on the specified source
            var values = _weakResults.GetOrCreateValue(source);

            object res;

            var key = new { selector, param };

            // Get the result for the combination source/selector/param
            bool cached = values.TryGetValue(key, out res);

            if (!cached)
            {
                res = selector(source, param);

                values[key] = res;
            }

            return (TResult)res;
        };
    }
        
    // Since is not possible to implicitly make a Func&lt;T,U&gt; out
    // of a method group, let's use the source as a function type inference.
    public static TResult ApplyMemoized&lt;TSource, TResult, TParam&gt;(
        this TSource source, Func&lt;TSource, TParam, TResult&gt; selector,
        TParam param
    )
    {
        return selector.AsWeakMemoized(source)(param);
    }
}
</pre>
<p>And is used like this :</p>
<pre class="brush: c-sharp">class Program
{
    static void Main(string[] args)
    {
        var data = new Data(42);
        var wr = new WeakReference(data.DoStuff(42));
        var wr2 = new WeakReference(data.DoStuff(42));

        GC.Collect(2);
        Console.WriteLine(wr.IsAlive);

        data = null;

        GC.Collect(2);
        Console.WriteLine(wr.IsAlive);
    }
}

// An immutable type
public class Data
{
    public Data(int value) { Value = value; }
    ~Data() { Console.WriteLine("~Data({0})", Value); }

    public int Value { get; private set; }
}

public static class DataExtensions
{
    public static Data DoStuff(this Data data, int someValue)
    {
        return data.ApplyMemoized(InternalDoStuff, someValue);
    }

    private static Data InternalDoStuff(Data data, int someValue)
    {
        Console.WriteLine("Computing {0}", someValue);
        return new Data(data.Value + someValue);
    }
}

//
// Output: 
//      Computing 42
//      True
//      False
//      ~Data(84)
//      ~Data(42)
//
</pre>
<p>The ConditionalWeakDictionary takes the instance to attach the result to, a function to memoize, and will return a new delegate that will cache the results for a combination of the source and a parameter.</p>
<p>Note that to reduce the verbosity of the code, since it's not possible to use a method group as a delegate, the ApplyMemoized method&nbsp;is used to take advantage of the few places where the delegate type inference is available.</p>
<p>&nbsp;</p>
<h2>AsWeakMemoized, the caveats</h2>
<p>This helper is far from perfect, and it has to do with the implementation of the ConditionalWeakTable implementation.</p>
<p>&nbsp;</p>
<h3>Referential Transparency</h3>
<p>The documentation says <em>"Two keys are equal if passing them to the Object.ReferenceEquals method returns true.". </em>This means that the even though two instances of the same data may be equal in their value, through <a href="http://msdn.microsoft.com/en-us/library/ms131187.aspx">IEquatable</a> or GetHashCode/Equals, they will be considered as different. This means that through this helper, type instances&nbsp;are not <a href="http://en.wikipedia.org/wiki/Referential_transparency_(computer_science)">referentially transparent</a>.</p>
<p>When&nbsp;using AsWeakMemoized, this is an implementation&nbsp;detail that needs to be taken into account. The same computation may be performed twice because the computation will be memoized on a <a href="http://msdn.microsoft.com/en-us/library/bb548891.aspx">projection</a> of an immutable instance.</p>
<p>I mentioned earlier that implementing a ConditionalWeakTable can be tricky, but implementing referential transparency could be a reason to do it. This would allow for the memoizer to&nbsp;compare values instead of references.</p>
<p>But it would also&nbsp;make memory management a bit more complex,&nbsp;regarding&nbsp;the owner of the computation result. For instance, there is the question of&nbsp;how long a&nbsp;referentially transparent result for a single value should be&nbsp;kept alive, even if the original reference that was used to create the resulting value has been collected.</p>
<p>You might want to go down that road :)</p>
<p>&nbsp;</p>
<h3>Static delegates</h3>
<p>You'll notice that the memoization is using the delegate as a key for the storage. This means that if the provided delegate&nbsp;changes, either because it is a lambda that creates a closure, or because the delegate is created from an instance method, the memoization will not happen.</p>
<p>At the very worse, the memoization will do nothing, or maybe simply allocate a few more bytes every time for the storage of the result.</p>
<p>&nbsp;</p>
<h3>Occasional Duplicate Work</h3>
<p>Finally, you'll notice that there are no locks in this implementation. This means that the computation for a set of inputs may occasionally&nbsp;be executed multiple times in case of a race condition. While this can be a performance&nbsp;issue on systems that do not have enough CPUs (&lt; 4), on greater multicore CPUs,&nbsp;it is acceptable to have the CPUs occasionally compute the same data even if it is discarded, just for the sake of avoiding acquiring&nbsp;locks.</p>
<p>&nbsp;</p>
<h2>Final words</h2>
<p>I personally think that Functional Programming concepts such as Immutability and method purity,&nbsp;along with techniques such as Memoization, are part of the answer to harness the power of multi-core CPUs. CPU <a href="https://en.wikipedia.org/wiki/Scalability">Scaling out</a> is not going away, so we'd better make good use of this hardware, using&nbsp;the appropriate development&nbsp;tools.</p>
<p>&nbsp;</p>
<p><em></em>&nbsp;</p>
{% include imported_disclaimer.html %}
