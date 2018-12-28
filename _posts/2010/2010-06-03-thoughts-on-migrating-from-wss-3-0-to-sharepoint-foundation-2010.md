---
layout: post
title: "Thoughts on Migrating from WSS 3.0 to SharePoint Foundation 2010"
date: 2010-06-03 21:31:00 -0400
comments: true
category: Archive
tags: ["Sharepoint"]
redirect_from: ["/post/2010/06/03/Thoughts-on-Migrating-from-WSS-3-0-to-SharePoint-Foundation-2010.aspx", "/post/2010/06/03/thoughts-on-migrating-from-wss-3-0-to-sharepoint-foundation-2010.aspx"]
author: admin
---
<!-- more -->
<p><a href="http://blogs.codes-sources.com/jay/archive/2010/06/04/notes-sur-une-migration-de-wss-3-0-vers-sharepoint-foundation-2010.aspx"><em>Cet article est disponible en francais.</em></a></p>
<p><em><br /></em></p>
<p>I upgraded recently a WSS 3.0 Farm to Sharepoint Foundation 2010, I tought I&rsquo;d share some of notes and pitfalls I found during the upgrade.</p>
<p>My setup is built around two Windows Server 2008 R2 64 Bits VMs hosted on Hyper-V Server R2, one VM for the frontend, one for the Database (SQL Server 2008 SP1 64 Bits) and the Search Server Express.</p>
<p>&nbsp;</p>
<h2><strong>Hyper-V Assisted Upgrade</strong></h2>
<p>Having the setup built on Hyper-V saved me a great deal of time, primarily by the use of snapshots taken at the same time on both machines. This helped a lot to be able to experiment directly on the production system during an expected downtime for the users.</p>
<p>The snapshots allow a trial and error process that lead to a somehow &ldquo;perfect&rdquo; environment where mistakes can be reversed pretty easily. Ugrading using this snapshot technique is however pratical only with enough disk space and a reasonable content database size depending on the physical hardware.</p>
<p>&nbsp;</p>
<h2><strong>Pre-Requisites</strong></h2>
<p>Here are the steps I followed to perform the upgrade :</p>
<ul>
<li>Made clones of both machines in a VM library, just to be safe in case the Hyper-V messed up the VMs because of the snapshots (you never know)</li>
<li>Upgraded WSS 3.0 with latest Cumulative Updates (<a href="http://support.microsoft.com/kb/978396" target="_blank">KB978396</a>)</li>
<li>Downloaded <span style="color: #ff0000;"><a href="http://www.microsoft.com/downloads/details.aspx?familyid=49C79A8A-4612-4E7D-A0B4-3BB429B46595&amp;displaylang=en" target="_blank">Sharepoint Foundation</a></span> and <span style="color: #ff0000;"><a href="http://www.microsoft.com/downloads/details.aspx?familyid=CEA31A4F-A8B4-4864-B520-BE612BECDCFA&amp;displaylang=en" target="_blank">Search Server Express</a></span> packages</li>
<li>Installed the prerequisites of Sharepoint Foundation on both VMs (not Search Servers prerequisistes, which do not install properly, and are seemingly the same as the SPF package)</li>
<li>Installed SQL Server 2008 SP1 Cumulative Updates <a href="http://support.microsoft.com/kb/970315" target="_blank">KB970315</a> and <a href="http://support.microsoft.com/kb/976761" target="_blank">KB976761</a> (in this order)</li>
</ul>
<p>That&rsquo;s the easy part, where updates do not impact the running farm.</p>
<p>I took a snapshot of both VMs at this point.</p>
<p>&nbsp;</p>
<h2><strong>Upgrading SharePoint and Search Server</strong></h2>
<p>You may want to <a href="http://technet.microsoft.com/en-us/sharepoint/ee517215.aspx" target="_blank">read information on this on technet</a>, which is very extensive.</p>
<p>Now, the Sharepoint upgrade :</p>
<ul>
<li>Put a site lock in place (just in case a user might try to update stuff he&rsquo;d probably lose)    
<ul>
<li><span style="font-family: Courier New;">stsadm -o setsitelock -url </span><a href="http://site"><span style="font-family: Courier New;">http://site</span></a><span style="font-family: Courier New;"> -lock readonly</span></li>
</ul>
</li>
<li>Detached the content databases using : (See later for the explanation of this step)&nbsp; :    
<ul>
<li><span style="font-family: Courier New;">stsadm.exe -o deletecontentdb -url </span><a href="http://site"><span style="font-family: Courier New;">http://site</span></a><span style="font-family: Courier New;"> &ndash;databasename WSS_Content</span></li>
</ul>
</li>
<li>Backup the content DB so they can be upgraded on an freshly installed SPF2010 setup. </li>
<li>Executed the Search Server Express setup on both machines, without configuring it</li>
<li>Upgraded Sharepoint Foundation setup, without configuring it</li>
<li>On the frontend VM (so the admin site can track the update jobs), ran the Sharepoint Configuration wizard to perform the upgrade. I selected the Visual Upgrade so the site collections templates would use the new visual style (Ribbon powered !)</li>
<li>After the configuration ended on the frontend machine, ran the same wizard on the database VM.</li>
<li>Let the jobs run and finish properly.</li>
<li>On a temporary empty SPF 2010 setup on a spare VM, mounted the backed up content DB and ran this powershell command :    
<ul>
<li><span style="font-family: Courier New;">Mount-SPContentDatabase -Name WSS_Content -DatabaseServer db.server.com -WebApplication </span><a href="http://site"><span style="font-family: Courier New;">http://site</span></a><span style="font-family: Courier New;"> &ndash;Updateuserexperience</span></li>
<li>You may require to install the templates used on your production environment to perform the upgrade properly.</li>
</ul>
</li>
<li>After the content db upgrade ended, detached the content database using the admin site (beware of the dependencies between content DBs if you have more than one)</li>
<li>Backuped up the content DB.</li>
<li>On the production Farm, dropped and recreated the appropriate Web Application without any site collection. I did this step to make sure the AppPools and sites were configured properly.</li>
<li>Restored and attached the content DB on the production server using the SPF admin site.</li>
</ul>
<p>Now, I did perform the &ldquo;Attach/detach&rdquo; procedure because it can be performed in parallel on multiple farms if the content databases are huge, and on my production setup, the in-place upgrade did not work properly. The images libraries where not upgraded properly (images where not displayed), and the default pages did not render properly for some obscure reason.</p>
<p>&nbsp;</p>
<h2><strong>A few additional gotchas</strong></h2>
<ul>
<li>I had a few other issues with the search SSP, where I needed to remove completely the Search SSP and recreate it to avoid this error :    
<ul>
<li><span style="font-family: Courier New;">CoreResultsWebPart::OnInit: Exception initializing: System.NullReferenceException</span></li>
</ul>
</li>
<li>The search SSP needs uses of the Security Token Service Application, which by default does use the &ldquo;Extended Security&rdquo; setting, which <a href="http://social.technet.microsoft.com/Forums/en-US/sharepoint2010setup/thread/9cdf5def-5019-4e82-83a2-e62cf0785bf6" target="_blank">needs to be turned off in IIS</a>.</li>
<li>Since I use Search Server Express, Content Databases must not select the search provider named &ldquo;Sharepoint Foundation Search Server&rdquo; for the search to work properly.</li>
</ul>
<p>&nbsp;</p>
<h3><strong>Upgraded Wikis</strong></h3>
<p>You&rsquo;ll find the the wiki editor has been greatly enhanced, and you&rsquo;ll find it even more powerful when you select the &ldquo;Convert to XHTML&rdquo; option in the html menu of the ribbon. The original pages were using the very loose HTML 4.0, which does not seem to work very well.</p>
<p>Other than that, everything else works very fine in this area.</p>
<p>&nbsp;</p>
<h3><strong>Upgraded Discussion Boards</strong></h3>
<p>I had a few discussion boards that had issues with the view of individual conversations using the threaded view. The conversation were defaulted to the default &ldquo;subject&rdquo; view, which is not applicable to view single threads. To fix this, make a new &ldquo;subject&rdquo; view and delete the previous one the normal behavior will come back.</p>
<p>&nbsp;</p>
<p>Happy SharePointing ! Now I can go back to my Immutability and F# stuff :)</p>
{% include imported_disclaimer.html %}
