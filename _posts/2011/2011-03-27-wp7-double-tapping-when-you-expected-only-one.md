---
layout: post
title: "[WP7Dev] Double tap when you expect only one"
date: 2011-03-27 19:24:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/03/27/WP7-Double-tapping-when-you-expected-only-one", "/post/2011/03/27/wp7-double-tapping-when-you-expected-only-one"]
author: jay
---
<!-- more -->
<p>I've been developing a <a href="http://jaylee.org/rc/wp7">free application to do some PC&nbsp;remote control</a> on Windows Phone 7, and it's been very instructive in many ways.</p>
<p>To improve the quality of the software, and be notified when an unhandled exception occurs somewhere in my code, or in someone else's code executed on my behalf, I've added a small opt-in unhandled exception reporting feature. This basically sends me back&nbsp;information about the device, most of what's available in <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.info.deviceextendedproperties(v=vs.92).aspx">DeviceExtendedProperties</a>&nbsp;for device aggregation of exceptions, plus some informations like the culture and, of course, the exception stacktrace and details.</p>
<p>&nbsp;</p>
<h2>The MarketplaceDetailTask exception</h2>
<p>A few recurring exceptions have popped up a lot&nbsp;recently, and one coming often is the following :</p>
<p><span style="font-family: courier new,courier;">Exception : System.InvalidOperationException: Navigation is not allowed when the task is not in the foreground. Error: -2147220989&nbsp; </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Shell.Interop.ShellPageManagerNativeMethods.CheckHResult(Int32 hr)&nbsp; </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Shell.Interop.ShellPageManager.NavigateToExternalPage(String pageUri, Byte[] args)&nbsp; </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Tasks.ChooserHelper.Navigate(Uri appUri, ParameterPropertyBag ppb)&nbsp; </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Tasks.MarketplaceLauncher.Show(MarketplaceContent content, MarketplaceOperation operation, String context)&nbsp; </span><br /><span style="font-family: courier new,courier;">at Microsoft.Phone.Tasks.MarketplaceDetailTask.Show()&nbsp;</span></p>
<p>This code is called when a user clicks on the purchase image located on some page of the software, and it looks like this :</p>
<p>[code:c#]<br />void&nbsp;PurchaseImage_ManipulationCompleted(object&nbsp;sender,&nbsp;ManipulationCompletedEventArgs&nbsp;e)<br />{<br />&nbsp;&nbsp;&nbsp; var&nbsp;details&nbsp;=&nbsp;new&nbsp;MarketplaceDetailTask();<br />&nbsp;&nbsp;&nbsp; details.ContentIdentifier&nbsp;=&nbsp;"d0736804-b0f6-df11-9264-00237de2db9e";<br />&nbsp;&nbsp;&nbsp; details.Show();<br /> }<br />[/code]</p>
<p>The call is performed directly on the image's ManipulationCompleted event.</p>
<p>I've been trying to reproduce it for a few times and I finally got it: <strong>The user is tapping more than once on the image.</strong></p>
<p>I can see a few reasons why:</p>
<ul>
<li>The ManipulationCompleted event is fairly sensitive and is raised multiple times when the user did not tap twice</li>
<li>The user did actually tap twice because the action did not answer fast enough.</li>
<li>The user tapped twice because he is used to always&nbsp;tap twice, as some PC users do... (You know, double clicking on hyperlinks in browsers, things like that)</li>
</ul>
<p>&nbsp;</p>
<h2>What do we do about it ?</h2>
<p>There may be actually more, be this actually tells me a lot.</p>
<p>First, I should be having some kind of visual&nbsp;feedback on the click of that image, to tell the user that he has done something (and also to actually follow the design guidelines)</p>
<p>Second,&nbsp;that even if the feedback is there, that there may always be two subsequent clicks, and one that may be executed after the first has called the <span style="color: #2b91af;">MarketplaceDetailTask.Show()</span>, and the application has been deactivated.&nbsp;I cannot do much about it, except handle the exception silently or track the actual application state and not call the method.</p>
<p>I'll go with the exception handler for now as it is not&nbsp;a very critical peace of code, but I'd rather have some way of that tell me that the application cannot do that reliably and not have to handle an exception. The API is rather limited on that side, where the <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.shell.phoneapplicationservice_members(v=VS.92).aspx">PhoneApplicationService</a> is only raising events and does not expose the current "activation" state.</p>
<p>&nbsp;</p>
<h2>Any other examples of exceptions ?</h2>
<p>I'll talk more about some other findings this opt-in exception reporting feature has brought me, with&nbsp;some that seem to be pretty tricky.&nbsp;</p>
{% include imported_disclaimer.html %}
