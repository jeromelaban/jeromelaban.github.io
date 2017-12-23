---
layout: post
title: "Xaml integration with WinRT and the IXamlMetadataProvider interface"
date: 2012-03-07 21:04:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2012/03/07/Xaml-integration-with-WinRT-and-the-IXamlMetadataProvider-interface", "/post/2012/03/07/xaml-integration-with-winrt-and-the-ixamlmetadataprovider-interface"]
author: jay
---
<!-- more -->
<p><em>TL;DR: This article talks about the internals of the WinRT/Xaml implementation in Windows 8 and how it deals with databinding, its use of the IXamlMetadataProvider interface, tips &amp; tricks around it, and how to extend the resolver to create dynamic databinding scenarios.</em></p>
<p>&nbsp;</p>
<p>Xaml has been around for a while, and it&rsquo;s been a big part of Silverlight and WPF. Both frameworks are mostly managed, and use a CLR feature known as Reflection, or type introspection.</p>
<p>This is a very handy feature used by Silverlight/WPF to enable late binding to data types, where strings can be used to find their actual classes counter-parts, either for <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.data.ivalueconverter.aspx">value converters</a>, UserControls, Data-Binding, etc...</p>
<p>&nbsp;</p>
<h2>The burden of .NET reflection</h2>
<p>It comes with a cost, though. Reflection is a very expensive process, and up until very recently in Silverlight, there was no way to avoid the use of Reflection. The recent addition of the <a title="ICustomTypeProvider" href="http://msdn.microsoft.com/en-sg/library/system.reflection.icustomtypeprovider.aspx">ICustomTypeProvider</a> interface allows for late binding without the use of reflection, which is a big step what I think is the right direction. Having this kind of interface allows for custom types that define pre-computed lists of fields and properties, without having the runtime to load every metadata available for an object, and perform expensive type safety checks.</p>
<p>This burden of the reflection is particularly visible on Windows Phone, where it is suggested to limit the use of DataBinding, which is performed on the UI thread. The Silverlight runtime needs to walk the types metadata to find observable properties so that it can properly perfrom one or two-way bindings, and this is very expensive.</p>
<p>There are ways to work around this without having&nbsp;<a title="ICustomTypeProvider" href="http://msdn.microsoft.com/en-sg/library/system.reflection.icustomtypeprovider.aspx">ICustomTypeProvider</a>, mainly by generating code that does everything the Xaml parser and DataBinding engines do, but it&rsquo;s mainly experimental, yet it gives great results.</p>
<p>&nbsp;</p>
<h2>WinRT, native code and the lack of Reflection</h2>
<p>In Windows 8, WinRT is pure native code, and now integrates what used to be the WPF/Xaml engine. This new engine can be seen at the cross roads of Silverlight, WPF and Silverlight for Windows Phone. This new iteration takes a bit of every framework, with some tweaks.</p>
<p>These tweaks are mainly related to the fact that WinRT is a native COM based API, that can be used indifferently from C# or C++.</p>
<p>For instance, xml namespaces have changed form and cannot reference assemblies anymore. Declarations that used to look like this :</p>
<p><span style="font-family: 'courier new', courier;">&nbsp; &nbsp; &nbsp;xmlns:common="</span><span style="font-family: 'courier new', courier;">clr-namespace</span><span style="font-family: 'courier new', courier;">:Application1.Common"</span></p>
<p>Now look like this :</p>
<p><span style="font-family: 'courier new', courier;">&nbsp; &nbsp; &nbsp;xmlns:common="</span><span style="font-family: 'courier new', courier;">using</span><span style="font-family: 'courier new', courier;">:Application1.Common"</span></p>
<p>Where the using only defines the namespace to be used to find the types specified in the inner xaml.</p>
<p>Additionally, WinRT does not know anything about .NET and the CLR, meaning it cannot do reflection.&nbsp;This means that the Xaml implentation in WinRT, to be compatible with the way we all know Xaml, needs to be able to do some kind of reflection.</p>
<p>&nbsp;</p>
<h2>Meet the IXamlMetadataProvider interface</h2>
<p>To be able to do some kind reflection, the new Metro Style Applications profile generates code based on the types that are used in the Xaml files of the project. It takes the form of a hidden file, named XamlTypeInfo.g.cs.</p>
<p>That file can be found in the &ldquo;obj&rdquo; folder under any Metro Style project that contains a Xaml file. To find it, just click on the &ldquo;Show all files&rdquo; button at the top of the Solution Explorer file. You may need to compile the project for it to be generated.</p>
<p>In the entry assembly, the file contains a partial class that extends the App class to make it implement the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.markup.ixamlmetadataprovider">IXamlMetadataProvider</a> interface. WinRT uses this interface to query for the details of types it found while parsing Xaml files.</p>
<p>This type acts as map for every type used in all Xaml files a project, so that WinRT can get a definition it can understand, in the form of <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.markup.ixamltype">IXamlType</a> and&nbsp;<a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.markup.ixamlmember">IXamlMember</a> instances. This takes the form of a big switch/case construct, that contains string representation of fully qualified types. See this example :</p>
<pre class="brush: c-sharp">private IXamlType CreateXamlType(string typeName) 
{ 
  XamlSystemBaseType xamlType = null; 
  XamlUserType userType;

  switch (typeName) 
  { 
    case "Windows.UI.Xaml.Controls.UserControl": 
      xamlType = new XamlSystemBaseType(typeName, typeof(Windows.UI.Xaml.Controls.UserControl)); 
      break;

    case "Application1.Common.RichTextColumns": 
      userType = new XamlUserType(this, typeName, typeof(Application1.Common.RichTextColumns), GetXamlTypeByName("Windows.UI.Xaml.Controls.Panel")); 
      userType.Activator = Activate_3_RichTextColumns; 
      userType.SetContentPropertyName("Application1.Common.RichTextColumns.RichTextContent"); 
      userType.AddMemberName("RichTextContent", "Application1.Common.RichTextColumns.RichTextContent"); 
      userType.AddMemberName("ColumnTemplate", "Application1.Common.RichTextColumns.ColumnTemplate"); 
      xamlType = userType; 
      break; 

  } 
  return xamlType; 
} 
</pre>
<p>It also creates hardcoded methods that can explicitly get or set the value of every properties a DependencyObject, like this :</p>
<pre class="brush: c-sharp">case "Application1.Common.RichTextColumns.RichTextContent": 
    userType = (XamlUserType)GetXamlTypeByName("Application1.Common.RichTextColumns"); 
    xamlMember = new XamlMember(this, "RichTextContent", "Windows.UI.Xaml.Controls.RichTextBlock"); 
    xamlMember.SetIsDependencyProperty(); 
    xamlMember.Getter = get_1_RichTextColumns_RichTextContent; 
    xamlMember.Setter = set_1_RichTextColumns_RichTextContent; 
    break;
</pre>
<p>Note that if you want to step into this code without the debugger ignoring you, you need to disable the &ldquo;Just my code&rdquo; feature in the debugger options.</p>
<p>Also, in case you wonder, the Code Generator scans for all referenced assemblies for implementations of the&nbsp;<a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.markup.ixamlmetadataprovider">IXamlMetadataProvider</a>&nbsp;interface, and will generate code that will query these providers to find Xaml type definitions.</p>
<p>&nbsp;</p>
<h2>Code Generation is good for you</h2>
<p>Now, this code generation approach is very interesting for some reasons.</p>
<p>The first and foremost is performance, because the runtime does not need to use reflection to determine what can be computed at runtime. This is a enormous performance gain, and this will be beneficial for the perceived performance as the runtime will not waste precious CPU cycles to compute data that can be determined at compile time.</p>
<p>More generally, in my projects, I've been using this approach of generating as much code as possible, to avoid using reflection and waste time and battery on something that can be only done once and for all.</p>
<p>The second reason is extensibility, as this IXamlMetadataProvider can be extended to add user-provided types that are not based on DependencyObject. This is an other good impact on performance.</p>
<p>&nbsp;</p>
<h2>Adding custom IXamlMetadataProvider</h2>
<p>It is possible to extend the lookup behavior for standard types that are not dependency objects. This opens the same range of scenarios that <a title="ICustomTypeProvider" href="http://msdn.microsoft.com/en-sg/library/system.reflection.icustomtypeprovider.aspx">ICustomTypeProvider</a> provides.</p>
<p>All that is needed is to implement the IXamlMetadataProvider interface somewhere in an assembly, and the code generator used for XamlTypeInfo.g.cs will pick those up and add them in the Xaml type resolution chain. Note that for some unknown reason, it does not work in the main assembly but only for referenced assemblies.</p>
<p>Every time the databinding engine will need to get the value behind a databinding expression, it will call the IXamlMetadataProvider.GetXamlType method to get the definition of that type, then get the databound property value.</p>
<p>A very good feature, if you ask me.</p>
<p>&nbsp;</p>
<h2>The case of hidden DependencyObject</h2>
<p>By hidden dependency properties, I&rsquo;m talking about DependencyObject types that are not directly referenced in Xaml files. This can be pretty useful for complex controls that generate convention based databinding, such as the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/xaml/windows.ui.xaml.controls.semanticzoom.aspx">SemanticZoom</a> control, that provides a implicit &ldquo;Key&rdquo; property to perform the Zoomed out view.</p>
<p>Since this XamlTypeInfo.g.cs code is generated from all known Xaml files, this means that these hidden DependencyObject types that do not have code generated for them. This forces the CLR to intercept these failed requests and fallback on actual .NET reflection based property searching for databinding, which is not good for performance.</p>
<p>This fallback behavior was not implemented in the Developer Preview, and the binding would just fail with a NullReferenceException without any specific reason given to the developer.</p>
<p>&nbsp;</p>
<h2>The case of Xaml files located in another assembly</h2>
<p>If your architecting a bit your solution, you&rsquo;re probably using MVVM or a similar pattern, and you&rsquo;re probably putting your views in another assembly.</p>
<p>If you do that, this means that there will not be any xaml file in your main assembly (aside from the App.xaml file), leading to an empty XamlTypeInfo.g.cs file. This will make any type resolution requested by WinRT fail, and your application will mostly likely not run.</p>
<p>In this case, all you need to do is create a dummy Xaml file that will force the generation of the XamlTypeInfo.g.cs, and basically make your layer separation work.</p>
<p>&nbsp;</p>
<p>Until next time, happy WinRT'ing !</p>
{% include imported_disclaimer.html %}
