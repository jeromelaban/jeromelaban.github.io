---
layout: post
title: "DataBinding performance in WinRT and the Bindable attribute"
date: 2012-11-26 20:51:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/11/26/DataBinding-performance-in-WinRT-and-the-Bindable-attribute", "/post/2012/11/26/databinding-performance-in-winrt-and-the-bindable-attribute"]
author: jay
---
<!-- more -->
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;"><em>tl;dr: The Bindable attribute can be placed on standard C# classes in Metro Apps&nbsp;to make them appear in the generated IXamlMetadataProvider class, to create static metadata. This technique allows for a 10% increase in data-binding performance over reflection based binding, but also adds a temporary cost in JITting, until Windows generates native images 24 hours later.</em></p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">Databinding in WPF/WinRT is very easy to use. Just put the name of the field you want to bind, set the <a href="http://msdn.microsoft.com/en-us/library/system.windows.frameworkelement.datacontext.aspx">DataContext</a>, and voila, it is displayed on screen. Yet, this is a tricky feature under the hood. It relies on the presence of an arbitrary string that may exist in the current DataContext to get the data to be displayed.</p>
<p>In WPF and Silverlight, this is fairly easy to do because everything is in managed code. Resolving that data&nbsp;member was performed using a bit of type Reflection, where the string "<span style="font-family: courier new,courier;">{Binding SomeValue}</span>" would result in a sequence of <a href="http://msdn.microsoft.com/en-us/library/system.type.getproperty.aspx">Type.GetProperty</a> to get a <a href="http://msdn.microsoft.com/en-us/library/system.reflection.propertyinfo.aspx">PropertyInfo</a> instance, then call <a href="http://msdn.microsoft.com/en-us/library/hh194385.aspx">GetValue</a> to get the actual value.</p>
<p>But in WinRT, all this is a lot different, mainly because WinRT is purely native and there is no reflection or metadata there.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">[more]</p>
<h2>Reflection for C#/XAML apps using WinRT</h2>
<p>How does that work then? Well, that&rsquo;s where some magic happens during the compilation of a C#/XAML project, a subject that <a href="http://jaylee.org/post/2012/03/07/Xaml-integration-with-WinRT-and-the-IXamlMetadataProvider-interface.aspx">I covered a while back</a>.</p>
<p>The compilation process parses the Xaml files to find nodes that reference actual C# or WinMD defined types, and generates a fairly big class that implements the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.markup.ixamlmetadataprovider.aspx">IXamlMetadataProvider</a> class. This class will provide detailed information about the types such as how to create them, their members, whether they are read-only or not, enum values, if there is a content property, etc&hellip;</p>
<p>This generated class is basically a static reflection engine for WinRT. At runtime, this class is provided to WinRT which in turn, calls it to determine what to do with the nodes it just parsed.</p>
<p>&nbsp;</p>
<h2>The magic with DataBinding POCO classes</h2>
<p>There&rsquo;s actually one interesting scenario, when using databinding POCO classes. Let&rsquo;s say you&nbsp;write this:</p>
<pre class="brush: xml">&lt;TextBlock x:Name=&rdquo;test&rdquo; Text={Binding MyValue}&rdquo; /&gt;</pre>
<p>Create the following class:</p>
<pre class="brush: c-sharp">public class MyClass
{
   public string MyValue { get; set; }
}</pre>
<p>And DataBind it using this code :</p>
<pre class="brush: c-sharp">test.DataContext = new MyClass { MyValue = &ldquo;Hello world !&rdquo; };</pre>
<p>Everything will work as expected.</p>
<p>But if you look at the generated IXamlMetadataProvider class, you&rsquo;ll notice that there no reference to the MyClass type.</p>
<p>I stated previously that WinRT can&rsquo;t do reflection, so how does that work? Simple. .NET Runtime magic !</p>
<p>Digging a bit deeper, by placing a breakpoint behind the C# auto property, we&rsquo;ll find this stack trace:</p>
<pre class="brush: text">--&gt; MyApp.exe!MyApp.MyClass.MyValue.get()
    [Native to Managed Transition]
    mscorlib.dll!System.Runtime.InteropServices.WindowsRuntime.CustomPropertyImpl.InvokeInternal(object target, object[] args, bool getValue)
    mscorlib.dll!System.Runtime.InteropServices.WindowsRuntime.CustomPropertyImpl.GetValue(object target)
    [Native to Managed Transition]
    [Managed to Native Transition]
    MyApp.exe!MyApp.MainPage.Test()
&nbsp;
&nbsp;
&nbsp;</pre>
<p>The runtime provides a way for WinRT to actually be able to perform reflection on arbitrary types that are not known at compile time by the IXamlMetadataProvider implementation, using the internal CustomPropertyImpl class.</p>
<p>This is completely transparent to the developer, and works very efficiently.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;"><span style="font-size: small; font-family: Calibri;">&nbsp;</span></p>
<h2>Adding a POCO to the IXamlMetadataProvider</h2>
<p>Knowing that DataBinding also works in a completely native environment using C++/CX, tells some very interesting information.</p>
<p>To be able to databind a type in a C++/XAML app, it is required to place the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.data.bindableattribute.aspx">Bindable</a> attribute. This will force the C++ compiler to generate code that performs the same static reflection class found in C#/Xaml apps. A similar IXamlMetadataProvider implementing class will be generated with this metadata, allowing the Xaml parser to work properly, the same way it does&nbsp;with C#.</p>
<p>When looking at the documentation for Bindable, here what can be read:</p>
<pre><span style="font-family: courier new,courier;">&ldquo;Specifies&nbsp;that&nbsp;a&nbsp;type&nbsp;defined&nbsp;in&nbsp;C++&nbsp;can&nbsp;be&nbsp;used&nbsp;for&nbsp;binding.&rdquo;</span></pre>
<p>Which implies that it should only be used for C++ created types.</p>
<p>Yet, because this attribute is defined in WinRT, it can be used on C# classes, and surprise, classes marked with it also get added to the generated IXamlMetadataProvider !</p>
<p><strong>This attribute can also be used to mark types as&nbsp;Bindable when provided&nbsp;in another assembly, but that are never referenced in Xaml.</strong></p>
<p>I&rsquo;m a big fan of code generation over reflection, particularly if what&rsquo;s being reflected upon is known at compile time. I firmly believe that it is most of the time a way better use of CPU power to perform this code generation a compile time, over doing it countless times on the client at runtime.</p>
<p>This is why I&rsquo;ll favor generating code, and this "Bindable" finding sound like music to my ears&hellip;</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">&nbsp;</p>
<h2>&nbsp;</h2>
<h2>Profiling the Binding performance using Reflected and Bindable marked types</h2>
<p>But let&rsquo;s be clear. Microsoft did not think this technique would be interesting, because they would have otherwise told to add that attribute to classes. Still, let&rsquo;s see how both binding infrastructure perform.</p>
<p>Before that, there are actually two things to consider.</p>
<p>First, the generated code for the IXamlMetadataProvider is not free. Even if it is mostly non-conditional, it still needs to be executed when the app starts to build the type definitions to provide to the WinRT Xaml engine. If this class grows very big, this may adversely impact the performance of the metadata lookup. A big switch/case on strings construct is translated into a standard <a href="http://msdn.microsoft.com/en-us/library/s4ys34ea.aspx">IDictionary&lt;string, int&gt;</a>, and reading this dictionary is not free.</p>
<p>Second, if there are lots of bindable types, even if the provider is composed of a single big switch/case and lots of small get/set methods, this means that this big method will need to be JITed. This can take a substantial amount of time at the startup of the app. This begs for NGEN though, as <a href="http://jaylee.org/post/2012/11/24/Reducing-apps-startup-time-with-Pre-JITing-and-NGEN-on-a-Surface-RT.aspx">I&rsquo;ve discussed in a previous article</a>, to eliminate this overhead.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;"><span style="font-size: small; font-family: Calibri;">&nbsp;</span></p>
<h3>Testing the startup speed</h3>
<p>Testing the startup speed can be done using code generation using T4. Considering an app that has around 30 databind-able types, all comprised of 15 fields, here&rsquo;s the startup time up to the data observed on the screen of a Surface RT:</p>
<ul>
<li>Reflection, 1.9s with JIT, 1.8s with NGEN</li>
<li>Bindable, 3.1s with JIT, 1.8s with NGEN</li>
</ul>
<p>&nbsp;</p>
<h3>Testing the binding speed</h3>
<p>Testing the binding speed be done using two simple types (one with the Bindable attribute, the other without), with one field, both databound to a TextBlock Text property. The loop is about setting 5000 times the DataContext to a new instance of the type:</p>
<ul>
<li>Reflection: 2.00s with a string, 2.45s with an int</li>
<li>Bindable: 1.78s with a string, 2.20s with an int</li>
</ul>
<p>&nbsp;</p>
<p>This makes for improvement of about 10%, which is interesting, particularly considering the fact that ARM tablets are not that powerful.</p>
<p>Unsurprisingly, since everything is passed out as an object both in the IXamlMetadataProvider&nbsp;and CustomPropertyImpl classes, binding to value types is a bit more expensive because of boxing operations.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">I'll provide the code for these two basic&nbsp;tests, if someone's interested.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">&nbsp;</p>
<h2>Is this good for you ?</h2>
<p>It may or may not be useful to you, depending on how much the non-NGENed startup is important for you, but also how much you rely on DataBinding.</p>
<p>I for one think that an added second to the startup to gain a rough 10% in binding speed seems a fairly reasonable deal, particularly when NGEN passes by afer a while to suppress this overhead.</p>
<p><strong>Remember that an app only has 16.6ms to perform operations on the dispatcher</strong> between screen refreshes to maintain 60 Frames per second and keep a smooth user experience.</p>
<p>On an ARM tablet, added to other performance tweaks, this can overall make a difference.</p>
{% include imported_disclaimer.html %}
