---
layout: post
title: "[VS2010] “Object reference not set to an instance of an object” when opening a file"
date: 2009-10-23 20:19:43 -0400
comments: true
category: Archive
tags: []
redirect_from: ["/post/2009/10/23/VS2010-e2809cObject-reference-not-set-to-an-instance-of-an-objecte2809d-when-opening-a-file.aspx", "/post/2009/10/23/vs2010-e2809cobject-reference-not-set-to-an-instance-of-an-objecte2809d-when-opening-a-file.aspx"]
author: admin
---
<!-- more -->
<p>If you’re trying out Visual Studio 2010, and you try to open source code file with the “Solution Explorer”, you might encounter a nice exception like this one :</p>  <p><font face="Courier New">---------------------------      <br />Microsoft Visual Studio       <br />---------------------------       <br />Object reference not set to an instance of an object.       <br />---------------------------       <br />OK       <br />---------------------------</font> </p>  <p>That could not be more vague..</p>  <p>After a small debugger analysis, it seems like VS2010 does not support TrueType fonts, but does not prevent their selection in the font selection dialog.</p>  <p>I was using <a title="Proggy Opti Small Font" href="http://www.proggyfonts.com/index.php?menu=download" target="_blank">“Proggy Opti Small”</a> with VS2008, which is not TrueType... So to be able to open you source files, use a font like <a href="http://www.microsoft.com/typography/fonts/family.aspx?FID=300" target="_blank">Consolas</a>, and restart VS2010. </p>  <p>A bug report <a href="https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=500551" target="_blank">exists on Connect</a>.</p>
{% include imported_disclaimer.html %}
