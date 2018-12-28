---
layout: post
title: ".NET 2.0 InternalsVisibleTo Attribute and Unsigned Assemblies"
date: 2008-09-02 07:19:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/09/02/NET-2-0-InternalsVisibleTo-Attribute-and-Unsigned-Assemblies.aspx", "/post/2008/09/02/net-2-0-internalsvisibleto-attribute-and-unsigned-assemblies.aspx"]
author: jay
---
<!-- more -->
<p>
<em>Ce post est disponible <a href="http://blogs.codes-sources.com/jay/archive/2008/09/02/L-attribut-InternalsVisibleTo-en-dot-NET-2-0-et-les-Assemblies-Non-Signees.aspx">en francais ici</a>.</em>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">In a recent project, for the purpose of unit testing, I had to use the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute.aspx">InternalsVisibleTo</a> attribute, which extends the scope of the <a href="http://msdn.microsoft.com/en-us/library/7c5ka91b.aspx">internal</a> qualifier. This allows the separation of the unit testing code assembly from the actual code, without publishing the internals to the &quot;public&quot;. This way, you can avoid shipping your unit testing code. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This is an interesting attribute from many points of view, but when using it, you may face this nice error message :</font>
</p>
<p>
<font face="courier new,courier" size="2">error CS1726: Friend assembly reference &#39;Dummy&#39; is invalid. Strong-name signed assemblies must specify a public key in their InternalsVisibleTo declarations.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">Problem was, my current assembly nor the target assemblies were signed. I tried adding a dummy PublicKey or PublicKeyToken as indirectly suggested <a href="http://www.sturmnet.org/blog/archives/2005/05/10/internalsvisibleto-sn/">here</a> and <a href="http://statestreetgang.net/post/2008/04/InternalsVisibleToAttribute-and-Strong-Named-Assemblies-Step-by-Step.aspx">here</a>, but as many people out here, I don&#39;t want to mess with assembly signing at this point of my project.</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">It turns out that the compiler considers your assembly as &quot;signed&quot; if there is either an AssemblyKeyFile or AssemblyKeyName defined on the assembly, even though both of them are empty.&nbsp;</font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">So, to be able to use AssemblyKeyName InternalsVisibleTo with unsigned assemblies, just remove AssemblyKeyFile or AssemblyKeyName attributes if you don&#39;t use them.</font>
</p>

{% include imported_disclaimer.html %}
