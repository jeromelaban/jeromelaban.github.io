---
layout: post
title: "Team Build and Windows Phone 7"
date: 2011-05-01 00:00:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/05/01/Team-Build-and-Windows-Phone-7.aspx", "/post/2011/05/01/team-build-and-windows-phone-7.aspx"]
author: jay
---
<!-- more -->
<p>Building Windows Phone 7 applications in an agile way encourages the use of <a href="http://en.wikipedia.org/wiki/Continuous_integration">Continuous Integration</a>, and that can be done using Team System 2010.</p>
<p>There are a few pitfalls to avoid to get there, but this can be acheived quite easily with great results.</p>
<p>I won't cover the goodness of automated builds, this has already been <a href="http://www.joelonsoftware.com/articles/fog0000000023.html">covered a lot</a>.</p>
<p>&nbsp;</p>
<h2>Adding Unit Tests</h2>
<p>Along with the continous integration to create hopefully successful builds out of every check-in of source code, you'll also find the automated execution of unit tests. Team System has the ability to provide nice code coverage and unit tests success rates in <a href="http://msdn.microsoft.com/en-us/library/bb668964.aspx">Reporting Services reports</a>, where statistics can be viewed, which gives good health indicators of the project.</p>
<p>Unfortunately for us, at this point&nbsp;there are no ways to automatically execute tests using the WP7 .NET runtime.&nbsp;But&nbsp;if you successfuly use an <a href="http://blog.galasoft.ch/archive/2011/04/13/deep-dive-mvvm-samples-mix11-deepdivemvvm.aspx">MVVM approach</a>, your&nbsp;view models and non UI code&nbsp;can be compiled&nbsp;for multiple platforms, because they do not rely on the UI components that may be specific to the WP7 platform. That way, we are still able to test our code using the .NET 4.0 runtime with MSTest and <a href="http://msdn.microsoft.com/en-us/library/ms182470.aspx">Visual Studio 2010 test projects</a>.</p>
<p>To avoid repeating code, multi-targeted files can be integrated into single target projects either by :</p>
<ul>
<li>Using the <a href="http://msdn.microsoft.com/en-us/library/ff921108(v=pandp.20).aspx">Project Linker tool</a> and link multiple projects,</li>
<li>Creating project files in the same folder and use the "include file" command when showing all files in the solution explorer. Make sure to change the output assembly name to something like "MyAssembly.Phone.dll" to avoid conflicts.</li>
</ul>
<p>Multi-targeted files are using the <a href="http://msdn.microsoft.com/en-us/library/4y6tbswk.aspx">#if directive</a> and the WINDOWS_PHONE define, or the lack thereof, to compile code for the current runtime target.</p>
<p>There is also the option of creating projects with the <a href="http://visualstudiogallery.msdn.microsoft.com/b0e0b5e9-e138-410b-ad10-00cb3caf4981/">Portable Library</a> add-in, but there are some caveats on that side, and there are a few constraints when using this method. You may need to <a href="http://blogs.msdn.com/b/sburke/archive/2011/02/03/using-observablecollection-with-the-portable-library-tools-ctp.aspx">externalize code that is not supported</a>, like UrlDecode.</p>
<p>Testing with MSTest ensures that your code runs successfully on .NET 4.0 runtime, but&nbsp;this does not test on the WP7 runtime. So to be sure that your code is successfully running on it, and since Windows Phone 7 is build on Silverlight 3, tools like the <a href="http://archive.msdn.microsoft.com/silverlightut">SL3 Unit Test framework</a> can be used to manually run tests in the emulator. This cannot be integrated into the build for now, unfortunately; you'll have to place that in your QA tests.</p>
<p>&nbsp;</p>
<h2>TeamBuild with the WP7 toolkit</h2>
<p>To be able to buid WP7 applications on your build machine, you need to install the SDK on your build machine, and a <a href="http://msdn.microsoft.com/en-us/library/ms181710(v=vs.80).aspx">Team Build</a> agent and/or controller.</p>
<p>Creating a build definition is done the same way as for any other build definition, except for one detail. You need to set the <a href="http://msdn.microsoft.com/en-us/library/dd647553.aspx">MSBuild platform</a> to x86 instead of Auto if your build machine is running on a 64 Bits Windows. This forces the MSBuild runtime to use the 32 bits runtime, and not 64 bits, where the SDK does not work properly.</p>
<p>If you don't, when building your WP7 application, you'll find that intriguing message&nbsp;:</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier; font-size: small;"> Could not load file or assembly 'System.Windows, Version=2.0.5.0'</span></p>
<p>Which is particularly odd considering that you've already installed the SDK, and that dll is definitely available.</p>
<p>You may also find that if you install that DLL from the SDK in the GAC, you'll get that other&nbsp;nice message :</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier; font-size: small;">Common Language Runtime detected an invalid program.</span></p>
<p>Which is most commonly found when mixing 32 bits and 64 bits assemblies, for which the <a href="http://msdn.microsoft.com/en-us/library/kb4wyys2(v=vs.80).aspx">architecture has been explicitly specified</a> instead of "Any CPU". So don't install that DLL in the GAC and set the MSBuild architecture to x86.</p>
<p>&nbsp;</p>
<p>That's it&nbsp; for now, and Happy WP7 building !</p>
{% include imported_disclaimer.html %}
