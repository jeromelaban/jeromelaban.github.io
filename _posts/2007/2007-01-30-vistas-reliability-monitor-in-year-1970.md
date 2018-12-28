---
layout: post
title: "Vista's Reliability Monitor in year 1970"
date: 2007-01-30 17:56:00 -0500
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2007/01/30/Vistas-Reliability-Monitor-in-year-1970.aspx", "/post/2007/01/30/vistas-reliability-monitor-in-year-1970.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">Vista&#39;s new reliability monitor is kind of great when you want check on your computer&#39;s overall performance. You get an index from 0 to 10 giving an overall performance rating of the system, which seems to be pretty accurate for what I can tell so far. It tracks hardware failures, software failures, windows failures, software (un)installs and some other failures, then gives you an index based on that info.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">Actually, on a developer machine, this is a little biaised because any exception you raise from any of your program ends up lowering the index. That should invite you not to let unhandled exceptions exiting your programs... :)</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">Anyway, a few days ago, I went to look at that index and found that the only history I had was on the 01/01/1970... Rings any bell ? :)</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">Well, if you ever encounter this particular problem in case vista&#39;s not detecting that by itself and resets its database, just go there :</font>
</p>
<p align="justify">
<font face="Courier New" size="2">%ProgramData%\Microsoft\RAC</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">and remove the two directories that should be there. The Reliability monitor should detect it and restart just fine.</font>
</p>

{% include imported_disclaimer.html %}
