---
layout: post
title: "Using Precompiled Headers"
date: 2004-03-01 11:50:00 -0500
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2004/03/01/Using-Precompiled-Headers.aspx", "/post/2004/03/01/using-precompiled-headers.aspx"]
author: jerome
---
<!-- more -->
<div id="nstext">
<table border="0" width="100%">
	<tbody>
		<tr>
			<td align="left"><em><font face="Tahoma" size="2">(How to speed up the C/C++ compilation step with Visual Studio .NET 2003)</font></em></td>
		</tr>
	</tbody>
</table>
<h4><font face="Tahoma" size="2">Visual C++ Pre compilation feature</font></h4>
<p>
<font face="Tahoma" size="2">Microsoft C/C++ compiler features for a long time now something called Header Precompilation, also known as PCH. You may have already encountered it or used it without knowing it.</font>
</p>
<p>
<font face="Tahoma" size="2">The concept is quite simple : Why having to parse header files for each C/C++ file to compile that includes them ? For a given compilation, the header files used will not likely change and even for any subsequent compilations. Standard include files like stdio.h, stdlib.h and many system header files do not need to be parsed at each inclusion.</font>
</p>
<p>
<font face="Tahoma" size="2">The C/C++ compiler uses a special set of files (default is stdafx.cpp and stdafx.h) where it can find all the files to precompile, and then reuse these pre-compiled headers in an efficient way in any future compilation. This avoids to recompile all these header files, especially the ones from the STL that can be really huge.</font>
</p>
<br />
<h4><font face="Tahoma" size="2">1. Using precompiled headers, the default way</font></h4>
<p>
<font face="Tahoma" size="2">At first, the C/C++ compiler searches for a file named stdafx.cpp and tries to compile it. This file only includes the stdafx.h file. This compilation generates a file called $(ProjectName).pch that will be used by other files being compiled&nbsp;to find pre-compiled symbols quickly.</font>
</p>
<p>
<font face="Tahoma" size="2">Any other C/C++ file must then have a line like this one :</font>
</p>
<pre class="code">
<font size="2">#include &quot;stdafx.h&quot;</font>
</pre>
<p>
<font face="Tahoma" size="2">Be aware that this line must be the <strong>very first line</strong> of your C/C++&nbsp;file. Anything that you will place before will be ignored, not even parsed.</font>
</p>
<h4><font face="Tahoma" size="2">2. Using precompiled headers, from scratch</font></h4>
<p>
<font face="Tahoma" size="2">PCH feature can also be activated from a empty project, by adding files one by one. Here is the way to do this.</font>
</p>
<p>
<font face="Tahoma" size="2">First, create an empty project and add 3 new&nbsp;files : <em>main.cpp</em>, <em>stdafx.h</em> and <em>stdafx.cpp</em>.</font>
</p>
<li>
<p>
<font face="Tahoma" size="2">File <em>stdafx.h</em> :</font>
</p>
<pre class="code">
<font size="2">#ifndef __STDAFX_H#define __STDAFX_H#include #include #include #include #endif // __STDAFX_H</font>
</pre>
</li>
<li>
<p>
<font face="Tahoma" size="2">File <em>stdafx.cpp</em>, quite simple :</font>
</p>
<pre class="code">
<font size="2">#include &quot;stdafx.h&quot;</font>
</pre>
</li>
<li>
<p>
<font face="Tahoma" size="2">File <em>main.cpp</em>, also quite simple :</font>
</p>
<pre class="code">
<font size="2">#include &quot;stdafx.h&quot;int main(){ std::string myString(&quot;Hello, World !&quot;); std::cout &lt;&lt; myString.c_str() &lt;&lt; std::endl; return 0;}</font>
</pre>
</li>
<p>
<font face="Tahoma" size="2">Once these files are added in the solution explorer into your C++ project, follow these steps :</font>
</p>
<li>
<p>
<font face="Tahoma" size="2">Select the<em> stdafx.cpp</em> file, right click and select Properties :</font>
</p>
<font face="Tahoma" size="2"><img src="/blogs//images/msdn_labtech_epitech_net/jaylee/18/o_create_pch.jpg" alt="" width="28" height="30" /> </font>
<p>
<font face="Tahoma" size="2">The two fields &quot;<em>Create/Use PCH Through File</em>&quot; and &quot;<em>Precompiled Header File</em>&quot; will be filled automatically.</font>
</p>
</li>
<li>
<p>
<font face="Tahoma" size="2">Then select the <strong>project item </strong>in the solution explorer right click and select Properties :</font>
</p>
<font face="Tahoma" size="2"><img src="/blogs//images/msdn_labtech_epitech_net/jaylee/18/o_proj_pch.jpg" alt="" width="28" height="30" /> </font>
<p>
<font face="Tahoma" size="2">The two fields &quot;<em>Create/Use PCH Through File</em>&quot; and &quot;<em>Precompiled Header File</em>&quot; will be filled automatically.</font>
</p>
<p>
<font face="Tahoma" size="2">Note that changing the C/C++ properties for the project propagates them to C/C++ files that have not been customized. Here it is <em>main.cpp</em>, but not <em>stdafx.cpp</em> because we&#39;ve customized settings in the previous step.</font>
</p>
</li>
<p>
<font face="Tahoma" size="2">After setting all this, the first file to be compiled is <em>stdafx.cpp</em> and then the other files in the project.</font>
</p>
<p>
<font face="Tahoma" size="2">You will see that big projects compile much faster when PCH features is enabled. Also note that you can use multiple precompiled header files in one project, although it is not recommended. If you feel like you need to make multiple PCH files, it is time for you to make a static or a dynamic library.</font><font color="#808080"><br />
</font>
</p>
</div>

{% include imported_disclaimer.html %}
