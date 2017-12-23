---
layout: post
title: "Remote Control 0.7.0-Beta2"
date: 2007-03-17 14:43:00 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2007/03/17/Remote-Control-070-Beta2", "/post/2007/03/17/remote-control-070-beta2"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Verdana" size="2">Download </font><a href="/files/BTRemoteSetup-0.7.0-Beta2.msi"><font face="Verdana" size="2">here</font></a><font face="Verdana" size="2">&nbsp;this new&nbsp;release 0.7.0-Beta2.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Since it&#39;s snowing a bit in Montreal, I&#39;ve had a bit of time to look around problems on the 0.7.0-Beta1 problems.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Most of the reported problems were about Widcomm support. The stack was crashing randomly on server shutdown... I hate those API that don&#39;t do what they should : Close means close, and you should be able to destroy the object. Well, not with the widcomm API. Some worker thread is still trying to tell by callback that the connection is being closed... This is a bit annoying where there&nbsp;is no connection !</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Anyway, this new Beta should fix this. There&#39;s also now in the stats bar a &quot;bluetooth&quot; status, whether it is using Microsoft Stack or Widcomm Stack.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">I&#39;m also working on some remote display of the PC screen, and to have some commands executed upon Bluetooth status change... This should be cool ! Wait and see !<br />
</font>
</p>

{% include imported_disclaimer.html %}
