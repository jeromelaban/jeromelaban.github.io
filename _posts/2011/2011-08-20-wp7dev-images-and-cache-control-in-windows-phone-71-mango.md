---
layout: post
title: "[wp7dev] Images and cache control in Windows Phone 7.1 (Mango)"
date: 2011-08-20 16:39:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/08/20/wp7dev-Images-and-cache-control-in-Windows-Phone-71-Mango.aspx", "/post/2011/08/20/wp7dev-images-and-cache-control-in-windows-phone-71-mango.aspx"]
author: jay
---
<!-- more -->
<p><strong><em>TLDR: Windows Phone 7.1 Mango's Image control now&nbsp;respects HTTP/1.1 server cache directives, and particularly the max-age, meaning better performing apps. And it is doing it better than you could ever do&nbsp;:)</em></strong></p>
<p>Image loading&nbsp;is one of the weakest parts of third party apps on Windows Phone 7.0 (NoDo).</p>
<p>The Flickr app in WP7 is a good demonstration of this. The app is basically stalling during the loading of images, and there are (obviously) a lot of images loaded by this app. (The Mango release will make this app really awesome and responsive, I can tell you :))</p>
<p>There are two main reasons for this, being&nbsp;the image loading happening on the UI Thread and&nbsp;partial cache persistence.</p>
<p>&nbsp;</p>
<h2>Hacking around image loading in NoDo</h2>
<p>So if you look around to mitigate those two issues, you'll find a few things like the <a href="http://blogs.msdn.com/b/delay/archive/2010/09/02/keep-a-low-profile-lowprofileimageloader-helps-the-windows-phone-7-ui-thread-stay-responsive-by-loading-images-in-the-background.aspx">LowProfileImageLoader from a Microsoftee</a>. This removes a lot of burden from the UI thread by not&nbsp;using the WebClient, and&nbsp;queueing requests to avoid having too many dowloads at the same time.</p>
<p>But <a href="http://jaylee.org/post/2011/04/22/WP7-HttpWebRequest-and-the-Flickr-app-Black-Screen-issue.aspx">like I've discussed before</a>, this is still not&nbsp;the perfect solution because HttpWebRequest still goes on to the UI thread, and when there are many images loaded the UI becomes easily sluggish.</p>
<p>For the image cache part, Silverlight will cache BitmapImage instances based on the Url, will persist&nbsp;them across application runs but will ignore the <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.3">HTTP/1.1 max-age</a> directive. This means that each time you run the application, the app will try to refresh the image again. It may not be downloaded again, but it still is checked. This may delay a lot the display of the image, because of the wait for the server to check if the image has changed.</p>
<p>If you still want to do some sort of caching without asking the server every time, then you need to handle the storage of downloaded streams and use <a href="http://msdn.microsoft.com/en-us/library/system.windows.media.imaging.bitmapsource.setsource(v=vs.95).aspx">BitmapSource.SetSource</a>, and perform some in-memory caching of BitmapSource instances to still use the Silverlight cache even if you can't provide an Url. And all this has to be performed on the UI thread. It <strong>really</strong>&nbsp;does not help.</p>
<p>These are many roadblocks that hurt badly the perceived performance of the application.</p>
<p>&nbsp;</p>
<h2>Images in Mango</h2>
<p>If you try to the same in Mango, doing the background download and caching by yourself, you end up making the matters worse.</p>
<p>Mango has changed everything on that front, and the Product Team has addressed these issues in a very nice way. Loading images is now seamless, you can download as many as you want and UI will not lag a bit.</p>
<p>If you observe the loading of images in mango, you'll quickly see that cached images are displayed almost instantly, primarily because of the cache engine respecting server cache directives. This means that an image will not be checked for a refresh nor downloaded again if the cache duration has not expired.</p>
<p>All this&nbsp;means that you pretty much don't need to do anything to display images in Mango, unless you need to bypass server caching directives.</p>
<p>This is good news :)</p>
<p>Also, Silverlight seems to be doing some work off&nbsp;the UI thread the "user code" (us,&nbsp;mere&nbsp;mortals)&nbsp;cannot do because it needs to be on the UI thread, meaning that you have to let Silverlight do its magic to load images the fastest and seamless way possible.</p>
<p>&nbsp;</p>
<h2>Image caching in Mango</h2>
<p>By looking closer to the way Mango does caching, I've noticed a few things :</p>
<ul>
<li>Images seem to be downloaded once per application run, meaning that server cache directives are ignored until a restart of the application (Fast-resume does not seem to count as a restart),</li>
<li>Images that need to be refreshed are checked for modifications on the server, and if an HTTP 302 is sent back, the cached image stays.</li>
<li>ETag is supported, the If-None-Match header is sent when the max-age has been reached.</li>
<li>If-Modified-Since is also sent when the max-age time span has been reached,</li>
<li>When using <a href="http://msdn.microsoft.com/en-us/library/system.windows.media.imaging.bitmapcreateoptions(v=vs.96).aspx">BitmapCreateOptions.IgnoreImageCache</a>
<ul>
<li>Server cache directives do not seem to be bypassed, the cache is not refreshed until max-age has been reached</li>
<li>If Cache-Control max-age and Expires headers are not specified the cache does not seem to be ever refreshed</li>
<li>If Expires is specified but not max-age, then the server is called to check for a newer version with If-Modified-Since</li>
</ul>
</li>
</ul>
<p>These are very good news, since most web server implementation support and respect the <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.4">HTTP/1.1 Cache-Control directives</a>, meaning that images will be displayed and refreshed properly by default.</p>
{% include imported_disclaimer.html %}
