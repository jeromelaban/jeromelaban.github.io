---
layout: post
title: "Revisited with the Reactive Extensions: DataBinding and Updates from multiple Threads"
date: 2010-07-26 18:42:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/07/26/Revisited-with-the-Reactive-Extensions-DataBinding-and-Updates-from-multiple-Threads.aspx", "/post/2010/07/26/revisited-with-the-reactive-extensions-databinding-and-updates-from-multiple-threads.aspx"]
author: admin
---
<!-- more -->
<p><em><a href="http://blogs.codes-sources.com/jay/archive/2010/07/26/revisit-avec-les-reactive-extensions-databinding-et-mise-jour-depuis-plusieurs-threads.aspx">Cet article est disponible en francais.</a></em></p>
<p>Recently, I wrote an article about <a href="http://jaylee.org/post/2010/01/02/WinForms-DataBinding-and-Updates-from-multiple-Threads.aspx">WinForms, DataBinding and Updates from multiple threads</a>, where I showed how to externalize the execution of event handler code on the UI thread.</p>
<p>I used a technique based on Action&lt;Action&gt; that takes advantage of closures, and the fact that an action will carry its context down to the point where it is executed. All this with no strings attached.</p>
<p>This concept of externalization can be revisited with the <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx">Reactive Extensions</a>, and the <a href="http://msdn.microsoft.com/en-us/library/system.concurrency.ischeduler%28VS.92%29.aspx">IScheduler</a> interface.</p>
<p>&nbsp;</p>
<h2>The Original Sample<br /></h2>
<p>But let's get right to the original code sample :</p>
<p>[code:c#]</p>
<p>&nbsp;&nbsp;&nbsp; public MyController(Action&lt;Action&gt; synchronousInvoker)<br />&nbsp;&nbsp;&nbsp; {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _synchronousInvoker = synchronousInvoker;<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ...<br />&nbsp;&nbsp;&nbsp; }<br /><br />[/code]</p>
<p>This code is the constructor of the controller for my form, and the synchronous invoker action will take something like this :</p>
<p>[code:c#]</p>
<p>&nbsp;&nbsp;&nbsp; _controller = new MyController(a =&gt; Invoke(a));<br />[/code]</p>
<p>And the invoker lambda is used like this :</p>
<p>[code:c#]<br />&nbsp;&nbsp;&nbsp; _synchronousInvoker(<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; () =&gt; PropertyChanged(this, new PropertyChangedEventArgs("Status"))<br />&nbsp;&nbsp;&nbsp; );<br /><br />[/code]</p>
<p>Where Invoke is actually <a href="http://msdn.microsoft.com/en-us/library/zyzhdc6b.aspx">Control.Invoke()</a>, used to execute code on the UI Thread where updates to the UI controls can be safely executed.</p>
<p>While the Action&lt;Action&gt; trick is working just fine in acheiving the isolation of concerns, it is not very obvious just by looking at the constructor signature what you are supposed to pass to it.</p>
<p>&nbsp;</p>
<h2>Using the IScheduler interface</h2>
<p>To be able to abstract the location used to execute the content of  Reactive operators, the Rx team introduced the concept of Scheduler,  with a bunch of <a href="http://msdn.microsoft.com/en-us/library/system.concurrency.scheduler_members%28VS.92%29.aspx">default  scheduler</a> implementations.</p>
<p>It basically exposes an abstracted way for users of the IScheduler instance to schedule the execution of a method in the specific way defined by the scheduler. In our case, we want to execute our handler code on the WinForms message pump, and "there's a scheduler for that".</p>
<p>The sample can be easily updated to use the IScheduler instead of the Action&lt;Action&gt; delegate, and make use of the <a href="http://msdn.microsoft.com/en-us/library/ff431922%28VS.92%29.aspx">IScheduler.Schedule()</a> method.</p>
<p>[code:c#]</p>
<p>&nbsp;&nbsp;&nbsp; public MyController(ISheduler scheduler)<br />&nbsp;&nbsp;&nbsp;  {<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _scheduler = scheduler;<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ...<br />&nbsp;&nbsp;&nbsp;  }<br /><br />[/code]</p>
<p>And replace the call by :</p>
<p>[code:c#]<br />&nbsp;&nbsp;&nbsp; _scheduler.Schedule(<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; () =&gt;  PropertyChanged(this, new PropertyChangedEventArgs("Status"))<br />&nbsp;&nbsp;&nbsp; );<br /><br />[/code]</p>
<p>Not a very complex modification, but it is far more readable.</p>
<p>And we can use the provided scheduler for the Winforms, the yet undocumented System.Concurrency.ControlScheduler which is not in the <a href="http://msdn.microsoft.com/en-us/library/system.concurrency%28VS.92%29.aspx">Scheduler</a> class because it cannot be created statically and requires a Control instance :</p>
<p>[code:c#]</p>
<p>&nbsp;&nbsp;&nbsp; _controller = new MyController(new ControlScheduler(this));<br />[/code]</p>
<p>where this is an instance of a control.</p>
<p>This is much better, and for the unit testing of the Controller, you can easily use the <a href="http://msdn.microsoft.com/en-us/library/system.concurrency.currentthreadscheduler%28VS.92%29.aspx">System.Concurrency.CurrentThreadScheduler</a>, because you don't need to switch threads in this context.</p>
<p>&nbsp;</p>
<h2>What about the Reactive Extensions and Silverlight for Windows Phone 7 ?</h2>
<p>In a very strange move, the WP7 team moved the IScheduler class from System.Concurrency to the very specific <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.reactive%28VS.92%29.aspx">Microsoft.Phone.Reactive</a> namespace.</p>
<p>I do not quite understand the change of namespace, and it makes code that used to compile easily on both platforms not compatible.</p>
<p>Maybe they considered the Reactive Extensions implementation for Windows Phone too different from the desktop implementation... But the compact framework was built around that assertion, and most of the common code stays in the same namespaces.</p>
<p>If someone has an explanation for that strange refactoring, I'm listening :)</p>
{% include imported_disclaimer.html %}
