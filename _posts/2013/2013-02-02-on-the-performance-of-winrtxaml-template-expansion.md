---
layout: post
title: "On the Performance of WinRT/Xaml Template Expansion"
date: 2013-02-02 15:11:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2013/02/02/On-the-Performance-of-WinRTXaml-Template-Expansion.aspx", "/post/2013/02/02/on-the-performance-of-winrtxaml-template-expansion.aspx"]
author: jay
---
<!-- more -->
<p><em>TL;DR: Expanding data-bound item templates in Xaml/WinRT in Windows 8&nbsp;is about a hundred times&nbsp;slower than with Xaml/WPF. This article details how this was measured and a possible explanation.</em></p>
<p>&nbsp;</p>
<p>In Windows 8, Microsoft has a introduced a whole new Xaml stack, codenamed Jupiter, completely re-written to be native only.</p>
<p>This allows the creation of Xaml controls using C++ as well as C#.</p>
<p>I will not discuss the philosophical&nbsp;choice of ditching managed WPF in favor of a native rewrite, but make a simple comparison of the performance between the two.</p>
<h2>&nbsp;</h2>
<h2>Template Expansion Performance</h2>
<p>I worked on a project that had performance issues for a UI-Virtualized control, where the initial binding of data as well as the realization of item templates, was having a significant impact on the fluidity of the scrolling of a GridView control.</p>
<p>To isolate this, I created a simple UI: [more]&nbsp;</p>
<pre class="brush: xml">&lt;GridView x:Name="testGrid"&gt;
    &lt;GridView.ItemsPanel&gt;
        &lt;ItemsPanelTemplate&gt;
            &lt;VariableSizedWrapGrid /&gt;
        &lt;/ItemsPanelTemplate&gt;
    &lt;/GridView.ItemsPanel&gt;
    &lt;GridView.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;TextBlock Text="{Binding}" Foreground="Red" /&gt;
        &lt;/DataTemplate&gt;
    &lt;/GridView.ItemTemplate&gt;
&lt;/GridView&gt;</pre>
<p>Initialized with the following code in a button click :</p>
<blockquote>
<pre class="brush: c-sharp">testGrid.ItemsSource = Enumerable.Range(1, 100);</pre>
</blockquote>
<p>On a Surface RT tablet, <strong>the time during which the UI was not responding was about 2.1 seconds</strong>.</p>
<p>Looking a the profiler revealed that a lot of time was spent in this function :</p>
<pre>?MeasureOverride@FrameworkElementGenerated@DirectUI@@UAAJUSize@Foundation@Windows@@PAU345@@Z</pre>
<p>Measuring is expected to take some time to execute, but it was very odd that it was that much.</p>
<p>&nbsp;</p>
<h2>Measuring between WPF and WinRT</h2>
<p>As an experiment, I tried to create approximately same layout in WPF, and test the rendering performance on my PC.</p>
<p>To measure more precisely, I created an override of both the GridView and the ListBox controls so that the call to Measure could be profiled, that way :</p>
<pre class="brush: c-sharp">protected override Size MeasureOverride(Size availableSize)
{
   var w = Stopwatch.StartNew();
   try 
   {             
      return base.MeasureOverride(availableSize); 
   }
   finally        
   {                 
      MeasureTime = w.Elapsed;    
      Debug.WriteLine("Measure " + w.Elapsed);         
   }       
}</pre>
<p>It turns out that the initial measure duration for 500 element is quite different: <strong>WPF measures for 15ms, WinRT does in 1500ms !</strong> And it appears visually, because the UI is not responding a long time in WinRT, where it does not in WPF. Subsequent measures are roughly the same for both platforms.</p>
<p>Interestingly, while the VS2012 profiler does not show stack traces below the manaded/native boundary, the concurrency visualizer does. A stacktrace element that comes back <strong>very</strong> often is the following :</p>
<p>&nbsp;</p>
<pre>windows.ui.xaml.dll!XamlWriter::WriteNode+0x53c</pre>
<pre>&nbsp;</pre>
<p>Which seems to point in the direction of the lack of template parsing result caching&hellip;</p>
<p>&nbsp;</p>
<h2>What then ?</h2>
<p>That&rsquo;s the problem. Not that much, apart from try to&nbsp;not realize too many collection&nbsp;items in WinRT. (and guessing that for the next update of WinRT that may address this)</p>
<p>Another solution can be to Databind an ObservableCollection and add items slowly to yield the template creation on the UI Thread... But this requires some plumbing code to avoid race conditions and sorting issues.</p>
<p>This makes UI virtualization very important with Virtualizing Panels, to reuse as many control instances as possible. This also implies that your visual design should not display too many items at once on the screen, with small logical dimensions.</p>
{% include imported_disclaimer.html %}
