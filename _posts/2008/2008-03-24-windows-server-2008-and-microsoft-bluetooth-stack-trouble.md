---
layout: post
title: "Windows Server 2008 and Microsoft Bluetooth Stack trouble"
date: 2008-03-24 13:49:00 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control", "Misc"]
redirect_from: ["/post/2008/03/24/Windows-Server-2008-and-Microsoft-Bluetooth-Stack-trouble", "/post/2008/03/24/windows-server-2008-and-microsoft-bluetooth-stack-trouble"]
author: jerome
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2">There&#39;s been <a href="http://blogs.msdn.com/vijaysk/archive/2008/02/11/using-windows-server-2008-as-a-super-desktop-os.aspx" title="Vijayshinva Karnure's blog">a lot</a>&nbsp;of&nbsp;<a href="http://blogs.zdnet.com/microsoft/?p=1218" target="_blank" title="Mary Jo Foley's Blog">Buzz</a>&nbsp;<a href="http://weblogs.asp.net/israelio/archive/2008/02/21/windows-server-2008-as-workstation.aspx" target="_blank">lately</a> about a <a href="http://www.win2008workstation.com/wordpress/" title="Windows Workstation 2008">&quot;Windows Workstation 2008&quot;</a>, which actually does not quite exist, but that should.&nbsp;It is actually installing Windows Server 2008 and making it a workstation platform, by enabling every workstation component that is disabled by default. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">From my point of view, Vista is definitely interesting, though it has too many services that are enabled by default and that do not make sense in every situation. For a computer savvy user, all this stuff is not really interesting, and Windows Server 2008 with its &quot;do not enable unused components&quot; policy, is quite interesting. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I decided to give it a shot by installing it as my main (and only)&nbsp;laptop OS, and quite frankly, I&#39;m pleasantly surprised !&nbsp;I do get the same user experience that I did have with Windows Vista with Aero, the nifty new features like the new start menu, and I seem to get a performance improvement over Vista. (Performance improvement is only a feeling; I don&#39;t have any numbers to show, though some did). </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Everything works as expected, except for the bluetooth part, for which <a href="http://www.sharepoint-tips.com/2008/02/installing-bluetooth-stack-in-windows.html" target="_blank" title="Ishai Sagi's blog">I do not seem to be the only one</a> having problem with. The microsoft stack does not seem to install completely, as there are three &quot;unknown devices&quot; left : BTH\MS_RFCOMM, BTH\MS_BTHBRB and BTH\MS_BTHPAN. All three of them are core components of the bluetooth stack, and are obiously needed to get bluetooth related software working properly. The interesting part is that there are actually all the driver and metadata files required to install these devices, but for some reason, Win2008 does not want to use them. The driver files seem to be identical to the files Vista SP1, so this is one bit of a mystery to me. Added to that, this installation issue seems to be related to the <a href="http://support.microsoft.com/kb/940199" target="_blank" title="KB940199">KB940199</a> where the infcache.1 file could not be found. Screwing with that file did not help either...</font> 
</p>
<p>
<font face="Trebuchet MS" size="2">So as a backup plan, I decided to fall back on the Widcomm/Broadcom Stack&nbsp;<a href="http://www.dev-toast.com/2007/01/05/uncrippling-bluetooth-in-vista-rtm" target="_blank" title="Uncrippling Bluetooth In Vista RTM">with this guide</a>, which seems to work fine, at least for the part I&#39;m interested in, <a href="/remotecontrol" title="Bluetooth Remote Control for Windows Mobile">Bluetooth Remote Control</a> . I still don&#39;t understand the licensing policy on this software... You need the hardware to get that software to work, why bother having an licensing scheme over this, haven&#39;t you already paid for it buying the hardware&nbsp;?</font> 
</p>
<p>
<font face="Trebuchet MS" size="2">Anyway, if you&#39;re a tech savvy user, give Windows Server 2008 a try as your workstation OS, you might be surprised :)</font> 
</p>
<p>
<font face="Trebuchet MS" size="2">Now, I&#39;m going back to adding new features to <a href="/remotecontrol" title="Bluetooth Remote Control for Windows Mobile">Bluetooth Remote Control</a> !</font> 
</p>

{% include imported_disclaimer.html %}
