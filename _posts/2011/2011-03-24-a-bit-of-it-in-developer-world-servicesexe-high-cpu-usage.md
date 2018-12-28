---
layout: post
title: "A bit of IT in developer's world: services.exe high CPU usage"
date: 2011-03-24 12:36:00 -0400
comments: true
category: Archive
tags: [".NET", "Misc"]
redirect_from: ["/post/2011/03/24/A-bit-of-IT-in-developer-world-servicesexe-high-CPU-usage.aspx", "/post/2011/03/24/a-bit-of-it-in-developer-world-servicesexe-high-cpu-usage.aspx"]
author: jay
---
<!-- more -->
<p>One of the advantages of virtualization is the P2V (Physical to Virtual)&nbsp;process: Converting an "old" build machine to&nbsp;a VM so it can be moved around with the load&nbsp;as-is, snapshotted, backed-up and so on.</p>
<p>This is particularly useful when say, you have a build machine that's been there for a very long time, has a lot of dependencies over old third party software, has been customized by so many people (that have long left the company)&nbsp;that if you wanted to rebuild that machine from scratch, it would literally take you weeks of tweaking to get it to work properly. And that machine is running out of very old hardware that may break at any time. And that the edition of Windows that does not migrate easily to new hardware because of HAL or Mass Storage&nbsp;issues, requiring a reinstallation. That a lot of "ands".</p>
<p>That's the kind of choice you do not need to make: You just take the machine and virtualize it using SCVMM 2008 R2.</p>
<p>But still, even virtualized, the machine been there that long, and things have started falling apart, like having the services.exe process taking 100% of the CPU. And I did not want to have to rebuild that machine just because of that strange behavior.</p>
<p>If you read <a href="http://www.hanselman.com/blog/FiguringOutWhyMySVCHOSTEXEIsAt100CPUWithoutComplicatedToolsInWindows7.aspx">scott hanselman's blog</a>, you've been recalled that Windows Server 2008 and later has the resource monitor that gives a wealth of information about the services running under services.exe. But if you're out of luck, like running under Windows Server 2003, you can still use <a href="http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx">Process Explorer</a>. This will give you the similar kind of insight in the Windows Services that are running.</p>
<p>For my particular issue, this was actually the Event Log service that was taking all the resources.</p>
<p>&nbsp;</p>
<h3>How about I get my CPU back ?</h3>
<p>After some digging around, I noticed that :</p>
<ul>
<li>All Event Viewer logging sections in the MMC snap-in were all displaying the same thing, which was actually a mix of all the System, Application and Security logs. </li>
<li>Displaying any of these logs was taking a huge amount of time to display.</li>
<li>The HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\System\Source was containing something like "System System System System System System System" a hundred of times, the same thing for the keys for the other event logs</li>
<li>A whole bunches (thousands)&nbsp;of interesting sources named like some .NET&nbsp;application domain created by the application being built on this machine</li>
</ul>
<p>To fix it, a few steps in that order :</p>
<ul>
<li>Disable the Event Log service and reboot. You won't be able to stop it, but at the next reboot it will not start.</li>
<li>In the&nbsp;C:\WINDOWS\system32\config folder, move the files *.evt to a temporary folder, so they don't get picked up by the service when it'll restart</li>
<li>In the registry, for each HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\[System|Security|Application]\Source, replace the content with the one found on the same key on another very similar Windows Server&nbsp;2003 machine. You can install a brand new&nbsp;machine and pick up the content.</li>
<li>If you have, like I did, a whole bunch of sources that look familiar and should not be there under HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\[System|Security|Application], remove their keys if you removed them from the "Source" value.</li>
<li>Set the Event Log service to "Automatic" and reboot.</li>
</ul>
<p>&nbsp;</p>
<p>The interesting part about the virtualization of that build&nbsp;machine&nbsp;is like in many other occasions, the snapshots, where you can make destructive changes and go back if they were actually <strong>too </strong>destructive.</p>
<p>&nbsp;</p>
<h3>What's with the event log "interesting sources" ?</h3>
<p>The application being built and tested is running tests on the build machine, and it makes use of application domains and log4net. Log4net has an <a href="http://logging.apache.org/log4net/release/sdk/log4net.Appender.EventLogAppender.html">EventLogAppender</a> that allows the push of specific content to the Windows Event Log. Log4net defaults the name of the source to the application domain name, if there is no entry assembly.</p>
<p>Those tests were actually using a default configuration, and were logging Critical messages to the event log, but the domains were created using a new GUID to avoid supposed name collisions. This is&nbsp;something that did actually more harm than good in the long run, because each new appdomain that was logging to the event log was creating a new event source.</p>
<p>And the build system has been there for a long time. Hence the thousands of "oddly named" event sources.</p>
<p>&nbsp;</p>
{% include imported_disclaimer.html %}
