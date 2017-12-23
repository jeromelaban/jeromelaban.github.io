---
layout: post
title: "ReaderWriterLockSlim on Windows Phone 8 and the seemingly random MethodAccessException"
date: 2013-01-13 15:02:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2013/01/13/ReaderWriterLockSlim-on-Windows-Phone-8-and-the-seemingly-random-MethodAccessException", "/post/2013/01/13/readerwriterlockslim-on-windows-phone-8-and-the-seemingly-random-methodaccessexception"]
author: jay
---
<!-- more -->
<p><em>TL;DR: Don't use the <a href="http://msdn.microsoft.com/en-us/library/System.Threading.ReaderWriterLockSlim.aspx">ReaderWriterLockSlim</a>&nbsp;class&nbsp;on Windows Phone 8 RTM, it has a bug&nbsp;that appears only under contention.</em></p>
<p>Windows Phone 8&rsquo;s move to the NT kernel has had a lot of advantages for the developer, such as the move to the same .NET CLR as the Desktop Windows, but also the ability to have multi-core based environment.</p>
<p>More specifically, there is one access synchronization &ndash; the <a href="http://msdn.microsoft.com/en-us/library/System.Threading.ReaderWriterLockSlim.aspx">ReaderWriterLockSlim</a> &ndash; which makes a lot of sense in real multi-core environment.</p>
<p>I&rsquo;ve been using this class to synchronize access to a dictionary abstraction&nbsp;for performance reasons and also for legacy reasons,&nbsp;since that the <a href="http://msdn.microsoft.com/en-us/library/system.collections.concurrent.aspx">Concurrent Collections</a> are available. Note that we do have a new tool in the toolbox, the <a href="http://blogs.msdn.com/b/bclteam/archive/2012/12/18/preview-of-immutable-collections-released-on-nuget.aspx">BCL Immutable Collections</a>, that are&nbsp;becoming my new preferred way for creating collections.</p>
<p>[more]</p>
<h2>The (seemingly) random System.MissingMethodException</h2>
<p>After moving a project to Windows Phone 8, I started seeing this <a href="http://msdn.microsoft.com/en-us/library/system.methodaccessexception.aspx">MethodAccessException</a> sporadically, seemingly without any reason :</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">[System.MethodAccessException]</span><br /><span style="font-family: courier new,courier;">Attempt by method 'System.Threading.ReaderWriterLockSlim.WaitOnEvent(System.Threading.EventWaitHandle, UInt32 ByRef, TimeoutTracker)' to access method 'System.Threading.WaitHandle.WaitOne(Int32, Boolean)' failed.</span></p>
<p>Which seems a bit odd.</p>
<p>I was able to reproduce the issue this way :</p>
<pre class="brush: c-sharp">var r = new ReaderWriterLockSlim();
r.TryEnterUpgradeableReadLock(-1); 
r.TryEnterWriteLock(-1);
ThreadPool.QueueUserWorkItem(_ =&gt; { r.TryEnterWriteLock(-1); });</pre>
<p>With this code pasted in the constructor of the app.xaml.cs file of an empty app.</p>
<p>This means that until there is actual contention on the lock, the bug will not show up. This makes it show up seemingly randomly.</p>
<p>&nbsp;</p>
<h2>A bit of geeky digging</h2>
<p>As always, I just can't let an issue like this not deeply understood, and I've dug a bit deeper to understand why this is happening.</p>
<p>In Windows Phone 8, the ReaderWriterLockSlim type is in the System.Core.dll assembly, and considering the exception stack trace and message, the ReaderWriterLockSlim.WaitOnEvent method is trying to access the WaitHandle.WaitOne(Int32, Boolean) method which is located in the mscorlib.dll assembly.</p>
<p>The problem is actually pretty simple: The WaitHandle.WaitOne(Int32, Boolean) method is declared as private in another assembly, making the runtime fail when linking the method.</p>
<p>This brings another interesting insight on how&nbsp;Microsoft handles the Framework, because this code could not have been compiled using the current set of assemblies. Somehow, the System.Core.dll was built using a different mscorlib.dll which has this method either public or most probably&nbsp;internal (because mscorlib.dll has the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute.aspx">InternalsVisibleTo</a>("System.Core") attribute set).</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<h2>A workaround</h2>
<p>To my knowledge, this issue is present in Windows Phone 8 RTM (Apollo), and I&rsquo;ve not been able to validate that this is still present in the Portico update.</p>
<p>For the time being, I&rsquo;ve moved back to a Monitor based synchronization mechanism until this gets fixed.</p>
<p>So in short, don&rsquo;t use ReaderWriterLockSlim on Apollo.</p>
{% include imported_disclaimer.html %}
