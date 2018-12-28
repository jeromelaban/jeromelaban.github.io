---
layout: post
title: "Pocket IE and Setting IMG Src via JavaScript"
date: 2008-09-07 09:28:00 -0400
comments: true
category: Archive
tags: ["Bus Stop Locator"]
redirect_from: ["/post/2008/09/07/Pocket-IE-and-Setting-IMG-Src-via-JavaScript.aspx", "/post/2008/09/07/pocket-ie-and-setting-img-src-via-javascript.aspx"]
author: jay
---
<!-- more -->
<p>
<em>Cet article est disponible en <a href="http://blogs.codes-sources.com/jay/archive/2008/09/07/pocketie-et-assignation-du-src-d-un-element-img.aspx">francais ici</a>.</em> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">With the <a href="http://jaylee.org/stm">Montreal&#39;s Bus Stop Locator</a> web site, to be able to use Google Maps effectively, it&#39;s imperative to let the client&#39;s browser fetch the image by itself. The Google API does not allow a single key to grab a lot of maps from a single host, so redirecting the image is not an option.
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
I wanted to have the image Url generated so I could use the Width of an <font face="courier new,courier">HTML</font> body to be embedded in the Map query, so I created a small script that builds the URL and sets as the <font face="courier new,courier">SRC</font> of the <font face="courier new,courier">IMG</font> element that will contain the Map.
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
Well, that seems easy, but it&#39;s not... The <font face="courier new,courier">IMG</font> does fetch the image, but the element&#39;s size is not updated with the size of the new image. 
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
In my case, I&#39;m placing the image in a <font face="courier new,courier">TABLE</font> element, so I&#39;m getting the default cell size for my image. To fix that I also had to set the width and height of the <font face="courier new,courier">IMG</font> element, to get it to have a proper size.
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
PocketIE has little support for scripting, but it&#39;s somehow clear that it has been a hard to fit addition to the engine... Come on microsoft ! You can do better than that ! Have a look at what <a href="http://www.opera.com/products/mobile/">Opera&#39;s</a> been doing... </font>
</p>

{% include imported_disclaimer.html %}
