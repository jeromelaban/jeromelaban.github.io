---
layout: post
title: "Unprotecting Protected Processes"
date: 2007-04-07 15:09:00 -0400
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2007/04/07/Unprotecting-Protected-Processes.aspx", "/post/2007/04/07/unprotecting-protected-processes.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<a href="http://www.alex-ionescu.com/?p=35"><font face="Verdana" size="2">Alex Ionescu&#39;s</font></a><font face="Verdana" size="2"> been searching a bit about Protected Processes, and he&#39;s managed to get around that protected state.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">I&#39;m no expert on that part nor have I read enough documentation on how that works, but since a goal of that particular feature is the &quot;Protected Media Path&quot; (PMP)&nbsp;to prevent anyone from eavesdropping a protected media, this is not good.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Since that implementation is based on a driver, that won&#39;t work on Vista 64, well, as long as you don&#39;t boot in that particular mode that allows you to load unsigned drivers. That&#39;s a good news for malware protection, since a virus shoud not be able to hide itself under normal conditions, but this is not for PMP. It seems that protected processes can check for a &quot;tainted&quot; environment, but how long is it going to take for someone to fool programs into thinking the system is clean... ? As always, that won&#39;t prevent evil dvd rippers to copy the media... but that&#39;ll piss a legitimate user.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Moreover, since it is easily possible in Vista 32, as alex is pointing it out, it probably won&#39;t take long for viruses to hide themselves using this technique and just a bit longer for antiviruses to unprotected any running process.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">What a mess :)</font>
</p>

{% include imported_disclaimer.html %}
