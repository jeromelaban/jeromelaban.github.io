---
layout: post
title: "A C# Traverse extension method, with an F# detour"
date: 2009-05-17 09:10:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2009/05/17/A-csharp-Traverse-extension-method-with-an-Fsharp-detour.aspx", "/post/2009/05/17/a-csharp-traverse-extension-method-with-an-fsharp-detour.aspx"]
author: jay
---
<!-- more -->
<p>
<em><a href="http://blogs.codes-sources.com/jay/archive/2009/05/18/extension-method-traverse-et-un-detour-par-fsharp.aspx" target="_blank">Cet article est disponible en Fran&ccedil;ais. </a></em>
</p>
<h1>The Traverse extension method in C# <br />
</h1>
<p>
Occasionally, you&#39;ll come across data structures that take the form of single linked lists, like for instance the MethodInfo class and its GetBaseDefinition method.
</p>
<p>
Let&#39;s say for a virtual method you want, for a specific type, discover which overriden method in the hierarchy is marked with a specific attribute.
I assume in this example that the expected attribute is not inheritable. 
</p>
<p>
You could implement it like this :
</p>
<p>
[code:c#]&nbsp;&nbsp;&nbsp;&nbsp; 
</p>
<p>
&nbsp;&nbsp;&nbsp; private static MethodInfo GetTaggedMethod(MethodInfo info)<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MethodInfo ret = null;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; do<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var attr = info.GetCustomAttributes(typeof(MyAttribute), false) as MyAttribute[];<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (attr.Length != 0)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return info;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ret = info;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; info = info.GetBaseDefinition();<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; while (ret != info);<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return null;<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code] 
<br />
</p>
<p>
This method has two states variables and a loop, which makes it a bit harder to stabilize. This is a method that could easily be expressed as a LINQ query, but (as far as I know) there is no way to make a enumeration of a data structure which is part of a linked list.
</p>
<p>
To be able to do this, which is &quot;traverse&quot; a list of objects of the same type that are linked from one to the next, an extension method containing a generic iterator can be written like this :
</p>
<p>
[code:c#]
<br />
</p>
<p>
&nbsp;&nbsp;&nbsp; public static class Extensions<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; public static IEnumerable&lt;T&gt; Traverse&lt;T&gt;(this T source, Func&lt;T, T&gt; next)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; while (source != null)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yield return source;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; source = next(source);<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br />
&nbsp;&nbsp;&nbsp; } 
</p>
<p>
[/code] 
<br />
</p>
<p>
This is a really simple iterator method, which calls a method to get the next element using the current element and stops if the next value is null.
</p>
<p>
It can be used easily like this, using the GetBaseDefinition example :
</p>
<p>
[code:c#]
<br />
</p>
<p>
&nbsp;&nbsp; var methodInfo = typeof(Dummy).GetMethod(&quot;Foo&quot;); 
</p>
<p>
&nbsp;&nbsp; IEnumerable&lt;MethodInfo&gt; methods = methodInfo.Traverse(m =&gt; m != m.GetBaseDefinition() ? m.GetBaseDefinition() : null);
</p>
<p>
[/code] 
<br />
</p>
<p>
Just to be precise, the lambda is not exactly perfect, as it is calling GetBaseDefinition twice. It can definitely be optimised a bit.
</p>
<p>
Anyway, to go back at the first example, the GetTaggedMethod function can be written as a single LINQ query, using the Traverse extension :
</p>
<p>
[code:c#]<br />
</p>
<p>
&nbsp;&nbsp;&nbsp; private static MethodInfo GetTaggedMethod(MethodInfo info)<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var methods = from m in methodInfo.Traverse(m =&gt; m != m.GetBaseDefinition() ? m.GetBaseDefinition() : null)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let attributes = m.GetCustomAttributes(typeof(MyAttribute), false)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where attributes.Length != 0<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select m;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return methods.FirstOrDefault();<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code] 
<br />
</p>
<p>
I, for one, find this code more readable... But this is a question of taste :)
</p>
<p>
Nonetheless, the MethodInfo linked-list is not the perfect example, because the end of the chain is not a null reference but rather the same method we&#39;re testing. Most of the time, a chain will end with a null, which is why the Traverse method uses null to end the enumeration. I&#39;ve been using this method to perform queries on a hierarchy of objects that have parent objects of the same type, and the parent&#39;s root set to null. It has proven to be quite useful and concise when used in a LINQ query.
</p>
<h1>An F# Detour <br />
</h1>
<p>
As I was here, I also tried to find out what an F# version of this code would be. So, with the help of recursive functions, I came up with this :
</p>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;
let rec traverse(m, n) =<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let next = n(m)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if next = null then<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [m] <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; else<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp; [m] @ traverse(next, n)
</p>
<p>
[/code]<br />
</p>
<p>
The interesting part here is that F# does not require to specify any type. &quot;m&quot; is actually an object, and &quot;n&quot; a (obj -&gt; obj) function, but returns a list of objects. And it&#39;s used like this :
</p>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;
let testMethod = typeof&lt;Dummy&gt;.GetMethod(&quot;Foo&quot;)<br />
<br />
&nbsp;&nbsp;&nbsp;
for m in&nbsp; traverse(testMethod, fun x -&gt; if x = x.GetBaseDefinition() then null else x.GetBaseDefinition()) do<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Printf.printfn &quot;%s.%s&quot; m.DeclaringType.Name m.Name
</p>
<p>
[/code] 
<br />
</p>
<p>
Actually, the F# traverse method is not exactly like the C# traverse method, because it is not an extension method, and it is not lazily evaluated. It is also a bit more verbose, mainly because I did not find an equivalent of the ternary operator &quot;?:&quot;. 
</p>
<p>
After digging a bit in the F# language spec, I found out it exists an somehow equivalent to the yield keyword. It is used like this :
</p>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;
let rec traverse(m, n) =<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; seq {<br />
&nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; let next = n(m)<br />
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp; if next = null then<br />
&nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yield m<br />
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp; else<br />
&nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yield m<br />
&nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yield! traverse(next, n)<br />
&nbsp; &nbsp; &nbsp;&nbsp; }
</p>
<p>
[/code]
<br />
</p>
<p>
It is used the same way, but the return value is not a list anymore but a sequence.
</p>
<p>
I also find interesting that F# is able to return tuples out of the box, and for my attribute lookup, I&#39;d have the method and I&#39;ll also have the attribute instance that has been found. Umbrella also defines tuples useable from C#, but it&#39;s an addon. 
</p>
<p>
F# is getting more and more interesting as I dig into its features and capabilities... 
</p>

{% include imported_disclaimer.html %}
