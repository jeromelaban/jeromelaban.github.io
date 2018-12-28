---
layout: post
title: "The file is not a valid Windows CE Setup file"
date: 2007-02-07 04:35:16 -0500
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2007/02/07/The-file-is-not-a-valid-Windows-CE-Setup-file.aspx", "/post/2007/02/07/the-file-is-not-a-valid-windows-ce-setup-file.aspx"]
author: jerome
---
<!-- more -->
<P><FONT face=Verdana size=2>If you've tried installing Bluetooth Remote Control on your device, and you are having trouble, such as getting this message :</FONT></P>
<P><FONT face="Courier New" size=2>The file RemoteControl.ARMV4.CAB is not a valid Windows CE Setup file</FONT></P>
<P><FONT face=Verdana size=2>You must be working on some Windows Moblile 2003 device. I did not have the time to fix this but if you are interested in getting it to work, you can proceed as follows, if you're familiar with a CE cab file structure :</FONT></P>
<UL>
<LI><FONT face=Verdana size=2>Expand the files on the PC using WinRar for instance,</FONT> 
<LI><FONT face=Verdana size=2>Rename 0SmartBT.001 to smarbt.exe</FONT> 
<LI><FONT face=Verdana size=2>Rename BTHelper.002 to bthelper.dll</FONT> 
<LI><FONT face=Verdana size=2>Just put&nbsp;those two files on the PPC wherever you want.</FONT></LI></UL>
<P><FONT face=Verdana size=2>You won't have a nice icon on your programs list, but at least it should help you run it.</FONT></P>
<P><FONT face=Verdana size=2>Enjoy&nbsp;!</FONT></P>
{% include imported_disclaimer.html %}
