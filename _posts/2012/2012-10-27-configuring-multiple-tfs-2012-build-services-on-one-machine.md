---
layout: post
title: "Configuring Multiple TFS 2012 Build Services on one Machine"
date: 2012-10-27 12:18:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2012/10/27/Configuring-Multiple-TFS-2012-Build-Services-on-one-Machine.aspx", "/post/2012/10/27/configuring-multiple-tfs-2012-build-services-on-one-machine.aspx"]
author: jay
---
<!-- more -->
<p>Since TFS2010, because there&rsquo;s been the introduction of project collections, build agents are quite the trouble for IT admins.</p>
<p>On one hand, there&rsquo;s the fact that putting every project in a single collection can be a maintenance and project isolation nightmare, and on the other hand the fact that there can officially be one build controller per machine tied to a single project collection, forcing to have many machines to support continuous integration build environments.</p>
<p>Luckily, there&rsquo;s <a href="http://blogs.msdn.com/b/jimlamb/archive/2010/04/13/configuring-multiple-tfs-build-services-on-one-machine.aspx">been a workaround published two years ago</a> on how to run multiple build services on the same machine, and it&rsquo;s working pretty fine.</p>
<p>&nbsp;</p>
<p>However with TFS2012, this hack needs to be adjusted to work properly.</p>
<p>Prior to running the configuration tool, instead of setting this:</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">set TFSBUILDSERVICEHOST=buildMachine-collection2</span></p>
<p>The new environment variable is:</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">set TFSBUILDSERVICEHOST<span style="background-color: #ffff00;">.2012</span>=buildMachine-collection2</span></p>
<p>I&rsquo;m guessing that this can be used to run both 2010 and 2012 build services on the same machine without interferences, though I haven&rsquo;t tried it.</p>
<p>Just as a reminder, to use this feature, you need to set non-overlapping build working folders and different listening ports for every new instance. This will avoid conflicts&hellip;</p>
<p>Works pretty great, now with TFS2012 !</p>
{% include imported_disclaimer.html %}
