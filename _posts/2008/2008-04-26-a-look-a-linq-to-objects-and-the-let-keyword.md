---
layout: post
title: "A look at Linq to objects and the 'let' keyword"
date: 2008-04-26 13:08:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/04/26/A-look-a-Linq-to-objects-and-the-let-keyword", "/post/2008/04/26/a-look-a-linq-to-objects-and-the-let-keyword"]
author: jerome
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2"><em>Cet article est aussi&nbsp;disponible en </em><a href="http://blogs.developpeur.org/jay/archive/2008/05/10/aventures-avec-le-mot-cl-let-dans-linq-to-objects.aspx" title="Aventures avec le mot cl&eacute; &quot;let&quot; dans LINQ to Objects "><em>fran&ccedil;ais ici</em></a><em>.</em>&nbsp;</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I&#39;ve had some time lately to use LINQ a bit more intensively and in particular to use the let keyword. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I had to process a lot of XML files placed in multiple folders, and I wanted to filter them using a regex. First, here&#39;s </font><font face="trebuchet ms,geneva" size="2">how to get all files from a directory tree : </font>
</p>
<p>
<font face="courier new,courier">[code:c#]<br />
<br />
&nbsp; var files = from dir in Directory.GetDirectories(rootPath, SearchOption.AllDirectories)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; from file in Directory.GetFiles(&quot;&quot;, &quot;*.*&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select new { Path = dir, File = Path.GetFileName(file) };<br />
<br />
[/code] </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">At this point, I could have omitted the <font face="courier new,courier">Directory.GetDirectories</font> call because <font face="courier new,courier">GetFiles</font> can also search recursively.&nbsp;</font><font face="trebuchet ms,geneva" size="2">But since the <font face="courier new,courier">GetFiles</font> method only returns a string array and not an enumerator, it means that all my files would have been </font><font face="trebuchet ms,geneva" size="2">returned in one array, which is not memory effective. I&#39;d rather have an iterator based implementation of <font face="courier new,courier">GetDirectories</font> </font><font face="trebuchet ms,geneva" size="2">and <font face="courier new,courier">GetFiles</font> for that matter, but the finest grained enumeration can only be done this way... </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Anyway, having all my files, I now wanted to filter the collection with a specific Regex, as my legitimate files need to </font><font face="trebuchet ms,geneva" size="2">observe a specific pattern. So, I updated my query to this : </font>
</p>
<font face="courier new,courier" size="2">[code:c#]<br />
<br />
&nbsp;&nbsp;&nbsp; Regex match = new Regex(@&quot;(?&lt;value&gt;\d{4}).xml&quot;);<br />
&nbsp;&nbsp;&nbsp; var files2 = from dir in Directory.GetDirectories(args[0], &quot;*&quot;, SearchOption.AllDirectories)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; from file in Directory.GetFiles(dir, &quot;*.xml&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let r = match.Match(Path.GetFileName(file))<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where r.Success<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select new {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Path = dir, <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; File = Path.GetFileName(file),<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Value = r.Groups[&quot;value&quot;].Value<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };<br />
<br />
[/code]<br />
</font><font face="trebuchet ms,geneva" size="2">This time, I&#39;ve introduced the let keyword. This keyword is very interesting because it allows the creation of a query-</font><font face="trebuchet ms,geneva" size="2">local variable that can contain either collections or single objects. The content of this variable can be used in the </font><font face="trebuchet ms,geneva" size="2">where clause, as the source of another &quot;from&quot; query, or in the select statement. </font>
<p>
<font face="trebuchet ms,geneva" size="2">In my case, I just wanted to have the result of the Regex match, so I&#39;m just calling Regex.Match to validate the file </font><font face="trebuchet ms,geneva" size="2">name, and I&#39;m placing the content of a Regex group in my resulting anonymous type. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Now, with all my files filtered I found that some XML files were not valid because they were not containing a specific node. </font><font face="trebuchet ms,geneva" size="2">So I filtered them again&nbsp;using this query : </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2"><font face="courier new,courier">[code:c#]<br />
<br />
&nbsp;&nbsp;&nbsp; var files2 = from dir in Directory.GetDirectories(args[0], &quot;*&quot;, SearchOption.AllDirectories)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; from file in Directory.GetFiles(dir, &quot;*.xml&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let r = match.Match(Path.GetFileName(file))<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let c = XElement.Load(file).XPathSelectElements(&quot;//dummy&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where r.Success &amp;&amp; c.Count() != 0<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select new {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Path = dir, <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; File = Path.GetFileName(file),<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Value = r.Groups[&quot;value&quot;].Value<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };<br />
<br />
[/code]</font> </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I&#39;ve added a new let clause to add the loading of the file, and make sure there is a node named &quot;dummy&quot; somewhere in the </font><font face="trebuchet ms,geneva" size="2">xml document. By the way, if you&#39;re looking XPath in XLinq, just look <a href="http://msdn2.microsoft.com/en-us/library/bb342176.aspx" target="_blank" title="XElement.XPathSelectElements">over there</a>. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">You may wonder by looking at this query when the load is actually evaluated... Well, it is only evaluated when the </font><font face="trebuchet ms,geneva" size="2"><font face="courier new,courier">c.Count()</font> call is performed, which is only after the regex has matched the file ! This&nbsp;way, I&#39;m&nbsp;not trying to load&nbsp;all the files returned by <font face="courier new,courier">GetFiles</font>.&nbsp;You need to always remember that </font><font face="trebuchet ms,geneva" size="2">queries are evaluated only when enumerated. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">In conclusion, Linq is a very interesting piece of technology, definitely NOT reserved to querying databases. What I like </font><font face="trebuchet ms,geneva" size="2">the most is that one can write code almost without any loop, therefore reducing side effects. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">If you haven&#39;t looking at Linq yet, just give it a try, you&#39;ll probably like it :) </font>
</p>

{% include imported_disclaimer.html %}
