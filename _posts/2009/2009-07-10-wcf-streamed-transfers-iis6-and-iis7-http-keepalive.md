---
layout: post
title: "WCF Streamed Transfers, IIS6 and IIS7 HTTP KeepAlive"
date: 2009-07-10 21:18:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2009/07/10/WCF-Streamed-Transfers-IIS6-and-IIS7-HTTP-KeepAlive.aspx", "/post/2009/07/10/wcf-streamed-transfers-iis6-and-iis7-http-keepalive.aspx"]
author: jay
---
<!-- more -->
<p>
<a href="http://blogs.codes-sources.com/jay/archive/2009/07/11/wcf-transferts-streamed-iis6-et-iis7-http-keepalive.aspx" target="_blank"><em>Ce billet est disponible en francais.</em></a> 
</p>
<p>
A while back, I was working on a client issue where I was having some kind of unusual socket exception from a WCF client connecting to an IIS6 hosted WCF service. 
</p>
<p>
<strong>To get a long story short, </strong><strong>if you&#39;re using the .NET 3.5 WCF streamed transfer on IIS6 and making a lot of transfers in a small time, disable the KeepAlive feature on your web site. The performance will be lower, but it will last longer (without a client support call). </strong>
</p>
<p>
Still here with me ? :) If you have a bit more time to read, here some detail about what I found on this issue... 
</p>
<p>
The setup is pretty simple : A WCF client that is sending a stream over a WCF service that has the transferMode set to streamed. This allows the transfer of a lot of information using genuine streaming, which means that the client writes to a System.IO.Stream instance, and the server reads from an other System.IO.Stream, and the data does not need to be transferred all at once, like in a &quot;normal&quot; SOAP communication. I&#39;m using the required basicHttpBinding for both ends.
</p>
<p>
The strange thing is that after having made more than 15000 requests to transfer streams,&nbsp; I was receivig this exception :
</p>
<p><code>
	<font face="courier new,courier">System.ServiceModel.CommunicationException: Could not connect to http://server/streamtest/StreamServiceTest.Service1.svc.<br />
	TCP error code 10048: Only one usage of each socket address (protocol/network address/port) is normally permitted 10.0.0.1:80.&nbsp; <br />
	---&gt; System.Net.WebException: Unable to connect to the remote server <br />
	---&gt; System.Net.Sockets.SocketException: Only one usage of each socket address (protocol/network address/port) is normally permitted 10.0.0.1:80</font>
	</code></p>
<p>
This is a rather common issue, which is mostly found where an application tries to bind to a TCP port but cannot do so, either because it is already being bound to an other application or because it does not use the SO_REUSEADDR socket option and the port was closed very recently.
</p>
<p>
What is rather unusual is that this exception is raised on the client side and not on the server side !
</p>
<p>
After a few netstat -an, I found out that an awful lot of sockets were lingering with the following state :
</p>
<p><code>
	<font face="courier new,courier">TCP&nbsp;&nbsp;&nbsp; the.client:50819&nbsp;&nbsp;&nbsp;&nbsp; the.server:80&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TIME_WAIT</font>
	</code></p>
<p>
There were something like 15000 lines of this, with incrementing numbers for the local port. This state is normal, it&#39;s meant to be that way, but it&#39;s generally more found lingering on a server, much less on a client. 
</p>
<p>
That could mean only one thing, considering that IIS6.0 is an HTTP/1.1 compliant web server: WCF is requesting that the connection to be closed at after a streamed transfer.
</p>
<p>
Wireshark being my friend, I started looking up at the content of the dialog between IIS6 and my client application : 
</p>
<p><code>
	<font face="courier new,courier">POST /streamtest/StreamServiceTest.Service1.svc HTTP/1.1<br />
	MIME-Version: 1.0<br />
	Content-Type: multipart/related; type=&quot;application/xop+xml&quot;;start=&quot;&lt;http://tempuri.org/0&gt;&quot;;boundary=&quot;uuid:41d2cf74-aaa6-4a80-a6c4-0ec37692a437+id=1&quot;;start-info=&quot;text/xml&quot;<br />
	SOAPAction: &quot;http://tempuri.org/IService1/Operation1&quot;<br />
	Host: the.server<br />
	Transfer-Encoding: chunked<br />
	Expect: 100-continue<br />
	Connection: Keep-Alive</font> 
	</code></p>
<p>
The server answers this : 
</p>
<p><code>
	HTTP/1.1 100 Continue 
	</code></p>
<p>
&nbsp;Then the stream transfer takes place, gets the SOAP response, then at the end :
</p>
<blockquote>
	<font face="courier new,courier">HTTP/1.1 200 OK<br />
	Date: Sat, 11 Jul 2009 01:40:16 GMT<br />
	Server: Microsoft-IIS/6.0<br />
	X-Powered-By: ASP.NET<br />
	X-AspNet-Version: 2.0.50727<br />
	Connection: close<br />
	MIME-Version: 1.0<br />
	Transfer-Encoding: chunked<br />
	Cache-Control: private</font><br />
</blockquote>
<p>
I quickly found out that IIS6.0, or the WCF handler is forcing the connection to close on this last request. That&#39;s not particularly unusual, since a server may explictly deny an HTTP client to keep alive the connection.
</p>
<p>
What&#39;s even more unusual is that out of luck by trying to deactivate the IIS6.0 keep alive setting on my web site, I noticed that all the connections were properly closed on the client...!
</p>
<p>
I tried analysing a bit deeper the dialog between the client and the server, and I noticed two differences :
</p>
<ol>
	<li>The content of the final answer of the IIS contains <strong>two </strong>&quot;Connection: close&quot; headers, which could mean one by the WCF handler, and one by IIS itself. I&#39;m not sure if repeating headers is forbidden in the RFC, I&#39;d have to read it again to be sure.</li>
	<li>It looks like the order of the FIN/ACK, ACK packets is a bit different, but I&#39;m not sure either where that stands. Both the client and the server are sending FIN packets to the other side, probably the result of calling Socket.Close().<br />
	</li>
</ol>
<p>
But then I found out something even stranger : It all works on IIS7 ! And the best of all, the KeepAlive status is honored by the web server. That obviously means that the global performance of the web service is better on IIS7 than it is on IIS6, since there is only one connection opened for all my 15000 calls, which is rather good. Too bad my client cannot switch to IIS7 for now...
</p>
<p>
It also seems that the WCF client is not behaving the same way it does with IIS6, because at the TCP level only the client is sending a TCP FIN packet and the server is not when the keep alive is disabled.
</p>
<p>
I think I&#39;ll be posting this on Microsoft Connect soon, but I&#39;m not sure where the problem lies, whether it is in IIS6, the WCF
client or the WCF server handler, but there is definitely an issue here. 
</p>

{% include imported_disclaimer.html %}
