---
layout: post
title: "The (non generic) System.Action delegate"
date: 2007-12-09 22:33:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/12/09/The-(non-generic)-SystemAction-delegate", "/post/2007/12/09/the-(non-generic)-systemaction-delegate"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Tahoma" size="2">There&#39;s been one delegate I wish would have been integrated in .NET 2.0 :</font> <br />
[code:c#;ln=on]<br />
namespace System<br />
{<br />
&nbsp; public delegate void Action();<br />
}<br />
[/code] 
</p>
<p align="justify">
<font size="2"><font face="Tahoma">Well, it&#39;s been added to the .NET framework 3.5. That will avoid me to create here and there an empty delegate that returns nothing and takes nothing in parameter. It&#39;s particularly useful with anonymous methods.</font></font> 
</p>
<p>
<font size="2"><font face="Tahoma">My discovery of that particular type is a bit odd though. </font></font><font size="2"><font face="Tahoma">A big project I&#39;m currently working on is defining this type :</font></font> 
</p>
<p>
[code:c#;ln=on]<br />
namespace T1 { public enum Action { A } } <br />
[/code] 
</p>
<font size="2">
<p>
<font size="2"><font face="Tahoma">&quot;T1&quot; is made up, &quot;Action&quot; is not. And it&#39;s being used like this :</font> </font>
</p>
</font>
<p>
[code:c#;ln=on]<br />
Action&nbsp;a = Action.A;<br />
[/code] 
</p>
<font size="2"><font size="2">
<p align="justify">
<font size="2"><font face="Tahoma">And it compiled just fine using .NET 2.0. During a migration to the .NET Framework 3.5, I came across some compilation problems telling me that the resolution of &quot;Action&quot; was ambiguous. I thought at first that this was because of a change in the resolutions of types in C# 3.0, but after a bit of digging I found out about that non-generic <strong>System.Action</strong> delegate, which is defined in System.Core.dll. By the way, they also added some other <strong>System.Action</strong> overloads&nbsp;with two, three and four generic parameters.</font></font> 
</p>
<p align="justify">
<font size="2"><font face="Tahoma">Reflector tells me that the non generic version is being used by <strong>System.Linq.Expressions.Expression</strong>. I&#39;m guessing that might be used by LINQ in some way... maybe by some generated code.</font></font> 
</p>
</font></font>

{% include imported_disclaimer.html %}
