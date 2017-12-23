---
layout: post
title: "Bluetooth Remote Control 0.8.3 - Broadcom Bluetooth Support for Windows Mobile"
date: 2007-09-10 11:26:00 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2007/09/10/Bluetooth-Remote-Control-083-Broadcom-Bluetooth-Support-for-Windows-Mobile", "/post/2007/09/10/bluetooth-remote-control-083-broadcom-bluetooth-support-for-windows-mobile"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Verdana" size="2">This time, I&#39;ve added the support for the Broadcom (formerly Widcomm)&nbsp;stack on the Windows Mobile side.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">This should add support for the&nbsp;<strong>Motorola Q9</strong>, which uses this particular stack.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">On the stability and compatibility&nbsp;side, I&#39;ve only tested on a version of the widcomm stack for the HTC Wizard, but only on WM6 and I don&#39;t know if it actually works on other devices :)</font>
</p>
<p align="justify">
<font face="Verdana" size="2">If it works for you, please let me know !</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Download <a href="http://www.jaylee.org/files/btremotesetup-0.8.3.msi">Bluetooth Remote Control 0.8.3</a>&nbsp;now !</font>
</p>
<p align="left">
<font face="Verdana" size="2">If you ever want to install the Broadcom stack on WM6 for the HTC Wizard&nbsp;and you get a &quot;Not enough driver memory&quot;, try changing the &quot;StackMode&quot; value from &quot;0&quot; to &quot;1&quot; in HKLM/Widcomm/BTConfig/General. It worked out for me :)</font>
</p>

{% include imported_disclaimer.html %}
