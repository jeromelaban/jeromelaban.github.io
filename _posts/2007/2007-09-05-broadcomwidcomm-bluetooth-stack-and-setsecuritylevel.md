---
layout: post
title: "Broadcom/Widcomm Bluetooth Stack and SetSecurityLevel"
date: 2007-09-05 15:37:00 -0400
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2007/09/05/BroadcomWidcomm-Bluetooth-Stack-and-SetSecurityLevel", "/post/2007/09/05/broadcomwidcomm-bluetooth-stack-and-setsecuritylevel"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Verdana" size="2">C++, my almost favorite language. This language had good intentions, really. But this is such a pain --&nbsp;</font><a href="http://blogs.msdn.com/ericlippert/archive/2007/08/14/c-and-the-pit-of-despair.aspx"><font face="Verdana" size="2">a&nbsp; pit of&nbsp;despair</font></a><font face="Verdana" size="2"> --&nbsp;to have to deal with library compatibility and compilation&nbsp;parameters matching&nbsp;issues, where it should be handled by the system.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Anyway, if you&#39;re trying to use the Broadcom Bluetooth SDK for Windows CE with&nbsp;VS2005 or VS2008, you might encounter this pretty message :</font>
</p>
<p>
<font face="Courier New" size="2">error LNK2019: unresolved external symbol &quot;__declspec(dllimport) public: int __cdecl CRfCommIf::SetSecurityLevel(wchar_t *,unsigned char,int)&quot; (</font><font face="Courier New" size="2">__imp_?SetSecurityLevel@CRfCommIf@@QAAHPA_WEH@Z</font><font face="Courier New" size="2">)</font>
</p>
<p align="justify">
<font face="Verdana" size="2">The symbol actually exists in the broadcom provided library (BtSdkCE30.lib or BtSdkCE30.lib)&nbsp; but is defined like this :</font>
</p>
<p>
<font face="Courier New" size="2">SetSecurityLevel@CRfCommIf@@QAAHPAGEH@Z</font>
</p>
<p>
<font face="Verdana" size="2">which, undecorated, means :</font>
</p>
<p>
<font face="Courier New" size="2">int CRfCommIf::SetSecurityLevel(unsigned short *,unsigned char,int)</font>
</p>
<p align="justify">
<font face="Verdana" size="2">See the difference ? The&nbsp;wchar_t type the compiler is using should&nbsp;be&nbsp;an unsigned short. Basically, both types are binary equivalents but the compiler is treating them as different, hence the different signature.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">To fix this, just set the option &quot;Treat wchar_t as Built-in Type&quot; to No.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">This way, the wchar_t will go back to being a short, and then match the method built in the static library.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">I&#39;m going back to my&nbsp;C# code, now.</font>
</p>

{% include imported_disclaimer.html %}
