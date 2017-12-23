---
layout: post
title: "[WPDev] The hidden cost of IL Jitting"
date: 2011-12-02 22:35:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/12/02/WPDev-The-hidden-cost-of-IL-Jitting", "/post/2011/12/02/wpdev-the-hidden-cost-of-il-jitting"]
author: jay
---
<!-- more -->
<p>We&rsquo;ve gotten used to it. Method jitting is negligible. Or is it really ?</p>
<p>&nbsp;</p>
<h2>IL JITing</h2>
<p>The compilation from IL to the native architecture assembly language (or <a href="http://msdn.microsoft.com/en-us/library/ht8ecch6(v=vs.90).aspx" target="_blank">JITting</a>) has been part of the CLR from the very beginning. That&rsquo;s an operation that was added to make the code execute faster, as interpreting the IL was too slow. By default, it&rsquo;s happening on the fly, when the code path comes to a method that needs to be jitted, and that impacts the code speed when executing the method the first time.</p>
<p>That compilation step is not exactly free. A lot of code analysis and CPU specific optimizations are performed during this step. This is what arguably makes already JITted code run faster than generic compiled code, where the compiler has no knowledge of the target architecture.</p>
<p>This analysis takes a bit of time, but it is taking less and less time to execute, due to CPUs getting faster, or <a href="http://www.asp.net/vnext/overview/getting-started-with-the-next-version-of-aspnet/what's-new-in-aspnet-45-and-visual-web-developer-11-developer-preview#_Toc_perf_4" target="_blank">multi-core JITting</a> features like the one found in .NET 4.5.</p>
<p>We&rsquo;ve come to a point, on desktop and server machines, where the JIT time is negligible, since it&rsquo;s gotten fast enough not to be noticed, or be an issue, in the most common scenarios.</p>
<p>Still, if there were times when JITing would be an issue, like it used to be around .NET 1.0, <a href="http://msdn.microsoft.com/en-us/library/ht8ecch6(v=vs.90).aspx" target="_blank">NGEN</a> would come to the rescue. This tool (available in a standard .NET installation) pre-compiles the assemblies for the current platform, and creates native images stored on the disk. When an assembly is NGENed, they appear in the debugger&rsquo;s &ldquo;module&rdquo; window named as &ldquo;your_assembly.il.dll&rdquo;, along with some other fancy decorations.</p>
<p>But while there are some caveats, like the restrictions with cross assembly method inline being ignored. It always comes down to a balance between start-up speed and code execution speed.</p>
<p>&nbsp;</p>
<h2>JITing on Windows Phone</h2>
<p>On the phone though, CPU is very limited, especially on Generation 1 (Nodo) phones. The platform is too, considering is relative young age. At least on surface.</p>
<p>We&rsquo;ve got used to create quite a bit of code to ease the development, add levels of abstraction for many common concepts, and lately, for asynchrony.</p>
<p>I&rsquo;ll take the example of <a href="http://msdn.microsoft.com/en-us/data/gg577609" target="_blank">Reactive Extensions</a> (Rx) in this article, just to make a point.</p>
<p>If you execute the following code on a Samsung Focus:</p>
<pre class="brush: c-sharp">    
    List&lt;timespan&gt; watch = new List&lt;timespan&gt;();

    var objectObservable = Observable.Empty&lt;object&gt;();

    var w = Stopwatch.StartNew();
    Observable.Timeout&lt;object&gt;(objectObservable, TimeSpan.FromSeconds(1));
    watch.Add(w.Elapsed);

    w = Stopwatch.StartNew();
    Observable.Timeout&lt;object&gt;(objectObservable, TimeSpan.FromSeconds(1));
    watch.Add(w.Elapsed);

    output.Text = string.Join(", ", watch.Select(t =&gt; t.TotalMilliseconds.ToString()));</pre>
<p>You'll consistently get something similar to this :</p>
<pre>    20.60, 1.19</pre>
<p><br />Calling an Rx method like this does almost nothing, it&rsquo;s basicallt just setup. But 20ms is a long time ! Particularly when done on the UI thread, or any other thread for that matter.</p>
<p>These rough measurements tend to show that the Windows Phone platform (as of Mango at least) is not performing any NGEN like pre-jitting, leaving the app the burden of jitting code on the fly.</p>
<p>Still, not everything can be accounted to JITing, there must be type metadata loading, type constructors that are called.</p>
<p>&nbsp;</p>
<h2>Generating code with T4</h2>
<p>So to sort that out a bit more, let&rsquo;s use some <a href="http://msdn.microsoft.com/en-us/library/bb126445.aspx" target="_blank">T4 templates</a> to generate code and isolate the JIT a bit more :</p>
<pre class="brush: c-sharp">&lt;#@ template language="C#" #&gt;
using System;
using System.Collections.Generic;

public class Dummy
{
   public static void Test()
   {
      List&lt;int&gt; list = new List&lt;int&gt;();

      &lt;#for (int i = 0; i &lt; 100; i++) { #&gt;
	  list.Add(&lt;#= i.ToString() #&gt;);
      &lt;#} #&gt;
   }
}</pre>
<p>&nbsp;</p>
<p>For different values of iterations, here's what gets out, when timing the call to the method :</p>
<table style="width: 400px;" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td valign="top" width="133"><strong>Calls</strong></td>
<td valign="top" width="133"><strong>First call</strong></td>
<td valign="top" width="133"><strong>Subsequent calls</strong></td>
</tr>
<tr>
<td valign="top" width="133">100</td>
<td valign="top" width="133">1.6ms</td>
<td valign="top" width="133">&gt; 0.03ms</td>
</tr>
<tr>
<td valign="top" width="133">1000</td>
<td valign="top" width="133">15.7ms</td>
<td valign="top" width="133">&gt; 0.09ms</td>
</tr>
<tr>
<td valign="top" width="133">5000</td>
<td valign="top" width="133">72.8ms</td>
<td valign="top" width="133">&gt; 2 ms</td>
</tr>
<tr>
<td valign="top" width="133">10000</td>
<td valign="top" width="133">148ms</td>
<td valign="top" width="133">&gt; 2ms</td>
</tr>
</tbody>
</table>
<p>&nbsp;</p>
<p>While this type of code is not exactly a good real-life scenario, this shows a bit the cost the IL jitting step. These are <strong>very</strong> simple method calls, no branching instructions, no virtual calls, &hellip; in short, nothing complex.</p>
<p>But with real code, the IL is a bit more intricate, and there&rsquo;s got to be more logic involved in the JIT when generating the native code.</p>
<p>&nbsp;</p>
<h2>Wrapping up</h2>
<p>Unfortunately, there&rsquo;s not much that can be done here, except by reducing the amount of actual lines of IL that are generated. But that can be a though job, particularly when customers are expecting a lot from applications.</p>
<p>One could suggest to push as much code as&nbsp; possible on a background thread, even code that seemingly does nothing particularly expensive. But that cannot always be performed on an other thread, particularly if that code depends on UI elements.</p>
<p>Finally, pre-jitting assemblies when installing the applications could be an interesting optimization for the Windows Phone platform, and I&rsquo;m wondering why this has not made its way to the platform yet&hellip;</p>
{% include imported_disclaimer.html %}
