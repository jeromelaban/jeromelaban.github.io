---
layout: post
title: "Windows 8, Developers, Hyper-V and the new VHDX format"
date: 2012-07-13 22:03:00 -0400
comments: true
category: Archive
tags: ["Windows 8"]
redirect_from: ["/post/2012/07/13/Windows-8-Developers-Hyper-V-and-the-new-VHDX-format.aspx", "/post/2012/07/13/windows-8-developers-hyper-v-and-the-new-vhdx-format.aspx"]
author: jay
---
<!-- more -->
<p><em>TL;DR: Windows 8 added support for SSD TRIM commands in its VHDX format, making very easy to boot from a virtual drive on an SSD drive. This allows to never install Windows directly on a physical disk anymore, easing backups, cloning and virtualization. It also allows for very easy migration between different versions of Windows 8.</em></p>
<p>It's been a very interesting year with Windows 8. We've been fed with&nbsp;two new releases since the Build conference, the Consumer Preview and Release Preview, and now it's <a href="http://windowsteamblog.com/windows/b/bloggingwindows/archive/2012/07/09/upcoming-windows-milestones-shared-with-partners-at-wpc.aspx">the RTM that's coming along in August</a>.</p>
<p>With all these releases to play with, it's a game of install, re-install and&nbsp;co-existence of multiple windows instances on my machine.</p>
<p>I do not like losing to much time staring at my computer while it reinstalls everything from scratch, particularly Visual Studio, and&nbsp;both booting a physical machine&nbsp;from VHD&nbsp;and/or virtualizing it Hyper-V has been very helpful to save time.</p>
<p>[more]&nbsp;</p>
<h1>Moving through Windows Installations</h1>
<p>Here&rsquo;s what I&rsquo;ve been doing to save some time:</p>
<ul>
<li>Install Windows 8 Consumer Preview (CP) from a USB drive in a VHD file</li>
<li>While running the Windows 8 CP, install the Windows 8 Release Preview (RP) in an Hyper-V Virtual Machine,</li>
<li>Boot on the Windows 8 RP VHD, run the Windows 8 CP in a Virtual Machine to be able to compare behaviors, and migrate settings between both installations.</li>
</ul>
<p>This is a real time saver, because both instances can run at the same time, meaning that I can continue working with my previous setup while I create the new setup. Cool stuff!</p>
<p>&nbsp;</p>
<h1>Boot on VHDX, The new VHDX format and SSDs</h1>
<p>Windows has had the feature to <a href="http://www.hanselman.com/blog/HowToGuideToInstallingAndBootingWindows8ConsumerPreviewOffAVHDVirtualHardDisk.aspx">boot a VHD drive since Windows 7</a>. It is very useful to avoid scrapping your actual hard drive to install a new instance of Windows. Your Windows instance runs directly on the hardware, except for the boot disk that is virtualized. It adds a small cost to the performance of the storage device, but the flexibility is a big gain. That is, unless you're on an SSD drive.</p>
<p>In Windows 7, the aging&nbsp;VHD format was not made to support the <a href="http://en.wikipedia.org/wiki/TRIM">new TRIM command</a>, which allows the system to report unused blocks of a partition back to the drive, so it can improve the lifetime of the drive. Windows 7 not supporting TRIM through VHD means that putting a VHD on an SSD drive would not be a very&nbsp;good thing&nbsp;for the performance of the SSD drive. The reason for this is that the VHD file may be totally unused, but the file is still placed on the disk, reserving space and ultimately degrades the disk performance (and its life-time).</p>
<p>In Windows 8, the <a href="http://blogs.technet.com/b/aviraj/archive/2012/05/06/windows-server-2012-convert-vhd-to-vhdx-using-hyper-v-manager.aspx">new VHDX format has been introduced</a>, and it now supports TRIM compatible drives, making it a viable solution to put a VHDX directly on an SSD drive.</p>
<p>&nbsp;</p>
<h1>Never installing on a bare disk anymore</h1>
<p>The VHDX abstraction adds a layer on top of the original drive. This adds a performance hit, particularly when using dynamically expanding drives. This can make contiguous content in the VHD be non-contiguous on the physical drive, which forces the head to seek a bit more than it should.</p>
<p>However, on an SSD drive where the seek time is very small, impacts of running in a dynamically expanding VHDX are almost irrelevant.</p>
<p>I&rsquo;ve then decided to never install windows directly on a disk, now that I know that the VHDX supports SSDs. This will make backup, cloning and virtualization a lot easier to do. If I need more performance for a particular Windows instance, I can move the VHDX files around and place them on an SSD drive, and store the other on a multi-terabytes drive.</p>
<p>&nbsp;</p>
<h1>Virtualizing your current drive</h1>
<p>I had installed the Windows 8 RP directly on my SSD drive, and I wanted to virtualize the disk to be able to install another Windows instance.</p>
<p>Hyper-V has the ability to do just that, and it is fairly easy to do. Here&rsquo;s how to do it:</p>
<ul>
<li>Install Windows 8 RP in a VHD, so you can access your original drive exclusively</li>
<li>Launch the Hyper-V manager</li>
<li>Create a new virtual machine, but do not attach a new drive</li>
<li>In the virtual machine settings, create a new VHDX drive, set its size, type and location</li>
<li>When a the &ldquo;Configure Disk&rdquo; step, use the &ldquo;Copy the contents of the physical disk&rdquo; and select the drive where your windows is installed</li>
<li>Click finish, wait a bit until the copy finishes and you&rsquo;re done!</li>
<li>Run the VM and your originally bare installation of windows is now working.</li>
</ul>
<p>Note that you should not be booting your computer on the copied VHDX, unless the partition that contains the original windows has been formatted.</p>
<p>If you do not do this, the Windows installation in your VHDX will be completely destroyed by the fact that the same partition ID exists twice on the system, and that the original partition will take over, forcing the Windows instance on the VHD to remap all the installation paths&hellip; An unrecoverable mess, in other words.</p>
<p>Also, you may not be able to mount the VHDX file (using the awesome out-of-the-box explorer right-click extension in Windows 8) unless you <a href="http://technet.microsoft.com/en-us/library/cc730793(v=ws.10)">change its Unique ID using the Diskpart tool</a>.</p>
<p>Finally, <strong>while I do not recommend doing it</strong>, I&rsquo;ve been able to virtualize the disk for which the current Windows was running on, and it seems to work perfectly. I&rsquo;m a bit puzzled on how that is actually working, because the content of the drive can change during its copy, but maybe the transactional features of the NTFS or VSS are at work&hellip;</p>
<p>Happy Windows 8&rsquo;ing :)</p>
<p>&nbsp;</p>
{% include imported_disclaimer.html %}
