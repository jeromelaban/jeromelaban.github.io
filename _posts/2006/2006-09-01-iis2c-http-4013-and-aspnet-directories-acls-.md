---
layout: post
title: "IIS, HTTP 401.3 and ASP.NET directories ACLs"
date: 2006-09-01 22:09:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2006/09/01/IIS2c-HTTP-4013-and-ASPNET-directories-ACLs-.aspx", "/post/2006/09/01/iis2c-http-4013-and-aspnet-directories-acls-.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2" color="#000000">A few days ago, on a newly installed web server with all the appropriate security patches applied, I kept having the same error on every ASP.NET 1.1 application&nbsp;I was running :</font>
</p>
<p align="justify">
<font face="Courier New" size="2" color="#000000">HTTP Error 401.3 - Unauthorized: Access is denied due to an ACL set on the requested resource.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2" color="#000000">At first, the reflex is to check all the permissions of the mapped physical directory, that they match the Application Pool identity, the guest identity (IUSR_<em>Machine</em> on my server) and for some configurations, the impersonated identity any ASP.NET configuration. Even with all these checks, any ASP.NET application was returning the same 401.3 error for anonymous users... </font>
</p>
<p align="justify">
<font face="Tahoma" size="2" color="#000000">Well, it turns out that the ACL of the <font face="Courier New">%SystemRoot%\Microsoft.NET\Framework\v1.1.4322</font> is important too... I don&#39;t know how the ACL got changed in the first place, and I don&#39;t know either how I came to check on these ACL, but that can waste a lot of time...</font>
</p>

{% include imported_disclaimer.html %}
