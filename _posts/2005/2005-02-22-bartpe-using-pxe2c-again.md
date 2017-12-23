---
layout: post
title: "BartPE using PXE, Again..."
date: 2005-02-22 11:31:09 -0500
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2005/02/22/BartPE-using-PXE2c-Again", "/post/2005/02/22/bartpe-using-pxe2c-again"]
author: jerome
---
<!-- more -->
<P><FONT face=Tahoma size=2>If you are in the Windows world, you might know that there is an upcoming <A href="http://www.microsoft.com/windowsserver2003/downloads/servicepacks/sp1/default.mspx" mce_href="http://www.microsoft.com/windowsserver2003/downloads/servicepacks/sp1/default.mspx">Service Pack for Windows 2003 Server</A>, which has reached the RC2 stage quite recently. This Service Pack brings a lot of improvements in the area of security, management and scaleability.</FONT></P>
<P><FONT face=Tahoma size=2>However, there are a few other interesting new features that have been added : <STRONG>The ability to boot an ISO image from the network</STRONG>. </FONT></P>
<P><FONT face=Tahoma size=2>The interesting thing about all this is that you don't have to install a RIS server anymore, only a simple TFTP server is needed (even under linux ;)). </FONT><FONT face=Tahoma size=2>The procedure is quite simple and can be found <A href="http://www.911cd.net/forums/index.php?showtopic=9685&amp;st=0" mce_href="http://www.911cd.net/forums/index.php?showtopic=9685&amp;st=0">here</A>. </FONT><FONT face=Tahoma size=2>The interesting part of this procedure is the content of the <STRONG>winnt.sif</STRONG> file&nbsp;:</FONT></P>
<P><FONT face=Tahoma color=#808080 size=2><EM>[SetupData]<BR>BootDevice = "<FONT color=#000000><STRONG>ramdisk(0)</STRONG>"</FONT><BR>BootPath = "\i386\System32\"<BR>OsLoadOptions = "/noguiboot /fastdetect /minint <STRONG><FONT color=#000000>/rdexportascd /rdpath=&lt;image&gt;.iso</FONT></STRONG>"</EM></FONT></P>
<P><FONT face=Tahoma size=2>I don't know where the guy got his informations from, but&nbsp;this solves <STRONG>a lot</STRONG> of my problems.</FONT></P>
<P><FONT face=Tahoma size=2>Now, we're going to be able to allow a multi network-boot, Win/NetBSD/Linux from any workstation on the network. All I have to do now is populate the iso image with the necessary software and we're good to go !</FONT></P>
<P><FONT face=Tahoma size=2>I know some people that are going to be&nbsp;happy :)</FONT></P>
{% include imported_disclaimer.html %}
