---
layout: post
title: "Vista's Sound Mixer in Bluetooth Remote Control"
date: 2007-05-26 12:04:00 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2007/05/26/Vistas-Sound-Mixer-in-Bluetooth-Remote-Control", "/post/2007/05/26/vistas-sound-mixer-in-bluetooth-remote-control"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">If you&#39;re using Vista, you might have noticed that you can&#39;t change the volume using the &quot;Windows Mixer&quot; remote control section.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">Actually, the volume do change, but for the application itself.&nbsp;BTRC is not generating any sound, this is&nbsp;particularly <strong>not</strong> useful.&nbsp;Vista&#39;s got a new feature that allows the user to control the volume of applications independently, which is pretty cool from the user point of view.</font>
</p>
<p align="justify">
<font face="Tahoma"><font size="2">But from the developer&#39;s perspective, this modifies a bit the way for setting the sound volume. The &quot;old&quot; api still works but for the current application; and if you set the application in Windows XP compatibility mode, the behavior will be restored. Since I don&#39;t want to set that compatibility mode, I had to add specific support for vista using the new <font color="#000000">IAudioEndpointVolume, COM interface <em>(COM, when&#39;ll that&nbsp;disapear...)</em>&nbsp;that allows&nbsp;getting back the original behavior, which is changing the master volume.</font></font></font>
</p>
<p align="justify">
<font color="#2b91af"><font face="Tahoma" size="2" color="#000000">It&#39;ll be in the next release.</font></font><font color="#2b91af"></font>
</p>

{% include imported_disclaimer.html %}
