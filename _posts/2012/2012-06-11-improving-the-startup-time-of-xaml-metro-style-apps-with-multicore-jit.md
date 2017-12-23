---
layout: post
title: "Improving the Startup Time of Xaml Metro Style Apps with Multicore JIT"
date: 2012-06-11 05:15:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/06/11/Improving-the-Startup-Time-of-Xaml-Metro-Style-Apps-with-Multicore-JIT", "/post/2012/06/11/improving-the-startup-time-of-xaml-metro-style-apps-with-multicore-jit"]
author: jay
---
<!-- more -->
<p><em><a href="http://blogs.developpeur.org/jay/archive/2012/06/11/ameliorer-le-demarrage-des-applications-xaml-style-metro-avec-le-jit-multicoeurs.aspx">Ce billet est disponible en francais.</a></em></p>
<p><em>TL;DR: Microsoft introduced the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.profileoptimization(v=vs.110).aspx">Multicore JIT</a>, which allows the recording of JITted methods during the startup of the app. This recording can be packaged in a Metro Style app for faster startup on Multicore CPUs by performing background compilation. Improvements range between 20% to 50%.</em></p>
<p>&nbsp;</p>
<p>Since the beginning of the year, I&rsquo;ve had the chance to work with some very interesting people at Microsoft, and one of the feature that came out from them was about the use of a new .NET 4.5 feature called <a href="http://msdn.microsoft.com/en-us/library/system.runtime.profileoptimization(v=vs.110).aspx">Multicore JIT</a> in Metro Style apps.</p>
<p>[more]</p>
<h2>The JIT Compiler</h2>
<p>The <a href="http://msdn.microsoft.com/en-us/library/ht8ecch6(v=vs.90).aspx">JIT compiler step</a> has been there in .NET since the version 1.0. It takes the Intermediate Language (MSIL) that the C# compiler generates, and compiles it to native instruction of the CPU.</p>
<p>This extra step to generate IL has proven to be useful in many occasions, and because IL is platform agnostic, this means that an assembly can be compiled once, and executed on multiple CPUs.</p>
<p>This has once again come very handy with <a href="http://windowsteamblog.com/windows/b/bloggingwindows/archive/2012/04/16/announcing-the-windows-8-editions.aspx">WoA (or Windows RT)</a>, where it is very easy to produce an app package compatible with the three platforms supported by Metro apps (x86, x64 and ARM).</p>
<p>But it has also its drawbacks. It&rsquo;s an extra synchronous step to execute code, meaning that it will slow down the execution path of your program, particularly at startup.</p>
<p>That is, unless you&rsquo;re using NGEN to precompile everything before hand. But NGEN is somehow cumbersome, as it requires your assemblies to be strongly signed and put in the GAC. This has the unfortunate effect of having NGEN not that much used, except for libraries.</p>
<p>&nbsp;</p>
<h2>JIT and Hardware Performance Bumps</h2>
<p>In 2000, computers were not that fast, and taking the route of JITing, with the technologies available at that time, required the use of NGEN to have acceptable performance.</p>
<p>As time passes, desktop computers became faster and faster, and the JIT tended to become transparent. Nowadays, it can almost be ignored, unless you do have a <strong>huge </strong>amount of code to execute.</p>
<p>But as history tends to repeat itself, and now that we&rsquo;re running after always longer battery life, devices like Windows Phone and WoA tablets that use low-powered CPUs appear again, with performance in the range of what could be found in 2000, JIT is becoming yet again an issue as <a href="http://www.jaylee.org/post/2011/12/02/WPDev-The-hidden-cost-of-IL-Jitting.aspx">I&rsquo;ve painfully discovered on Windows Phone</a>.</p>
<p>I&rsquo;m expecting to see the same kind of JIT performance problems on WoA, and while we&rsquo;re all playing with Metro-style apps on super-fast i5 or i7 computers, we do not see the impact of JITing.</p>
<p>&nbsp;</p>
<h2>Multicore JIT (MCJ)</h2>
<p>Microsoft seems to have taken the bull by its horns, and attacked the JIT problem using recent technologies.</p>
<p>Historically, the JIT was happening on the thread calling the method to be JITted, impacting the performance of that code path. These days, most CPUs have multi-cores, even on the ARM platform, and having the JIT take advantage of it would almost certainly be a win for performance.</p>
<p>On .NET 4.5, this feature will allow the recording of &ldquo;profiles&rdquo;, where the JITted methods signatures are persisted in a file. This allows, in later reruns of the app, to pre-JIT methods ahead of time on multiple cores, so that the startup path does not wait for them to be compiled.</p>
<p>The feature does not seem to be based on NGEN, meaning that it is not required to strongly-sign assemblies nor put them in the GAC. A big win, if you ask me.</p>
<p>&nbsp;</p>
<h2>Multicore JIT and Metro Style apps</h2>
<p>The .NET Core Profile that is used to run XAML/C# based apps is evidently using most of the .NET 4.5 code base, and this new feature is also available. It is specifically directed at first runs of apps because after a 24 hours, the app is automatically fully JITted into native images. At that point, Multicore JIT is not used anymore.</p>
<p>While there is an API to generate JIT profile recording files on the full .NET 4.5, the technique for Metro Style apps is a bit hidden, but straight forward.</p>
<p>Microsoft has recently published the <a href="http://support.microsoft.com/kb/2715214">KB2715214 article named &ldquo;<em>How to reduce JIT time during initial start of Windows Metro style apps</em>&rdquo;</a> on how to use it.</p>
<p>I&rsquo;ll give you a short version :</p>
<ul>
<li>In your main assembly, create empty file named &ldquo;[assembly_name]_[EntryPoint].profile&rdquo;. The entry point is commonly &ldquo;App&rdquo;. Make sure the file is really empty, it should show 0KB in the Windows Explorer.</li>
<li>Run your app in release without the debugger, and try to go as far as possible into your app, so you can grab many methods. The profiler takes 10 seconds of data.</li>
<li>Navigate to %localappdata%\Packages\[YOUR_APP_PACKAGE]\AC</li>
<li>Get the profile file and replace the empty one in your project</li>
</ul>
<p>&nbsp;</p>
<p>You&rsquo;re done !</p>
<p>If the file is not generated, this can be because :</p>
<ul>
<li>You named it wrong in the project</li>
<li>You did not set it to &ldquo;Content&rdquo; in its properties</li>
<li>Or you&rsquo;re running on a single-core machine.</li>
</ul>
<p>&nbsp;</p>
<h2>Measures on Published Apps</h2>
<p>I&rsquo;ve been working on 3 Metro apps (Photobucket, Allrecipes and TuneIn) that have been released in the Windows 8 Release Preview Store, and using this feature has greatly improved apps startup time. The improvements range from 25% to 50% startup time, depending on the amount of code executed during startup.</p>
<p>In practice, on a Core Duo Mobile (1.8GHz) machine, the app takes 3.20s to startup without MCJ, and about 1.73s with MCJ enabled. On a Quad Core i7 @ 2.8 GHz, the app startup is well under a second, which makes performance measuring very unreliable, particularly when talking about actual user perceived performance.</p>
<p>I&rsquo;m very eager to see what will happen on WoA devices&hellip; Oh well, we&rsquo;ll all that see this fall.</p>
<p>&nbsp;</p>
<h2>Caveats of MCJ</h2>
<p>While this feature is very interesting, it records a snapshot of the startup path of an app. This means that as soon as your code changes, the MCJ may start JITing methods that are not anymore in your startup path.</p>
<p>This will not destabilize the application, but you may progressively loose the benefits of MCJ, and at the very worse, degrade the startup path of your app. But that&rsquo;s unlikely.</p>
<p>You&rsquo;ll then need to think of re-creating your JIT profile files, right before shipping your app, and not rely on a profile you&rsquo;ve made before.</p>
{% include imported_disclaimer.html %}
