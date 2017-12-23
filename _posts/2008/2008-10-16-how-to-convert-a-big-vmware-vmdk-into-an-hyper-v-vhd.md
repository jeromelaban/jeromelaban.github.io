---
layout: post
title: "How to convert a (big) VMWare VMDK into an Hyper-V VHD"
date: 2008-10-16 19:36:00 -0400
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2008/10/16/How-to-convert-a-big-VMWare-VMDK-into-an-Hyper-V-VHD", "/post/2008/10/16/how-to-convert-a-big-vmware-vmdk-into-an-hyper-v-vhd"]
author: jay
---
<!-- more -->
<p>
<em>Cet article est disponible en <a href="http://blogs.codes-sources.com/jay/archive/2008/10/17/comment-convertir-un-gros-vmdk-vmware-en-vhd-pour-hyperv.aspx">francais ici</a>.</em>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I&#39;ve been playing a lot with Hyper-V lately and quite frankly, I&#39;m very pleased with it. 
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
VMs a very responsive, the IO is not the bottleneck it was before, the impact of VMs on the host is far lower than with VPC or VMWare, it supports snapshots and fine grained ACLs on each VMs.The performance part is subjective as always, but I find it faster, and better integrated in the OS that other products. </font><font face="trebuchet ms,geneva" size="2">And now, there is a free version of Windows 2008 Server named <a href="http://www.microsoft.com/servers/hyper-v-server/default.mspx" target="_blank">Hyper-V Server</a>, which allows to run a bare minimum text-mode only version of windows Just Hyper-V and the VMs.
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">By the way, I partially agree with <a href="http://www.winsupersite.com/" target="_blank">Paul Thurrott</a> on <a href="http://twit.tv/ww78" target="_blank">the &quot;GUI user experience&quot; of Hyper-V</a> which is not very elegant, involving some scripting and information found on blogs, but hey, this is a server product. This is definitely <strong>not </strong>for the average joe that does not know a bit of what he is doing.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
I&#39;ve had recently to port a VMWare VM to Hyper-V and the main disk was created as a fixed length (flat) -- disk of 80GB, which is obviously a VMDK file of 80GB.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2"> First, I tried converting the file with the <a href="http://vmtoolkit.com/files/folders/converters/entry8.aspx" target="_blank">VMDK to VHD converter</a>, which unfortunately does not seem to support big flat disks. I already tried converting disks with this tool, I know for a fact that is does work, so it must be the size of the file.
</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">
Then I tried using the <a href="http://www.vmware.com/support/ws55/doc/ws_disk_manager_running.html">VMWare Virtual Disk Manager</a> to convert the flat VMDK to a multiple 2GB VMDK spanned file. After the conversion, the VMDK to VHD converter worked perfectly by converting my spanned VMDK to a flat VHD disk compatible with Hyper-V.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This does not end here however, because mass storage drivers installed by Windows at install time for VMWare VMs are not compatible with the one Hyper-V is using. This leads to a nice BSOD saying INACCESSIBLE_BOOT_DEVICE, described <a href="/post/2004/10/Multiple-MassStorage-Drivers-with-Windows-2000XP2003-and-INACCESSIBLE_BOOT_DEVICE.aspx">here</a> and by the <a href="http://support.microsoft.com/default.aspx?scid=kb;en-us;314082">KB314082</a>.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">There&#39;s two ways to fix this : Create the reg file from the KB article and merge it when the VM is running under VMWare, or mount the VHD disk into an other Hyper-V VM and merge the reg file in the SYSTEM hive of the target OS.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">In my case the first possibility was out of the question; Moving the disk again to another machine would have been a waste of time.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">That left me with the registry hive mounting solution. Here&#39;s how to integrate the registry file : </font>
</p>
<ul>
	<li><font face="trebuchet ms,geneva" size="2">Use another VM to mount the target VHD, to be able to see the SYSTEM hive file.</font></li>
	<li><font face="trebuchet ms,geneva" size="2">Using the regedit, load the system hive from <em>System32\config\system</em> into <em>HKLM\temp.</em></font></li>
	<li><font face="trebuchet ms,geneva" size="2">Modify the KB reg file replace every reference to &quot;<em>SYSTEM\CurrentControlSet</em>&quot; by &quot;<em>temp\ControlSet001</em>&quot;, and import it to update the default boot configuration</font></li>
	<li><font face="trebuchet ms,geneva" size="2">Modify again the KB reg file to replace  every reference to &quot;<em>SYSTEM\CurrentControlSet</em>&quot; by &quot;<em>temp\ControlSet002</em>&quot;, to modify the &quot;Last Known Good Configuration&quot;, and import it , just in case.</font></li>
	<li><font face="trebuchet ms,geneva" size="2">Unload the hive.</font></li>
	<li><font face="trebuchet ms,geneva" size="2">stop the current VM.</font></li>
	<li><font face="trebuchet ms,geneva" size="2">Boot the new VM using the converted VHD disk, and voil&agrave; !</font></li>
</ul>
<font face="trebuchet ms,geneva" size="2">This is time consuming, but worth the trouble, the VM is now working properly under Hyper-V</font><font size="2">.<br />
</font>

{% include imported_disclaimer.html %}
