---
layout: post
title: "IEnumerable<T>.Any() vs. IEnumerable<T>.Count() != 0"
date: 2008-05-10 11:10:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/05/10/IEnumerable3cT3eAny()-vs-IEnumerable3cT3eCount()-neq-0", "/post/2008/05/10/ienumerable3ct3eany()-vs-ienumerable3ct3ecount()-neq-0"]
author: jay
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2"><em>This post is also <a href="http://blogs.developpeur.org/jay/archive/2008/05/11/ienumerable-t-any-vs-ienumerable-t-count-0.aspx" title="IEnumerable&lt;T&gt;.Any() vs. IEnumerable&lt;T&gt;.Count() != 0 ">available in french here</a>.</em>&nbsp;</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">After reading </font><a href="http://blogs.msdn.com/ericlippert/archive/2008/05/09/computers-are-dumb.aspx" title="Computers are Dumb"><font face="trebuchet ms,geneva" size="2">this post from Eric Lippert</font></a><font face="trebuchet ms,geneva" size="2">, and reminding me that in the samples for&nbsp;</font><a href="/post/2008/04/A-look-a-Linq-to-objects-and-the-let-keyword.aspx" title="A look at Linq to objects and the &quot;let&quot; keyword"><font face="trebuchet ms,geneva" size="2">this post</font></a><font face="trebuchet ms,geneva" size="2">, I&#39;m using IEnumerable&lt;T&gt;.Count() where I&#39;m actually not using the return value, therefore I&#39;m enumerating through the whole collection for almost nothing. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I&nbsp;could have used IEnumerable&lt;T&gt;.Any(), which after looking at the IL code, just starts the enumeration, and stops just after one element if there is one and returns true, or if there isn&#39;t, returns false. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This is more effective,&nbsp;even without <a href="http://blogs.msdn.com/pfxteam/" title="Parallel Programming with .NET">PLinq</a>&nbsp;:) </font>
</p>

{% include imported_disclaimer.html %}
