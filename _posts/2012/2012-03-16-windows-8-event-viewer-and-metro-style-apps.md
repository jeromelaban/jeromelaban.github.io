---
layout: post
title: "Windows 8 Event Viewer’s Immersive-Shell and Metro Style apps"
date: 2012-03-16 20:41:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/03/16/Windows-8-Event-Viewer-and-Metro-Style-apps", "/post/2012/03/16/windows-8-event-viewer-and-metro-style-apps"]
author: jay
---
<!-- more -->
<p><em>TL;DR: This article talks about an app startup error that can happen with Metro Style apps in Windows 8, how the presence of an app.config file can prevent the app from starting and how the Windows event log viewer&rsquo;s new Immersive-Shell section can help.</em></p>
<p>&nbsp;</p>
<p>The Windows 8 Metro style Xaml/C# application development is an interesting experience.</p>
<p>Since .NET is merely on top of a WinRT and its native structure, you&rsquo;re left in a bit of a darkness sometimes, when it comes to debugging problems that come from WinRT.</p>
<p>Silverlight and Windows Phone also have their fair share of blind issues of this kind, either by having the application that exits with no apparent reason (<a href="http://jaylee.org/post/2011/09/21/WP7Dev-Diagnosing-StackOverflowExceptions-or-the-lack-thereof.aspx" target="_blank">when it is in fact a StackOverflow</a>) or because <a href="http://jaylee.org/post/2011/07/29/wp7dev-Error-code-0xc00cee65.aspx" target="_blank">you&rsquo;ve set two namespaces names with the same content</a>.</p>
<p>You&rsquo;ve basically left at guessing, particularly on Windows Phone and Silverlight for the desktop, and if you&rsquo;re lucky enough you&rsquo;re having a error code that specific enough so that you can narrow your solution to a dozen google can find for you. If you&rsquo;re not, well you&rsquo;ve got a E_ERROR. Fail, as they say.</p>
<p>Windows 8 is actually a bit better at that, because of the Event Viewer. There&rsquo;s a lot of details that appear there, and it&rsquo;s very informative.</p>
<p>[more]</p>
<p>But let&rsquo;s dive into one specific problem I encountered recently.</p>
<p>&nbsp;</p>
<h2>Diagnosing the &ldquo;Unable to active Windows Metro Style application&rdquo; issue</h2>
<p>I&rsquo;ve been upgrading an application&rsquo;s <a href="http://msdn.microsoft.com/en-us/data/gg577609" target="_blank">Reactive Extensions</a> via NuGet to the <a href="http://blogs.msdn.com/b/rxteam/archive/2012/03/12/reactive-extensions-v2-0-beta-available-now.aspx" target="_blank">latest and greatest 2.0-beta</a>, and after doing that update, it turned out the application was not starting at all. The debugger was not very helpful either, showing me this cryptic message :</p>
<p><code><span style="font-family: 'Courier New';">Unable to activate Windows Metro style application &lsquo;xxxxxxxxx!App&rsquo;.</span></p>
<p><span style="font-family: 'Courier New';">Windows was unable to communicate with the target application. This usually indicates that the target application&rsquo;s process aborted. "More information may be available in the Debug pane of the Output window.</span></code></p>
<p>There was not much more details in the output window, and worse, the managed code was not even called. The output window showed that no assembly from my app were even loaded.</p>
<p>In a normal startup, the App.xaml.cs type constructor should be at least called, but this time it was not. This meant that it could not be anything related to C# code, but only related to the configuration of the application.</p>
<p>&nbsp;</p>
<h2>Using the event viewer to troubleshoot metro apps</h2>
<p>The event viewer is very well known in the IT administration side of computing, but less on the development side, which actually is a shame. <strong>This is the first place to dig into when the debugger can&rsquo;t tell you what&rsquo;s happening.</strong></p>
<p>Microsoft&rsquo;s been publishing a lot of information in these logs, and Metro style apps are no exceptions. There is a new section specifically for this type of apps that contains a lot of information to diagnose issues.</p>
<p>Here&rsquo;s how to get there :</p>
<ul>
<li>Open the start menu and type <strong>Event</strong>, select the Settings section</li>
<li>Open the <strong>View event logs</strong> item that shows up</li>
<li>Navigate to the <strong>Applications and Services Logs / Microsoft / Windows</strong></li>
<li>Open the <strong>Immersive-Shell</strong> node</li>
</ul>
<p>You&rsquo;ll find there a lot of details about what&rsquo;s happening in the new shell for Metro Style Apps.</p>
<p>For the specific case of my crashing app, I found out this message :</p>
<p><code><span style="font-family: 'Courier New';">Activation of the Metro style application xxxxxxxxx!App for the Windows.Launch contract failed with error: Server execution failed</span></code></p>
<p>This helped a bit, but not that much. The Windows.Launch contract is actually the creation of the App.xaml.cs file, which I already knew was not called.</p>
<p>There&rsquo;s actually a lot more information in that file, like live tiles notifications, but that probably a subject for another blog post.</p>
<p>&nbsp;</p>
<h2>Friends don&rsquo;t let friends have an app.config file</h2>
<p>After doing a lot of file removal, basically returning to a bare naked application, without any dependencies, the application was still not staring, until I removed the app.config file that had been added recently.</p>
<p>That file was added by the NuGet package update procedure, from 1.0 to 2.0 as a compatibility feature and it seems to be a very annoying file for WinRT&hellip; A bug, I assume.</p>
<p>So next time you find an app.config file in your Windows 8 Metro style app, remove it :)</p>
<p>&nbsp;</p>
<p>Happy WinRT&rsquo;ing !</p>
{% include imported_disclaimer.html %}
