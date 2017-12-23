---
layout: post
title: "[VS2012] Temporarily disable the C# static code analysis for a whole VS instance"
date: 2013-03-30 11:52:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2013/03/30/VS2012-Temporarily-disable-the-C-static-code-analysis-for-a-whole-VS-instance", "/post/2013/03/30/vs2012-temporarily-disable-the-c-static-code-analysis-for-a-whole-vs-instance"]
author: jay
---
<!-- more -->
<p><em>TL;DR: It is possible to disable the Static Analysis phase in VS2012 projects by setting the <strong>DevDivCodeAnalysisRunType</strong> environment variable to &ldquo;<strong>Disabled</strong>&rdquo;.</em></p>
<p>&nbsp;</p>
<p>This will be a quick post, but which might save you a lot of time if you rely heavily on <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh441471(v=VS.85).aspx">Code Analysis (FxCop)</a>.</p>
<p>FxCop is definitely not known for its analysis speed, and when ran in every build, this takes a lot of time. I usually work on projects where FxCop is only enabled in the Release Configuration, which helps during development in Debug configuration.</p>
<p>But if you&rsquo;re bound to run in Release configuration, such as when profiling the app, or any other task that requires to build in that configuration, then having FxCop running every single time can be time consuming.</p>
<p>To avoid this, there are multiple choices :</p>
<ul>
<li>Use find and replace to change <strong>&lt;RunCodeAnalysis&gt;true&lt;/RunCodeAnalysis&gt;</strong> to <strong>&lt;RunCodeAnalysis&gt;false&lt;/RunCodeAnalysis&gt;</strong>, but then you have to remember to not check that into your source control (even if the <strong>Perform Code Analysis</strong> setting is set to <strong>Always</strong> in&nbsp;your CI Build definition),</li>
<li>Create an alternate configuration similar to <strong>Release</strong> that does not have the Static Code Analysis enabled, but changing configurations in VS2012&nbsp;can take time (even with the Update 2) and you&rsquo;ll have to maintain that configuration with the others,</li>
<li>Or you can use a little trick to disable the static code analysis for a whole Visual Studio instance.</li>
</ul>
<p>&nbsp;</p>
<p>That trick is a bit hidden, but here&rsquo;s how you can do this :</p>
<ul>
<li>Open a <strong>Developer Command Prompt for VS2012</strong></li>
<li>type <strong>set</strong> <strong>DevDivCodeAnalysisRunType=Disabled</strong></li>
<li>type <strong>devenv</strong></li>
</ul>
<p>&nbsp;</p>
<p>Build your solution and you won&rsquo;t have the static analysis running, without any modification to your solutions&rsquo; configuration or projects. Easy.</p>
<p>That said, remember that this is not documented and <strong>might change at any point in the future</strong> so don't rely on it too much.</p>
<p>&nbsp;</p>
{% include imported_disclaimer.html %}
