---
layout: post
title: "Working with Umbrella in .NET 3.5"
date: 2008-11-08 16:06:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/11/08/Working-with-Umbrella-in-NET-3-5", "/post/2008/11/08/working-with-umbrella-in-net-3-5"]
author: jay
---
<!-- more -->
<div align="justify">
<font face="trebuchet ms,geneva" size="2">If you&#39;ve been using .NET 3.5, and the new features that are provided by C# 3.0, and especially LINQ, you must have wondered why, oh why, there isn&#39;t an extension named ForEach on an IEnumerable&lt;T&gt;.</font><br />
<font face="trebuchet ms,geneva" size="2">
</font><br />
<font face="trebuchet ms,geneva" size="2">
Well, I still haven&#39;t figured that out, though it appears that it might have something to do with the fact that by nature an <a href="http://msdn.microsoft.com/en-us/library/018hxwa8.aspx" target="_blank" title="Action&lt;T&gt; Delegate - MSDN">Action&lt;T&gt;</a> is most of the time not &quot;pure&quot;, which means that it modifies some states, somewhere. I can&#39;t remember where I&#39;ve found that explanation, though it &quot;might&quot; make sense from a functional point of view.</font><br />
<font face="trebuchet ms,geneva" size="2">
</font><br />
<font face="trebuchet ms,geneva" size="2">
But you may wonder why, then, is there a <a href="http://msdn.microsoft.com/en-us/library/bwabdf9z.aspx" target="_blank" title="List&lt;T&gt;.ForEach">ForEach</a> method on List&lt;T&gt; ? Well, I don&#39;t know why there is that kind of consistency issue, but I know there is one library that somehow tries to fix this :</font>
</div>
<div align="justify">
&nbsp;
</div>
<div align="center">
<font face="trebuchet ms,geneva" size="5">
<a href="http://www.codeplex.com/umbrella" target="_blank" title="Umbrella on CodePlex">Umbrella</a></font><br />
</div>
<div align="center">
&nbsp;
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">
This library fills the gaps left by the BCL, and adds a whole bunch of new extension methods that eases the use of .NET 3.5 classes. This library is not a framework though, mainly because you don&#39;t need to redesign your whole application to use it.</font><br />
<font face="trebuchet ms,geneva" size="2">
</font><br />
<font face="trebuchet ms,geneva" size="2">
A few weeks ago during a presentation of Umbrella at the <a href="http://www.dotnetmontreal.com/" target="_blank" title=".NET Montreal User Group">.NET User Group of Montreal</a>, the creators Francois Tanguay and Erik Renaud from <a href="http://www.nventive.net/dnn/" target="_blank" title="nVentive">nVentive</a>, were asked the question <em>&quot;Where do we start ?&quot;</em>. This is a tough question to answer, because of the nature of umbrella, which &quot;plugs&quot; itself everywhere it can.</font>
</div>
<h1>Using Umbrella</h1>
<p>
<font face="trebuchet ms,geneva" size="2">Let&rsquo;s see a simple example:</font><br />
<br />
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; Enumerable<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .Range(20, 30)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .ForEach(<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (index, value) =&gt; &quot;Value #{0}={1}&quot;.InvariantCultureFormat(index, value)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );
</p>
<p>
[/code]
<font face="trebuchet ms,geneva" size="2"></font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">As you can see, using Umbrella requires most of the time the use of <a href="/admin/Pages/lambda%20expressions" target="_blank" title="Lambda Expressions">lambda expressions</a>. Here, this example is an answer to the question &quot;<em>How can I get the index of the enumerated value in a foreach statement ?&quot;</em>.</font><font face="trebuchet ms,geneva" size="2"></font>
</p>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">
Another extension provided by Umbrella is the Remove method on an ICollection by specifying a predicated to select elements to be removed. </font>
</div>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var values = new List&lt;int&gt;(Enumerable.Range(1, 10));<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; values.Remove(v =&gt; v &gt; 4);
</p>
<p>
[/code]
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Sure, you can write some code that does this, but you&#39;ll have to create a temporary list to store elements to be removed, then remove them from the collection.</font>
</p>
<div align="justify">
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">There is also an extension on IDictionary&lt;&gt; that simplifies the use of TryGetValue which requires an &ldquo;out&rdquo; argument, which is particularly annoying to write. Umbrella provides a TryGetValueOrDefault that either gives you the value for the key you requested, or default(TValue) if the key is not found.</font>
</div>
<div align="justify">
&nbsp;
</div>
<h2>
The Action.ToDisposable() extension method </h2>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">One particularly interesting extension is the ability to encapsulate an Action delegate into an anonymous disposable instance. Let&rsquo;s say that we need to profile the duration of a particular scope of code. We would need to write something like this :</font><br />
</div>
<p>
<br />
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; var w = Stopwatch.StartNew();<br />
&nbsp;&nbsp;&nbsp; for (int i = 0; i &lt; 1000; i++)&nbsp; { }<br />
&nbsp;&nbsp;&nbsp; Console.WriteLine(w.Elapsed);
</p>
<p>
[/code] <br />
<br />
</p>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">We would have to write the enclosing code to time for each portion of code we would need to profile. You could of course write a class that would do this on purpose :</font><br />
</div>
<p>
<br />
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; class ProfileScope : IDisposable<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Stopwatch w = Stopwatch.StartNew();<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; void IDisposable.Dispose()<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Console.WriteLine(w.Elapsed);<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code] <br />
</p>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">And it&rsquo;s being used like with a &ldquo;using&rdquo; statement :</font><br />
</div>
<p>
<br />
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; using (new ProfileScope())<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; for (int i = 0; i &lt; 1000; i++)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code] 
</p>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">This time, timing a block of code is somehow easier to write, however, there is a way to simplify the writing of the writing of the utility class by doing this:</font><br />
</div>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; static IDisposable CreateProfileScope()<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var w = Stopwatch.StartNew();<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return Actions.Create(() =&gt; Console.WriteLine(w.Elapsed)).ToDisposable();<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code]
</p>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">We can use here the ToDisposable extension to be able to call the specified action in the dispose method of some disposable class.</font><br />
<font face="trebuchet ms,geneva" size="2">
</font><br />
<font face="trebuchet ms,geneva" size="2">
The point of this code is to avoid exposing the inner details of the profiling code, and just expose a known and generic way for executing something at the end of a scope.</font><br />
</div>
<h2>Extension Points<br />
</h2>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">There is one drawback when using extension methods: They tend to &ldquo;pollute&rdquo; the original type. By pollute I mean that they appear in the IntelliSense window. Because it is easy to add extension methods, you end up having hundreds of new methods on a type, which make IntelliSense much less useable.</font>
</div>
<div align="justify">
<br />
<font face="trebuchet ms,geneva" size="2">
The Umbrella guys came up with the idea of Extension Points, which are a way to group extension methods the namespaces do for types.</font><br />
<font face="trebuchet ms,geneva" size="2">
</font><br />
<font face="trebuchet ms,geneva" size="2">
This is somehow a hack from a code perspective, but is rather ingenious from a usability perspective. For instance, extending the serialization applies to every type that exists, and placing all serialization extension methods directly on System.Object is not a good idea.</font><br />
<font face="trebuchet ms,geneva" size="2">
</font><br />
<font face="trebuchet ms,geneva" size="2">
Extensions points are used like this:</font><font face="trebuchet ms,geneva" size="2"></font><br />
</div>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; using (MemoryStream stream = new MemoryStream())<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; int a = 0;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a.Serialization().Binary(stream);<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // ...<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code] <br />
<br />
</p>
<div align="justify">
<font size="2">Serialization is an extension method placed on every object that returns an SerializationExtensionPoint instance, and the &ldquo;Binary&rdquo; method extends that type instead of System.Object.</font>
</div>
<div align="justify">
</div>
<p align="justify">
<font size="2">It is important to remember that Umbrella is for most of its code is not &quot;rocket science&quot;, as it probably contains code that you may already have partially developed for your own project. It&#39;s only the amount of small but useful utility methods that Umbrella provides that makes it worth using. </font>
</p>
<div align="justify">
</div>
<div align="justify">
<font size="2">
I&rsquo;ll be writing a few other posts about Umbrella; there are a lot of extensions in there that are great time savers.</font>
</div>
<p>
&nbsp;
</p>
<font size="2"></font>

{% include imported_disclaimer.html %}
