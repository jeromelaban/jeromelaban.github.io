---
layout: post
title: "Reducing apps startup time with Pre-JITing and NGEN on a Surface RT"
date: 2012-11-24 21:13:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/11/24/Reducing-apps-startup-time-with-Pre-JITing-and-NGEN-on-a-Surface-RT.aspx", "/post/2012/11/24/reducing-apps-startup-time-with-pre-jiting-and-ngen-on-a-surface-rt.aspx"]
author: jay
---
<!-- more -->
<p><em>TL;DR: The JIT can take over a third of the startup time of a managed Metro App, and using&nbsp;Native Image Generation (NGEN)&nbsp;can greatly improve the startup time of these apps. There is also a way to check for these native images to act accordingly.</em></p>
<p><em></em>&nbsp;</p>
<p>A while back, <a href="http://jaylee.org/post/2012/06/11/Improving-the-Startup-Time-of-Xaml-Metro-Style-Apps-with-Multicore-JIT.aspx">I&rsquo;ve had the chance to work with the guys that are behind the Pre-JIT </a>feature of the CLR 4.5 for&nbsp;Metro Apps. Back then, I was only able to work on x86/x64 architectures, as ARM/Windows RT devices were not available.</p>
<p>Now that the Surface RT devices&nbsp;are available, we&rsquo;re facing quite a few challenges in terms of code execution performance, and I&rsquo;m going to discuss a few tips and tricks about the Managed Code JIT on Windows RT.</p>
<p>&nbsp;</p>
<h2>Profiling a slow starting app on a Surface RT</h2>
<p>Running apps on the Surface can be troubling. Having an app that is useable after 16 to 18 seconds is definitely not acceptable, let alone the fact that the Splash Screen can disappear after 6 to 8 seconds.</p>
<p>Profiling such an app that starts slowly is very interesting, when looking a the <a href="http://msdn.microsoft.com/en-us/library/z9z62c29.aspx">Visual Studio profiler</a>, where during these 17 seconds, about a third is spent in a &ldquo;clr.dll&rdquo; module in exclusive time (time spent only in this module and not its descendants). This is a very big number.</p>
<p>This time is actually spent in the JIT, where big methods tend to take more time to be JITed, sometimes on the UI thread, making the app sluggish.</p>
<p>[more]</p>
<p>This JIT phase is expected though, and is since .NET 1.0. Microsoft chose not to JIT metro&nbsp;apps on installation supposedly to improve the user's interaction flow with apps, and did not choose to make it in the cloud like the Windows Phone team did, starting from Windows Phone 8. The drawback of this approach&nbsp;is that managed code apps do have an extremely poor first launch experience, where the JIT kicks in every time.</p>
<p>The good news is that there are workaround for this slow startup, Pre-JIT and NGEN.</p>
<p>&nbsp;</p>
<h2>Pre-JITing applications</h2>
<p><a href="http://jaylee.org/post/2012/06/11/Improving-the-Startup-Time-of-Xaml-Metro-Style-Apps-with-Multicore-JIT.aspx">Pre-JIT</a> is based on profiling how an app JITs code, then creating a file out of it that can be bundled with the final app package. In later startups of the app, that file is used of JIT code ahead of time, substantially improving the app&rsquo;s startup time.</p>
<p>Yet, this technique is not the panacea, because it still requires the CPU to JIT code every time, preventing the rest of the app to use that CPU-time for some other useful tasks.</p>
<p>While this profile technique works like a charm on Windows 8 desktop, I&rsquo;ve yet to notice any difference in the startup time of apps on Windows RT. I&rsquo;m not sure if it&rsquo;s the CPU that&rsquo;s not powerful enough to JIT ahead of the normal execution path, or if it&rsquo;s simply not used at all.</p>
<p>Interestingly, I&rsquo;ve been able to generate that profile on a Surface device, but that&rsquo;s pretty much it.</p>
<p>&nbsp;</p>
<h2>NGEN on Windows RT</h2>
<p>Fortunately, apps are NGENed roughly after 24 hours. This technique, which has been around since .NET 1.0, has the ability to take managed code and generate a &ldquo;native&rdquo; image which is basically a fully JITed assembly that targets the current CPU architecture.</p>
<p>That same app that takes 17 seconds to start and be useable drops down to 12 seconds. This makes the startup a lot more interesting! (12 seconds is still not acceptable, but using other optimizations like using the 4 logical cores, that same app is now down to 8 second, which is a lot better)</p>
<p>This NGEN tool is ran as part of a &ldquo;Maintenance Task&rdquo;, located in the standard Windows Scheduler. It can be found when searching for &ldquo;Schedule tasks&rdquo; in the start menu, then under &ldquo;Microsoft / Windows / .NET Framework&rdquo;. Right clicking and selecting Run for both &ldquo;.NET Framework NGEN v4.0.30319&rdquo; and &ldquo;.NET Framework NGEN v4.0.30319 64&rdquo; will generate native images for applications that have been run at least once for what seems to be about 10 to 15 seconds.</p>
<p>On the Surface RT, generating these images can take quite some time depending on the size of the app. I&rsquo;ve seen apps that take about 30 seconds to a minute to fully generate. Also note that there are space constraints, such as available free space. If your system drive has limited free space (such as below 1GB), NGEN will not generate anything.</p>
<p>Microsoft recommends <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh994639.aspx">running this tool</a> by hand using a command line, but I&rsquo;ve yet to see it work properly on a Surface RT device&hellip; Only running&nbsp;the scheduled task manually actually generates native images.</p>
<p>&nbsp;</p>
<h2>Checking&nbsp;for native images</h2>
<p>Knowing that you&rsquo;re running with or without native images can allow you to act differently to mitigate&nbsp;the terrible initial startup time.</p>
<p>The generated native images are located under the following folder:</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">%LOCALAPPDATA%\Packages\[AppID]\AC\Microsoft\CLR_v4.0_32\NativeImages</span></p>
<p>Testing for its presence is enough to guess it the app is running without JIT.</p>
<p>The following code can tell you this :</p>
<pre class="brush: c-sharp">    private async Task IsNativeImage()
    {
        var source = Windows.Storage.ApplicationData.Current.LocalFolder.Path + "\\..\\AC\\Microsoft\\";
            
        try
        {
            await StorageFolder.GetFolderFromPathAsync(source + "CLR_v4.0_32\\NativeImages");
            return true;
        }
        catch (Exception ex) { }

        try
        {
            await StorageFolder.GetFolderFromPathAsync(source + "CLR_v4.0\\NativeImages");
            return true;
        }
        catch (Exception ex) { }

        return false;
    }
</pre>
<p>So when testing for your app&rsquo;s performance, don&rsquo;t forget to manually run NGEN&nbsp;and create a JIT profile&nbsp;!</p>
<p>&nbsp;</p>
{% include imported_disclaimer.html %}
