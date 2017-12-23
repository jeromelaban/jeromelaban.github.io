---
layout: post
title: "VS2012: Create multiple file links with a drag and drop between two projects"
date: 2012-09-26 19:08:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2012/09/26/VS2012-Create-multiple-file-links-with-a-drag-and-drop-between-two-projects", "/post/2012/09/26/vs2012-create-multiple-file-links-with-a-drag-and-drop-between-two-projects"]
author: jay
---
<!-- more -->
<p><em><a href="http://blogs.codes-sources.com/jay/archive/2012/09/26/VS2012-Creer-plusieurs-fichiers-lies-avec-un-drag-and-drop-entre-deux-projets.aspx">Ce billet est disponible en francais.</a></em></p>
<p>With more and more platforms appearing in the Microsoft landscape, multi-targeting is getting more and more common.</p>
<p>Multi-targeting is used when the <a href="http://msdn.microsoft.com/en-us/library/gg597391.aspx">Portable Libraries</a> don't cut it. It's interesting when you're not in the mood of making too many abstractions to cope with missing classes and methods for your selected platforms intersection.</p>
<p>If you go with the multi-targeting, you've got multiple choices, such as using <a href="http://msdn.microsoft.com/en-us/library/ff921108(v=pandp.20).aspx">Project Linker</a>,&nbsp;and create multiple projects for the same source files and link the files between your two projects.</p>
<p>At this point though, Project Linker does not support VS2012, so you're left with the <a href="http://msdn.microsoft.com/en-us/library/9f4t9t92(v=vs.100).aspx">Add as Link</a> trick in Visual Studio, which does not support folders.</p>
<p>Well, in Visual Studio 2012, there a new feature that allows just that.</p>
<p>[more]</p>
<p>Here's what to do :</p>
<ul>
<li>In your source project, select your files and folders to link to the other project</li>
<li>Drag then to the new location in the other project, but don't drop just yet</li>
<li>Hold Ctrl + Shift, then drop.</li>
</ul>
<p>&nbsp;</p>
<p>There you have it, a replica of your folders and files.&nbsp;You'll notice that folders are not links (folder links do not exist), but files are. Oh well, it's better that nothing !</p>
<p>You'll also notice that if you do the operation again, with new files or folders, only the new files and folders will be linked, and no error will be thrown because the other files or folders already exist.</p>
<p>Now you can multi-target your source files very easily with a drag and drop.</p>
<p>Not a ground breaking feature, but definitely another time saver !</p>
{% include imported_disclaimer.html %}
