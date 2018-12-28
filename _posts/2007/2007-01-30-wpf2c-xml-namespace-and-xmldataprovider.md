---
layout: post
title: "WPF, Xml namespace and XmlDataProvider"
date: 2007-01-30 16:15:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/01/30/WPF2c-Xml-namespace-and-XmlDataProvider.aspx", "/post/2007/01/30/wpf2c-xml-namespace-and-xmldataprovider.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">I was playing with the <a href="http://msdn2.microsoft.com/en-us/library/system.windows.data.xmldataprovider.aspx">XmlDataProvider</a> in WPF and I wanted to bind the content of inline XML from&nbsp;that provider&nbsp;to a ComboBox.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">So, with I wrote this little piece of code :</font>
</p>
<div class="csharpcode">
<pre>
&lt;XmlDataProvider x:Key=&quot;ops&quot; XPath=&quot;/operations/op&quot;&gt;  &lt;x:XData&gt;    &lt;operations&gt;      &lt;op&gt;A&lt;/op&gt;      &lt;op&gt;B&lt;/op&gt;      &lt;op&gt;C&lt;/op&gt;    &lt;/operations&gt;  &lt;/x:XData&gt;&lt;/XmlDataProvider&gt;[...]&lt;ComboBox   ItemsSource=&quot;{Binding Source={StaticResource ops}}&quot; /&gt;
</pre>
</div>
<p align="justify">
<font face="Tahoma" size="2">Turns out that the combobox does not display anything, even though the XPath for&nbsp;the XmlDataProvider is correct and the ItemSource binding as well.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">In reality, the problem is not in the binding but rather in the XPath, where the XML &quot;operations&quot; node in the inline xml inherits from the <font face="Courier New">System.Windows</font> XML namespace, which renders the XPath <font face="Courier New">&quot;/operations/op&quot;</font> ineffective. In that case, the XPath expression selects nodes from the &quot;&quot; (empty) namespace.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">I&nbsp;just needed to write this instead :</font>
</p>
<div class="csharpcode">
<pre>
&lt;XmlDataProvider x:Key=&quot;ops&quot; XPath=&quot;/operations/op&quot;&gt;  &lt;x:XData&gt;    &lt;operations <strong><font color="#ff0000">xmlns=&quot;&quot;</font></strong>&gt;
</pre>
</div>
<p align="justify">
<font face="Tahoma" size="2">to reset the namespace used for that node, and my ComboBox was filled !&nbsp;</font> 
</p>
<p align="justify">
<font face="Tahoma" size="2">As a side note on WPF, I am just wondering on how this technology will be adopted. Many concepts are fairly innovative and differ from the usual concepts found in Winforms or even MFC.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">I&#39;m convinced that it is definitely going in the right direction with the data and design separation, but I&#39;m not sure on how beginners are going to apprehend all this. Maybe a new release of Cider&nbsp;is going to ease development with this technology...Wait and see.&nbsp;</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">Meanwhile, I&#39;m continuing to enjoy the fact that I can databind a TreeView very easily :)</font>
</p>

{% include imported_disclaimer.html %}
