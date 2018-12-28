---
layout: post
title: "Using the Remote Debugger"
date: 2010-07-22 20:05:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/07/22/Using-the-Remote-Debugger.aspx", "/post/2010/07/22/using-the-remote-debugger.aspx"]
author: jay
---
<!-- more -->
<p><em><a href="http://blogs.codes-sources.com/jay/archive/2010/07/20/utiliser-le-remote-debugger-de-visual-studio.aspx">Cet article est disponible en francais.</a></em></p>
<p>To continue in the same kind of articles about Visual Studio features that have been available for a while now, but are commonly under-used, I'll talk in this post about the Remote Debugger.</p>
<p>&nbsp;</p>
<h3>Local Debugging<br /></h3>
<p>Visual Studio has a debugger that allows the debugging of program when running it using F5, or "Debug / Start Debugging". Visual Studio will then start in a special mode that allows step by step execution of the program, use features like <a href="http://msdn.microsoft.com/en-us/library/ktf38f66.aspx">BreakPoints</a>, <a href="http://msdn.microsoft.com/en-us/library/232dxah7.aspx">TracePoint</a>, Watches, <a href="http://msdn.microsoft.com/en-us/library/dd264915.aspx">IntelliTrace</a>, <a href="http://msdn.microsoft.com/en-us/library/fk551230.aspx">create MiniDumps</a> and many more.</p>
<p>The debugger runs the program on the local machine, and uses the permissions of the locally logged on user.</p>
<p>Nothing out of the ordinary. Well, maybe the Reverse Debugging with IntelliTrace in VS2010, which is <strong>very</strong> cool.</p>
<p>&nbsp;</p>
<h3>Hardware Specific and CrapWare</h3>
<p>I don't know about you, but I keep my development PC as stable as possible. I rarely install new software, so that I keep the overall performance stable over time. I will most of the time install new software versions only after having tested them on other PCs to determine their behavior.</p>
<p>Call me maniac, that's what it is :)</p>
<p>But then, what to do when the need for testing an installation program comes up ? Or when you need to debug plugins for NI TestStand or Labview ? Or when the software needs a very specific kind of hardware that cannot be installed on your development PC ? (Rainbow Keys, anyone ?)</p>
<p><br />The answer is simple : The Remote Debugger ! When possible, I will test and debug my software on a virtual machine, or on a physical machine that has the appropriate environment to execute the software.<br /><br />That way, the development environment stays stable, and I don't need to make installation of software that could add some crapware and eat up the few bytes of RAM left :)</p>
<h3>The Remote Debugger ?<br /></h3>
<p>The idea is to continue using the development machine, where the source code is and to connect via the network on a machine that will execute the program. After that, the remote debugging session is very similar to a local session, with the exception of the "Edit and Continue" that is not supported. But most of the time, we can live without it.</p>
<p>&nbsp;</p>
<h4>Running the debugger from Visual Studio</h4>
<p>It is possible to run the execution on the remote machine by using the "Use Remote Machine" option in the "Debug" tab of a C# project. It is important to note that checking this option implies that all paths specified in "Working Directory" or "External Program" are those of the remote machine.</p>
<p>Aditionnally, Visual Studio will not copy the binaries and PDB files on the remote machine. You have to make the copy of the files at the appropriate location, by using a "Post Build Action", a UNC path in the form of&nbsp;"\\mymachine\c$\temp".</p>
<p>&nbsp;</p>
<h4>Attach to a Running Process</h4>
<p>It is also possible to attach to a running process, by using the "Debug / Attach To Process" option. You just need to fill in the "Qualifier" and set the name of the remote debugger, and to choose the process to debug.</p>
<p>Quick hint: The option "Show processes from all users" is not enabled by default. This means that is you want to debug a Windows Service, you will not see it in the list until you enable it.<br /><br />Finally, the "Attach To Process" window is also very useful with local processes. It is a very handy feature to create a memory dump of a process that takes too much memory, and analyze it.</p>
<p>&nbsp;</p>
<h3>Installing the Remote Debugger</h3>
<p>The Remote Debugger is an additional Visual Studio component that is located on the installation media, in the "Remote Debugger" folder. Three versions exist : x86, x64 and ia64 (RIP, Itanium...). If you have to debug a 32 process on 64 bits machine, I advise that you install both the x86 and x64 versions. You will have to choose which remote debugger to run depending on the .NET runtime that is used. You can see which version to use in the "Type" column of the "Attach to Process" window.</p>
<p>Here's what to do :</p>
<ul>
<li>If you are using VS2008 SP1, you can <a style="text-decoration: none; color: #3db2ff;" href="http://www.microsoft.com/downloads/details.aspx?FamilyID=440ec902-3260-4cdc-b11a-6a9070a2aaab&amp;displaylang=en">download it here</a>, and for VS2010 you can use the install located on the DVD</li>
<li>Once installed on the remote machine, install the RDBG service with the wizard, using the LocalSystem account.</li>
<li>You may have a message about a security issue. If you do, follow these steps :      
<ul>
<li>Open the "Local Security Policy" section of the "Administrative Tools" control panel</li>
</ul>
<ul>
<li>Go to the "Local Policies" / "Security Options"</li>
</ul>
<ul>
<li>Double click on "Network access: Sharing and security model for local  accounts" and set the value to "Classic : Local users authenticate as  themselves"</li>
</ul>
<ul>
<li>Close the window</li>
</ul>
</li>
<li>If your machine is not on the same domain as your development machine, or even if it's not on a domain at all, add a local use account on the remote machine that has the same name as your current username, and make it a member of the administrators group. The password also has to be the same.</li>
<li>Start the remote debugger on the remote machine. Note that to debug a 32 bits process, you have to run the 32 Bits version of the debugger.</li>
<li>On the development machine, open the "Attach to process" window, and type the identifier of the remote debuger (shown on the remote debugger window). It should look like this: administrator@my-machine.</li>
</ul>
<p>Note that the firewall on both the development and the remote machine can prevent the remote debugger from working properly. You can temporarily disable it, but make sure to enable it back after. If you only want to enable specific ports, the port 135/TCP is used. The Remote Debugger uses DCOM as its communication protocol.</p>
<p>&nbsp;</p>
<h3>And if my breakpoints stay empty red circles ?</h3>
<p>This is a very common situation that means that the pdb files do not match the loaded binaries. Make sure that you've copied the pdb files at the same time you did the dlls.</p>
<p>The "Debug / Windows / Modules" shows if the debug symbols have been loaded properly, and if it's not the case, the "View / Output / Debug" window will most of the time show why.</p>
<p><br />Happy debugging !</p>
{% include imported_disclaimer.html %}
