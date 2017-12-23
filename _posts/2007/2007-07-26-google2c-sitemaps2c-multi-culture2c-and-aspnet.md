---
layout: post
title: "Google, Sitemaps, Multi-Culture, and ASP.NET"
date: 2007-07-26 13:48:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/07/26/Google2c-Sitemaps2c-Multi-Culture2c-and-ASPNET", "/post/2007/07/26/google2c-sitemaps2c-multi-culture2c-and-aspnet"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">A&nbsp;while ago, by watching my web server&nbsp;logs and some statistics on <a href="http://analytics.google.com">Google Analytics</a>, I discovered that writing content&nbsp;in english and having a server based&nbsp;in the US has the effect of my site being mostly visible from within the US. It is also a bit visible when using localized google sites, but the page rank is quite low... So I&#39;m tried to improve my page rank.</font><font face="Tahoma" size="2"><br />
<br />
<a href="https://www.google.com/webmasters">Google Webmaster Tools</a>&nbsp;is&nbsp;an interesting tool shows rather precisely how Google scans a website. It can&nbsp;alert&nbsp;if it encounters server errors (500), broken links (404) or any other unusual error for links that point to a website. That tool helped me fixing an URL&nbsp;rewriting bug a while ago.<br />
<br />
There&#39;s also a section called Sitemap, which allows you to provide explicitly a list of urls from your site. This is more of a hint than an order, but that helps to have the content being indexed a bit more quickly.<br />
<br />
To have my content displayed in multiple languages, namely English, French and Spanish, I tried multiple techniques, one being the use of the automatic culture selection&nbsp;of ASP.NET. This is fine when a &quot;real&quot; user is browsing a site, but for the indexing this is not effective. The automatic culture is based on the accept-language HTTP header, which is not set by the google crawler... Google then only indexes the default language, which is English for me.<br />
<br />
Then I tried the HTTP query parameters, by adding a <font face="Courier New">&amp;lang=fr-FR</font> at the end of each page. Well, that not good enough either, since it seems that Google is not indexing dynamic content pretty well.<br />
<br />
Finally,&nbsp;I tried adding the culture just before the aspx extension (<font face="Courier New">default.fr-FR.aspx</font>), which &quot;fools&quot; google into thinking this is static content. This is an extension of the url rewriting technique I use to have my urls containing the titles of my blog posts. <br />
<br />
This time it worked! I have my remote control page listed in the first page of Google France and Spain with &quot;Bluetooth Remote Control&quot; query. Also, to make sure that I have links to each and every language available without explicitly placing them in the sitemap, I also placed &quot;language&quot; flag image links at the top of each page. This is not useful for real users, since culture will probably be selected correctly in the first place, but it&#39;ll help indexing.</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">I also noted that having <font face="Courier New">Hn</font> HTML tags improves indexing.&nbsp;Search engine rank optimization&nbsp;is a strange and obscure&nbsp;science... :)</font>
</p>
<p align="justify">
<font face="Tahoma" size="2"><em>PS. :&nbsp;Sorry for the bad spanish language, it&#39;s a&nbsp;raw Google translator cut and paste. I&#39;m in the process of having the text checked by a spanish&nbsp;speaker :)</em></font>
</p>

{% include imported_disclaimer.html %}
