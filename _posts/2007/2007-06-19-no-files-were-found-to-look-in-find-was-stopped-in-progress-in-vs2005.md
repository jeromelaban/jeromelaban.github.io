---
layout: post
title: "No files were found to look in. Find was stopped in progress in VS2005"
date: 2007-06-19 07:08:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/06/19/No-files-were-found-to-look-in-Find-was-stopped-in-progress-in-VS2005.aspx", "/post/2007/06/19/no-files-were-found-to-look-in-find-was-stopped-in-progress-in-vs2005.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font size="2"><font face="trebuchet ms,geneva">There are bugs, mystical bugs I may add, that no one has been able to reproduce, nor find a fix for, well at least that do work on every occurrence. VS2005 has one of these, where when you try to use &quot;find in files&quot;, you get a <strong>&quot;No files were found to look in. Find was stopped in progress.&quot;</strong> and that&#39;s it. No disk access, no registry relevant access, no relevant file access.<br />
<br />
Some have found that pressing <strong>Ctrl+Scroll</strong> Lock may help, well, it did not help for me. Some other tried the reboot, which seems to work better. You can also </font><a href="http://blogs.ugidotnet.org/franny/archive/2005/12/08/31303.aspx"><font face="trebuchet ms,geneva">spin around&nbsp;three times on your chair</font></a><font face="trebuchet ms,geneva"> before doing that, that might also&nbsp;help.<br />
<br />
Then I remembered that some times, because it seems that the CLR maintains some kind of &quot;cross process state&quot; (I don&#39;t know what it acutally&nbsp;is, so I&#39;m just guessing) that cripples all the .NET processes that are running. The only way to reset that state is to kill all processes that are using the CLR.<br />
<br />
That means killing every standard application, plus IIS&#39;s aspnet_wp, and if you have .NET 3.0, PresentationFontCache.exe, ...<br />
<br />
That did the trick for me, after killing all these processes and running VS2005, my search was up&nbsp;and running :)</font></font>
</p>

{% include imported_disclaimer.html %}
