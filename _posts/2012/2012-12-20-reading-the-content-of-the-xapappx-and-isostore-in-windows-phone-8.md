---
layout: post
title: "Reading the content of the Xap/Appx and IsoStore in Windows [Phone] 8"
date: 2012-12-20 09:57:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2012/12/20/Reading-the-content-of-the-XapAppx-and-IsoStore-in-Windows-Phone-8", "/post/2012/12/20/reading-the-content-of-the-xapappx-and-isostore-in-windows-phone-8"]
author: jay
---
<!-- more -->
<p>In Windows Phone 8, things got better of the front of reading the file system. It&rsquo;s actually gotten closer to the Windows 8 with the inclusion of WinRT, and it now possible to read the content of files that were pretty hard (or impossible)&nbsp;to access directly before.</p>
<p>&nbsp;</p>
<h2>Reading the app package content files</h2>
<p>There are some times when you need to access the content of the xap directly, and in Windows Phone 7, you were pretty limited to the use of <a href="http://msdn.microsoft.com/en-us/library/3ak841sy.aspx">the IsolatedStorage</a>, but really did not have access to the content of the Xap, except for some location such as images <a href="http://www.developer.nokia.com/Community/Wiki/How_to_access_Application_Manifest_(WMAppManifest.xml)_file_at_runtime">or XmlDocument</a> and the SQL CE engine <a href="http://msdn.microsoft.com/en-us/library/hh202861%28v=vs.92%29.aspx">using the appdata:/ scheme</a>.</p>
<p>Things have changed in Windows Phone 8, and here&rsquo;s how to read the WMAppManifest.xml file by yourself :</p>
<p>[more]</p>
<pre class="brush: c-sharp">   var file = await StorageFile.GetFileFromPathAsync(
      Path.Combine(
         Windows.ApplicationModel.Package.Current.InstalledLocation.Path,   
         "WMAppManifest.xml"
      ) 
   );

   using (var stream = new StreamReader(await file.OpenStreamForReadAsync()))
   {
      var content = await stream.ReadToEndAsync();
   }</pre>
<p>That way, you can access anything located in your package, and use it like any other file. Also note that this technique is also working on Windows 8.</p>
<p>&nbsp;</p>
<h2>Reading the content of the IsolatedStorage</h2>
<p>The IsolatedStorage class is just a wrapper around the file system that was introduced in .NET a while back, but since it is now possible to access the file system using WinRT, it is possible to read the isolated storage folder content directly.</p>
<p>It&rsquo;s just a matter of using <a href="http://msdn.microsoft.com/en-us/library/windows/apps/xaml/Hh700361.aspx">Windows.Storage.ApplicationData.Current.LocalFolder</a> and replicating the code above, which will make for a very portable code between Windows 8 and Windows Phone 8.</p>
{% include imported_disclaimer.html %}
