---
layout: post
title: "Don't get C# volatile the wrong way"
date: 2004-08-05 11:45:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2004/08/05/Dont-get-C-volatile-the-wrong-way", "/post/2004/08/05/dont-get-c-volatile-the-wrong-way"]
author: jerome
---
<!-- more -->
<p>
<font face="Tahoma" size="2">Don&#39;t get the C# volatile the wrong way. There is a lot of blurriness around synchronization issues in .NET, especially around the Memory Barriers, System.Monitor, the lock keyword and stuff like this.</font>
</p>
<p>
<font face="Tahoma" size="2">It is common to have objects that are able to create Unique identifiers, by mean of a index incremented each time a new value is retreived. It would, using a simplistic view, look like this :</font>
</p>
<span style="font-size: 8pt; font-family: sixbyten"><font face="Courier New"><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span>&nbsp;&nbsp; </span>public</span><span style="font-size: 8pt; font-family: 'Courier New'"> <span style="color: blue">class</span> UniqueIdentifier</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;&nbsp; <br />
&nbsp;&nbsp; </span></span></font></span><span style="font-size: 8pt; font-family: sixbyten"><font face="Courier New"><span style="font-size: 8pt; font-family: 'Courier New'">{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;</span><span>&nbsp;&nbsp;</span><span>&nbsp;&nbsp;<br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span></font></span><span style="font-size: 8pt; font-family: sixbyten"><font face="Courier New"><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">private</span> <span style="color: blue">static</span> <span style="color: blue">int</span> _currentIndex = <span style="color: fuchsia">0</span>;</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">public</span> <span style="color: blue">int</span> NewIndex<br />
</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;&nbsp;</span><span>&nbsp;&nbsp;&nbsp;</span>{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;&nbsp; </span><span>&nbsp;&nbsp;&nbsp;</span><span>&nbsp;&nbsp;&nbsp;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="color: blue">get</span>&nbsp;</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;&nbsp;&nbsp;</span><span>&nbsp;&nbsp;&nbsp;</span><span>&nbsp;&nbsp;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="color: blue">return</span> _currentIndex++;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="font-size: 8pt; font-family: 'Courier New'">}<br />
</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;&nbsp;</span><span>&nbsp;&nbsp;&nbsp;</span>}<br />
</span><span style="font-size: 8pt; font-family: 'Courier New'"><span>&nbsp;&nbsp; </span>}</span></font></span><span style="font-size: 8pt; font-family: sixbyten"><font face="Courier New"></font></span> 
<p>
<font face="Tahoma" size="2">This is pretty straightforward: Each time the NewIndex property is called, a new index is returned.</font>
</p>
<p>
<font face="Tahoma" size="2">But there is a problem in a multithreaded environment, where multiple threads can call the NewIndex at the same time. If we look at the code generated for the getter, here is what we have:</font>
</p>
<p>
<font face="Courier New" size="2">&nbsp; IL_0000:&nbsp; ldsfld&nbsp;&nbsp;&nbsp;&nbsp; int32 UniqueIdentifier::_currentIndex<br />
&nbsp; IL_0005:&nbsp; dup<br />
&nbsp; IL_0006:&nbsp; ldc.i4.1<br />
&nbsp; IL_0007:&nbsp; add<br />
&nbsp; IL_0008:&nbsp; stsfld&nbsp;&nbsp;&nbsp;&nbsp; int32 UniqueIdentifier::_currentIndex</font>
</p>
<p>
<font face="Tahoma" size="2">One thing about the multithreading in general, the system can stop the execution anywhere it wants, especially between the operation at 0 and the end of the operation at 8. The effect is pretty obvious : If during this stop time, some other thread executes that same piece of code, each thread ends up with the same &quot;new&quot; index, each one incrementing from the same index. This scenario is in the presence of a uni-processor system, which interleaves the execution of running threads. On multi-processor systems, threads do not even need to be stopped to have this exact same problem. While this is harder that kind of race condition on a uniprocessor, this is far more easier to fall into with multiple processors.</font>
</p>
<p>
<font face="Tahoma" size="2">This is a very common problem when programming in multithreaded environments, which is generally fixed by means of synchronization mecanisms like Mutexes or CriticalSections. The whole operation needs to be atomic which&nbsp;means executed by at most one thread at a time.</font>
</p>
<p>
<font face="Tahoma" size="2">In the native world, in C/C++ for instance, the language does not provide any &quot;built-in&quot; synchronization mecanisms and the programmers have to do all the work by hand. The .NET framework with C#, on the other hand, provides that kind of mecanisms integrated in the language : the volatile and lock keywords.</font>
</p>
<p>
<font face="Tahoma" size="2">A&nbsp;common and incorrect&nbsp;interpretation of the volatile keyword is to think that all <strong>operations</strong> (opposed to&nbsp;accesses)&nbsp;on a volatile variables are synchronized. This generally leads to this kind of code :</font>
</p>
<span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span>&nbsp;&nbsp; </span>public</span><span style="font-size: 8pt; font-family: 'Courier New'"> <span style="color: blue">class</span> UniqueIdentifier</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp; </span>{<br />
&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">private</span> <span style="color: blue">static</span> <span style="color: blue">volatile</span> <span style="color: blue">int</span> _currentIndex = <span style="color: fuchsia">0</span>;</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">public</span> <span style="color: blue">int</span> NewIndex<br />
&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="font-size: 8pt; font-family: 'Courier New'">{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="color: blue">get</span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="font-size: 8pt; font-family: 'Courier New'">{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="color: blue">return</span> _currentIndex++;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="font-size: 8pt; font-family: 'Courier New'">}<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="font-size: 8pt; font-family: 'Courier New'">}</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp; </span>}</span><span style="font-family: 'Courier New'"></span> 
<p>
<font face="Tahoma" size="2">While this code is valid, it does not fix the synchronization problem encountered. The correct interpretation of the volatile keyword is that read and write operations to a volatile fields must not be reordered and, that the value of the variable must not be cached. </font>
</p>
<p>
<font face="Tahoma" size="2">On a single x86 processor system, the only effect of the volatile keyword is that the value is never cached in something like a register and is always fetched from the memory. Since there is only one set of&nbsp;caches and one processor, there is no risk to have inconsistencies where memory would have been modified elsewhere. (This is called processor Self-Consistency)<br />
</font><font face="Tahoma" size="2">But, on a multiprocessor system, each processor has a data cache set&nbsp;and depending on the cache policy, an updated value for the variable might not be written back immediatly into the main memory to&nbsp;make it available to the other&nbsp;threads requesting it. In fact it may never be updated, depending on the cache policy. Actually, this kind of situation is really hard to reproduce&nbsp;because of the high utilization of the cache and frequent flushes.</font>
</p>
<p>
<font face="Tahoma" size="2">Back to volatile, it means that read/write operations will always target the main memory. In practice, a volatile read or write is called a Memory Barrier. Then, when using a volatile variable&nbsp;the thread&nbsp;is&nbsp;sure to have the latest value. <br />
Back to our example, while we are sure to have the latest value, the read/increment/write operation is still not atomic and can be interrupted right in the middle.</font>
</p>
<p>
<font face="Tahoma" size="2">To have a correct implementation of this UniqueIndentifier generator, we have to make it atomic :</font>
</p>
<span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span>&nbsp;</span><span>&nbsp; </span>public</span><span style="font-size: 8pt; font-family: 'Courier New'"> <span style="color: blue">class</span> UniqueIdentifier</span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'">{</span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">private</span> <span style="color: blue">static</span> <span style="color: blue">int</span> _currentIndex = <span style="color: fuchsia">0</span>;</span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">private</span> <span style="color: blue">object</span> _syncRoot = <span style="color: blue">new</span> <span style="color: blue">object</span>();</span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">public</span> <span style="color: blue">int</span> NewIndex </span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'">{</span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'"><span style="color: blue">get</span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="font-size: 8pt; font-family: 'Courier New'">{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="color: blue">lock</span>(_syncRoot)</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>{</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span><span style="color: blue">return</span> _currentIndex++;</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>}</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span>}</span><span style="font-size: 8pt; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>}</span><span style="font-size: 8pt; color: blue; font-family: 'Courier New'"><span><br />
&nbsp;&nbsp; </span></span><span style="font-size: 8pt; font-family: 'Courier New'">}</span><span style="font-family: 'Courier New'"></span> 
<p>
<font face="Tahoma" size="2">In this example, we are using the lock keyword. This is pretty nice because it uses the System.Threading.Monitor class to create a scope that can be entered by one thread at a time. </font><font face="Tahoma" size="2">While this solves the atomicity problem, you might notice that the volatile keyword is not present anymore. This is where the Monitor does something under the hood : It does a memory barrier at the acquisition of the lock and an other one a the release of the lock.</font>
</p>
<p>
<font face="Tahoma" size="2">A lot of implicit stuff done by the CLR and it can be pretty hard to catch up on all this. Besides, the x86 memory model is pretty strong and does&nbsp;not introduce a lot of race conditions, while it would on a weaker memory model like the <font face="Tahoma" size="2"></font>one on Itanium.</font>
</p>
<p>
<font face="Tahoma" size="2">As a conclusion,&nbsp;use <strong>lock</strong> and forget <strong>volatile</strong>. :)</font>
</p>
<p>
<font face="Tahoma" size="2">By the way, this </font><a href="http://www.yoda.arachsys.com/csharp/multithreading.html"><font face="Tahoma" size="2">article</font></a>&nbsp;<font face="Tahoma" size="2">is pretty interesting on the subject.</font>
</p>

{% include imported_disclaimer.html %}
