---
layout: post
title: "Bluetooth Remote Control 0.8.2"
date: 2007-08-22 15:44:00 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2007/08/22/Bluetooth-Remote-Control-082", "/post/2007/08/22/bluetooth-remote-control-082"]
author: jerome
---
<!-- more -->
<p>
<font face="Verdana" size="2">This kind of a fix only release. I&#39;m trying to fix the &quot;MissingMethodException&quot;. This is a bit weird since the function I&#39;m calling should always be here, as said in the SDK documentation... Some documentation or ISV&nbsp;glich, maybe.</font>
</p>
<p>
<font face="Verdana" size="2">I&#39;ve also added a more visual bluetooth availability check. Now, if the background of the desktop server is in some kind of red color, then bluetooth is not available. Of course, nothing won&#39;t work until it becomes gray again.</font>
</p>
<p>
<font face="Verdana" size="2">On a side note, I&#39;m trying some of the new features of Visual Studio 2008, like the multi-targeting of frameworks. Too bad they did not include the&nbsp;.NET Compact Framework 1.0. This is not good news for Bluetooth Remote Control, since I&#39;m keeping .NET CF 1.0 to run on older devices like Smartphones with WM2003. I guess I&#39;m going to&nbsp;have to drop support of these devices in future version.</font>
</p>
<p>
<font face="Verdana" size="2">Get the new release&nbsp;here : </font><a href="http://www.jaylee.org/remotecontrol"><font face="Verdana" size="2">http://www.jaylee.org/remotecontrol</font></a>
</p>

{% include imported_disclaimer.html %}
