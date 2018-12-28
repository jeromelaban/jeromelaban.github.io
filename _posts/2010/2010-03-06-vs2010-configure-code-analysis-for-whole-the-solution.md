---
layout: post
title: "[VS2010] Configure Code Analysis for the Whole Solution"
date: 2010-03-06 17:38:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/03/06/VS2010-Configure-Code-Analysis-for-Whole-the-Solution.aspx", "/post/2010/03/06/vs2010-configure-code-analysis-for-whole-the-solution.aspx"]
author: jay
---
<!-- more -->
<p><a href="http://blogs.codes-sources.com/jay/archive/2010/03/06/vs2010-Configurer-analyze-de-code-pour-toute-la-solution.aspx"><em>Cet article est disponible en francais.</em></a></p>
<p>In Visual Studio, configuring Code Analysis was a bit cumbersome. If you had more than a bunch (say 10), this could take a long time to manage and have a single set of rules for all your solution. You had to resort to update all the projects files by hand, or use a small tool that would edit the csproj file to set the same rules everywhere.</p>
<p>Not very pleasant, nor efficient. Particularly when you have hundreds of projects.</p>
<p>In Visual Studio 2010, the product team added two things :</p>
<ol>
<li>Rules are now in external files, and not embedded in the project file. That makes the rules reuseable in other projects in the solution. Nice.</li>
<li>There&rsquo;s a new section in the Solution properties named &ldquo;Code Analysis Settings&rdquo;, that allows to set rule files to use for single projects, and even better, for all projects ! Very nice.</li>
</ol>
<p>That option is also available from the &ldquo;Analyze&rdquo; menu, with &ldquo;Configure Code Analysis for Solution&rdquo;.</p>
<p>One gotcha there though, to be able to select all files, you can&rsquo;t use Ctrl+A but you have to select all files by selecting the first item, then hold Ctrl while selecting the last item. Maybe the Product Team will fix that for the release...</p>
<h3>Migrating Rules from VS2008</h3>
<p>If you&rsquo;re migrating your projects from VS2008, and were using code analysis there, you&rsquo;ll notice that the converter will generate a file named &ldquo;Migrated rules for MyProject.ruleset&rdquo; for every project in the solution. That&rsquo;s nice if all your projects don&rsquo;t have the same rules. But if they do, you&rsquo;ll have to manage all of them...</p>
<p>Like all programmers, I&rsquo;m lazy, and I wrote a little macro that will remove all generated ruleset files for the current solution, and use a single rule set.</p>
<p>This is not a very efficient macro, but since it won&rsquo;t be used that often... You&rsquo;ll probably live with the bad performance, and bad VB.NET code :)</p>
<p>Here it is :</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">Sub RemoveAllRuleset()</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp; For Each project As Project In DTE.Solution.Projects   <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FindRuleSets(project)    <br />&nbsp;&nbsp;&nbsp; Next</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">End Sub</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">Sub FindRuleSets(ByVal project As Project)</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp; For Each item As ProjectItem In project.ProjectItems</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Not item.SubProject Is Nothing Then   <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Not item.SubProject.ProjectItems Is Nothing Then    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FindRuleSets(item.SubProject)    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; End If    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; End If</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Not item.SubProject Is Nothing Then   <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If Not item.SubProject.ProjectItems Is Nothing Then</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Dim ruleSets As List(Of ProjectItem) = New List(Of ProjectItem)</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; For Each subItem In item.SubProject.ProjectItems   <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If subItem.Name.StartsWith("Migrated rules for ") Then    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ruleSets.Add(subItem)    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; End If    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Next</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; For Each ruleset In ruleSets   <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ruleset.Remove()    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Next    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; End If    <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; End If    <br />&nbsp;&nbsp;&nbsp; Next</span></p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">End Sub</span></p>
{% include imported_disclaimer.html %}
