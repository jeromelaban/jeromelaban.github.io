---
layout: post
title: "Toying around with F# Queries, Rx, Portables Libraries, Windows [Phone] 8 and the Zip operator"
date: 2013-01-02 22:42:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2013/01/02/Toying-around-with-FSharp-Queries-Rx-Portables-Libraries-Windows-Phone-8-and-the-Zip-operator", "/post/2013/01/02/toying-around-with-fsharp-queries-rx-portables-libraries-windows-phone-8-and-the-zip-operator"]
author: jay
---
<!-- more -->
<p>Agreed, that&rsquo;s a lot of keywords. Yet, they fit one another in a very interesting way.</p>
<p>I find more and more that F#, particularly regarding the way my development trends are going, is getting more and more of a fit regarding the immutability and flexibility needs.</p>
<p>Given that, I thought I&rsquo;d give a try at running some F# Query Expressions using custom Rx operators, on Windows Phone 8, using Portable Libraries.</p>
<p>[more]</p>
<h2>Getting F# to run on Windows Phone 8</h2>
<p>Interestingly, the F# team made a <a href="http://msdn.microsoft.com/en-us/library/gg597391.aspx">portable library</a>&nbsp;version of their runtime, so that it can run under the intersection of .NET 4.5, Windows Store Apps and Silverlight 5 frameworks. Windows Phone 8 projects accept this intersection, which make F# available to work with.</p>
<p>That was easy.</p>
<p>&nbsp;</p>
<h2>Getting Rx to run on F#&rsquo;s PLib intersection</h2>
<p>That one is a bit more tricky. Out of the box, <a href="http://msdn.microsoft.com/en-us/data/gg577609.aspx">Rx</a>&nbsp;supports a different PLib frameworks intersection, being .NET 4.5 and Windows Store Apps.</p>
<p>Fortunately, the <a href="http://rx.codeplex.com/">Rx team published the source code</a> of the library, making this experiment a lot easier to do.</p>
<p>I&rsquo;ve updated a bit the Rx sources to match a set of new defines :</p>
<pre class="brush: xml">&lt;DefineConstants&gt;
$(DefineConstants);NO_RXINTERFACES;PREFER_ASYNC;
HAS_APTCA;NO_HASHSET;NO_CDS;NO_SEMAPHORE;NO_STOPWATCH;
NO_REMOTING;NO_SERIALIZABLE;NO_THREAD;PLIB
&lt;/DefineConstants&gt;</pre>
<p>I&rsquo;ve also changed the PLib profile&nbsp;from Profile7 and Profile47, to match the same profile as the F# portable library. There&rsquo;s actually a new define I introduced, NO_SEMAPHORE, because that new profile does not support the SemaphoreSlim class.</p>
<p>I had to make a small update to the code to make that code Semaphore-less, if you&rsquo;re interested let me know, I&rsquo;ll share that.</p>
<p>&nbsp;</p>
<h2>Playing with the F# Query Expressions and Rx</h2>
<p>I&rsquo;ve come across an interesting article from <a href="http://mnajder.blogspot.ca/2011/09/when-reactive-framework-meets-f-30.html">Marcin Najder about F# 3.0 and Query Expressions</a> using the Reactive Extensions and this got me started.</p>
<p>The interesting side is to be able to have comprehensions that simplify the writing of queries that contain LINQ or Rx operators that may not be supported by C#, such as IObservable.Take() :</p>
<pre>type Dummy() = 
 
        let rxquery = new RxQueryBuiler()
        let obs = new ObsBuiler()
 
        let values =
            obs {
                for value in [0..15] do
                    yield value
            }
           
        member this.Foo = 
            rxquery {
                for value in values do
                where (value &gt; 1)
                select value
                take 10 into trimmed
                select trimmed 
            }
    </pre>
<p>The Foo property can be used directly from a C# program, and observed like any other IObservable instance.</p>
<h2><span style="font-family: Calibri;">The ZipLike custom operator</span></h2>
<p>With Rx and their enumerable counterparts (such as Linq), the <a href="http://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.zip(v=VS.103).aspx">Zip operator</a> allows to get two streams of values and produce another using each tuple of values.</p>
<p>The problem with this operator is that in C# it cannot be written using the &ldquo;from xx in yy&rdquo;&nbsp;LINQ syntax, because there is no keyword for it, and the language does not allow the creation of new ones (though some are <a href="http://social.msdn.microsoft.com/Forums/en/rx/thread/155cd697-5341-465c-98c3-268793c4a653">trying to get that into future C# versions</a>).</p>
<p>There&nbsp;are no examples or documentation anywhere to use a custom Zip operator (using the <a href="http://msdn.microsoft.com/en-us/library/hh289776.aspx">CustomOperation.IsLikeZip</a>&nbsp;property), so digging a bit into the F# test suite got me started on how to use it.</p>
<p>So, instead of writing a Zip like this :</p>
<pre>o1.Zip(o2, (lv, rv) =&gt; new Tuple(lv, rv));</pre>
<pre>&nbsp;</pre>
<p>It can be written like this in F#, by defining a custom zip operator :</p>
<pre>    let values2 =
        obs {
            for value in [5..20] do
                yield value
        }</pre>
<pre>    member this.Combined = 
        rxquery {
            for test in values do
            zip test2 in values2
            select (test, test2)
        }</pre>
<pre>&nbsp;</pre>
<p>This returns a simple tuple for the combined values.</p>
<p>But this also has the advantage of providing what the <a href="http://msdn.microsoft.com/en-us/library/bb383976(v=VS.100).aspx">C# let keyword</a> does, by creating query variables :</p>
<pre>        member this.Combined = 
            rxquery {
                for test in values do
                zip test2 in values2
                let values = (test * 2, test2)
                where (fst(values) &gt; 5)
                select (test, test2)
            }
    </pre>
<p>There is no need to create temporary anonymous types to forward the state of the query to downstream Rx operators manually.</p>
<p>Cool stuff, really.</p>
{% include imported_disclaimer.html %}
