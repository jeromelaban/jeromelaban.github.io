---
layout: post
title: "[VS2010] How to disable the Power Tools Ctrl+Click Go to Definition"
date: 2010-06-13 17:17:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/06/13/VS2010-How-to-disable-the-Power-Tools-Ctrl2bClick-Go-to-Definition", "/post/2010/06/13/vs2010-how-to-disable-the-power-tools-ctrl2bclick-go-to-definition"]
author: admin
---
<!-- more -->
<p>Last week, Microsoft released the <a href="http://weblogs.asp.net/scottgu/archive/2010/06/09/visual-studio-2010-productivity-power-tool-extensions.aspx" target="_blank">Visual Studio 2010 Productivity Power Tool Extensions</a>, which includes a lot of features that probably should have made it to the VS2010 RTM, but somehow did not.</p>
<p>&nbsp;</p>
<p>A must install, really. Just for the fixed Add Reference dialog that <strong>includes a search filter</strong>. A big time saver.</p>
<p>&nbsp;</p>
<p>But there&rsquo;s also an other feature, the Ctrl+Click Go to Definition that allows to go to the definition with a single left click (the F12 key in the default keyboard bindings).</p>
<p>&nbsp;</p>
<p>If you&rsquo;re like me, you may be lazy enough to let the text editor select complete words and you&rsquo;re probably using Ctrl + left lick to select words so you don&rsquo;t have to aim too much with the mouse pointer. That particular Power Tools feature conflicts directly with it and you end up constantly going to the definition of types when you want to select text... And that&rsquo;s pretty annoying.</p>
<p>&nbsp;</p>
<p>There does not seem to be a way to disable that feature from the IDE, so you may well disable it the hard way :</p>
<ul>
<li>Go to <span style="font-family: Courier New;">C:\Users\USER_NAME\AppData\Local\Microsoft\VisualStudio\10.0\Extensions\Microsoft\Visual Studio 2010 Pro Power Tools\10.0.10602.2200</span></li>
<li>Remove or rename the <span style="font-family: Courier New;">GoToDefProPack.dll</span> file.</li>
<li>Enjoy your complete word selection again ! </li>
</ul>
<p>Have fun :)</p>
{% include imported_disclaimer.html %}
