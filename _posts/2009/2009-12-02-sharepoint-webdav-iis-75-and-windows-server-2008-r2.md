---
layout: post
title: "SharePoint WebDAV, IIS 7.5 and Windows Server 2008 R2"
date: 2009-12-02 18:02:49 -0500
comments: true
category: Archive
tags: ["Sharepoint"]
redirect_from: ["/post/2009/12/02/SharePoint-WebDAV-IIS-75-and-Windows-Server-2008-R2", "/post/2009/12/02/sharepoint-webdav-iis-75-and-windows-server-2008-r2"]
author: jay
---
<!-- more -->
<p>A neat feature of Sharepoint 2007 (or WSS 3.0) is the ability to browse the content of a site as if it were a network drive. This is done under the hood by using WebDAV, a standard protocol that Microsoft used to implement this feature.</p>  <p>If you happen to have to install WSS 3.0 on a Windows 2008 R2 Box, you’ll quickly find out that this feature does not work properly, with interesting messages like “Access Denied” or “The network path could not be found” when trying to map a folder.</p>  <p>Using IIS 6.0, you’d simply need to make sure that the WebDAV Web Service Extension is “Prohibited”.</p>  <p>With IIS 7.5, there are multiple places dealing with WebDAV but only one to look at :</p>  <ul>   <li>Open the “Modules” configuration section for the Sharepoint web site </li>    <li>Find the “WebDAVModule” entry </li>    <li>Remove it, your’re done !</li> </ul>  <p>The interesting bit about this is that even though the WebDAV component is disabled in every possible section of the site, the module seems to intecept the WebDAV PROPFIND verb and returns a 405 (Not Allowed) error.</p>  <p>Since the verb is handled by an ASP.NET httpHandler, it never gets the chance to deal with it... and you can’t see your files in the Windows Explorer.</p>
{% include imported_disclaimer.html %}
