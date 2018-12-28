---
layout: post
title: "Bluetooth Remote Control for Windows Mobile 0.7.0-Beta1 : Widcomm Support"
date: 2007-03-08 18:01:00 -0500
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2007/03/08/Bluetooth-Remote-Control-for-Windows-Mobile-070-Beta1-Widcomm-Support.aspx", "/post/2007/03/08/bluetooth-remote-control-for-windows-mobile-070-beta1-widcomm-support.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">There it is ! The support for the Widcomm bluetooth&nbsp;stack on the desktop side. </font>
</p>
<p align="justify">
<font face="Tahoma" size="2">Download it here : <a href="/files/BTRemoteSetup-0.7.0-Beta1.msi">Bluetooth&nbsp;Remote Control for Windows Mobile 0.7.0-Beta1</a></font>
</p>
<p align="justify">
<font face="Tahoma" size="2">For now, this is a beta release, but this<em> should</em> work fine. </font><font face="Tahoma" size="2">You can now use Bluetooth Remote Control along with your A2DP headphones :)</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">On the dev side, working with a event driven API with C++ can pose some problems... The widcomm API is pretty different from the Microsoft stack. I have to admit, I prefer the Microsoft way of doing things, where you can call a blocking Receive method, instead of being notified asynchronously by some random thread, where I prefer managing my own threads.&nbsp;Also, handles are not closed if you application crashes, leaving a connection or an advertised service as an orphan. Annoying&nbsp;during&nbsp;the&nbsp;debugging...</font>
</p>
<p>
<font face="Tahoma" size="2">Well, it works now. Maybe next time I&#39;ll try to updated the mobile side...</font>
</p>

{% include imported_disclaimer.html %}
