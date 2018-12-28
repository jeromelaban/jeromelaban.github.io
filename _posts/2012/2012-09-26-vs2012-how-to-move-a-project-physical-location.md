---
layout: post
title: "VS2012: How to change a project’s physical location"
date: 2012-09-26 19:02:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2012/09/26/VS2012-How-to-move-a-project-physical-location.aspx", "/post/2012/09/26/vs2012-how-to-move-a-project-physical-location.aspx"]
author: jay
---
<!-- more -->
<p><em><a href="http://blogs.codes-sources.com/jay/archive/2012/09/26/VS2012-Comment-changer-l-emplacement-physique-d-un-projet.aspx">Ce billet est disponible en francais.</a></em></p>
<p>Physically moving a project in a Visual Studio solution has always been cumbersome. It&rsquo;s often needed if you rename your [insert favorite language] project&rsquo;s folder.</p>
<p>In previous versions of Visual Studio, you would move the project into the new location and either :</p>
<ul>
<li>Remove the project from the solution and add it back using the new location</li>
<li>Go the warrior way, and hack around the very user-friendly .sln file format and change all the references</li>
</ul>
<p>That&rsquo;s been an annoying situation, but that could be worked around.</p>
<p>But every time, you&rsquo;d lose your solution wide settings, such as default startup project or build configuration. And if you have many build configuration, I think you understand the pain of restoring all these settings properly.</p>
<p>Fortunately, there&rsquo;s been a very small change in VS2012 that allows to provide a new location of a project file, if it failed to load because it was not found.</p>
<p>[more]</p>
<p>To make this appear :</p>
<ul>
<li>Close your solution in VS2012</li>
<li>Move your project to the new location</li>
<li>Open your solution</li>
<li>Select the project that failed to load</li>
<li>In the Properties tool window, there an editable &ldquo;File path&rdquo; entry that allows you to select the new project location</li>
<li>Set the new path</li>
<li>Right click on the project and click reload</li>
</ul>
<p>&nbsp;</p>
<p>You&rsquo;re done !</p>
<p>Small improvement, big time saver when refactoring your solution !</p>
{% include imported_disclaimer.html %}
