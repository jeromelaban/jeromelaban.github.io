---
layout: post
title: "[WP7] Using an Exchange Account With a Custom Certificate"
date: 2010-07-31 15:07:00 -0400
comments: true
category: Archive
tags: ["Windows Phone Dev"]
redirect_from: ["/post/2010/07/31/WP7-Using-Exchange-Account-With-a-Custom-Certificate.aspx", "/post/2010/07/31/wp7-using-exchange-account-with-a-custom-certificate.aspx"]
author: jay
---
<!-- more -->
<p>Depending on which corporation you work for, you may have to connect to your exchange server using a <a href="http://weblogs.asp.net/scottgu/archive/2007/04/06/tip-trick-enabling-ssl-on-iis7-using-self-signed-certificates.aspx">self-signed server certificate</a> to be used with HTTPS protocol (using either TLS or SSL).</p>
<p>If you're unlucky enough to be in this situation, but are using a modern browser, you can install the certificate in either your windows certificate store, or using your browser's store. You can do that using <a href="http://stackoverflow.com/questions/681695/what-do-i-need-to-do-to-get-ie8-to-accept-a-self-signed-certificate">this lengthy technique</a> for IE8.</p>
<p>But if you're on a Windows Phone 7, if you try to connect to your exchange account, you'll get a nice  message telling you that there is a problem with the server certificate. Well, neither Internet Explorer or the bundled Exchange tools give you the ability to install that custom certificate. And there is no access to the file system either.</p>
<p>Luckily, you can email your certificate on your GMail account for instance, and the WP7 mail client has the ability to install certificates !</p>
<p>So, use the <a href="http://stackoverflow.com/questions/681695/what-do-i-need-to-do-to-get-ie8-to-accept-a-self-signed-certificate">lengty technique</a> to export your certificate in the ".cer" format by connecting to your exchange server using its HTTPS address in Internet Explorer on your PC, email it to yourself, and tap on it on your Windows Phone 7 to install it.</p>
<p>Now you can enjoy having your work emails and calendars on your weekends, in case you don't have anything else better to do :)</p>
<p>&nbsp;</p>
<p>EDIT: If it still does not work, you may need to also import the full chain of certificates, up to the root. To do so, in the certificate from your exchange server, open the "Certification Path" tab, then for each item in the tree, click "View Certificate", then "Details", then "Copy to File...". Email each certificate to your Windows Phone and you're done !</p>
{% include imported_disclaimer.html %}
