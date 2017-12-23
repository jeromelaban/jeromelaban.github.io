---
layout: post
title: "C# 2.0, Closures and Anonymous Delegates"
date: 2005-04-04 11:33:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2005/04/04/C-202c-Closures-and-Anonymous-Delegates", "/post/2005/04/04/c-202c-closures-and-anonymous-delegates"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">I was looking around the web about new features in C# 2.0, and I came across <a href="http://joe.truemesh.com/blog/000390.html">this article</a>&nbsp;about the support for closures in C# 2.0. The article explains that the support for closures in C# 2.0 takes the form of anonymous delegates.</font>
</p>
<p>
<font face="Tahoma" size="2">There are some examples of closures like this one :</font>
</p>
<p>
<font face="Courier New"><font size="1"><font color="#0000ff">public</font> <font color="#008080">List</font>&lt;<font color="#008080">Employee</font>&gt; Managers(<font color="#008080">List</font>&lt;<font color="#008080">Employee</font>&gt; emps)<br />
</font></font><font face="Courier New" size="1">{<br />
</font><font face="Courier New"><font size="1"><font color="#0000ff">&nbsp; return</font> emps.FindAll(<br />
</font></font><font face="Courier New"><font size="1"><font color="#0000ff">&nbsp;&nbsp;&nbsp; delegate</font>(<font color="#008080">Employee</font> e)<br />
</font></font><font face="Courier New" size="1">&nbsp;&nbsp;&nbsp; {<br />
</font><font face="Courier New"><font size="1"><font color="#0000ff">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return</font> e.IsManager;<br />
</font></font><font face="Courier New" size="1">&nbsp;&nbsp;&nbsp; }<br />
</font><font face="Courier New" size="1">&nbsp; );<br />
</font><font face="Courier New" size="1">}</font>
</p>
<p>
<font face="Tahoma" size="2">Which is interesting, but less than this one :</font>
</p>
<p>
<font size="1"></font>
</p>
<font face="Courier New"><font size="1"><font color="#0000ff">public</font> <font color="#008080">List</font>&lt;<font color="#008080">Employee</font>&gt; HighPaid(<font color="#008080">List</font>&lt;<font color="#008080">Employee</font></font></font><font face="Courier New" size="1">&gt; emps)<br />
</font><font size="1"><font face="Courier New">{<br />
</font><font face="Courier New"><font color="#0000ff">&nbsp; int</font> threshold = <font color="#ff00ff">150</font></font></font><font face="Courier New" size="1">;<br />
</font><font face="Courier New"><font size="1"><font color="#0000ff">&nbsp; return</font> emps.FindAll(<br />
&nbsp;&nbsp;&nbsp; <font color="#0000ff">delegate</font>(<font color="#008080">Employee</font></font></font><font face="Courier New" size="1"> e)<br />
</font><font size="1"><font face="Courier New">&nbsp;&nbsp;&nbsp; {<br />
</font><font face="Courier New" color="#0000ff">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return</font></font><font face="Courier New" size="1"> e.Salary &gt; threshold;<br />
</font><font face="Courier New" size="1">&nbsp;&nbsp;&nbsp; }<br />
&nbsp; </font><font size="1"><font face="Courier New">);<br />
</font><font face="Courier New">}</font> </font>
<p>
<font face="Tahoma" size="2">The interesting part here is that the delegate is actually allowed to use a variable that is local to the method where it is defined. </font><font size="2"><font face="Tahoma">You might wonder how this is implemented by the C# compiler.<br />
</font><font face="Tahoma">It may become even less obvious with this example :</font></font><font size="1"><font color="#0000ff"></font></font>
</p>
<font face="Courier New"><font size="1"><font color="#008080"><font color="#0000ff">public </font>Predicate</font>&lt;<font color="#008080">Employee</font>&gt; PaidMore(<font color="#0000ff">int</font></font></font><font face="Courier New" size="1"> amount)<br />
{<br />
<font color="#0000ff"><font color="#000000">&nbsp; </font>return</font> <font color="#0000ff">delegate</font>(<font color="#008080">Employee</font> e)<br />
&nbsp; {<br />
&nbsp;&nbsp;&nbsp; <font color="#0000ff">return</font> e.Salary &gt; amount;<br />
&nbsp; };<br />
}</font> 
<p>
<font face="Tahoma" size="2">Ok, where does the compiler stores the value of &quot;amount&quot; since the delegate method is only returned to be executed later... ?</font>
</p>
<p>
<font face="Tahoma" size="2">In fact, the compiler only generates a &quot;DisplayClass&quot; that containts <font face="Courier New">amount</font> as a field initialized when the anonymous delegate is created, and the implementation of the delegate itself.</font>
</p>
<p>
<font face="Tahoma" size="2">Easy.</font>
</p>

{% include imported_disclaimer.html %}
