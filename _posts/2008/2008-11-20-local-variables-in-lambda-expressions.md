---
layout: post
title: "Local Variables in Lambda Expressions"
date: 2008-11-20 19:58:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/11/20/Local-Variables-in-Lambda-Expressions.aspx", "/post/2008/11/20/local-variables-in-lambda-expressions.aspx"]
author: jay
---
<!-- more -->
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><em>Cet article est <a href="http://blogs.codes-sources.com/jay/archive/2008/11/21/variables-locales-et-expressions-lambda.aspx" target="_blank" title="Variables Locales et Expressions Lambda">disponible en fran&ccedil;ais</a>. </em></font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">
&nbsp;
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">
After a quick chat with <a href="http://blogs.msdn.com/ericlippert/" target="_blank" title="Eric Lippert">Eric Lippert</a> about a <a href="http://blogs.codes-sources.com/jay/archive/2008/11/19/expressions-lambda-et-boucles-foreach.aspx" target="_blank" title="Expressions Lambda et Boucles ForEach - Jerome Laban">post</a>&nbsp; on the use in lambda expressions of a local variable declared in a foreach loop, Eric pointed me out that this piece of code :</font><br />
</div>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;    int a = 0;<br />
&nbsp;&nbsp;&nbsp;    Action action = () =&gt; Console.WriteLine(a);<br />
&nbsp;&nbsp;&nbsp;    action();
</p>
<p>
[/code] 
</p>
<div align="justify">
&nbsp;<font face="trebuchet ms,geneva" size="2">
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">
Is actually not expanded by the compiler to this code :</font><br />
</div>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp; [CompilerGenerated]<br />
&nbsp;&nbsp;&nbsp;   private sealed class &lt;&gt;c__DisplayClass1<br />
&nbsp;&nbsp;&nbsp;   {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       public int a;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       public void &lt;Main&gt;b__0()<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;          Console.WriteLine(this.a);<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       }<br />
&nbsp;&nbsp;&nbsp;   }<br />
<br />
&nbsp;&nbsp;&nbsp; void Main()<br />
&nbsp;&nbsp;&nbsp; {<br />
&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; int a = 0;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            var display = new &lt;&gt;c__DisplayClass1();<br />
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; display.a = a;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var action = new Action(display.&lt;Main&gt;b__0);<br />
<br />
&nbsp;&nbsp; &nbsp; &nbsp;            action();<br />
&nbsp;&nbsp;&nbsp; }
</p>
<p>
[/code] 
</p>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">&nbsp;
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">
I made the assumption that a local variable was simply copied in the &quot;DisplayClass&quot; if it is not used after the creation of the lambda, which&nbsp; is not the case.
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><br />
If we take this slightly different sample :
</font>
</div>
<p>
<br />
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;      int a = 0;<br />
&nbsp;&nbsp;&nbsp;    Action action = () =&gt; Console.WriteLine(a);<br />
&nbsp;&nbsp;&nbsp;    a = 42;<br />
&nbsp;&nbsp;&nbsp;    action();
</p>
<p>
[/code]
</p>
<div align="justify">
&nbsp;
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">My assumption would have made this code display &quot;0&quot;. This is correct because lambda expressions (and anonymous methods) &quot;capture&quot; the variable and not the value; The execution must display 42.
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><br />
Actually, this latter piece of code is expanded like this:</font><br />
</div>
<p>
[code:c#]
</p>
<p>
&nbsp;&nbsp;&nbsp;     var display = new &lt;&gt;c__DisplayClass1();<br />
<br />
&nbsp;&nbsp;&nbsp;    display.a = 0;<br />
<br />
&nbsp;&nbsp;&nbsp; var action = new Action(display.&lt;Main&gt;b__0);<br />
<br />
&nbsp;&nbsp;&nbsp;    display.a = 42;<br />
<br />
&nbsp;&nbsp;&nbsp;    action();
</p>
<p>
[/code] 
<br />
</p>
<div align="justify">
&nbsp;
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">We can see that in fact, the variable that was previously local, on the stack, has been &quot;promoted&quot; as a field memberr of the &quot;Display Class&quot;. This means that all references to this &quot;local&quot; variable, inside or outside of the lambda, are replaced to point to the current instanc e of the &quot;DisplayClass&quot;.
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><br />
This is quite simple actually, but we can feel that the &quot;magic&quot; behind the C# 3.0 syntactic sugar is the result of a lot of thinking !
</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><br />
I will end this post by a big thanks to Eric Lippert, who took the time to answer me, even though he&#39;s probably under heavy load with the developement of C# 4.0. (With the contravariance of generics, yey !)
</font>
</div>
<span style="color: Black; background-color: transparent; font-family: Courier New; font-size: 11px; font-weight: normal"></span>

{% include imported_disclaimer.html %}
