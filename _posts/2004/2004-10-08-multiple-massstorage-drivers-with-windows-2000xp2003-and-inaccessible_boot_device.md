---
layout: post
title: "Multiple MassStorage Drivers with Windows 2000/XP/2003 and INACCESSIBLE_BOOT_DEVICE"
date: 2004-10-08 11:42:00 -0500
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2004/10/08/Multiple-MassStorage-Drivers-with-Windows-2000XP2003-and-INACCESSIBLE_BOOT_DEVICE.aspx", "/post/2004/10/08/multiple-massstorage-drivers-with-windows-2000xp2003-and-inaccessible_boot_device.aspx"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">One annoying thing about Windows is the Mass Storage drivers management. The Windows Setup only installs the necessary drivers for the current system, which is generally fine most of the time. </font><font face="Tahoma" size="2">As long as you change your hardware but not the Mass Storage chipset, there is no problem. Windows is just restarting its Plug and Play stage to re-detect all the new devices and peripherals and this works really fine.</font>
</p>
<p>
<font face="Tahoma" size="2">Here at Epitech, computers were Via based for two years in a row and changing from one to the other was not a problem. This year&#39;s new computers are now Intel based. Nice computers, really. </font>
</p>
<p>
<font face="Tahoma" size="2">But one problem : Via based Windows installation don&#39;t boot anymore. There is a nice 0x7B stop mode (a bsod) which means INACCESSIBLE_BOOT_DEVICE. Windows was unable&nbsp;to find any suitable boot device, because it does not have the appropriate drivers for the current hardware.</font>
</p>
<p>
<font face="Tahoma" size="2">Microsoft has a KBase article (<a href="http://support.microsoft.com/default.aspx?scid=kb;en-us;314082">KB314082</a>)&nbsp;about this particular issue, which states that you can force a Windows installation to try every known MassStorage driver during the startup. Since the procedure implies the copying of some Intel drivers, I assumed a while ago that it would only work in the Via to Intel direction. </font><font face="Tahoma" size="2">Well, apparently not. It also works in the Intel to Via direction, which is really nice :) Actually, it works for any to any chipset, as long as the hardware is natively known by Windows. </font>
</p>
<p>
<font face="Tahoma" size="2">This solves a lot of problems for many people here that do really want to reinstall their Windows.</font>
</p>

{% include imported_disclaimer.html %}
