---
layout: post
title: "SharePoint : The database connection string is not available. (0xc0041228)"
date: 2009-03-31 20:21:00 -0400
comments: true
category: Archive
tags: ["Sharepoint"]
redirect_from: ["/post/2009/03/31/Sharepoint-The-database-connection-string-is-not-available-(0xc0041228)", "/post/2009/03/31/sharepoint-the-database-connection-string-is-not-available-(0xc0041228)"]
author: jay
---
<!-- more -->
<p>
A Sharepoint Services 3.0 setup I&#39;m managing had a few issues lately, and I had to bring back up an old version of the system. The original setup had a Search Server Express 2008 installed, and the backup I restored did not, even though I had the databases for it. 
</p>
<p>
After reinstalling everything that was needed, and having the Search Server properly indexing content, I kept having a lot of messages like<font face="courier new,courier"> &quot;The database connection string is not available.&quot;</font> in the event log, and<font face="courier new,courier"> &quot;Your search cannot be completed because of a service error.&quot;</font> in the search tool in any Sharepoint site. I had the content database properly associated with the correct indexer.
</p>
<p>
I did not notice at first that the service named &quot;Windows SharePoint Services Search&quot; was not started, and when I tried to start it, I had a nice <font face="courier new,courier">&quot;The handle is invalid.&quot;</font> error message... Not very helpful.
</p>
<p>
A few posts around the web were suggesting to stop that service, then restart it. One suggested to check the user account of that service, which was &quot;Network Service&quot; for me. I changed it to the same domain account that the &quot;Windows SharePoint Services Timer&quot; service is using. At this point, the service was starting properly, but I was still having the<font face="courier new,courier"> &quot;The database connection string is not available.&quot;</font> message.
</p>
<p>
In the &quot;Services on Server&quot;, I tried stopping the &quot;Windows SharePoint Services Search&quot; service, (telling me that it was going to delete the index content), which succeeded. But trying to restart the service gave me an error saying that the database already had content, and that I had to create a new one.
</p>
<p>
I did create a new database, but the service would still not start, this time giving an other error message that I enjoy so much : <font face="courier new,courier">&quot;An unknown problem occured&quot;</font>.
</p>
<p>
I went back to some forum posts, and I came across a command to &quot;initialize&quot; the service from the command line with STSADM :
</p>
<p>
<font face="courier new,courier">&nbsp;stsadm -o spsearch -action start -farmserviceaccount [domain_account] -farmservicepassword [domain_account_password]</font>
</p>
<p>
Which at first gave me this :
</p>
<p>
<font face="courier new,courier">&nbsp;The specified database has an incorrect collation.&nbsp; Rebuild the database with the Latin1_General_CI_AS_KS_WS collation or create a new database.</font>
</p>
<p>
I did re-create the database with the proper collation, then ran stsadm again and it gave me this :&nbsp;
</p>
<p>
<font face="courier new,courier">&nbsp;The Windows SharePoint Services Search service was successfully started.</font>
</p>
<p>
Hurray ! That did the trick, and indeed, my searches in any Sharepoint sites were not returing any error. I just had to wait for the service to refresh its index, and my search was running again !
</p>
<p>
This is a long and verbose post, but I hope this will help someone with this cryptic message... 
</p>
<p>
&nbsp;
</p>
<p>
&nbsp;
</p>

{% include imported_disclaimer.html %}
