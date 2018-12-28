---
layout: post
title: "BTRemote Control and Windows XP 64 Bits "
date: 2006-05-26 23:21:05 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2006/05/26/BTRemote-Control-and-Windows-XP-64-Bits-.aspx", "/post/2006/05/26/btremote-control-and-windows-xp-64-bits-.aspx"]
author: jerome
---
<!-- more -->
<P align=justify><FONT face=Tahoma><FONT size=2>Quite a few people have now been using the </FONT><A href="http://btremote.selfip.net/"><FONT size=2>latest beta</FONT></A><FONT size=2> for a while, and I get most of the time feature requests, sometimes non critical bugs like the Remote Client not connecting if the host PC is not discoverable, and not so often blocking bugs.</FONT></FONT></P>
<P align=justify><FONT face=Tahoma size=2>The one Krijn Wijnands (thanks !)&nbsp;has uncovered might involve Windows XP&nbsp;64 Bits.&nbsp;The remote client does not connect to the server, and he's using an hardware configuration that's proved to work fine.</FONT></P>
<P align=justify><FONT face=Tahoma size=2>I do not own a 64 bits machine and if any of readers of this blog or user of this utility uses Windows XP 64 bits, please leave me a message. I'll be glad to hear from you.</FONT></P>
<P align=justify><FONT face=Tahoma size=2>The client version of this software do log much out of the debug build for now, but I'll add some more for extensive diagnostics.</FONT></P>
<P align=justify><FONT face=Tahoma size=2>From a pure developer point of view, .NET 2.0 64 Bits should not change a thing, but I suspect a P/Invoke issue that I've not covered. That should give me a reason to upgrade my server to 64 Bits ;) For the sake of the testing, of course.</FONT></P>
<P align=justify><FONT face=Tahoma size=2>For the next release, I'll be adding generic support for applications, configurable using a static xml file (there's no point on modifying this at runtime)</FONT></P>
<P align=justify><FONT face=Tahoma size=2>If you have some other&nbsp;wishes that come to mind, feel free to speak !</FONT></P>
{% include imported_disclaimer.html %}
