---
layout: post
title: "Visual Studio 2008 Solution Tree Items Collapse"
date: 2008-03-17 19:26:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/03/17/Visual-Studio-2008-Solution-Tree-Items-Collapse", "/post/2008/03/17/visual-studio-2008-solution-tree-items-collapse"]
author: jerome
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2">Even though there is a way to expand all nodes in the solution tree of Visual Studio 2008, there is no way to do the opposite, which is collapse all. Not collapse all top level nodes, but collapse all&nbsp;child nodes, one by one.</font> 
</p>
<p>
<font face="Trebuchet MS" size="2">There&#39;s a bug in VS2005/2008 that prevents a node from being collapsed properly for some obscure reason. The Expanded property keeps on being &quot;true&quot; even if set to &quot;false&quot;.</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Fortunately,&nbsp;a </font><a href="http://geekswithblogs.net/scottkuhl/archive/2007/04/09/111195.aspx" title="Scott Kuhl Blog"><font face="trebuchet ms,geneva" size="2">fix by Scott Kuhl</font></a><font face="trebuchet ms,geneva" size="2"> which was working with Visual Studio 2005 is also working with Visual Studio 2008. The script is doing some trick to simulate a DefaultAction each node, which seems to collapse a node without using the Expanded property.</font><font face="trebuchet ms,geneva" size="2">&nbsp;</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Nice trick :) It avoids me the&nbsp;burden of hitting the minus and enter keys numerous times...</font> 
</p>

{% include imported_disclaimer.html %}
