---
layout: post
title: "[WP7Dev] Beware of the [ThreadStatic] attribute on Silverlight for Windows Phone 7"
date: 2010-06-19 21:36:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/06/19/WP7Dev-Beware-of-the-ThreadStatic-attribute-on-Silverlight-for-Windows-Phone-7", "/post/2010/06/19/wp7dev-beware-of-the-threadstatic-attribute-on-silverlight-for-windows-phone-7"]
author: admin
---
<!-- more -->
<p><em><a href="http://blogs.codes-sources.com/jay/archive/2010/06/20/wp7dev-attention-attribut-threadstatic-dans-silverlight-pour-windows-phone-7.aspx">Cet article est disponible en francais.</a></em></p>
<p>In other words, it is not supported !</p>
<p>And the worst in all this is that you don&rsquo;t even get warned that it&rsquo;s not supported... The code compiles, but the attribute has no effect at all ! Granted that you can read the msdn article about <a href="http://msdn.microsoft.com/en-us/library/ff426930%28VS.96%29.aspx#Threads" target="_blank">the differences between silverlight on Windows and Windows Phone</a>, but well, you may still miss it. Maybe a <a href="http://msdn.microsoft.com/en-us/library/dd380660.aspx" target="_blank">custom code analysis rule</a> could prevent this.</p>
<p>Still, you want to use ThreadStatic because you probably need it, somehow. But since it is not supported, you could try the <a href="http://msdn.microsoft.com/en-us/library/system.threading.thread.getnameddataslot.aspx" target="_blank">Thread.GetNamedDataSlot</a>, mind you.</p>
<p>Well, too bad. It&rsquo;s not supported either.</p>
<p>That leaves us implementing or own TLS implementation, by hand...</p>
<p>&nbsp;</p>
<h3>Updating Umbrella for Silverlight on Windows Phone</h3>
<p>I&rsquo;m a big fan of Umbrella, and the first time I had to use <a href="http://msdn.microsoft.com/en-us/library/bb347013.aspx" target="_blank">Dictionary&lt;&gt;.TryGetValue</a> and its magically aweful out parameter in my attempt to rewrite my <a href="http://jaylee.org/rc" target="_blank">Remote Control</a> app for Windows Phone 7, I decided to port <a href="http://umbrella.codeplex.com" target="_blank">Umbrella</a> to it. So I could use <a href="http://umbrella.codeplex.com/wikipage?title=IDictionary.GetValueOrDefault&amp;referringTitle=Home" target="_blank">GetValueOrDefault</a> without rewriting it, again.</p>
<p>I managed to get almost all the desktop unit tests to pass, except for those who emit code, use web features, use xml and binary serializers, call private methods using reflection, and so on.</p>
<p>There are a few parts where the code needed to be updated, because <a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.typedescriptor.aspx" target="_blank">TypeDescriptor</a> class is not available on WP7, you have to crash and burn to see if a value is convertible from one type to the other. But that&rsquo;s not too bad, it works as expected.</p>
<p>&nbsp;</p>
<h3>Umbrella&rsquo;s ThreadLocalSource</h3>
<p>Umbrella has this nice ThreadLocalSource class that wraps the <a href="http://en.wikipedia.org/wiki/Thread-local_storage" target="_blank">TLS</a> behavior, and you can easily create a static variable of that type instead of the <a href="http://msdn.microsoft.com/en-us/library/system.threadstaticattribute.aspx" target="_blank">ThreadStatic</a> static variable.</p>
<p>The Umbrella quick start samples make that kind of use for it :</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp; ISource&lt;int&gt; threadLocal = new ThreadLocalSource&lt;int&gt;(1);

&nbsp;&nbsp;&nbsp; int valueOnOtherThread = 0;

&nbsp;&nbsp;&nbsp; Thread thread = new Thread(() =&gt; valueOnOtherThread = threadLocal.Value);
&nbsp;&nbsp;&nbsp; thread.Start();
&nbsp;&nbsp;&nbsp; thread.Join();

&nbsp;&nbsp;&nbsp; Assert.Equal(1, threadLocal.Value);
&nbsp;&nbsp;&nbsp; Assert.Equal(0, valueOnOtherThread);
</pre>
<p>The main thread set the value to 1, and the other thread tries to get the same value from the other thread and it should be different (the default value of an int, which is 0).</p>
<p>&nbsp;</p>
<h3>Updating the ThreadLocalSource to avoid the use of ThreadStatic</h3>
<p>The TLS in .NET is basically a dictionary of string/object pairs that is attached to each running threads. So, to mimic this, we just need to make a list of all threads that want to store something for themselves and wrap it nicely.</p>
<p>We can create a variable of this type :</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp; private static Tuple&lt;WeakReference, IDictionary&lt;string, T&gt;&gt;[] _tls;</pre>
<p>That variable is intentionally an array to try to make use of memory spacial locality, and since on that platform we won&rsquo;t get a lot of threads, this should be fine when we got through the array to find one. This approach is trying to be lockless, by using a retry mechanism to update the array. The <a href="http://msdn.microsoft.com/en-us/library/system.weakreference.aspx" target="_blank">WeakReference</a> is used to avoid keeping a reference to the thread after it has been terminated.</p>
<p>So, to update the array, we can do as follows :</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp; private static IDictionary&lt;string, T&gt; GetValuesForThread(Thread thread)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Find the TLS for the specified thread
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var query = from entry in _tls

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Only get threads that are still alive
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let t = entry.T.Target as Thread

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Get the requested thread
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where t != null &amp;&amp; t == thread
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select entry.U;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var localStorage = query.FirstOrDefault();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (localStorage == null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bool success = false;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // The storage for the new Thread
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; localStorage = new Dictionary&lt;string, T&gt;();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; while(!success)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // store the original array so we can check later if there has not
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // been anyone that has updated the array at the same time we did
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var originalTls = _tls;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var newTls = new List&lt;Tuple&lt;WeakReference, IDictionary&lt;string, T&gt;&gt;&gt;();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Add the slots for which threads references are still alive
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; newTls.AddRange(_tls.Where(t =&gt; t.T.IsAlive));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var newSlot = new Tuple&lt;WeakReference, IDictionary&lt;string, T&gt;&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; T = new WeakReference(thread),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; U = localStorage
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; newTls.Add(newSlot);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // If no other thread has changed the array, replace it.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; success = Interlocked.CompareExchange(ref _tls, newTls.ToArray(), originalTls) != _tls;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return localStorage;
&nbsp;&nbsp;&nbsp; }</pre>
<p>Instead of the array, another dictionary could be created but I&rsquo;m not sure of the actual performance improvement that would provide, particularly for very small arrays.</p>
<p>Using a lockless approach like this one will most likely limit the contention around the use of that TLS-like class. There may be, from time to time, computations that are performed multiple times in case of race conditions on the update of the _tls array, but that is completely acceptable. Additionally, <a href="http://en.wikipedia.org/wiki/Deadlock#Livelock" target="_blank">livelocks</a> are also out of the picture on that kind of preemptive systems.</p>
<p>I think developing on that platform is going to be fully of little workarounds like this one... This is going to be fun !</p>
{% include imported_disclaimer.html %}
