---
layout: post
title: "On the Startup Performance of a WPF ElementHost in Winforms"
date: 2009-08-09 17:34:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2009/08/09/On-the-Startup-Performance-of-a-WPF-ElementHost-in-Winforms", "/post/2009/08/09/on-the-startup-performance-of-a-wpf-elementhost-in-winforms"]
author: jay
---
<!-- more -->
<span class="Apple-style-span" style="font-size: 12px">
<p>
<em>Ce billet est aussi <a href="http://blogs.codes-sources.com/jay/archive/2009/08/09/performance-de-d-marrage-d-un-elementhost-de-wpf.aspx">disponible en francais</a>.</em>&nbsp; 
</p>
<p>
Imagine you&#39;re dealing with a relatively complex WinForms application, and you&#39;ve been so tempted for a while by the charms of WPF that you want to integrate a WPF control somewhere in a form lost in the application. The form in question is opened and closed a lot. 
</p>
<p>
A solution is to develop a WPF control, and integrate it in the WinForms application by&nbsp;using of the&nbsp;<a style="color: #be5007; text-decoration: underline" href="http://msdn.microsoft.com/en-us/library/system.windows.forms.integration.elementhost.aspx">ElementHost</a>, this little piece of &quot;magic&quot; that saves a great deal of time. 
</p>
<p>
Soon enough, you&#39;ll discover that loading time of the WPF control takes a lot of time... On my machine, a Core 2 Duo 3GHz, that takes something like 3 to 4 seconds before displaying properly the form that contains the ElementHost. And as if it was not enough, the display of the form is done by chunks... Not that fancy... Simple intuition, but that seems to be the loading time of WPF that is too long... 
</p>
<p>
The solution to speed things up is rather simple : <strong>Just keep an &quot;empty&quot; ElementHost visible at all times.</strong>&nbsp;Place a control of the ElementHost type sized 1x1 somehere on a form that stays visible during the application execution. 
</p>
<p>
The result, a form that shows up in no time and without any visual glitch. Of course, the loading time of the &quot;initial&quot; form that contains the empty ElementHost will still take some time to load, but after that all other forms that contain WPF controls will show up instantly.&nbsp; 
</p>
<p>
From a more technical point of view, it seems that the initialization of WPF is done when the first ElementHost of the application is initialized, and is released when the last ElementHost of the application is closed. A small analysis using reflector did not show the existence of a method named &quot;InitializeAndKeepWPFInitialized()&quot;, and it probably just a matter of instanciating the proper WPF type to intialize WPF... But the empty ElementHost is more than enough ! 
</p>
</span>

{% include imported_disclaimer.html %}
