---
layout: post
title: "Montreal Bus Stop Locator Site Updates"
date: 2008-09-07 09:06:00 -0400
comments: true
category: Archive
tags: ["Bus Stop Locator"]
redirect_from: ["/post/2008/09/07/Montreal-Bus-Stop-Locator-Site-Updates.aspx", "/post/2008/09/07/montreal-bus-stop-locator-site-updates.aspx"]
author: jay
---
<!-- more -->
<p>
<em>Cet article est disponible en <a href="http://blogs.codes-sources.com/jay/archive/2008/09/07/mise-a-jour-du-moteur-de-recherche-des-arrets-de-bus-de-montreal.aspx">fran&ccedil;ais ici</a>. </em>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
So far, the response has been great for my <a href="http://jaylee.org/stm">little utility</a>, even though I&#39;m not making an active advertising.
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
Using some Microsoft terminology, I&#39;m <a href="http://en.wikipedia.org/wiki/Eat_one%27s_own_dog_food">dogfooding</a> my own application and I found a few points that could be enhanced a bit :
</font>
</p>
<ul>
	<li><font face="trebuchet ms,geneva" size="2">Now, if using Google Gears, you are not automatically redirected to the stop list for your location. You simply see a small map of where you&#39;re supposed to be, and you&#39;re offered a other button to choose to use that location.</font></li>
	<li><font face="trebuchet ms,geneva" size="2">Some error messages are displayed when typing incorrect or missing data. That could be confusing...</font></li>
	<li><font face="trebuchet ms,geneva" size="2">All maps are now using all the available width of the screen. That means that the image is not resized poorly on a mobile device. Still, I wonder how the iPhone wil behave with this...</font></li>
	<li><font face="trebuchet ms,geneva" size="2">I fixed a bit the distance display to display Km instead of meters if appropriate, even though if you are in an other city, you&#39;ll still be offered some reaaaally long distance.<br />
	I&#39;m still wondering if I need to filter out people that are outside Montreal. The point of this site is also to be a technological proof-of-concept for GeoLocation, so even if the distance is not usable, it&#39;s still a valuable information.</font></li>
	<li><font face="trebuchet ms,geneva" size="2">I&#39;ve also added some street name samples to help people that are not from Montreal to see what the site can do. I&#39;ve taken a random intersection and bus stop code that can be typed in directly. </font></li>
	<li><font face="trebuchet ms,geneva" size="2">I also made some XHTML compliance, for what it&#39;s worth :)</font></li>
</ul>
<p>
<font face="trebuchet ms,geneva" size="2">
Now, I think I&#39;ll try to make sure that Google search will find the site in both French and English languages because right now, I&#39;m having the same problems I did have with my Remote Control software pages. Google&#39;s crawling without specifying a language, so for this site, English is going to come out, since it is the default language. Maybe I&#39;ll add a &quot;/fr&quot; and &quot;/en&quot; virtual base for each of them. 
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
I&#39;m always interested in any suggestions or bug reports !
</font>
</p>

{% include imported_disclaimer.html %}
