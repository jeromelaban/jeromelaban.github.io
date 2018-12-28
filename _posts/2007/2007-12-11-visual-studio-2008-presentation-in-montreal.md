---
layout: post
title: "Visual Studio 2008 Presentation in Montréal"
date: 2007-12-11 21:46:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/12/11/Visual-Studio-2008-Presentation-in-Montreal.aspx", "/post/2007/12/11/visual-studio-2008-presentation-in-montreal.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">This last Monday, there&#39;s been a presentation&nbsp;for the </font><a href="http://www.guvsm.net/"><font face="Tahoma" size="2">GUVSM</font></a><font face="Tahoma" size="2">&nbsp;of Visual Studio 2008 at Microsoft Montr&eacute;al by </font><a href="http://guy.dotnet-expertise.com/"><font face="Tahoma" size="2">Guy Barrette</font></a><font face="Tahoma" size="2">, a Microsoft Regional Directory. The room was a bit crowded but there was a pretty good ambience. This was my first time there and I met a former distant colleague of mine from Winwise, </font><a href="http://weblogs.asp.net/lduveau/"><font face="Tahoma" size="2">Laurent Duveau</font></a><font face="Tahoma" size="2">. </font>
</p>
<p align="justify">
<font face="Tahoma" size="2">On the menu, an overview of only the tooling that can be found in Visual Studio 2008. Not that C# 3.0 and .NET 3.5 are not interesting, far from it, but this will probably be the point of another session.</font>
</p>
<h3 align="justify">Multi Targeting </h3>
<p align="justify">
<font face="Tahoma" size="2">VS2005 forced the migration to .NET 2.0 when coming from VS2003. VS2008 does not enforce the migration to .NET 3.5, and leaves a choice of targeting .NET 2.0, 3.0 and 3.5. Solution and Project files don&#39;t change much, which means a smoother transition from VS2005 to VS2008. Some developers can run VS2005 and some others VS2008 without any difficulties. The only downside about the multi-targeting is that Microsoft dropped the support of .NET CF 1.0 and forces the conversion of projects to .NET CF 2.0.<br />
The target framework can also be changed afterward, upgrade or downgrade. At that point, this is a good feature but if you have a solution that contains 100+ projects, this can be a repetitive task to migrate each and every one of them. I&#39;ll discuss that in an other post, but I wrote a small tool that converts every csproj file it can find to a .NET 3.5 project. That gave me the opportunity to test XLinq, by the way :)</font>
</p>
<h3 align="justify">VSTO (Visual Studio Tools for Office) </h3>
<p align="justify">
<font face="Tahoma" size="2">There is now no more the need to install add-ins to create VSTO addins, support for Office 2003 and 2007 is now included out of the box. Guy showed us how to create a simple plugin that creates a task pane which opens via a custom Excel Ribbon button, fills up from a database, and then fills the selected cell with some info. Pretty interesting stuff and easy to setup. It seems that now the deployment of VSTO addins has been greatly enhanced, compared to what could be done with VSTO and VS2005. </font>
</p>
<h4 align="justify">WCF</h4>
<p align="justify">
<font face="Tahoma" size="2">WCF&nbsp;now has an integrated webservice testing tool, that allows to call your WCF webservice without having to create the little testing project in Console that calls the webservice to test it. I&#39;m guessing that it&#39;ll be useful only in the first few minutes of the webservice development, but after that... Maybe for the demos :) </font>
</p>
<h3 align="justify">Javascript </h3>
<p align="justify">
<font face="Tahoma" size="2">Javascript has a better support, especially for debugging. It is now possible to step into methods defined in javascript in the actual files, not in their memory representation. It is also possible to have custom help for javascript methods ! This is a hack using xml comments like in the ones found in C#, but that&#39;s helpful.&nbsp;I kind of hate the javascript development partly for the lack of debugging features and intellisense, but with this, I might hate it a little bit less... </font>
</p>
<h3 align="justify">SQL Server CE </h3>
<p align="justify">
<font face="Tahoma" size="2">It is a port of the SQL CE engine found on Mobile Devices. It&#39;s a small database engine similar to Jet could have been back in its days, but is compatible with SQL server to some extents. There&#39;s no stored procedures, no security, but it&#39;s a cheap way to store data instead of maybe&nbsp;using XML files to store data. </font>
</p>
<h3 align="justify">Database Cache </h3>
<p align="justify">
<font face="Tahoma" size="2">which is a out of the box component that is able to create a snapshot of the database, and handle the synchronization of the snapshot later in time.</font>
</p>
<h3 align="justify">Mobile Development </h3>
<p align="justify">
<font face="Tahoma" size="2">And finally a few words on mobile side, for which it is now possible to test security scenarios, with third party executable signature enabled, for instance.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">This was a pretty rich session, with too much to say probably, but I learnt a few things along the way. I&#39;ll definitely attend the next one in january !</font>
</p>

{% include imported_disclaimer.html %}
