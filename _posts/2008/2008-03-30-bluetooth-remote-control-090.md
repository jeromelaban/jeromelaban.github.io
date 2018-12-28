---
layout: post
title: "Bluetooth Remote Control for Windows Mobile 0.9.0"
date: 2008-03-30 21:27:00 -0400
comments: true
category: Archive
tags: ["Bluetooth Remote Control"]
redirect_from: ["/post/2008/03/30/Bluetooth-Remote-Control-090.aspx", "/post/2008/03/30/bluetooth-remote-control-090.aspx"]
author: jay
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2">Yet another release of Bluetooth Remote Control for Windows Mobile. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This time, I&#39;ve added two features that were requested for a long time, which are the ability to control application that I did not include out of the box, and the ability to use the device as a simple mouse and keyboard device. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">About the first part, there is now a configuration file that allows the addition of new applications by defining each and every command that can be used on the mobile side, and their corresponding action. I intentionally did not comment the file, this is not meant to be modified by a standard user, but rather by an experienced user or developer that wants to control a custom application. The configuration file will hopefully be really simple to understand for a developer. So, if you understand how this file works, just feel free to add a new application, or update an existing if it does not fit your needs.</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Now about the part with the keyboard and mouse. I added a new &quot;application&quot; which actually is not, which is basically a pass-through for mouse and keyboard actions. You&#39;ll be able to move your mouse and type in any key you want in the focused window. On my TyTN II, the key mapping is a bit funky when using the function key, but I&#39;ll try to fix that for a future release.</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Both features are fairly stable, but I suspect minor issues to come up as users are using it. Feel free to comment ! </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Download version 0.9.0 at </font><a href="/remotecontrol" title="Bluetooth Remote Control"><font face="trebuchet ms,geneva" size="2">http://www.jaylee.org/remotecontrol</font></a><font face="trebuchet ms,geneva" size="2">. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Here is the change log : </font>
</p>
<ul>
	<li><font face="trebuchet ms,geneva" size="2">Fixed warning message when ActiveSync is not detected to install the mobile side program. </font></li>
	<li><font face="trebuchet ms,geneva" size="2">Added application definition file. </font></li>
	<li><font face="trebuchet ms,geneva" size="2">Added screen control application. </font></li>
	<li><font face="trebuchet ms,geneva" size="2">Fixed Widcomm PC side support when transmitting large chunks of data.</font></li>
</ul>

{% include imported_disclaimer.html %}
