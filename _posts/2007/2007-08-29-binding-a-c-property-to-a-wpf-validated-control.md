---
layout: post
title: "Binding a C# property to a WPF validated Control"
date: 2007-08-29 05:48:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/08/29/Binding-a-C-property-to-a-WPF-validated-Control", "/post/2007/08/29/binding-a-c-property-to-a-wpf-validated-control"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="trebuchet ms,geneva" size="2">The DataBinding in WPF allows the binding&nbsp;of control properties to many things, like other control properties, arrays, data providers, ... But it can also bind a control property to a property defined by code on the C# side. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">It is interesting to bind to C# property to be able to have the integrated WPF validation and still use a simple property from&nbsp;the code. In that case, I wanted to have the validation of a TextBox displaying a DateTime object. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Here&#39;s how to do this. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">On the C# side :</font> <br />
<font size="2" color="#0000ff"><br />
</font><font face="Courier New">[code:c#]<br />
public partial class Window1 : Window<br />
{&nbsp; <br />
&nbsp; public Window1()&nbsp; <br />
&nbsp; {&nbsp;&nbsp;&nbsp; <br />
&nbsp;&nbsp;&nbsp; MyDate = DateTime.Now;<br />
&nbsp;&nbsp;&nbsp; InitializeComponent();<br />
&nbsp; }<br />
&nbsp; public DateTime MyDate { get; set; }<br />
}<br />
[/code]</font>&nbsp; 
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Note that I&#39;m using the latest C# 3.0 syntax to declare variable-less properties. This is compiler trick, the variable is still declared at compile time, but since I don&#39;t need to have a specific code in the get or set accessor, it can stay in this short form. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Now on the XAML side : </font>
</p>
<font face="courier new,courier">[code:xml]<br />
&lt;Window ... &gt;<br />
&nbsp; &lt;TextBox Width=&quot;100&quot; Height=&quot;20&quot;&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;Binding Path=&quot;MyDate&quot; RelativeSource<strong>=&quot;{RelativeSource Mode=FindAncestor,AncestorType={x:Type Window}}</strong>&quot; UpdateSourceTrigger=&quot;PropertyChanged&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;Binding.ValidationRules&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;ExceptionValidationRule /&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/Binding.ValidationRules&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;/Binding&gt;<br />
&nbsp; &lt;/TextBox&gt;<br />
&lt;/Window&gt; <br />
[/code]</font> 
<p align="justify">
<font face="Verdana" size="2"><font face="trebuchet ms,geneva">The interesting part here is the use of</font> <strong><font face="Courier New">RelativeSource</font></strong> <font face="trebuchet ms,geneva">and the</font> <strong><font face="Courier New">FindAncestor</font></strong> <font face="trebuchet ms,geneva">mode. Here, we&#39;re looking for the property</font> <strong><font face="Courier New">MyDate</font> </strong><font face="trebuchet ms,geneva">from the nearest ancestor&nbsp;instance&nbsp;of the</font> <strong><font face="Courier New">Window</font></strong> <font face="trebuchet ms,geneva">type, from the current instance.</font></font><font face="trebuchet ms,geneva"> </font>
</p>
<p align="justify">
<font face="Verdana" size="2"><font face="trebuchet ms,geneva">This way, you&#39;ll have a validated date time in your property value. Just make sure you&#39;re checking that the value is really valid using the</font> <strong><font face="Courier New">System.Windows.Controls.Validation</font></strong> <font face="trebuchet ms,geneva">class.</font></font><font face="trebuchet ms,geneva"> </font>
</p>

{% include imported_disclaimer.html %}
