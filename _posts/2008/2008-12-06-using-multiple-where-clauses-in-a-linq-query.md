---
layout: post
title: "Using Multiple Where Clauses in a LINQ Query"
date: 2008-12-06 15:32:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/12/06/Using-Multiple-Where-Clauses-in-a-LINQ-Query", "/post/2008/12/06/using-multiple-where-clauses-in-a-linq-query"]
author: jay
---
<!-- more -->
<div align="justify">
<em>Cet article est <a href="http://blogs.codes-sources.com/jay/archive/2008/12/06/utiliser-plusieurs-clauses-where-dans-une-requete-linq.aspx" target="_blank" title="Utiliser plusieurs clauses Where dans une requ&ecirc;te LINQ - Jerome Laban">disponible en francais</a>.</em> <br />
</div>
<div align="justify">
&nbsp;
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">After writing my <a href="http://jaylee.org/post/2008/12/fsharp-TryWith-Maybe-and-Umbrella.aspx" target="_blank" title="F#, TryWith, Maybe and Umbrella - Jerome Laban">previous article</a> where I needed to intercept exceptions in a LINQ Query, I found out that it is possible to specify multiple where clauses in a LINQ Query.</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><br />
Here is the query :<br />
</font>
</div>
<br />
[code:c#]<br />
var q = from file in Directory.GetFiles(@&quot;C:\Windows\Microsoft.NET\Framework\v2.0.50727&quot;, &quot;*.dll&quot;)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    let asm = file.TryWith(f =&gt; Assembly.LoadFile(f))<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    where asm != null<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    let types = asm.TryWith(a =&gt; a.GetTypes(), (Exception e) =&gt; new Type[0])<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where types.Any()<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                    select new { asm, types };<br />
[/code]<br />
<font face="trebuchet ms,geneva" size="2"><br />
</font>
<div align="justify">
<font face="trebuchet ms,geneva" size="2">This query is able to find assemblies for which it is possible to list types. The point of using multiple Where clauses is to avoir evaluating chunks of a query if previous chunks can prevent it. By the way, the TryWith around the Assembly.GetTypes() is there to intercept exceptions raised when loading types, in case dependencies would not be available at the moment of the enumeration.</font>
</div>
<div align="justify">
<font face="trebuchet ms,geneva" size="2"><br />
A useful LINQ trick to remember !</font>
</div>

{% include imported_disclaimer.html %}
