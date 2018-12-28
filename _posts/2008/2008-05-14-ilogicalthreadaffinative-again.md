---
layout: post
title: "ILogicalThreadAffinative, again."
date: 2008-05-14 19:12:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/05/14/ILogicalThreadAffinative-again.aspx", "/post/2008/05/14/ilogicalthreadaffinative-again.aspx"]
author: jay
---
<!-- more -->
<p>
<font face="trebuchet ms,geneva" size="2"><em>Ce post est &eacute;galement </em><a href="http://blogs.codes-sources.com/jay/archive/2008/05/15/ilogicalthreadaffinative-suite.aspx"><em>disponible en francais</em></a><em>.</em>&nbsp;</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">In a <a href="/post/2008/02/NET-Threads-CallContext-and-ILogicalThreadAffinative.aspx">previous post</a>, I talked about a feature of the .NET framework that allows to automatically pass information from a thread to any thread it spawns. </font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">It turns out that not only the Call Context flows to the thread pool, but&nbsp;it also flows to any System.Threading.Timer and to IO completion related threads, such as in Sockets.</font> 
</p>
<p>
<font face="trebuchet ms,geneva" size="2">I was taken off-guard by the fact that in early stages of the server-side&nbsp;remoting pipeline, there were already information in the&nbsp;CallContext before the incoming message was deserialized. But the content was not exactly what I was expecting; it was the data that was in the CallContext when </font><a href="http://msdn.microsoft.com/en-us/library/system.runtime.remoting.remotingconfiguration.configure.aspx" title="RemotingConfiguration.Configure"><font face="courier new,courier" size="2">RemotingConfiguration.Configure()</font></a><font size="2"><font face="trebuchet ms,geneva">&nbsp;was invoked</font><font face="trebuchet ms,geneva"><font face="courier new,courier">.</font> </font></font>
</p>
<p>
<font face="trebuchet ms,geneva" size="2">This makes sense, all the sockets used by TcpChannel or HttpChannel are created when calling this method, and asynchronous operations are started at this time. So, to avoid having the data to flow to the other side, there are these two methods on the <a href="http://msdn.microsoft.com/en-us/library/system.threading.executioncontext.aspx" target="_blank" title="ExecutionContext">ExecutionContext</a> class to <a href="http://msdn.microsoft.com/en-us/library/system.threading.executioncontext.aspx" target="_blank" title="ExecutionContext.SuppressFlow">Suppress the flow</a> and <a href="http://msdn.microsoft.com/en-us/library/system.threading.executioncontext.restoreflow.aspx" target="_blank" title="ExecutionContext.RestoreFlow">Restore it</a>, which can&nbsp;prevent temporarily the Call Context from flowing.</font> 
</p>
<p>
<font face="Courier New" size="2"></font>
</p>

{% include imported_disclaimer.html %}
