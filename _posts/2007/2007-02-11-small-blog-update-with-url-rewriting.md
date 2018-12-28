---
layout: post
title: "Small blog update, with Url Rewriting"
date: 2007-02-11 16:33:40 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/02/11/Small-blog-update-with-Url-Rewriting.aspx", "/post/2007/02/11/small-blog-update-with-url-rewriting.aspx"]
author: jerome
---
<!-- more -->
<P align=justify><FONT face=Verdana size=2>Since this little website upgrade to add the blog, posts have been indexed by google with an ugly internal <FONT face="Courier New">uniqueidentifier</FONT> of mine. </FONT></P>
<P align=justify><FONT face=Verdana size=2>Incidentally, that does not index well in google. Since the url to a post does not explain what the destination is about, I decided to add an url rewriting HttpHandler, to have the url containing the title of each post or category.</FONT></P>
<P align=justify><FONT face=Verdana size=2>The part about writing the url rewriter has been covered over and over, oh and over, so I won't talk about it here.</FONT></P>
<P align=justify><FONT face=Verdana size=2>Actually, I'm using <A href="http://crystaltech.com/">CrystalTech</A> web hosting --&nbsp;like <A href="http://www.white-clouds.org/">my brother Matthieu</A>&nbsp;-- which works really fine, and does a fine job providing ASP.NET 2.0 services. </FONT></P>
<P align=justify><FONT face=Verdana size=2>Their service is providing a tweaked Medium trust level to users, so hosted application have be configured that way. By tweaked, I mean that they re-enabled Reflection, Web, ODBC and OLEDB permissions from the Medium trust profile.</FONT></P>
<P align=justify><FONT face=Verdana size=2>Anyway, you have to deal with Partially Trusted Callers. My application uses a class library in a separate assemnly, which needs to have the AllowPartiallyTrustedCallers attribute set, because of the Medium trust. That avoids this :</FONT></P>
<P align=left><FONT face="Courier New" size=2>[SecurityException: That assembly does not allow partially trusted callers.]</FONT></P>
<P align=justify><FONT face=Verdana size=2>So, if you choose CrystalTech web hosting, remember to try your website under the Medium trust profile to avoid suprises, with this :</FONT></P>
<P align=left><FONT face="Courier New" size=2>&lt;system.web&gt;<BR>&nbsp;&nbsp; &lt;trust level="Medium"/&gt;<BR>&lt;/system.web&gt;</FONT></P>
<P align=justify><FONT face=Verdana size=2>Also, on the side of differences between VS2005 web server and IIS 6.0 behavior :&nbsp;The default handler for any url in VS2005 is ASP.NET and in IIS 6.0 it is the file system. Since my urls are now generated, they definitely don't exist on the file system. I just had to add an ".aspx" at the end of my virtual url to get IIS 6.0 handing it to ASP.NET runtime.</FONT></P>
<P align=justify><FONT face=Verdana size=2>Now I have my nice URLs :)</P></FONT>
{% include imported_disclaimer.html %}
