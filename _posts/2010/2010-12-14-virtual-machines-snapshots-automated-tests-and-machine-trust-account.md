---
layout: post
title: "Virtual Machines, Snapshots, Automated Tests and Machine Trust Account"
date: 2010-12-14 21:47:00 -0500
comments: true
category: Archive
tags: []
redirect_from: ["/post/2010/12/14/Virtual-Machines-Snapshots-Automated-Tests-and-Machine-Trust-Account", "/post/2010/12/14/virtual-machines-snapshots-automated-tests-and-machine-trust-account"]
author: jay
---
<!-- more -->
<p>During the development of a project, you may need at some point to automate the testing of your whole&nbsp;application. Virtual Machines make that very easy, especially with <a href="http://msdn.microsoft.com/en-us/vstudio/ee712698">Visual Studio Lab Management</a>&nbsp;in the loop.</p>
<p>You can have a step in your nightly build to have your application installed silently, and then have some acceptance&nbsp;tests running on it.</p>
<p>To make all this repeatable and mostly predictable, you can use snapshots of a clean environment and restore that environment before starting your test scenarios. That way, any previous test run does not affect the new run, as long as the tests can run confined in a single machine. For multiple machines tests, like a web front end and DB backend, you may need to synchronize&nbsp;all of the snapshots, but this is definitely doeable.</p>
<h3>The case of the VM part of a Domain</h3>
<p>Your tests may also need to have a domain account to run. To do this, you join your VM to the domain, then make a snapshot.</p>
<p>Your automated&nbsp;tests run fine for about a month, then start to fail for reasons like :</p>
<div class="errormsg" style="PADDING-LEFT: 30px; FONT-FAMILY: "><span style="font-family: courier new,courier;"><span style="font-size: medium;">The trust relationship between this workstation and the primary domain failed.</span></span></div>
<div>&nbsp;</div>
<div>Or something a bit&nbsp;different like :</div>
<div>&nbsp;</div>
<div class="errormsg" style="PADDING-LEFT: 30px"><span style="font-family: courier new,courier;"><span style="font-size: medium;">This computer could not authenticate with </span></span><a href="file://\\MYDC"><span style="font-family: courier new,courier;"><span style="font-size: medium;">\\MYDC</span></span></a><span style="font-family: courier new,courier;"><span style="font-size: medium;">, a Windows domain controller for domain MYDOMAIN, and therefore this computer might deny logon requests.</span></span></div>
<div class="errormsg" style="PADDING-LEFT: 30px">&nbsp;</div>
<div class="errormsg" style="PADDING-LEFT: ">
<div>Which&nbsp;means that the machine account registered with the domain is out of sync.</div>
<div>&nbsp;</div>
<div>For a bit of background, the machine account&nbsp;is&nbsp;used by windows subsystem, but also the "NT AUTHORITY\System" account to communicate with the domain to apply <a href="http://en.wikipedia.org/wiki/Group_Policy">GPOs</a>, for instance. This account is also used for many other things, like in <a href="http://www.microsoft.com/systemcenter/en/us/virtual-machine-manager.aspx">SCVMM</a>, to ensure that the server has access to a host without requiring a "real" account that has administrative access to that host.</div>
<div>&nbsp;</div>
<h3>Password Renewal</h3>
<p>&nbsp;</p>
<div>That machine trust account has a password like a normal user account, even though it is not accessible to the users. That password is changed periodically,&nbsp;every 30 days or so, and that change is initiated by a request of the machine tied to this account. It is designed in such a way that&nbsp;it allows machines to be offline for long periods and not get out of sync because the DC would have changed the password unilaterally.</div>
<div>&nbsp;</div>
<div>At this point, you're probably seeing why that account information can be out of sync when using VMs and&nbsp;reverting to snapshots.</div>
<div>&nbsp;</div>
<div>When joining the domain, the account is in sync, and it is possible to revert to that snapshot until that 30 days window is reached. Then, the live snapshot of the machine asks for password renewal of the trust account, and then you revert to your original snapshot. This is where the problem occurs, the machine cannot authenticate on the domain anymore because it is using the previous password.</div>
<h3>&nbsp;</h3>
<h3>Keeping the password in sync</h3>
<p>&nbsp;</p>
<div>There are a few techniques to keep that password in sync, the first being a&nbsp;leave&nbsp;and join domain sequence. This is fairly easy to do, but there are some caveats.</div>
<div>&nbsp;</div>
<div>When you leave and re-join, you get a new SID for that machine, which means that if you gave access rights to&nbsp;the previous&nbsp;machine account on an other machine, you're forced to update your ACLs on that other machine.</div>
<div>&nbsp;</div>
<div>But there is a lesser known technique&nbsp;based on&nbsp;the <a href="http://technet.microsoft.com/en-us/library/cc737599(WS.10).aspx">netdom</a> command, to reset that trust account&nbsp;password :&nbsp;</div>
<div>&nbsp;</div>
<div><span style="font-family: courier new,courier;"><span style="font-size: medium;">netdom resetpwd /server:MYDC /userd:MYDOMAIN\myuser /passwordD:* /securepasswordprompt​</span></span></div>
<div><span style="font-family: courier new,courier;"><span style="font-size: x-small;"><br /></span></span>
<div>This needs to be run on the target machine, using an account that has the rights to update machine accounts, which is most probably the same account you used to add your machine to the domain.</div>
<h3>Making it permanent</h3>
<p>&nbsp;</p>
<div>You still don't want to have to reset that password every two weeks, so you can use&nbsp;the <a href="http://technet.microsoft.com/en-us/library/cc962289.aspx">DisablePasswordChange</a>&nbsp;registry setting for that, which will disable the update of the machine account password.</div>
<div>This will make your VM a bit more vulnerable to password based attacks to hijack your machine account, just so you know.</div>
</div>
</div>
{% include imported_disclaimer.html %}
