---
layout: post
title: ".NET Threads, CallContext and ILogicalThreadAffinative"
date: 2008-02-10 21:53:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2008/02/10/NET-Threads-CallContext-and-ILogicalThreadAffinative", "/post/2008/02/10/net-threads-callcontext-and-ilogicalthreadaffinative"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="trebuchet ms,geneva"><font size="2">I&#39;ve recently been looking for a way to automatically&nbsp;pass information from a thread&#39;s current call context to any thread that&#39;s been spawned from this thread. This can be useful for many reasons, and sometimes having some TLS information like the current user, or some custom context information.</font> </font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><font size="2">This can be done by wrapping any thread entry point around some custom code that will effectively pass the context to the new thread. This is a bit annoying, especially if this is information that is generated internally by a framework, because it requires the user of the framework to always use the wrapping method.</font> </font>
</p>
<p align="justify">
<font face="Verdana" size="2"><font face="trebuchet ms,geneva">There&#39;s a way around this by using the</font> <a href="http://msdn2.microsoft.com/en-us/library/system.runtime.remoting.messaging.callcontext.aspx"><font face="Courier New">CallContext</font></a> <font face="trebuchet ms,geneva">class, and particularly</font> <a href="http://msdn2.microsoft.com/en-us/library/system.runtime.remoting.messaging.callcontext.getdata.aspx"><font face="Courier New">GetData</font></a><font face="Courier New">/</font><a href="http://msdn2.microsoft.com/en-us/library/system.runtime.remoting.messaging.callcontext.setdata.aspx"><font face="Courier New">SetData</font></a> <font face="trebuchet ms,geneva">methods. Problem is, if you set some data in the CallContext, it will not pass onto a spawned thread. Actually, it will if the type you are placing in the CallContext implements the</font> <a href="http://msdn2.microsoft.com/en-us/library/system.runtime.remoting.messaging.ilogicalthreadaffinative.aspx"><font face="Courier New">ILogicalThreadAffinative</font></a> <font face="trebuchet ms,geneva">interface.</font></font> 
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><font size="2">This is a marker interface that is used to avoid context data that is not meant to &quot;flow&quot; through each spawned thread.</font> </font>
</p>
<p align="justify">
<font face="Verdana" size="2"><font face="trebuchet ms,geneva">It&#39;s also interesting to know that</font> <font face="Courier New">ILogicalThreadAffinative</font> <font face="trebuchet ms,geneva">flagged types will also be passed along&nbsp;to threads spawned by the thread pool, and incidentally to delegates enqueued via the</font> <font face="Courier New">BeginInvoke</font> <font face="trebuchet ms,geneva">compiler generated method.</font></font><font face="trebuchet ms,geneva"> </font>
</p>
<p align="justify">
<font face="Verdana" size="2"><font face="trebuchet ms,geneva">Finally, in case of a remoting call, any</font> <font face="Courier New">ILogicalThreadAffinative</font> <font face="trebuchet ms,geneva">flagged type will also serialized to the remote context and be serialized back to the local context.</font></font><font face="trebuchet ms,geneva"> </font>
</p>
<p align="justify">
<font size="2"><font face="trebuchet ms,geneva">Being able to </font><a href="http://blogs.msdn.com/sburke/archive/2008/01/16/configuring-visual-studio-to-debug-net-framework-source-code.aspx"><font face="trebuchet ms,geneva">step through the Framework&#39;s code</font></a><font face="trebuchet ms,geneva"> has been somehow a time saver to&nbsp;better understand this&nbsp;:)</font></font> 
</p>

{% include imported_disclaimer.html %}
