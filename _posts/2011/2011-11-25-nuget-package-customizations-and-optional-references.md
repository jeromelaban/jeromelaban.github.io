---
layout: post
title: "NuGet package customizations and optional references"
date: 2011-11-25 20:55:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2011/11/25/NuGet-package-customizations-and-optional-references", "/post/2011/11/25/nuget-package-customizations-and-optional-references"]
author: jay
---
<!-- more -->
<p><em>This article describes a bit of what NuGet does and why you should take a look at it, but also a package installation customization to a work around a problem with packages that contain optional assemblies.</em></p>
<p>&nbsp;</p>
<p><a href="http://nuget.org/" target="_blank">NuGet</a> is a fantastic tool. Its ability to ease package discovery, installation and update is a big time saver. As a solution maintainer, you can spend less time deploying and updating your external dependencies, particularly when they&rsquo;re often updated.</p>
<p>&nbsp;</p>
<h2>Private NuGet Repositories</h2>
<p>It can be used to easily install public packages exposed via Microsoft&rsquo;s servers, but it can also be used to create private package repositories.</p>
<p>I've been using it to publish internal framework that is often updated and used in many projects. The automatic creation of packages in a build script and the ease of deployment using the <a href="http://docs.nuget.org/docs/creating-packages/hosting-your-own-nuget-feeds" target="_blank">NuGet OData server</a> is, again, a big time saver.</p>
<p>Each time a check-in is performed to update the internal framework, a new package is automatically created and it appears as a package update for the projects that use it.</p>
<p>&nbsp;</p>
<h2>Existing libraries and NuGet</h2>
<p>The package model is built around some basic rules:</p>
<ul>
<li><em>Each package is installed using its version as the directory name <br /></em>The NuGet engine will update all the references &ldquo;HintPath&rdquo; nodes of projects files for them to point to the updated package location.</li>
<li><em>By default, the install action of a package adds references to all the available assemblies in the target project. <br /></em>There&rsquo;s a pretty good reason for that; you don&rsquo;t want to download and install assemblies that you don&rsquo;t need. There is no support for &ldquo;sub-packages&rdquo;, even with <a href="http://docs.nuget.org/docs/reference/nuspec-reference#Specifying_Explicit_Assembly_References" target="_blank">reference exclusions</a>. Big frameworks, like Enterprise Library, get chunked into small dependent packages, and you can install only those needed by your projects.</li>
<li><em>A project that installed a package that has references to </em><a href="http://docs.nuget.org/docs/reference/nuspec-reference#Specifying_Explicit_Assembly_References" target="_blank"><em>package-excluded references</em></a><em> will not have these manual references updated to the latest package. <br /></em>That is a side effect of the second rule. If you add reference-excluded assemblies to a package, the action of updating that package will not update manual references you created by referencing these assemblies.
<p>&nbsp;</p>
<p>For existing libraries that may not easily be split into small pieces, primarily for time constraints, moving to NuGet can be a tough job. If you make a package with all your assemblies and only reference a few of them by default in there, then updating the package can quickly become an annoying &ldquo;Find and Replace&rdquo; job to manually change references that did not get updated automatically by the NuGet engine.</p>
<p>&nbsp;</p>
<h2>Using the Install.ps1 script</h2>
<p>Fortunately, there is a file that can be included in the package, which is by convention named and located in <a href="http://docs.nuget.org/docs/creating-packages/creating-and-publishing-a-package#Automatically_Running_PowerShell_Scripts_During_Package_Installation_and_Removal" target="_blank">tools\install.ps1</a>.</p>
<p>It is a <a href="http://technet.microsoft.com/en-us/scriptcenter/dd742419" target="_blank">PowerShell</a> script that gets called automatically when the package is installed. But the interesting part is that this installer file gets called with a <a href="http://msdn.microsoft.com/en-us/library/51h9a6ew(v=VS.100).aspx" target="_blank">DTE.Project</a> instance that can be used to manipulate the actual project in VS2010 !</p>
<p>This means that when the package is installed, it is possible to update the references that were manually created to the previous package assemblies.</p>
<p>&nbsp;</p>
<h2>Updating the manual references</h2>
<p>This is not a straightforward as it may seem, but after working around a <a href="http://stackoverflow.com/a/6847955/26346" target="_blank">HintPath update issue</a>, here is a small helper to do the job:</p>
<pre class="brush: ps">param($installPath, $toolsPath, $package, $project)
    
    $packageName = $package.Id + '.' + $package.Version;
    $packageId = $package.Id;

    # Full assembly name is required
    Add-Type -AssemblyName 'Microsoft.Build, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'

    $projectCollection = [Microsoft.Build.Evaluation.ProjectCollection]::GlobalProjectCollection
    
    # There is no indexer on ICollection&lt;T&gt; and we cannot call
    # Enumerable.First&lt;T&gt; because Powershell does not support it easily and
    # we do not want to end up MethodInfo.MakeGenericMethod.
    $allProjects = $projectCollection.GetLoadedProjects($project.Object.Project.FullName).GetEnumerator(); 

    if($allProjects.MoveNext())
    {
        foreach($item in $allProjects.Current.GetItems('Reference'))
        {
            $hintPath = $item.GetMetadataValue("HintPath")
            
            $newHintPath = $hintPath -replace $packageId + ".*?\\", "$packageName\\"
        
            if ($hintPath -ne $newHintPath)
            {
                Write-Host "Updating $hintPath to $newHintPath"
                $item.SetMetadataValue("HintPath", $newHintPath);
            }
        }
    }</pre>
<p>This script is called for each project the package is installed onto, and scans all the references of the project that match the current package to update them.</p>
<p>You&rsquo;ll notice that the ICollection&lt;T&gt; interface is not particularly PowerShell friendly. Too bad the Powershell syntax does not allow the use of generic methods, otherwise that nasty GetEnumerator / MoveNext could have gone away. Still, Powershell is dynamically typed so using IEnumerable.Current is fine.</p>
</li>
</ul>
{% include imported_disclaimer.html %}
