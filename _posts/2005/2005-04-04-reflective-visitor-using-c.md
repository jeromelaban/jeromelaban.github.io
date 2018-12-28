---
layout: post
title: "Reflective Visitor using C#"
date: 2005-04-04 11:35:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2005/04/04/Reflective-Visitor-using-C.aspx", "/post/2005/04/04/reflective-visitor-using-c.aspx"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">There is a well known design pattern in the Object Oriented world: The Visitor pattern.</font>
</p>
<p>
<font face="Tahoma" size="2">This pattern, among other things, allows to extend&nbsp;an object without actually modifying it. It is fairly easy to implement in any good OO language such as C++, Java or C#. </font>
</p>
<p>
<font face="Tahoma" size="2">However, there is a problem with the implementation of this pattern, or rather an implementation limitation. It requires&nbsp;the base interface or abstract class for all visitors to define at most one method for each type that may visit it. This is not a problem by itself, but it requires to modify the visitor base each time you add a new type. The best way to do this would be to call the appropriate method based on the caller type.</font>
</p>
<p>
<font face="Tahoma" size="2">.NET provides that kind of behavior through reflection, as it is possible to find a method based on its parameters at runtime.</font>
</p>
<p>
<font face="Tahoma" size="2">I decided to try this out with the C# 2.0 and its generics :)</font>
</p>
<p>
<font face="Tahoma" size="2">Here what I came up with :</font>
</p>
<font size="2">
<p>
&nbsp;
</p>
</font><font face="Courier New"><font size="1"><font color="#0000ff">public</font> <font color="#0000ff">interface</font> </font><font size="1" color="#008080">IOperand<br />
</font></font><font size="1"><font face="Courier New">{<br />
&nbsp; </font><font face="Courier New"><font color="#008080">IOperand</font> Accept(<font color="#008080">Visitor</font> visitor, <font color="#008080">IOperand</font></font></font><font size="1"><font face="Courier New"> right);<br />
</font><font face="Courier New">}<br />
<br />
</font></font><font face="Courier New"><font size="1"><font color="#0000ff">public</font> <font color="#0000ff">class</font> <font color="#008080">Operand</font><font color="#000000">&lt; T&gt;</font>&nbsp;: </font></font><font face="Courier New" size="1" color="#008080">IOperand<br />
</font><font face="Courier New"><font size="1">{<br />
&nbsp; <font color="#0000ff">private</font></font></font><font face="Courier New"><font size="1"> T _value;<br />
<br />
<font color="#0000ff">&nbsp; public</font></font></font><font face="Courier New" size="1"> Operand(T value)<br />
&nbsp; </font><font face="Courier New" size="1">{<br />
&nbsp;&nbsp;&nbsp; </font><font face="Courier New" size="1">_value = value;<br />
&nbsp; </font><font face="Courier New"><font size="1">}<br />
&nbsp; <font color="#0000ff">public</font> <font color="#008080">IOperand</font> Accept(<font color="#008080">Visitor</font> v, <font color="#008080">IOperand</font></font></font><font face="Courier New" size="1"> right)<br />
&nbsp; </font><font face="Courier New"><font size="1">{<br />
&nbsp;&nbsp;&nbsp; <font color="#0000ff">return</font> v.Visit(<font color="#0000ff">this</font></font></font><font face="Courier New" size="1">, right);<br />
&nbsp; }<br />
</font><font face="Courier New" size="1" color="#0000ff">&nbsp; public</font><font face="Courier New" size="1"> T Value<br />
</font><font face="Courier New"><font size="1">&nbsp; {<br />
<font color="#0000ff">&nbsp;&nbsp;&nbsp; get</font> { <font color="#0000ff">return</font></font></font><font face="Courier New" size="1"> _value; }<br />
&nbsp; </font><font face="Courier New" size="1">}<br />
&nbsp;&nbsp;<font color="#0000ff">public</font> <font color="#0000ff">override</font> <font color="#0000ff">string</font> ToString()<br />
&nbsp; {<br />
&nbsp;&nbsp;&nbsp; <font color="#0000ff">return</font> <font color="#0000ff">string</font>.Format(<font color="#ff00ff">&quot;{0} ({1})&quot;</font>, _value, GetType());<br />
&nbsp; }<br />
}</font> 
<p>
<font face="Tahoma" size="2">This is the definition of an operand, which is used in a abstract machine to perform operations on abstract types. This class is generic, I did not want to implement all the possible types.</font>
</p>
<p>
<font face="Tahoma" size="2">Then here is the visitor :</font>
</p>
<font size="2">
<p>
&nbsp;
</p>
</font><font face="Courier New"><font size="1"><font color="#0000ff">public</font> <font color="#0000ff">class</font> <font color="#008080">Visitor<br />
</font></font></font><font face="Courier New" size="1">{<br />
&nbsp; </font><font size="1"><font face="Courier New"><font color="#0000ff">public</font> <font color="#0000ff">virtual</font> <font color="#008080">IOperand</font> Visit(<font color="#008080">IOperand</font> left, <font color="#008080">IOperand</font></font><font face="Courier New"> right)<br />
&nbsp; </font></font><font face="Courier New" size="1">{<br />
&nbsp;&nbsp;&nbsp; </font><font size="1"><font face="Courier New"><font color="#008080">MethodInfo</font> info = GetType().GetMethod(<font color="#ff00ff">&quot;Visit&quot;</font>, <font color="#0000ff">new</font> <font color="#008080">Type</font></font><font face="Courier New">[] { left.GetType(), right.GetType() });<br />
&nbsp;&nbsp;&nbsp; <br />
&nbsp;&nbsp;&nbsp; </font></font><font size="1"><font face="Courier New"><font color="#0000ff">if</font> (info != <font color="#0000ff">null</font> &amp;&amp; info.DeclaringType != <font color="#0000ff">typeof</font>(<font color="#008080">Visitor</font></font><font face="Courier New">))<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </font></font><font size="1"><font face="Courier New"><font color="#0000ff">return</font> info.Invoke(<font color="#0000ff">this</font>, <font color="#0000ff">new</font> <font color="#0000ff">object</font>[] { left, right }) <font color="#0000ff">as</font> <font color="#008080">IOperand</font></font><font face="Courier New">;<br />
<br />
&nbsp;&nbsp;&nbsp;&nbsp;</font></font><font size="1"><font face="Courier New"><font color="#008080">Console</font>.WriteLine(<font color="#ff00ff">&quot;Operation not supported&quot;</font></font><font face="Courier New">);<br />
&nbsp;&nbsp;&nbsp; </font></font><font size="1"><font face="Courier New"><font color="#0000ff">return</font> <font color="#0000ff">null</font></font><font face="Courier New">;<br />
&nbsp;&nbsp;</font></font><font face="Courier New" size="1">}<br />
</font><font face="Courier New" size="1">}</font> 
<p>
<font face="Tahoma" size="2">This method search in the current type all methods named &quot;Visit&quot; that take the actual type of the parameters left and right and tries to match a method with it. Also, to avoid looping through the same method we&#39;re in since it&#39;s matching everything, there is a test for the type declaring the method.</font>
</p>
<p>
<font face="Tahoma" size="2">Now the AdditionVisitor :</font>
</p>
<font size="2">
<p>
&nbsp;
</p>
</font><font face="Courier New"><font size="1"><font color="#0000ff">public</font> <font color="#0000ff">class</font> <font color="#008080">AdditionVisitor</font> : <font color="#008080">Visitor<br />
</font></font></font><font face="Courier New" size="1">{<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp; public</font> <font color="#008080">IOperand</font> Visit(<font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font>&gt; value, <font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font><font face="Courier New">&gt; right)<br />
</font><font face="Courier New" size="1">&nbsp; {<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp;&nbsp;&nbsp; return</font> <font color="#0000ff">new</font> <font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font></font><font face="Courier New">&gt;(value.Value + right.Value);<br />
</font><font face="Courier New" size="1">&nbsp; }<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp; public</font> <font color="#008080">IOperand</font> Visit(<font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font>&gt; value, <font color="#008080">Operand</font>&lt;<font color="#0000ff">short</font><font face="Courier New">&gt; right)<br />
</font><font face="Courier New" size="1">&nbsp; {<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp;&nbsp;&nbsp; return</font> <font color="#0000ff">new</font> <font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font></font><font face="Courier New">&gt;(value.Value + right.Value);<br />
</font><font face="Courier New" size="1">&nbsp; }<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp; public</font> <font color="#008080">IOperand</font> Visit(<font color="#008080">Operand</font>&lt;<font color="#0000ff">double</font>&gt; value, <font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font><font face="Courier New">&gt; right)<br />
</font><font face="Courier New" size="1">&nbsp; {<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp;&nbsp;&nbsp; return</font> <font color="#0000ff">new</font> <font color="#008080">Operand</font>&lt;<font color="#0000ff">double</font></font><font face="Courier New">&gt;(value.Value + right.Value);<br />
</font><font face="Courier New" size="1">&nbsp;&nbsp;}<br />
</font><font face="Courier New" size="1">}</font> 
<p>
<font face="Courier New" size="1"><font face="Tahoma" size="2">Which defines a bunch of visitable methods used to add different&nbsp;operations on IOperand-like types.</font></font>
</p>
<p>
<font face="Tahoma" size="2">And finally to use it :</font>
</p>
<font size="2">
<p>
&nbsp;
</p>
</font><font size="1"><font face="Courier New"><font color="#0000ff">class</font> <font color="#008080">Program<br />
</font></font></font><font face="Courier New" size="1">{<br />
</font><font size="1"><font face="Courier New"><font color="#0000ff">&nbsp; static</font> <font color="#0000ff">void</font> Main(<font color="#0000ff">string</font>[] args)<br />
</font></font><font face="Courier New" size="1">&nbsp; {<br />
</font><font size="1"><font face="Courier New"><font color="#008080">&nbsp;&nbsp;&nbsp; Operand</font>&lt;<font color="#0000ff">int</font>&gt; a = <font color="#0000ff">new</font> <font color="#008080">Operand</font>&lt;<font color="#0000ff">int</font>&gt;(<font color="#ff00ff">21</font>);<br />
<font size="1"><font face="Courier New"><font color="#008080">&nbsp;&nbsp;&nbsp; Operand</font>&lt;<font color="#0000ff">short</font>&gt; b = <font color="#0000ff">new</font> <font color="#008080">Operand</font>&lt;<font color="#0000ff">short</font>&gt;(<font color="#ff00ff">21</font>);<br />
<font size="1"><font face="Courier New"><font color="#008080">&nbsp;&nbsp;&nbsp; Console</font>.WriteLine(Add(a, b));<br />
</font></font><font face="Courier New" size="1">&nbsp; }<br />
<br />
&nbsp; </font><font size="1"><font face="Courier New"><font color="#0000ff">static</font> <font color="#008080">IOperand</font> Add(<font color="#008080">IOperand</font> a, <font color="#008080">IOperand</font> b)<br />
&nbsp; </font></font><font face="Courier New" size="1">{<br />
&nbsp;&nbsp;&nbsp; </font><font size="1"><font face="Courier New"><font color="#008080">AdditionVisitor</font> addVisitor = <font color="#0000ff">new</font> <font color="#008080">AdditionVisitor</font>();<br />
&nbsp;&nbsp;&nbsp; </font></font><font size="1"><font face="Courier New"><font color="#0000ff">return</font> a.Accept(addVisitor, b);<br />
&nbsp; </font></font><font face="Courier New" size="1">}<br />
</font><font face="Courier New" size="1">}</font><font size="2"></font> 
<p>
<font face="Tahoma" size="2">Using this Reflective Visitor, modifying the base visitor class is not needed anymore, which limits the modifications to one class only. Of course, there&#39;s room for optimization, for instance by avoiding the method lookup using the System.Reflection namespace, but you get the picture.</font>
</p>
</font></font></font></font></font></font></font></font></font></font></font></font></font>
<p>
<font face="Tahoma" size="2">Some asked me what could be done in .NET that could not be done in C++, this is an example of it :)</font>
</p>

{% include imported_disclaimer.html %}
