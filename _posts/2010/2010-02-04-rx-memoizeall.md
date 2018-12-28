---
layout: post
title: "Reactive Framework: MemoizeAll"
date: 2010-02-04 18:21:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/02/04/Rx-MemoizeAll.aspx", "/post/2010/02/04/rx-memoizeall.aspx"]
author: jay
---
<!-- more -->
<p><em>Cet article est <a href="http://blogs.codes-sources.com/jay/archive/2010/02/04/rx-framework-memoizeall.aspx">disponible en francais</a>.</em></p>
<p>For some time now, with the release of the <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx">Rx Framework</a> and the reactive/interactive programming, some new features were highlighted through a <a href="http://community.bartdesmet.net/blogs/bart/archive/2010/01/07/more-linq-with-system-interactive-functional-fun-and-taming-side-effects.aspx">very good article of Bart De Smet</a> dealing with System.Interactive and the &ldquo;lazy-caching&rdquo;.</p>
<p>When using LINQ, one can find two sorts of operators: The &ldquo;lazy&rdquo; operators that take elements one by one and forward them when they a requested (<a href="http://www.google.com/url?sa=t&amp;source=web&amp;ct=res&amp;cd=1&amp;ved=0CAcQFjAA&amp;url=http%3A%2F%2Fmsdn.microsoft.com%2Fen-us%2Flibrary%2Fsystem.linq.enumerable.select.aspx&amp;ei=LlRqS6ehH8WUtgeOoPTmBg&amp;usg=AFQjCNELVAkTl8_nMqU6BskLt2REQo6daQ&amp;sig2=4LGGLT7UllGauB0MLhIsWw">Select</a>, Where, SelectMany, First, &hellip;), and the operators that I would call &ldquo;Rendez-vous&rdquo; for which the entirety of the elements of the enumerators need to be enumerated (Count, ToArray/ToList, OrderBy, &hellip;) to produce a result.</p>
<p>&nbsp;</p>
<h3>&ldquo;Lazy&rdquo; Operators</h3>
<p>Lazy operators are pretty useful as they offer a good performance when it is not required to enumerate all the elements of an enumerable. This can also be useful when it may take a very long time to enumerate each element of an enumerator, and that we only want to get the first few elements.</p>
<p>For instance this : <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [code:c#]</p>
<p><code>static IEnumerable&lt;int&gt; GetItems()     <br />{      <br />&nbsp;&nbsp;&nbsp; for (int i = 0; i &lt; 5; i++)      <br />&nbsp;&nbsp;&nbsp; {      <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Console.WriteLine(i);      <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yield return i + 1;      <br />&nbsp;&nbsp;&nbsp; }      <br />}</p>
<p>static void Main()     <br />{      <br />&nbsp;&nbsp; Console.WriteLine(GetItems().First());      <br />&nbsp;&nbsp; Console.WriteLine(GetItems().First());<br />}</p>
<p>[/code]</code></p>
<p>Will output :</p>
<p><code>[code:c#]<br />0     <br /> 1<br />0     <br /> 1<br /><br />[/code]</code></p>
<p>Only the first element of the enumerator will be enumerated from GetItems().</p>
<p>However, these operators expose a behavior that is important to know about: Each time they are enumerated, they also enumerate their source again. That could either be a advantage (Enumerating multiple times a changing source) or a problem (enumerating multiple times a resource intensive source).</p>
<p>&nbsp;</p>
<h3>&ldquo;Rendez-vous&rdquo; Operators</h3>
<p>These operators are also interesting because they force the enumeration of all the elements of the enumerable, and in the particular case of ToArray, this allows the creation of an immutable version of the content of the enumerable. These are useful in conjunction with lazy operators to prevent them to enumerate their source again, when enumerated multiple times.</p>
<p>If we the previous sample, and update it a bit:</p>
<p><code>[code:c#]<br />static void Main()     <br />{<br />&nbsp;&nbsp; var items = GetItems().ToArray();</p>
<p>&nbsp;&nbsp; Console.WriteLine(items.Count());      <br />&nbsp;&nbsp; Console.WriteLine(items.Count());      <br />}<br /><br />[/code]</code></p>
<p>We get this result :</p>
<p><code>[code:c#]<br />0     <br />1      <br />2      <br />3      <br />4      <br />5<br />5<br /><br />[/code]</code></p>
<p>Because Count() needs to know all the elements of the source enumerator to determine the count.</p>
<p>These operators also enumerate their source with each use, but using ToArray/ToList prevents their result to enumerate the source again.</p>
<h3>The case of multiple enumerations<br /></h3>
<p>A concrete example of the problem posed by the multiple enumerations is the <a href="http://www.markhneedham.com/blog/2010/02/01/functional-c-writing-a-partition-function/">creation of an enumerable partitionning operator</a>. In this example, we can see that the enumerable passed as the source is used by to "Where" different operators, which implies that the source enumerable will be enumerated twice. Storing the whole content of the source enumerable by means of a ToArray/ToList is possible, but that would be a possible waste of resource, mainly because we can't know if the output enumerable will be enumerated completely (If that is possible, as in the case of an infinite enumerable, ToArray is not applicable).</p>
<p>An intermediate operator between "Lazy" and "Rendez-vous" would be useful.</p>
<h3>EnumerableEx.MemoizeAll</h3>
<p>The EnumerableEx class brings us an extension, MemoizeAll (built from the <a href="http://en.wikipedia.org/wiki/Memoization">Memoization</a> concept), that is just the middle ground we're looking for, and will cache elements from the source enumerator when they are requested. A sort of "lazy" ToArray.</p>
<p>If we take the example of Mark Needham, we would modify it like this :</p>
<p style="padding-left: 30px;">[code:c#]<br />var evensAndOdds = Enumerable.Range(1, 10)<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .MemoizeAll()<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .Partition(x =&gt; x % 2 == 0);<br /><br />[/code]</p>
<p>In this example, the MemoizeAll does not have a real benefit on the performance side, since Enumerable.Range is not a very expensive operator. But in the case where the source of the "Partition" operator would be a most expensive enumerable, like a Linq2Sql query, the lazy caching could be very effective.</p>
<p><a href="http://www.markhneedham.com/blog/2010/02/01/functional-c-writing-a-partition-function/#comment-31312">One of the comments</a> suggests that a GroupBy based implementation could be written, but this operator also evaluates the source operator when a group is enumerated. The MemoizeAll is then again appropriate for better performance, but as always, this is a tradeoff between processing and memory.</p>
<p>By the way, Bart de Smeth discusses the part of the elimination of side effects linked the multiple enumeration of enumerables by using Memoize and MemoizeAll, which is not really an issue in the previous example, but is nonetheless a very interesting subject.</p>
<p>&nbsp;</p>
<h3>.NET 4.5 ?</h3>
<p>On a side note, I find regrettable that the EnumerableEx extensions did not make their way in .NET 4.0... They are very useful, and not very complex. They may have arrived too late in the development cycle of .NET 4.0... Maybe in .NET 4.5 :)</p>
{% include imported_disclaimer.html %}
