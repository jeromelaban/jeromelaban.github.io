---
layout: post
title: "Switching a Windows 8 Consumer Preview from a VHD to Hyper-V"
date: 2012-03-04 13:39:00 -0500
comments: true
category: Archive
tags: ["Windows 8"]
redirect_from: ["/post/2012/03/04/Switching-a-Windows-8-Consumer-Preview-from-a-VHD-to-Hyper-V", "/post/2012/03/04/switching-a-windows-8-consumer-preview-from-a-vhd-to-hyper-v"]
author: jay
---
<!-- more -->
<p>Now that Windows 8 Consumer Preview (CP) is out the doors, I'll start publishing a series of articles about Windows 8 development in various areas, but primarily focused on development.</p>
<p>I'll start today with&nbsp;some tips and tricks that allow to run Windows 8 natively from a VHD, but also run that same VHD from Hyper-V, within Windows 8 Consumer Preview.</p>
<p>The common scenario is simple, you want to try out stuff that worked in the Developer Preview (DP) inside Win8 CP, or run a two instances of the Win8 CP for testing purposes. Chances are you installed the DP on a VHD, from the <a href="http://www.hanselman.com/blog/HowToGuideToInstallingAndBootingWindows8ConsumerPreviewOffAVHDVirtualHardDisk.aspx" target="_blank">excellent blog post from Scott Hanselman</a>.</p>
<h2>A few things about Hyper-V</h2>
<p>Hyper-V&rsquo;s been here for a while on the Server side, but chances are you never looked at it. Hyper-V will most likely completely replace Virtual PC, as it now supports 64 Bits virtual machines and the very useful dynamic memory feature. Dynamic memory basically add or removes physical memory assigned to the virtual machines depending on the actual demand. This allows to have higher virtual machine density on servers, but on client machines this is also pretty useful because you may not have 32GB of ram, and still want to use it for your primary OS.</p>
<p>This is why this tutorial suggests to have a machine starting with 512MB of RAM, so that it can grow to the appropriate amount, and not arbitrarily reserve 2 or 4GB of ram, just in case.</p>
<p>&nbsp;</p>
<h2>Running the VHD from Hyper-V</h2>
<p>So now that Hyper-V is now included natively in Windows 8, and even though the final SKU in which it will be included is unknown, we can use it from the CP.</p>
<p>If you&rsquo;re like me, you may have installed the CP directly on a SSD to make it pretty darn fast. Now, here&rsquo;s how you install Hyper-V :</p>
<ol><ol>
<li>Open the start menu, type &ldquo;<strong>Feature</strong>&rdquo; and open &ldquo;<strong>Turn Windows Features on and off</strong>&rdquo;</li>
<li>Select &ldquo;<strong>Hyper-V</strong>&rdquo; and OK. Reboot. (If the Hyper-V infrastructure is off, then either your CPU does not support hardware virtualization, or it is disabled in your bios)</li>
<li>Launch the &ldquo;<strong>Hyper-V Manager</strong>&rdquo;</li>
<li>In the right hand side panel, select &ldquo;<strong>Virtual Switch Manager</strong>&rdquo;</li>
<li>Click on &ldquo;<strong>Create a Virtual Switch</strong>&rdquo;</li>
<li>Set a relevant network or network adapter name</li>
<li>Select your network card, could be either a wireless or wired network card, then OK.</li>
<li>In the right hand side panel, select &ldquo;<strong>New</strong>&rdquo; and &ldquo;<strong>Virtual Machine</strong>&rdquo;</li>
<li>Give it 512MB of RAM and check the &ldquo;<strong>Dynamic Memory</strong>&rdquo; box, then next</li>
<li>Select the virtual network switch you previously created</li>
<li>Select &ldquo;<strong>Use an existing Virtual Hard Disk</strong>&rdquo; and select the Windows 8 DP VHD or VHDX file</li>
<li>Click next twice.</li>
<li>Now in the list of machines in the <strong>Hyper-V Manager</strong>, right click on your machine and start.</li>
</ol></ol>
<p>&nbsp;</p>
<p>Chances are that your machine will not boot and ask you to replace the disk with a bootable one.</p>
<p>&nbsp;</p>
<p>Here&rsquo;s how to fix that :</p>
<ol><ol>
<li>Double click on the virtual machine line to open a <strong>Display Console</strong></li>
<li>In the Media menu, select &ldquo;<strong>Insert Disk</strong>&rdquo; and select the Windows 8 CP iso file</li>
<li>Reboot the virtual machine using the reset button in the toolbar</li>
<li>Once on the Windows 8 install welcome screen, type <strong>Shift+F10</strong>, this will show a command line window</li>
<li>Run <strong>Diskpart</strong></li>
<li>Type &ldquo;<strong>list volume</strong>&rdquo;</li>
<li>Type &ldquo;<strong>select volume 1</strong>&rdquo;</li>
<li>Type &ldquo;<strong>list partition</strong>&rdquo; and find the &ldquo;<strong>System Boot</strong>&rdquo; partition or <strong>the primary partition</strong></li>
<li>Type &ldquo;<strong>select partition 1</strong>&rdquo; (replace the 1 with the partition listed at the previous step)</li>
<li>Type &ldquo;<strong>active</strong>&rdquo;</li>
<li>Then type &ldquo;<strong>exit</strong>&rdquo;</li>
<li>Back at the command line, you should be on the X: drive</li>
<li>Type the following &ldquo;<strong>bcdboot c:\windows /s c:</strong>&rdquo; (you may need to replace the second &ldquo;c:&rdquo; with the drive that is the System Boot, when installing on a physical machine)</li>
<li>Once done, close the installer window to exit the installation and reboot</li>
<li>You&rsquo;re done !</li>
</ol></ol>
<p>&nbsp;</p>
<p>Note that this section can be used to register pre-installed Windows 7 or 8 VHD to an existing Windows 7 or 8 physical machine. But in this case, you may need to find the actual hidden boot partition that the Windows Installation creates automatically. You&rsquo;ll find that partition with the &ldquo;list partition&rdquo; command.</p>
<p>Happy Windows 8 CP&rsquo;ing !</p>
{% include imported_disclaimer.html %}
