---
layout: post
title: "A bug in VS2008 Code Analysis, Generics normal and nested classes"
date: 2008-04-07 19:12:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/04/07/A-bug-in-Code-Analysis-Generics-normal-and-nested-classes.aspx", "/post/2008/04/07/a-bug-in-code-analysis-generics-normal-and-nested-classes.aspx"]
author: jerome
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2">I&#39;ve found a few days ago a small bug in the former &quot;FxCop&quot; now renamed Code Analysis part of Visual Studio 2008.</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">While compiling this little piece of code :</font> 
</p>
<p>
[code:c#]<br />
public class Dummy&lt;T&gt; where T : IDisposable<br />
&nbsp;&nbsp;&nbsp;&nbsp;{<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public T Test<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new NestedDummy&lt;T&gt;(default(T));<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;class NestedDummy&lt;U&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public NestedDummy(U item)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.Value = item;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public U Value { get; private set; }<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br />
&nbsp;&nbsp;&nbsp;&nbsp;}<br />
[/code] 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Which is a trimmed down version of the actual&nbsp;code, I saw a lot of errors like this :</font><br />
<br />
<font face="courier new,courier">MSBUILD : error : CA0001 : Rule=Microsoft.Reliability#CA2001, Target=ConsoleApplication1.Dummy`1+NestedDummy`1.#.ctor(!1) : The following error was encountered while reading module &#39;ConsoleApplication1&#39;: Could not resolve member reference: ConsoleApplication1.Dummy`1&lt;type parameter.T&gt;+NestedDummy`1&lt;type parameter.U&gt;::set_Value.<br />
</font><font face="courier new,courier"><br />
<font face="trebuchet ms,geneva" size="2">This means that for some reason, the Code Analysis tool is unable to parse the metadata to check for some analysis rule. This is not a blocking bug since it does not prevent the build from ending properly, but it displays a lot of error messages, which can be disturbing.<br />
<br />
To fix to, I found two solutions : Either move the nested class out of its parents class, or remove the&nbsp;generic constraint on the parent class.</font></font> 
</p>
<p>
<font face="courier new,courier"><font face="trebuchet ms,geneva" size="2">I <a href="https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=336142&amp;wa=wsignin1.0" target="_blank" title="Microsoft Connect">posted the bug on Microsoft Connect</a>, and I was pleasantly surprised to see&nbsp;that it has already been processed and <a href="http://blogs.msdn.com/fxcop/" title="FxCop Blog">David Kean</a> from Microsoft&nbsp;wrote that the fix will be available in the next Service Pack of Visual Studio 2008.</font></font> 
</p>
<p>
<font face="courier new,courier"><font face="Trebuchet MS" size="2">Not a big issue but still, nice to see that Connect has an impact.</font></font> 
</p>

{% include imported_disclaimer.html %}
