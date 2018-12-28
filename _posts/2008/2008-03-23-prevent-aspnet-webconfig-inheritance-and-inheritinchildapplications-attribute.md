---
layout: post
title: "Prevent ASP.NET web.config inheritance, and inheritInChildApplications attribute"
date: 2008-03-23 12:34:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/03/23/Prevent-ASPNET-webconfig-inheritance-and-inheritInChildApplications-attribute.aspx", "/post/2008/03/23/prevent-aspnet-webconfig-inheritance-and-inheritinchildapplications-attribute.aspx"]
author: jerome
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2">Since I&#39;ve changed my top level blog engine, I&#39;ve had some troubles with YAF, the forum engine I&#39;m using for my Remote Control software. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">The forum I&#39;m using is in a child directory, which is a child application defined in IIS as an other application. The </font><a href="http://dotnetblogengine.net/" title="BlogEngine.NET"><font face="trebuchet ms,geneva" size="2">BlogEngine.NET</font></a><font face="trebuchet ms,geneva" size="2"> disables the use of Sessions, and </font><a href="http://forum.yetanotherforum.net/" title="Yet Another Forum"><font face="trebuchet ms,geneva" size="2">YAF</font></a><font face="trebuchet ms,geneva" size="2"> requires sessions to be enabled, plus BlogEngine.NET adds some custom HTTP handlers, which incidentally are not known but the forum application. This is quite a mess, and to be able&nbsp;to get both applications running without fine tuning each one to work with the other, I had to use&nbsp;the little known attribute </font><a href="http://msdn2.microsoft.com/en-us/library/system.configuration.sectioninformation.inheritinchildapplications.aspx" title="SectionInformation::InheritInChildApplications Property "><font face="trebuchet ms,geneva" size="2">inheritInChildApplications</font></a><font face="trebuchet ms,geneva" size="2">. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This attribute prevents an application from passing its configuration as a default to child applications. Using this attribute is a little tricky, and has to be used this way :</font> 
</p>
<p>
<font face="courier new,courier">[code:xml]<br />
&lt;!-- Root web.config file --&gt;<br />
&lt;?xml version=&quot;1.0&quot;?&gt;<br />
&lt;configuration&gt;<br />
&nbsp; &lt;location path=&quot;.&quot; inheritInChildApplications=&quot;false&quot;&gt; <br />
&nbsp;&nbsp;&nbsp; &lt;system.web&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;compilation debug=&quot;false&quot; /&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;!-- other configuration attributes --&gt;<br />
</font><font face="courier new,courier">&nbsp;&nbsp;&nbsp; &lt;/system.web&gt; <br />
&nbsp; &lt;/location&gt;<br />
&lt;/configuration&gt;<br />
[/code]</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This way, any child application defined below this application will not use the current configuration. There&#39;s some mystery around the inheritInChildApplications&nbsp;attribute; it is not defined in the dotnetconfig.xsd file and it still is a rather helpful configuration option...</font>
</p>

{% include imported_disclaimer.html %}
