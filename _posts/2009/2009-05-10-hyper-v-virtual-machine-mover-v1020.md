---
layout: post
title: "Hyper-V Virtual Machine Mover 1.0.2.0"
date: 2009-05-10 15:20:00 -0400
comments: true
category: Archive
tags: []
redirect_from: ["/post/2009/05/10/Hyper-V-Virtual-Machine-Mover-v1020", "/post/2009/05/10/hyper-v-virtual-machine-mover-v1020"]
author: jay
---
<!-- more -->
<p>
Thanks to the guys at <a href="/admin/Pages/www.lakecomm.com">Lakewood Communications</a>, I&#39;ve updated my tool to move Hyper-V Virtual Machines to allow VMs that do not have any snapshots.
</p>
<p>
Version 1.0.2.0 also has a minor fix to try guessing the original VM path to replace it correctly in the configuration file. That would mean that you could have detached a VM and could not have attached it back.
</p>
<p>
On the Win2008 R2 front now the RC is out, for the time being since I don&#39;t have any spare hardware to test it on, I don&#39;t know if it works or if it still needed. If you do have tested my tool on this OS, please let me know. 
</p>
<p>
<a href="http://jaylee.org/page/Hyper-V-Virtual-Machine-Mover.aspx">Download the latest version here.</a>
</p>

{% include imported_disclaimer.html %}
