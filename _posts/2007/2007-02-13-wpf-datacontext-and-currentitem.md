---
layout: post
title: "WPF DataContext and CurrentItem"
date: 2007-02-13 15:23:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/02/13/WPF-DataContext-and-CurrentItem", "/post/2007/02/13/wpf-datacontext-and-currentitem"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="trebuchet ms,geneva" size="2">DataBinding is one of the technologies&nbsp;I like the most.&nbsp; </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Winforms did a fine job introducing the concept, but&nbsp;it was pretty easy to&nbsp;be stuck&nbsp;when trying to perform complex databinding. .NET 2.0 brought the concept of BindingSource, which eased the management of multiple &quot;Current items&quot; on a single form. You might have encountered this when creating multiple master/detail views on the same form. That was&nbsp;the case when you wanted to walk through a set of tables through their relations, say down to three levels. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">WPF has a far more advanced DataBinding engine and introduces the concept of DataContext. There&#39;s the notion of a hierarchy of DataContexts and a databound control uses the nearest DataContext available. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">An example is better than a thousand words. Here&#39;s a data source :</font> <br />
[code:xml]<br />
&lt;xmldataprovider x:key=&quot;ops&quot; xpath=&quot;/data/level1&quot;&gt;<br />
&nbsp; &lt;x:XData&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;data xmlns=&quot;&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level1 name=&quot;1&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level2 name=&quot;1-1&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3 name=&quot;test&quot;&gt;Some Value&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Yet an other value from level 3&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/level2&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level2 name=&quot;1-2&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Some other Value&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Yet an other Value&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/level2&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/level1&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level1 name=&quot;2&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level2 name=&quot;2-1&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Some Value&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Yet an other value from level 3&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/level2&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level2 name=&quot;2-2&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Some other Value&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;level3&gt;Yet an other Value&lt;/level3&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/level2&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/level1&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;/data&gt;<br />
&nbsp; &lt;/x:XData&gt;<br />
&lt;/xmldataprovider&gt;[/code]<br />
<br />
<font face="trebuchet ms,geneva" size="2">It&#39;s a three level hierarchy, and I want to display this data, by recursively selecting each level in a ListBox to see its content. </font>
</p>
<p align="justify">
<font size="2"><font face="Verdana"><font face="trebuchet ms,geneva">Now, let&#39;s bind this data to a list box, contained in a GroupBox to be a bit more readable : <br />
</font><br />
</font></font><font face="courier new,courier">[code:xml]<br />
&lt;GroupBox Header=&quot;Level 1&quot; DataContext=&quot;{Binding Source={StaticResource ops}}&quot;&gt;<br />
&nbsp; &lt;ListBox ItemsSource=&quot;{Binding}&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DisplayMemberPath=&quot;@name&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IsSynchronizedWithCurrentItem=&quot;True&quot; /&gt;<br />
&lt;/GroupBox&gt;<br />
[/code]</font> 
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">This performs a binding to the attribute &quot;name&quot; of the level1 node list. The IsSynchronizedWithCurrentItem tells the listbox to set the CurrentItem of the current DataContext, which can be used to fill the next ListBox for the level. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><font size="2">Now, to add a new DataContext level, let&#39;s add a new GroupBox, and a stack panel to have a nice layout :</font> </font>
</p>
<font face="courier new,courier">[code:xml]<br />
&lt;GroupBox Header=&quot;Level 1&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataContext=&quot;{Binding Source={StaticResource ops}}&quot;&gt;<br />
&nbsp; &lt;StackPanel Orientation=&quot;Horizontal&quot;&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;ListBox ItemsSource=&quot;{Binding}&quot; <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DisplayMemberPath=&quot;@name&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IsSynchronizedWithCurrentItem=&quot;True&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;GroupBox Header=&quot;Level2&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataContext=&quot;{Binding Path=CurrentItem}&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;ListBox ItemsSource=&quot;{Binding}&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;DisplayMemberPath=&quot;@name&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IsSynchronizedWithCurrentItem=&quot;True&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;/GroupBox&gt;<br />
&nbsp; &lt;/StackPanel&gt;<br />
&lt;/GroupBox&gt;<br />
[/code]</font> 
<p align="justify">
<font face="trebuchet ms,geneva"><font size="2">Now, there is a new DataContext for any children of the second group box, and is having the CurrentItem of the upper DataContext as a root. This is fairly easy to do, so let&#39;s do this for the final level.</font> </font>
</p>
<p>
<font face="courier new,courier">[code:xml]<br />
&lt;GroupBox Header=&quot;Level 1&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataContext=&quot;{Binding Source={StaticResource ops}}&quot;&gt;<br />
</font><font face="courier new,courier">&nbsp; &lt;StackPanel Orientation=&quot;Horizontal&quot;&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;ListBox ItemsSource=&quot;{Binding}&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DisplayMemberPath=&quot;@name&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IsSynchronizedWithCurrentItem=&quot;True&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;GroupBox Header=&quot;Level2&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataContext=&quot;{Binding Path=CurrentItem}&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;StackPanel Orientation=&quot;Horizontal&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;ListBox ItemsSource=&quot;{Binding}&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DisplayMemberPath=&quot;@name&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IsSynchronizedWithCurrentItem=&quot;True&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;GroupBox Header=&quot;Level3&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataContext=&quot;{Binding Path=CurrentItem}&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;StackPanel Orientation=&quot;Horizontal&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;ListBox ItemsSource=&quot;{Binding}&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IsSynchronizedWithCurrentItem=&quot;True&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;Label Content=&quot;{Binding Path=CurrentItem}&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/StackPanel&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/GroupBox&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/StackPanel&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;/GroupBox&gt;<br />
&nbsp; &lt;/StackPanel&gt;<br />
&lt;/GroupBox&gt;<br />
[/code]<br />
</font><font face="trebuchet ms,geneva" size="2">Each time a new DataContext is set, a new CurrentItem is created. That kind of behavior was hard to reproduce using DataBinding with WinForms; WPF allows it only by using a simple declarative syntax. Easy and powerful. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Also there is a fine feature that came up when using this DataContext and CurrentItem : The CurrentItem &quot;chain&quot;&nbsp;for a specific path is -- when the data source does not change --&nbsp;kept if you change the selection, and come back to that particular path. Pretty interesting. </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Here is a working&nbsp;</font><a href="/files/multiple_data_context.zip"><font face="trebuchet ms,geneva" size="2">xaml sample</font></a><font size="2"><font face="trebuchet ms,geneva"> of this post.</font><font face="trebuchet ms,geneva"> </font></font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva" size="2">Did I say that a really like WPF already ? :) </font>
</p>

{% include imported_disclaimer.html %}
