---
layout: post
title: "[wpdev] Tips and tricks about updating live tiles in Mango"
date: 2011-09-29 19:10:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/09/29/wpdev-tips-and-tricks-about-updating-live-tiles-in-mango.aspx"]
author: jay
---
<!-- more -->
<p><em>Cet article est aussi <a href="http://blogs.codes-sources.com/jay/archive/2011/10/01/wpdev-trucs-et-astuces-sur-la-mise-jour-tiles-dans-mango.aspx">disponible en francais</a>.</em></p>
<p>In the last published applications I've worked on, like <a href="http://www.windowsphone.com/en-US/apps/26cf3302-469f-e011-986b-78e7d1fa76f8">Foursquare</a>, <a href="http://www.windowsphone.com/en-US/apps/2e49fb07-592b-e011-854c-00237de2db9e">Flickr</a> or <a href="http://www.windowsphone.com/en-US/apps/7f7e3f68-ba3a-e011-854c-00237de2db9e">TuneIn</a>&nbsp;(and more are coming), all of them have live tiles, in both the Pull and locally generated tiles forms. But there are a few things to know to have a great experience with it, and you'll find it out by reading this article.</p>
<p>This is a very powerful feature, letting the user choose how to customize its own very personal experience, with no-one forcing the user into having a tile he does not want. This is the very same reasoning behind the absence of API to add items in Windows 7 task bar.</p>
<p>&nbsp;</p>
<h2>Live Tiles in Pull mode</h2>
<p>In the foursquare app there is the&nbsp;main tile, updated via the "pull" model, every hour or so (and the "or so" has a very strong meaning).</p>
<p>That&nbsp;tile that displays the Leaderboard is built in an Azure cloud service using a WPF offscreen rendering, based on&nbsp;the requests of the <a href="http://msdn.microsoft.com/en-us/library/ff769548(v=VS.92).aspx">tile Pull Engine</a>. This tile was built this way because of the limited capabilities of NoDo, where background agents were not available to render it locally on the device.</p>
<p>With&nbsp;Windows Phone Nodo, many users were complaining about the main tile not updating, and quite frankly, this has been a mystery up to the end. It seems like tiles would update on some devices, but not on others, but would <strong>only update if the battery power was more 50%</strong>.</p>
<p>Also, <strong>these tiles seemed to not update if the device is in standby</strong>, but only when the user sees the home screen, and when the suggested refresh delay has expired. I say "seem" because it seems like the rules behind this tile update were either not clear, or broken in some way.</p>
<p>This has changed with mango though. The Pull tiles are not updating almost all the time, but the 50% battery rule still&nbsp;seems to apply.</p>
<p>Also there's the rule of the 80KB file size JPEG, that if you go over, your tile won't be displayed.</p>
<p>&nbsp;</p>
<h2>Programmatic&nbsp;Live Tiles</h2>
<p>In Foursquare, the user may choose to pin a secondary "Tile" a specific place to its main screen for easy access.</p>
<p>Updating these tiles can be acheived with the <a href="http://msdn.microsoft.com/en-us/library/hh202979(v=VS.92).aspx">ShellTile API</a>, and with that you can set four thing:</p>
<ul>
<li>A&nbsp;title for the front and back</li>
<li>An image URL on the front and back</li>
<li>A&nbsp;four lines text on the back</li>
<li>A number on the front</li>
<li>(and you&nbsp;forget about the animated tiles like the people hub)</li>
</ul>
<p>&nbsp;</p>
<p>While all these features&nbsp;are interesting, only one of them is actually very useful: Images URL.</p>
<p>All the other properties are not stylable, they only&nbsp;follow the system's colors, and do not fit very well with user generated content. In the case of Foursquare, Flickr and TuneIn, the displayed images is user provided content, and having a white on white text is not very useful.</p>
<p>On the subject of image URLs, setting an external URL sets the image of the tile, but as long as the device does not reboot. <strong>If the device is rebooted, the tile looses its content.</strong> A pretty strange behavior, if you ask me.</p>
<p>&nbsp;</p>
<h2>Using the new isostore uri schema</h2>
<p>Fortunately, it is now&nbsp;possible to store the image locally in a special folder in the <a href="http://msdn.microsoft.com/en-us/library/hh202948(v=VS.92).aspx">isolated storage named&nbsp; /Shared/ShellContent</a>, and use the new <a href="http://msdn.microsoft.com/en-us/library/hh202861(v=VS.92).aspx">URI prefix "isostore"</a>, like this "isostore:/Shared/ShellContent/MyTile.jpg".</p>
<p>This means that you can download the image to display to the isolated storage, and use it from there.</p>
<p>But there's a big problem with using this technique : You do not control the size of the downloaded image. So if it is bigger than 80KB, you're stuck with the accent color background.</p>
<p>On a side note, I'd be curious to know the story behind this isostore prefix, because there are only two places that can use it, SQL CE Databases and Live Tiles. This prefix cannot be used as a Source property for Image controls, even though it would be <strong>very</strong> useful. But I digress.</p>
<p>&nbsp;</p>
<h2>Generating Live Tiles</h2>
<p>Hopefully, it's very much possible to generate a complete tile's content, using the <a href="http://msdn.microsoft.com/en-us/library/system.windows.media.imaging.writeablebitmap.render(v=vs.95).aspx">WriteableBitmap.Render</a> method. This method allows the offscreen rendering of any UIElement, then save it using the <a href="http://msdn.microsoft.com/en-us/library/system.windows.media.imaging.extensions.savejpeg(VS.92).aspx">SaveJpeg</a> extension method to persist it.</p>
<p>The tiles for Foursquare, Flickr and TuneIn are generated this way, using a&nbsp;user control that a real&nbsp;designer person&nbsp;created. This gives great looking tiles, and the layout and style can be updated depending on the dynamic content.</p>
<p>Here are a few things to generate tiles :</p>
<ul>
<li>The "new" (kinda)&nbsp;Silverlight 4 ViewBox control is very handy to resize text to fit the 173x173 layout,</li>
<li>You can use an Image control in your render source, but you need to wait for the BitmapImage (not the Image)&nbsp;to raise the ImageLoaded event, (The <a href="http://msdn.microsoft.com/en-us/data/gg577609">Reactive Extensions</a>&nbsp;can be very handy for that)</li>
<li>You'll also need to set the <a href="http://msdn.microsoft.com/en-us/library/system.windows.media.imaging.bitmapimage.createoptions(v=vs.96)">CreateOptions to None</a> on your&nbsp;BitmapImage to make sure the image is downloaded immediately,</li>
<li>If you download images, make sure you have a local fallback image underneath, just in case the remote image cannot be downloaded,</li>
<li>Before rendering the content, make sure to call <a href="http://msdn.microsoft.com/en-us/library/system.windows.uielement.measure(v=vs.96)">Measure</a> and <a href="http://msdn.microsoft.com/en-us/library/system.windows.uielement.arrange(v=vs.96)">Arrange</a> methods to force the layout to the 173x173 size required by the tiles.</li>
<li>You may need to call Measure and Arrange multiple times, because for some obscure reason, the control to be&nbsp; rendered may not honor these commands. Check for the <a href="http://msdn.microsoft.com/en-us/library/system.windows.frameworkelement.actualheight(v=vs.96)"><span style="color: #1364c4;">ActualHeight</span></a>&nbsp;and <a href="http://msdn.microsoft.com/en-us/library/system.windows.frameworkelement.actualwidth(v=vs.96)"><span style="color: #1364c4;">ActualWidth</span></a>&nbsp;properties values to see if they are correct.</li>
<li>Make sure to render your tile <strong>before</strong>&nbsp;pinning it to the&nbsp;home screen&nbsp;! The app is basically halted when you call the pin command, and the user may not come back&nbsp;to your app for you to finish the image rendering.</li>
<li>Don't take too long to render your tile though, if you wait too much, the user experience if pretty bad. That can definitely be a challenge when downloading content to be displayed on the tile.</li>
</ul>
<p>But then, you may only refresh your tiles when the application is running, unless you use the new Background Agents&nbsp;mango&nbsp;feature.</p>
<p>&nbsp;</p>
<h2>Updating the tiles with Background Agents</h2>
<p><a href="http://msdn.microsoft.com/en-us/library/hh202942%28v=VS.92%29.aspx">Background agents</a> are Microsoft's way of letting third party apps run code in the background, but with some&nbsp;big restrictions, like memory (4MB),&nbsp;schedule (30&nbsp;minutes)&nbsp;or&nbsp;duration limits (15 Seconds) for <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.scheduler.periodictask(v=VS.92).aspx">Periodic Tasks</a>.</p>
<p>Here are a few tricks about background agents :</p>
<ul>
<li>Periodic Agents run at a 30 minutes interval, and that is not configurable. So be gentle, you may want to add logic to avoid doing work too often, like not refreshing tiles during the night, and actually update the tile every 3 to 6 hours.</li>
<li>Don't wait too much to generate the tile, 15 seconds is very short. And your task may get killed by the OS before that.</li>
<li>Don't rely solely on the agent to run to update your tiles, the user may disable your agent using the Settings / Applications / Background Agent page. And the OS may prevent it&nbsp;from running, if it needs to.</li>
<li>Abuse of the <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.scheduler.scheduledactionservice.launchfortest(v=vs.92).aspx">ScheduledActionService<span>.</span>LaunchForTest</a>, to test your background agent,</li>
<li>A background agent runs your code in a different process than your application, meaning that both your app <strong>and</strong> the agent can run at the same time. Watch out for shared resources, like a SQL CE database or an isolated storage file.</li>
<li>If your are updating your tiles in both your application and your background agent, you may need to add some <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/aa365574(v=vs.85).aspx">IPC</a> using an old fashion&nbsp;<a href="http://msdn.microsoft.com/en-us/library/f55ddskf(v=VS.96).aspx">named-Mutex</a> (ahh, the good old days) and synchronize access to your resources.</li>
<li>Avoid referencing too many assemblies in your background agent, there are a lot of <a href="http://msdn.microsoft.com/en-us/library/hh202962(v=vs.92).aspx">Unsupported API</a> that may make your app fail certification. You can validate your app using the <a href="http://msdn.microsoft.com/en-us/library/hh394032(v=vs.92).aspx">Marketplace Test Kit</a> automated tests.</li>
</ul>
<p>About the first point, while I understand the power consumption concerns on running below 30 minutes, I still don't get why that interval cannot be set higher, to avoid that very same power consumption issue. There also must be a story behind this...</p>
<p>Then about the last point, during the Beta Phase of the Mango SDK, the <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.shell.standardtiledata.standardtiledata(v=VS.92).aspx">StandardTileData</a> class was considered an unsupported API, making the automatic background&nbsp;update of tiles impossible. Hopefully, this changed since the RC of the SDK and it is now possible to update tiles from background agents.</p>
<p>&nbsp;</p>
<p>That's it for now. Have fun with the tiles !&nbsp;</p>
{% include imported_disclaimer.html %}
