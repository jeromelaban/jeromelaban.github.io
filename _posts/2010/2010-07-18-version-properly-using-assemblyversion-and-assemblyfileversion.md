---
layout: post
title: "Version Properly using AssemblyVersion and AssemblyFileVersion"
date: 2010-07-18 15:25:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/07/18/Version-Properly-using-AssemblyVersion-and-AssemblyFileVersion", "/post/2010/07/18/version-properly-using-assemblyversion-and-assemblyfileversion"]
author: jay
---
<!-- more -->
<p><a href="http://blogs.codes-sources.com/jay/archive/2010/07/10/comment-versionner-efficacement-avec-les-attributs-assemblyversion-et-assemblyfileversion.aspx"><em>Cet article est disponible en francais.</em></a></p>
<p>We talk quite easily about new technologies and things we just learned about, because that's the way geeks work. But for newcomers, this is not always easy. This is a large recurring debate, but I find that it is good to step back from time to time and talk about good practices for these newcomers.</p>
<p>&nbsp;</p>
<h2><span style="font-size: 12px; font-weight: bold;">The AssemblyVersion and AssemblyFileVersion Attributes</span></h2>
<p>When we want to give a version to a .NET assembly, we can choose one of two common ways :</p>
<ul>
<li><a href="http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute%28VS.71%29.aspx" target="_blank">AssemblyVersion</a>, which is stored in the <a href="http://msdn.microsoft.com/en-us/library/1w45z383.aspx" target="_blank">assembly manifest</a>,</li>
<li><a href="http://social.msdn.microsoft.com/Search/en-US?query=AssemblyFileVersion&amp;ac=8" target="_blank">AssemblyFileVersion</a>, which is stored the <a href="http://msdn.microsoft.com/en-us/library/ms680547%28VS.85%29.aspx">PE image</a>&nbsp;resources.</li>
</ul>
<p>&nbsp;</p>
<p>Most of the time, and by default in Visual Studio 2008 templates, we can find the AssemblyInfo.cs file in the Properties section of a project. This file will generally only contain an AssemblyVersion attribute, which will force the value of the AssemblyFileVersion value to the AssemblyVersion's value. The AssemblyFileVersion attribute is now added by default in the Visual Studio 2010 C# project templates, and this is a good thing.</p>
<p>It is possible to see the value of the AssemblyFileVersion attribute in file properties window the Windows file explorer, or by adding the "File Version" column, still in the Windows Explorer.</p>
<p>We can also find a automatic numbering provided by the C# compiler, by the use of :</p>
<pre style="font-family: consolas;">[<span style="color: blue;">assembly</span>:&nbsp;<span style="color: #2b91af;">AssemblyVersion</span>(<span style="color: #a31515;">"1.0.0.*"</span>)]<br /></pre>
<p>&nbsp;</p>
<p>Each new compilation will create a new version.</p>
<p>This feature is enough at first, but when you start having somehow complex projects, you may need to introduce&nbsp;<a href="http://msdn.microsoft.com/en-us/library/ms364045%28VS.80%29.aspx">continuous integration</a>&nbsp;that will provide&nbsp;<a href="http://www.google.com/url?sa=t&amp;source=web&amp;cd=1&amp;ved=0CB0QFjAA&amp;url=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FNeutral_build&amp;ei=MfQ4TMXxLaDtnQfF17i8Aw&amp;usg=AFQjCNFuP77CKx5GrVJfFnUTSmXK4D5epg&amp;sig2=Pcimx6WDB3S_pyC-Wv0E3g">nightly builds</a>. You will want to give a version to the assemblies in such a way it is easy to find which revision has been used in the source control system to compile those assemblies.</p>
<p>You can then modify the Team Build scripts to use tasks such as the AssemblyInfo task of&nbsp;<a href="http://msbuildtasks.tigris.org/" target="_blank">MSBuild Tasks</a>, and generate a new AssemblyInfo.cs file that will contain the proper version.</p>
<p>&nbsp;</p>
<h2>Publishing a new version of an Assembly<br /></h2>
<p>To come back to the subject of versionning an assembly properly, we generally want to know quickly, when a project build has been published, which version has been installed on the client's systems. Most of the time, we want to know which version is used because there is an issue, and that we will need to provide an updated assembly that will contain a fix. Particularly when the software cannot be reinstalled completely on those systems.</p>
<h3><span style="font-weight: bold;">A somehow real world example</span></h3>
<p>Lets consider that we have a solution with two assemblies signed with <a href="http://msdn.microsoft.com/en-us/library/wd40t7ad.aspx">a  strong name</a> Assembly1 and Assembly2, with Assembly1 that uses types available in Assembly2, and that finally are versioned with an AssemblyVersion set to 1.0.0.458. These assemblies are part of an official build published on the client's systems.</p>
<p>If we want to provide a fix in Assembly2, we will create a branch in the source control from the revision 1.0.0.458, and make the fix in that branch which will give revision 460, so the version 1.0.0.460.</p>
<p>If we let the Build System compile that revision, we will get assemblies that will be marked as 1.0.0.460. If we only take Assembly2, and we place it on the client's systems, the CLR will refuse to load this new version if the assembly, because Assembly1 requires to have Assembly to of the version 1.0.0.458. We can use the <a href="http://msdn.microsoft.com/en-us/library/eftw1fys.aspx">bindingRedirect</a> parameter in the configuration file to get around that, but this is not always convenient, particularly when we update a lot of assemblies.</p>
<p>We can also compile this new version with the AssemblyVersion of 1.0.0.460 set to 1.0.0.458 for Assembly2, but this willl have the disadvantage of lying about the actual version of the file, and that will make diagnostics more complex in case of an other issue that could happen later.</p>
<h3>Adding AssemblyFileVersion</h3>
<p>To avoid having those issues with the resolution of assembly dependencies, it is possible to keep the AssemblyVersion constant, but use the AssemblyFileVersion to provide the actual version of the assembly.</p>
<p>The version specified in the AssemblyFileVersion is not used by the .NET Runtime, but is displayed in the file properties in the Windows Explorer.</p>
<p>We will then have the AssemlyVersion set to the original published version of the application, and set the AssemblyFileVersion to the same version, and later change only the AssemblyFileVersion when we published fixes of these assemblies.</p>
<p>Microsoft uses this technique to version the .NET runtime and BCL assemblies, and we take a look at System.dll for .NET 2.0, we can see that the AssemblyVersion is set to 2.0.0.0, and that the AssemblyFileVersion is set, for instance, to 2.0.50727.4927.</p>
<p>&nbsp;</p>
<h2>Other examples of versionning issues<br /></h2>
<p>We can find other cases of loading issues linked to the mismatch of the version for a loaded assembly that is different from the expected assembly version.</p>
<h3>Custom Behaviors in WCF</h3>
<p>WCF gives the developer a way to provide custom behaviors to alter the default behaviors for out-of-the-box bindings, and it is necessary to provide&nbsp;<a href="http://connect.microsoft.com/wcf/feedback/details/216431/wcf-fails-to-find-custom-behaviorextensionelement-if-type-attribute-doesnt-match-exactly">the fully qualified name, without errors</a>. This is a pretty annoying but in WCF 4.x because it is somewhat complex to debug, and it is a very good case of use for the <a href="http://jaylee.org/post/2010/07/05/VS2010-On-the-Impacts-of-Debugging-with-Just-My-Code.aspx">deactivation of "Just My Code"</a> to find out why the assembly is not being loaded.</p>
<p>A good new though, this pretty old bug has been fixed in WCF 4.0 !</p>
<h3>Dynamic Proxy Generators</h3>
<p>Some dynamic proxy generators like <a href="http://www.castleproject.org/dynamicproxy/index.html">Castle  Dynamic Proxy 2</a> or <a href="http://www.springframework.net">Spring.NET</a> use fully qualified types to generate the code for the proxy's, and loading issues can occur if the assembly referenced by the proxy is not exactly what is being loaded, with or without a Strong Name. These frameworks are heavily used with <a href="http://en.wikipedia.org/wiki/Aspect-oriented_programming">AOP</a>, or by frameworks <a href="http://en.wikipedia.org/wiki/NHibernate">nHibernate</a>, <a href="http://www.castleproject.org/activerecord/" target="_blank">ActiveRecords</a> or <a href="http://ibatis.apache.org/dotnet.cgi">iBatis</a>.</p>
<p>To be a bit more precise, the use of the <span style="font-family:  'Courier New';">ProxyGenerator.CreateInterfaceProxyWithTarget</span> method generates a proxy that targets the assembly that is referenced during the generation of the code for the proxied interface.<br /><br />To give an example, let's take an interface I1 in an assembly A1(1.0.0.0), which has a method that uses a type T1 in an assembly A2(1.0.0.0). If we change the assembly A2 and that its version becomes A2(2.0.0.0), the proxy will not be properly generated because the reference T1/A2(1.0.0.0) will be used because compiled in A1(1.0.0.0), regardless if we loaded A2(2.0.0.0)<br /><br />The best practice of not changing the AssemblyVersion avoid loading issues of this kind. These issues are not blocking, but this more work to do to get around it.</p>
<h2>And You ?<br /></h2>
<p>This is only a example of "Best Practice", which seems to have worked properly so far.</p>
<p>And you ? What do you do ? Which practices do you use to version your assemblies ?</p>
{% include imported_disclaimer.html %}
