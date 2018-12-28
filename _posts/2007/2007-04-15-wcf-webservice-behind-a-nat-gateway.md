---
layout: post
title: "WCF WebService behind a NAT Gateway"
date: 2007-04-15 11:06:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/04/15/WCF-WebService-behind-a-NAT-Gateway.aspx", "/post/2007/04/15/wcf-webservice-behind-a-nat-gateway.aspx"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Verdana" size="2">I&#39;m currently working on a WCF service that&#39;s exposed via a <a href="http://msdn2.microsoft.com/en-us/library/ms731361.aspx">basicHttpBinding</a>, with some exposition of the metadata via a WSDL url.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">The metadata generator has been improved a bit since it is now split into multiple files, with references from the first wsdl to other URI&#39;s. There&#39;s not much to do to have that WSDL generated, and the generator take the liberty of using the machine&#39;s name to create reference URIs.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Most of the time, this is a good idea, but sometimes when your machine is behind a NAT gateway doing some port forwarding, your machine probably won&#39;t have the public FQDN used to access your gateway. You then end up with a root document referencing URI&#39;s with a host name that is not valid on the other side of the gateway.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">The solution to change that behavior is quite simple : <a href="http://support.microsoft.com/kb/324287/">Just change the host name IIS is listening on</a>. Use the MMC snap-in, and specify that your port 80 (or whatever port you&#39;re using) is listening on your external FQDN. WCF will then use that host name when generating URI&#39;s. Also don&#39;t forget that if you specify a host name, you might need to add an other entry to be able to access your website using &quot;localhost&quot;.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">This operation is also needed for servers farm when doing some load balancing.</font>
</p>

{% include imported_disclaimer.html %}
