---
layout: post
title: "ASP.NET Remote Debugging, Windows XP SP2 and .NET Framework 2.0"
date: 2004-12-13 11:38:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2004/12/13/ASPNET-Remote-Debugging2c-Windows-XP-SP2-and-NET-Framework-20", "/post/2004/12/13/aspnet-remote-debugging2c-windows-xp-sp2-and-net-framework-20"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">I did not have to create and debug any ASP.NET application for a long time, but since I&#39;m creating an online Questions/Answers application, I had to use the really nice debugging features brought by Visual Studio .NET.</font>
</p>
<p>
<font face="Tahoma" size="2">To be specific, I did not have to debug any remote web site since I installed the Windows XP SP2. My configuration is quite simple, I host my application on a development Windows 2003 Server and I design with VS.NET on my Windows XP machine.</font>
</p>
<p>
<font face="Tahoma" size="2">So when I trying to debug my web application, all I could get was : <font face="Courier New">&quot;Unable to debug the application&quot;</font> or <font face="Courier New">&quot;The remote debugger could not connect to the local machine&quot;</font> or a really helpfull <font face="Courier New">&quot;Cannot debug process&quot;</font>.</font>
</p>
<p>
<font face="Tahoma" size="2">The first reaction when seeing things like this is to check the local and remote <strong>VS Developers </strong>and <strong>Debugger Users</strong>&nbsp;security groups. But every thing was fine there... In fact, the problem lies in the DCOM security configuration. Installing the SP2 removed the right for the <strong>anonymous</strong> account to use DCOM remotely... but not for the <strong>Everyone</strong> account. Odd.<br />
The only thing to do is to get there : <strong>Run / dcomcnfg.exe / Component Services / Computers / My Computer / Properties / COM Security / Edit Limits</strong>, and to check &quot;<strong>Remote Access / Allow</strong>&quot;. Easy.</font>
</p>
<p>
<font face="Tahoma" size="2">This solved my first problem, the remote debugging. The second one is still ASP.NET debugging, but locally this&nbsp;time.</font>
</p>
<p>
<font face="Tahoma" size="2">I have both VS.NET 2003 and 2005 installed on my local machine, and so are both 1.1 and 2.0 .NET Framework versions. Installing 2.0 over 1.1 changes the default framework used by the Windows XP IIS to version 2.0 which breaks the 1.1 debugger :)<br />
The simple thing to do here is this : <strong>IIS / Web Sites / [WebSite] / Properties / ASP.NET</strong>, then select <strong>1.1.4322</strong> for the ASP.NET Version field and that&#39;s it :)</font>
</p>
<p>
<font face="Tahoma" size="2">Man, I really like being able to debug my Web Applications, I really missed it ! (<em>And so was Karouman actually, who was debugging blindly by guessing exceptions :p</em>)</font>
</p>

{% include imported_disclaimer.html %}
