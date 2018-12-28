---
layout: post
title: "[wp7dev] Error code 0xc00cee65 and duplicate XML namespaces"
date: 2011-07-29 07:32:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2011/07/29/wp7dev-Error-code-0xc00cee65.aspx", "/post/2011/07/29/wp7dev-error-code-0xc00cee65.aspx"]
author: jay
---
<!-- more -->
<div>There are times when you feel almost alone, particularly when you type an error code in google, and nothing comes up.</div>
<div>I got that interesting and&nbsp;particularly verbose&nbsp;exception when doing some XAML refactoring for a Windows Phone&nbsp;7&nbsp;application :</div>
<div><br />
<p><span style="font-family: courier new,courier;">An unhandled exception of type 'System.Exception' occurred in System.Windows.dll</span><br /><span style="font-family: courier new,courier;">Additional information: 0xc00cee65</span></p>
<div>Turns out it was due to this :</div>
</div>
<p>&nbsp;</p>
<pre style="background: white; color: black; font-family: Consolas;"><span style="color: red;">xmlns</span><span style="color: blue;">:</span><span style="color: red;">ucontrols2</span><span style="color: blue;">=</span><span style="color: blue;">"clr-namespace:Company.Views.Controls;assembly=Company.Views.Phone"</span>&nbsp;&nbsp;
<span style="color: red;">xmlns</span><span style="color: blue;">:</span><span style="color: red;">ucontrols</span><span style="color: blue;">=</span><span style="color: blue;">"clr-namespace:Company.Views.Controls;assembly=Company.Views.Phone"</span>
</pre>
<p>&nbsp;</p>
<div>Where two xml namespaces were referencing the same CLR namespace﻿s and assembly. Merging the two namespaces fixed this issue.</div>
<p>&nbsp;</p>
<div>Tough one...</div>
{% include imported_disclaimer.html %}
