---
layout: post
title: "F#, TryWith, Maybe and Umbrella"
date: 2008-12-06 13:49:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/12/06/fsharp-TryWith-Maybe-and-Umbrella.aspx", "/post/2008/12/06/fsharp-trywith-maybe-and-umbrella.aspx"]
author: jay
---
<!-- more -->
<div align="justify">
<em>Cet article est <a href="http://blogs.codes-sources.com/jay/archive/2008/12/06/fsharp-trywith-maybe-et-umbrella.aspx" target="_blank" title="F#, TryWith, Maybe et Umbrella">disponible en Fran&ccedil;ais</a>.</em> <br />
</div>
<div align="justify">
&nbsp;
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">I&#39;ve thrown myself a bit in the discovery of F#, and even though I do not intend to make it my first language, I intend to use techniques and features found in it and try to port them into C#. New additions in C# 3.0 make it a good target for functional concepts.<br />
<br />
There seem to be a consensus for the fact that F# is not a multi-purpose language, as C# is also not, for instance with the writing of parallel code. C# is not a perfect language for this, but F# seems to be. At the opposite, F# does not seem to be a language of choice for writing GUI code. For my part, and considering that F# if not really official, reusing concepts will be enough for now.</font>
<br />
</div>
<h3 align="justify">TryWith Extension</h3>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">
Using F#, I had to write this:
</font>
</p>
<div align="justify">
</div>
<p align="justify">
[code:c#]<br />
&nbsp;&nbsp;        let AllValidAssemblies = [<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            for file in Directory.GetFiles(@&quot;C:\Windows\Microsoft.NET\Framework\v2.0.50727&quot;, &quot;*.dll&quot;) -&gt; <br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp; System.Reflection.Assembly.LoadFile(file)<br />
&nbsp;&nbsp;&nbsp;        ]<br />
[/code]
</p>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">This code creates a list of assemblies that can be loaded in the current AppDomain. There is however an issue with the invocation of the Assembly.LoadFile method, because it raises an exception when the file is not loadable for some reason. This is a non-modifiable behavior, even though we would like to return a null instead of an exception.
</font>
</p>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">
To work around this, there is a feature in F# that can do this :<br />
</font>
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;        let EnumAllTypes = [<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            for file in Directory.GetFiles(@&quot;C:\Windows\Microsoft.NET\Framework\v2.0.50727&quot;, &quot;*.dll&quot;) -&gt; <br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp; try System.Reflection.Assembly.LoadFile(file) with _ -&gt; null<br />
&nbsp;&nbsp;&nbsp;        ]<br />
[/code]
</p>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">The point of the try/with block is to transform any exception into a null reference.
</font>
</p>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">
To transpose the creation of this list in C# with a LINQ query, the same problem arises. We must intercept the exception raised by LoadFile and convert it to a null reference.<br />
<br />
Here is the equivalent in C#, without the exception handling :
</font>
</p>
<div align="justify">
</div>
<p align="justify">
[code:c#]<br />
&nbsp;&nbsp;&nbsp;            var q = from file in Directory.GetFiles(@&quot;C:\Windows\Microsoft.NET\Framework\v2.0.50727&quot;, &quot;*.dll&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    let asm = Assembly.LoadFile(file)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select asm;<br />
[/code]
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">When LoadFile raises an exception, the execution of the request is interrupted, which is a problem.
</font>
</p>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">
Extension Methods can be of a great value here, and even though a normal method could do the trick, we can write this :</font><br />
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;        public static class Extensions<br />
&nbsp;&nbsp;&nbsp;        {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;        public static TResult TryWith&lt;TInstance, TResult&gt;(this TInstance instance, Func&lt;TInstance, TResult&gt; action)<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;        {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            try {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                return action(instance);<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            }<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            catch {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                return default(TResult);<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            }<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp; }<br />
&nbsp;&nbsp;&nbsp;        }<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">
The idea behind this method is to reproduce the behavior of the try/with F# construct. With this method, we can update the LINQ query into this :</font><br />
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;            var q = from file in Directory.GetFiles(@&quot;C:\Windows\Microsoft.NET\Framework\v2.0.50727&quot;, &quot;*.dll&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    let asm = file.TryWith(f =&gt; Assembly.LoadFile(f))<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select asm;<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">This is creates the same list the F# code does, with null references for assemblies that could not be loaded.<br />
<br />
The TryWith method can be overloaded to be a bit more flexible, like calling a method for a specific exception :</font>
<br />
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;        public static TResult TryWith&lt;TInstance, TResult, TException&gt;(<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp; &nbsp;&nbsp; this TInstance instance, Func&lt;TInstance, TResult&gt; action, <br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                Func&lt;TException, TResult&gt; exceptionHandler<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            )<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            where TException : Exception<br />
&nbsp;&nbsp;&nbsp;        {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            try {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                return action(instance);<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            }<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            catch (TException e) {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                return exceptionHandler(e);<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            }<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            catch {<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                return default(TResult);<br />
&nbsp;&nbsp;&nbsp;        &nbsp;&nbsp;&nbsp;            }<br />
&nbsp;&nbsp;&nbsp;        }<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">By the way, there is an interesting bug with this code. If we execute this :</font><br />
<br />
[code:c#]<br />
&nbsp; &nbsp; string value = null;<br />
&nbsp;&nbsp;&nbsp;            var res = value.TryWith(s =&gt; s.ToString(), (Exception e) =&gt; &quot;Exception&quot;);<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">The behavior is different depending on whether it is executed with or without the debugger with the x86 runtime. It seems that the code generator &quot;forgets&quot; to add the handler for the TException typed exception, which is annoying. This is not a big bug, mainly because it only appears when the x86 debugger is present. With the x64 runtime debugger, there is no problem though. For those interesting in this, <a href="https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=386652" title="Microsoft Connect">the bug is on Connect</a>.</font><br />
</p>
<div align="justify">
</div>
<h3 align="justify">Maybe Extension<br />
</h3>
<div align="justify">
</div>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">I also added recently in <a href="http://www.codeplex.com/umbrella" target="_blank" title="Umbrella - CodePlex">Umbrella</a> an extension named Maybe, which has a behavior rather similar to TryWith, but without the exceptions :</font><br />
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;        public static TResult Maybe&lt;TResult, TInstance&gt;(<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; this TInstance instance, <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Func&lt;TInstance, TResult&gt; function)<br />
&nbsp;&nbsp;&nbsp;        {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            return instance == null ? default(TResult) : function(instance);<br />
&nbsp;&nbsp;&nbsp;        }<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">The point of this method is to be able to execute code if the original value is not null. For instance :</font><br />
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;            object instance = null;<br />
&nbsp;&nbsp;&nbsp;            Console.WriteLine(&quot;{0}&quot;, instance.Maybe(o =&gt; o.GetType());<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">This allows the evaluation of the GetType call, only if &quot;instance&quot; is not null. With a method call like this, it is possible to write an &quot;if&quot; block, with inside a LINQ query, this because a bit more complex.</font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">The idea for the code is not new and is similar to the functional Monad concept. It has been covered numerous times, and an implementation more in line with F# can be found on <a href="http://weblogs.asp.net/podwysocki/archive/2008/10/13/functional-net-linq-or-language-integrated-monads.aspx" title="Functional .NET - LINQ or Language Integrated Monads? ">Matthew Podwysocki&#39;s Blog</a></font>.
</p>
<div align="justify">
</div>
<h3 align="justify">Pollution ?</h3>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">When we&#39;re talking about pollution linked to Extension Methods, we&#39;re talking about Intellisense pollution. We can quickly find ourselves dealing with a bunch of extensions that are useful in the context of the current code, which renders Intellisense unusable. With Umbrella, these two extensions are somehow polluting all types, because they are generic without constraints.<br />
<br />
Although these are only two very generic extensions, this can apply to almost any block of code, but they could find themselves better placed in an Umbrella Extension Point, rather than directly on every types.<br />
<br />
We could have this :</font>
<br />
<br />
[code:c#]<br />
&nbsp;&nbsp;&nbsp;            object instance = null;<br />
&nbsp;&nbsp;&nbsp;            Console.WriteLine(&quot;{0}&quot;, instance.Control().Maybe(o =&gt; o.GetType());<br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">But I have some issues with this approach : To be able to create an extension point the Control methd has to create an instance of the IExtensionPoint. This add a new instantiation during the execution, although the lambda also creates itself a hidden instance, we&#39;re counting anymore... There is also the fact that it lengthens the code line, but is only aesthetics. We&#39;ll see what pops out ...<br />
<br />
Anyway, it is interesting to see the impact the learning a new language has on the style of writing code with another language that one&#39;s been using for a long time...
</font>
</div>

{% include imported_disclaimer.html %}
