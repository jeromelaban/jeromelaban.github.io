---
layout: post
title: "Building in Parallel Across Multiple Build Agents in TFS2012 for Metro Apps"
date: 2013-03-30 12:50:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8", "Windows Phone Dev"]
redirect_from: ["/post/2013/03/30/Building-in-Parallel-Across-Multiple-Build-Agents-with-TFS-2012-with-Gated-Checkin.aspx", "/post/2013/03/30/building-in-parallel-across-multiple-build-agents-with-tfs-2012-with-gated-checkin.aspx"]
author: jay
---
<!-- more -->
<p><em>TL;DR: Using an upgraded (and fixed) </em><a href="http://blogs.msdn.com/b/jimlamb/archive/2010/09/14/parallelized-builds-with-tfs2010.aspx"><em>Parallel Build Process Template</em></a><em> allows to use multiple TFS2012 build agents simultaneously, which can be more than welcome when building metro apps that target all three supported platforms. A build that took 11 minutes can go down to 3.5 minutes.</em></p>
<p><em>Download the Parallel Build Process Template for TFS2012 <a href="http://jaylee.org/files/ParallelTemplate.11.1.zip">here</a>.</em></p>
<p>&nbsp;</p>
<p>CI is a wonderful feature, especially when associated with <a href="http://msdn.microsoft.com/en-us/library/dd787631.aspx">Gated Checkins</a>.</p>
<p>You&rsquo;re certain that what&rsquo;s in your source control is in line with your build definition and constraints, and that there is always a binary that respects a minimum set of rules. This does not ensure that your app is bug free, but still, that&rsquo;s a minimum.</p>
<p>&nbsp;</p>
<h2>Build time matters</h2>
<p>The downside of this validation is that there cannot be multiple builds running at the same time. This can become a bottleneck when multiple developers checkin within the duration of a single build run.</p>
<p>This means that the longer your build gets, the longer a developer might wait for its task completion because of a long build queue,&nbsp;and increase its task switching cost. If a build fails, the developer needs to unshelve its changes, make the necessary adjustments, then check-in again.</p>
<p>Below 4 minutes of build time, this stays in the acceptable range where the developer&rsquo;s task context may not be lost if the build fails.</p>
<p>[more]</p>
<h2>The case of Metro Apps</h2>
<p>Metro apps are a bit tricky in this regard, because when a .NET based app needs a native dependency, such as <a href="http://visualstudiogallery.msdn.microsoft.com/bb764f67-6b2c-4e14-b2d3-17477ae1eaca">Bing Maps SDK</a>, then it is not possible to use the Any CPU configuration anymore.</p>
<p>In such case, building the app requires the target the three Metro Apps supported platforms: ARM, x86, x64.</p>
<p>The msbuild configuration for metro apps&nbsp;enforces that all projects for a single platform be compiled under the same platform, meaning that every assembly must be build three times to produce a complete app submission package.</p>
<p>If your build takes 4 minutes for one platform, then multiply it by three (or more)&nbsp;for all platforms. This is where the CI build time gets in the way of development efficiency.</p>
<p>And if you&rsquo;re multi-targeting to include Windows Phone apps, and desktop apps that share code with metro apps, Build Time can get out of hand very quickly. And I'm not even&nbsp;mentioning obfuscation...</p>
<p>&nbsp;</p>
<h2>Building in Parallel</h2>
<p>I&rsquo;m a fan of msbuild projects authoring (over Workflow authoring) but in this case, msbuild does not cut it. This will probably please my&nbsp;TFS MVP&nbsp;friend <a href="http://blogs.codes-sources.com/etienne/default.aspx">Etienne Margraff</a>, which gives a lot of love to customizing its workflows.</p>
<p>When building using msbuild, parallelization can occur inside a single solution, for a single Configuration/Platform combination. While this improve the build time a bit, this does not work at all for multiple combinations.</p>
<p>There&rsquo;s also the trick that the guys over a the <a href="http://msbuildextensionpack.codeplex.com/">MSBuild Extension Pack</a> that enables the execution of multiple tasks inside of a single MSBuild project file to <a href="http://mikefourie.wordpress.com/2012/02/29/executing-msbuild-targets-in-parallel-part-1/">run</a> in <a href="http://mikefourie.wordpress.com/2012/04/18/executing-msbuild-targets-in-parallel-part-2/">parallel</a>. The problem with this is that the same source tree is used for all builds at the same time, which can cause trouble over shared files or resources. You have to make sure that all projects include the platform in their respective outputs, but even with that, you can&rsquo;t be sure that a race condition might not happen at any time.</p>
<p>&nbsp;</p>
<h2>Using the Parallel Build Process Template</h2>
<p>A few years back, a <a href="http://blogs.msdn.com/b/jimlamb/archive/2010/09/14/parallelized-builds-with-tfs2010.aspx">Jim Lamb posted a custom Build Process Template</a> that allows to build multiple configurations in parallel. The beauty of this template is that it allows for each configuration to run on a separate build agent !</p>
<p>This means that the build can be split on multiple machines, running each on their own copy of the source tree, avoiding race conditions over shared resources from the source tree.</p>
<p>I&rsquo;ve migrated a few projects over to this updated template and the results are very interesting. <strong>A build that would take 11 minutes to run would fall down to 3.5 minutes when ran over three build agents on the same machine.</strong></p>
<p>One thing though, the original template is built for TFS2010, so I had to port it over to TFS2012 using <a href="http://blogs.msdn.com/b/jpricket/archive/2012/07/17/tfs-2012-cleaning-up-workflow-xaml-files-aka-removing-versioned-namespaces.aspx">this cleanup tool from Jason Prickett</a>. If you don&rsquo;t want to migrate it by yourself, download the file at the top of the post.</p>
<p>I also had to modify the original error handling, because if one of the configuration build failed, the overall build would not fail. In the context of a gated checkin, this can be problematic&hellip;</p>
<p>&nbsp;</p>
<h2>Getting a bit farther</h2>
<p>The interesting thing with this parallel build template is that it can be used to parallelize validation or non-output generating tasks. For instance, creating a zip file of the source tree, validating some portions of the xaml for best practices, running some non-interactive unit tests&hellip; and so on.</p>
<p>This relies on scaling out, instead of scaling up, which is more than welcome, particularly in virtualized environments.</p>
{% include imported_disclaimer.html %}
