---
layout: post
title: "A tool to move an Hyper-V Virtual Machine without exporting it"
date: 2009-02-20 20:26:00 -0500
comments: true
category: Archive
tags: []
redirect_from: ["/post/2009/02/20/A-tool-to-move-an-Hyper-V-Virtual-Machine-without-exporting-it.aspx", "/post/2009/02/20/a-tool-to-move-an-hyper-v-virtual-machine-without-exporting-it.aspx"]
author: jay
---
<!-- more -->
<p><em>Cet article est disponible <a href="http://blogs.codes-sources.com/jay/archive/2009/02/19/outil-pour-d-placer-une-vm-hyper-v-sans-l-exporter.aspx" target="_blank">en francais</a>.</em></p>
<p><span style="font-family: trebuchet ms,geneva; font-size: x-small;"> Hyper-V is a wonderful tool, and provides great performance and stability. But on the administration side, available tools are a bit scare, and even though most general operations are available, some are a bit hard to use. One can guess that this will improve in Windows Server 2008 R2. </span></p>
<p><span style="font-family: trebuchet ms,geneva; font-size: x-small;"> But for now, the administration tools do not provide any mean to import a VM that has not been previously exported. Exporting a VM can only be done on the original host server while it is still running. In the case of a crashed server, exporting a VM becomes a bit more complex. </span></p>
<p><span style="font-family: trebuchet ms,geneva; font-size: x-small;"> Some techniques do exist, <a href="http://blogs.msdn.com/robertvi/archive/2008/12/19/howto-manually-add-a-vm-configuration-to-hyper-v.aspx">here </a>and <a href="http://www.adopenstatic.com/cs/blogs/ken/archive/2008/01/14/15467.aspx">there</a>, that explain by means of mklink and icacls, </span><span style="font-family: trebuchet ms,geneva; font-size: x-small;">how to recreate symbolic links and file permissions for the VM configuration files. But that stays a particularly complex task, mostly because all files must be modified, and a specific order must be respected for all modifications. And this is especially true for the case of a running host server.</span></p>
<p><span style="font-family: trebuchet ms,geneva; font-size: x-small;"> After having dug in Hyper-V </span><span style="font-family: trebuchet ms,geneva; font-size: x-small;">symlinks and WMI interface, I created a GUI tool that allows to attach and detach VM that have not been previously epoxted.<br /> <br /> Some thougts on this tool : <br /> </span></p>
<ul>
<li><span style="font-family: trebuchet ms,geneva; font-size: x-small;">A VM can only be detached if it is in the "Saved" or "Stopped" state.</span></li>
<li><span style="font-family: trebuchet ms,geneva; font-size: x-small;">It is not necessary to stop the Hyper-V service and all modifications are detected live by the service.</span></li>
<li><span style="font-family: trebuchet ms,geneva; font-size: x-small;">A VM can only be imported if it contains at least on HDD on the IDE 0 controller.</span></li>
<li><span style="font-family: trebuchet ms,geneva; font-size: x-small;">All the VM files must be under the same directory, HDD and snapshots.</span></li>
<li><span style="font-family: trebuchet ms,geneva; font-size: x-small;">All files that are modified are backed-up next to the original files; All other files are not modified nor moved.</span></li>
<li><span style="font-family: trebuchet ms,geneva; font-size: x-small;">.NET 3.5 must be installed.<br /> </span></li>
</ul>
<p><span style="font-family: trebuchet ms,geneva; font-size: x-small;">I'll provide the sources for this tool in a near future, as well as a console version.</span></p>
<p><span style="font-family: trebuchet ms,geneva; font-size: x-small;"> Of course, there will be bugs, and do not hesitate to report them to me. I may not be able to do anything about it because it is a tool that performs a operation that is (I assume) not supported by Microsoft. </span><span style="font-family: trebuchet ms,geneva; font-size: x-small;"><br /> <br /> You can download the tool <a href="http://www.jaylee.org/page/Hyper-V-Virtual-Machine-Mover.aspx">here</a>.</span></p>
{% include imported_disclaimer.html %}
