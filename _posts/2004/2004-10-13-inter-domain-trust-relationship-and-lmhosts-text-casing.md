---
layout: post
title: "Inter-Domain Trust Relationship and lmhosts Text Casing"
date: 2004-10-13 11:40:00 -0500
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2004/10/13/Inter-Domain-Trust-Relationship-and-lmhosts-Text-Casing", "/post/2004/10/13/inter-domain-trust-relationship-and-lmhosts-text-casing"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">Among the things that are interesting in latest versions of Samba, there are NT4 Inter-Domain Trust Relationships. In the Windows world, the original NT4 domain model was not scalable enough when NT4 was omnipresent. Companies fusions did not mix well with the single windows domain to merge all user accounts and everything else. </font>
</p>
<p>
<font face="Tahoma" size="2">One solution brought back then was the ability to have an NT4 domain to trust users authenticated in an other NT4 domain. This comes in multiple flavors known as incoming, outgoing, bidirectional trust to allow users to be authenticated in both or only selected domains. Other things like transitive and non-transitive trusts can also be used in at least a 3 domains interaction to allow users from domain, say,&nbsp;A to be authenticated in domain C through B, if the trust between A and B is transitive.</font>
</p>
<p>
<font face="Tahoma" size="2">This facility is now fully integrated in Windows 2000/2003 domains and this allows to trust NT4 style domains, which includes Samba managed domains. </font>
</p>
<p>
<font face="Tahoma" size="2">Here at epitech we a growing need to interconnect a number of small domains to allow users to connect to a variety of services and to allow an unified password management, a samba domain is used to map users between the Windows and the Unix world.</font>
</p>
<p>
<font face="Tahoma" size="2">When establishing the trust between two domains , there are at least two possible scenarios : </font>
</p>
<ul>
	<li><font face="Tahoma" size="2">The domain can be identified using a Netbios broadcast or WINS resolution, which implies that the domain is known and has registerd itself to the wins or is located on the same physical IPv4 subnet,</font> </li>
	<li><font face="Tahoma" size="2">Or, the domain cannot be identified using a single broadcast and must be identified via a manual addition of the domain in the WINS or the domain and the associated domain controller have been added to the lmhosts file.</font></li>
</ul>
<p>
<font face="Tahoma" size="2">During a lot of trying, I have found that a combination of both the WINS and lmhosts modification are needed to establish the trust. If you don&#39;t do both, the domain controller that wants to establish the trust tries to partially resolve the remote DC by using a really weird NETLOGON/UDP packet that is of course rejected (and not even logged) by Samba. The rest is done by an attempt to locate the remote DC by a local subnet broadcast, which of course fails.</font>
</p>
<p>
<font face="Tahoma" size="2">You might say :&nbsp;&quot;Well, modifying the lmhosts file should be a piece of cake !&quot;. Actually yes, writing&nbsp;it is. So here is what you might want to add in your lmhosts file :</font>
</p>
<p>
<font face="Courier New">&nbsp;&nbsp;&nbsp;&nbsp; 10.0.0.1&nbsp;&nbsp;&nbsp;&nbsp;mysamba-dc&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#PRE&nbsp;&nbsp; #DOM:midhearth<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;10.0.0.1&nbsp;&nbsp; &quot;midearth&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; \0x1b&quot;&nbsp; #PRE</font> 
</p>
<p>
<font face="Tahoma" size="2">Which seems to be good. By the way, the \x1b is used to identify the midearth entry as a domain and should be placed at the 16 byte index. Historical laziness... What a shame.&nbsp; Anyway, this does not work and produces the strange behavior I described earlier. Here is a version of the same block of text that really works :</font>
</p>
<p>
<font face="Courier New">&nbsp;&nbsp;&nbsp;&nbsp; 10.0.0.1&nbsp;&nbsp;&nbsp;&nbsp;MYSAMBA-DC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#PRE&nbsp;&nbsp; #DOM:MIDEARTH<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;10.0.0.1&nbsp;&nbsp; &quot;MIDEARTH&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; \0x1b&quot;&nbsp; #PRE</font> 
</p>
<p>
<font face="Tahoma" size="2">Noticing anything ? Yes ! All names are uppercased... And no error or warning message notifies you about this when casing is not correct. After that, everything works fine and the trust can be establised. I kind of hate losing time with that kind of tricks, but hey, it works&nbsp;now&nbsp;:)</font>
</p>

{% include imported_disclaimer.html %}
