---
layout: post
title: "Mono 1.0.5 support for NetBSD 2.0"
date: 2004-12-15 11:37:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2004/12/15/Mono-105-support-for-NetBSD-20", "/post/2004/12/15/mono-105-support-for-netbsd-20"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">As I promised earlier, here is the <a href="http://msdn.labtech.epitech.net/mono/mono-netbsd-20041216.patch.gz">patch for Mono 1.0.5</a> to run on NetBSD 2.0-Release.</font>
</p>
<p>
<font face="Tahoma" size="2">I&#39;ve managed to get MonoDoc 1.0.5 to run on my box and near being able to run MonoDevelop too. It seems that somewhere, a mutex is unlocked twice and since the libpthread is asserting on that kind of invalid behavior, MonoDevelop stops. But even if assertions are disabled, MonoDevelop stops while not being able to read a perfectly valid file... Well :) There&#39;s some work to be done here.</font>
</p>
<p>
<font face="Tahoma" size="2">Earlier I talked about the fact that I forgot to save the stack&#39;s address of suspended threads. Under Linux, where signals are used to &quot;suspend&quot; threads, the signal handler looks like this :</font>
</p>
<font color="#0000ff"><font face="Courier New" size="2">&nbsp;&nbsp;&nbsp;void </font></font><font size="2"><font face="Courier New">GC_suspend_handler(<font color="#0000ff">int</font> sig)<br />
&nbsp;&nbsp; {<br />
<font color="#0000ff">&nbsp;&nbsp; &nbsp;&nbsp; int</font> dummy;</font> </font>
<p>
<font face="Courier New" size="2" color="#008000">&nbsp;&nbsp; &nbsp;&nbsp; /* some not important stuff... */</font>
</p>
<font face="Courier New" size="2">&nbsp;&nbsp; &nbsp;&nbsp; pthread_t my_thread = pthread_self();</font><font color="#008000"><font size="2"> </font>
<p>
<font face="Courier New" color="#000000"><font size="2">&nbsp;&nbsp;&nbsp; &nbsp; me -&gt; stop_info.stack_ptr = (ptr_t)(&amp;dummy);&nbsp; <font color="#008000">/* Get the top of the stack address */</font></font></font>
</p>
<p>
<font color="#000000"><font face="Courier New"><font size="2">&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;sigsuspend(&amp;suspend_handler_mask); </font><font size="2" color="#008000">/* Wait for signal to resume */<br />
</font></font></font><font size="2"><font face="Courier New" color="#000000">&nbsp;&nbsp; }</font><br />
</font><br />
<font face="Tahoma" size="2" color="#000000">The thread being stopped is held into its signal handler, waiting for the signal to resume and exit the signal handler. The main reason for this &quot;trick&quot; is that there is no standard way to suspend a thread with the libpthread.</font>
</p>
</font>
<p>
<font size="2"><font face="Tahoma" color="#000000">However, NetBSD&#39;s libpthread has two nice methods <font face="Courier New">pthread_suspend_np</font> and <font face="Courier New">pthread_resume_np</font>, which are able to suspend&nbsp;and resume&nbsp;arbitrarily a specific thread. So in the GC code, instead of raising signals to specific threads, calling these two methods is only necessary.</font></font>
</p>
<p>
<font size="2"><font face="Tahoma" color="#000000">The thing I missed in the first patch, is the storage of the top&nbsp;stack&#39;s address, retreived by using the <font face="Courier New">dummy</font> variable in the signal handler. So, to work around this problem, I used a third non standard function that is able to retreive the base adress, not the top address,&nbsp;of the stack for the suspended thread and give it to the GC.</font></font>
</p>
<p>
<font face="Tahoma" size="2">Although, the patch seems to please mono, which is running fine, the stack address given to the GC is not the perfect one. Unfortunately, there is no way, as far as I know, to get the top of stack pointer for suspended threads. I can&#39;t tell for now the impact of this modification, but still, mono does not seem to complain, neither does the GC.</font>
</p>
<p>
<font face="Tahoma" size="2">I also tried to mix signals and thread suspension, but it seems that signals in general are breaking GC&#39;s operations.</font>
</p>
<p>
<font face="Tahoma" size="2">Maybe some pthread/GC guru could enlighten me :)</font>
</p>
<p>
<font face="Tahoma" size="2"><strong>Update</strong> : It also works nicely with Mono 1.1.3. Actually, it runs better as it has a little bug that has been fixed in System.Timers.Timer, but this has nothing to do with NetBSD ;)</font>
</p>

{% include imported_disclaimer.html %}
