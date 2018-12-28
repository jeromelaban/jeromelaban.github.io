---
layout: post
title: "Using Google Gears to find Montreal's Bus Stops"
date: 2008-08-31 20:08:00 -0400
comments: true
category: Archive
tags: [".NET", "Misc", "Bus Stop Locator"]
redirect_from: ["/post/2008/08/31/Using-Google-Gears-to-find-Montreals-Bus-Stops.aspx", "/post/2008/08/31/using-google-gears-to-find-montreals-bus-stops.aspx"]
author: jay
---
<!-- more -->
<p>
<em>Cet article est &eacute;galement disponible <a href="http://blogs.codes-sources.com/jay/archive/2008/09/01/utiliser-google-gears-pour-trouver-les-arrets-de-bus-de-montreal.aspx">en francais ici</a>.</em>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">It&#39;s been a while since I&#39;ve posted on this blog. This time, I will not be talking about bluetooth, but still about some .NET powered code&nbsp;:) </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I&#39;ve been busy lately, but I&#39;ve found some time to work on something that will help me a lot, and I think a lot of Windows Mobile users and mobile users in general. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Montreal&#39;s Bus network is somehow large, but its representation in the digital world is quite poor, and inexistent when talking about mobile internet. The </font><a href="http://www.stm.info/"><font face="trebuchet ms,geneva" size="2">web site in question</font></a><font face="trebuchet ms,geneva" size="2"> is generating some quite large pages and is not suited for mobile web browsing. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Most of the time, you may want to know the schedule of the next bus, and this is quite hard to get this way. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">There&#39;s been some effort lately to offer </font><a href="http://directionfilms.net/stmmobile/index_en.html"><font face="trebuchet ms,geneva" size="2">this kind of service</font></a><font face="trebuchet ms,geneva" size="2"> on the iPhone, but I wanted to give the opportonity to other users to have the same information, with some Geo Localization features. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This is where </font><a href="http://gears.google.com"><font face="trebuchet ms,geneva" size="2">google gears</font></a><font face="trebuchet ms,geneva" size="2"> comes into action, where their latest release offers a </font><a href="http://code.google.com/apis/gears/api_geolocation.html"><font face="trebuchet ms,geneva" size="2">Geo-Location API</font></a><font face="trebuchet ms,geneva" size="2">, which approximates a position using the nearest GSM cells location. Unfortunately, it only works on Windows Mobile devices. But don&#39;t worry, if you don&#39;t have a Gears enable device, it will still work ! You&#39;ll only have to type a bit, by entering your streets intersection. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">After getting that location, I&#39;m querying a database (using Linq to SQL) to get the nearest Bus Stops and their next schedule. I&#39;m also querying </font><a href="http://code.google.com/apis/maps/"><font face="trebuchet ms,geneva" size="2">Google Maps</font></a><font face="trebuchet ms,geneva" size="2"> to get some markers pointing at the bus stops. That can be helpful since the Geo-Location is only an approximation by nature, because of the GSM &#39;triangulation&#39;. It can also be used to query the schedule of a specific bus stop, using the number placed at the bottom of the bus stop signs. A small plus here, compared to the original site, is that schedules from the past half hour are still visible, making possible to have determine if a bus has missed its schedule using a great long street.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2"><strong>Anyway, if you&#39;re in Montreal and have an internet connected device (or a normal PC), give it a try by connecting to this adress : </strong></font><a href="/stm"><font face="trebuchet ms,geneva" size="2"><strong>http://jaylee.org/stm</strong></font></a><font face="trebuchet ms,geneva" size="2"> </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Any comments or suggestions are welcome ! </font>
</p>

{% include imported_disclaimer.html %}
