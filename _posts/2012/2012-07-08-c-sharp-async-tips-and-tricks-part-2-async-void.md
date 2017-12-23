---
layout: post
title: "C# Async Tips and Tricks Part 2 : Async Void"
date: 2012-07-08 17:05:00 -0400
comments: true
category: Archive
tags: [".NET", ".NET", "Windows 8", "Windows 8"]
redirect_from: ["/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void"]
author: jay
---
<!-- more -->
<p><em>TL;DR: This article discusses the differences between <em>async Task</em> and <em>async void</em>, and how <em>async void</em> methods and <em>async void </em>lambdas, used outside the DispatcherSynchronizationContext, can crash the process if exceptions are not handled.</em></p>
<p><em>You can also read&nbsp;the <em><a href="http://jaylee.org/post/2012/09/29/C-Async-Tips-and-Tricks-Part-3-Tasks-and-the-Synchronization-Context.aspx"><span style="color: #00709f;">part 3, Tasks and the Synchronization Context</span></a>. </em></em></p>
<p>In the <a href="http://www.jaylee.org/post/2012/06/18/CSharp-5-0-Async-Tips-and-Tricks-Part-1.aspx">first part of the series</a>, we discussed the behavior of async methods.&nbsp;The second part&nbsp;discusses how <em>async Task</em> and <em>async void</em> methods can differ in behavior, while seemingly being similar.</p>
<p>&nbsp;</p>
<h2>Authoring Async Methods</h2>
<p>Async methods can be authored in three different ways:</p>
<pre class="brush: c-sharp">async Task MyMethod() { }</pre>
<p>which creates a method that can be awaited, but does not return any value,</p>
<pre class="brush: c-sharp">async Task&lt;T&gt; MyReturningMethod { return default(T); }</pre>
<p>which creates a method that can be awaited, and returns a value of the type T,</p>
<pre class="brush: c-sharp">async void MyFireAndForgetMethod() { }</pre>
<p>which is allows for fire and forget methods, and cannot be awaited.</p>
<p>You may be wondering why there are two ways to declare a void returning method. Read on.</p>
<p>[more]</p>
<h2><em>async void</em> methods</h2>
<p><em>async void</em> methods exist for the single purpose of being used as an event handler. More specifically, it has been introduced to support UI elements event handlers, nothing else.</p>
<p>It allows writing simple async methods such as this one:</p>
<pre class="brush: c-sharp">private async void Button1_Click(object sender, EventArgs args) { }</pre>
<p>All Microsoft provided events respect a <a href="http://msdn.microsoft.com/library/ms182133(VS.100).aspx">BCL guideline</a> for which all .NET event delegates return void, which is in part&nbsp;why the <em>async Task</em> declaration cannot not be used.</p>
<p>An <em>async void</em> method is handled differently under the hood by the compiler. Task returning async methods are managed by the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.asynctaskmethodbuilder(v=vs.110).aspx">AsyncTaskMethodBuilder</a> class, whereas <em>async void</em> methods are managed by the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.asyncvoidmethodbuilder(v=vs.110).aspx">AsyncVoidMethodBuilder</a> class.</p>
<p>Both builders handle the synchronization context capture the same way, where the ambient context is reused to execute each block of code between the awaits of an async method.</p>
<p>This means than in normal circumstances, the code executes the same way in both Task and void returning methods.</p>
<p>But there is one fundamental difference between <em>async Task</em> and <em>async void</em>: Exceptions handling.</p>
<h2><em></em>&nbsp;</h2>
<h2><em>async Task</em> methods and Exceptions</h2>
<p>Let&rsquo;s consider this example, using an <em>async Task</em>:</p>
<pre class="brush: c-sharp">static void Main(string[] args)
{
   Test();
   Console.ReadLine();
}

public static async Task Test()
{
   throw new Exception();
}</pre>
<p>When executed, this program will not end until a key is pressed. The exception will be stored in an anonymous&nbsp;Task instance, because the return value of call to Test is never awaited, nor stored in a variable.</p>
<p>It is a pretty bad practice to write this kind of code, because the exception is essentially silent, and may hide a bug. The compiler is a bit helpful in that regard to avoid writing&nbsp;that kind of code, and will provide you with the following warning message:</p>
<p style="padding-left: 30px;"><span style="font-family: courier new,courier;">warning CS4014: Because this call is not awaited, execution of the current method continues before the call is completed. Consider applying the 'await' operator to the result of the call.</span></p>
<p>As a best practice, you should never have this warning anywhere in your code, and handle exceptions properly.</p>
<p>&nbsp;</p>
<h2>TPL unhandled exceptions</h2>
<p>The TPL, which used behind async methods,&nbsp;handles exceptions differently than the ThreadPool, and unobserved exceptions will be thrown in the <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.taskscheduler.unobservedtaskexception(v=vs.100).aspx">UnobservedTaskException</a> event.</p>
<p>If we modify a bit our sample:</p>
<pre class="brush: c-sharp">static void Main(string[] args)
{
    TaskScheduler.UnobservedTaskException += TaskScheduler_UnobservedTaskException;
    Test();

    Console.ReadLine();

    GC.Collect();
    GC.WaitForPendingFinalizers();

    Console.ReadLine();
}

static void TaskScheduler_UnobservedTaskException(object sender, UnobservedTaskExceptionEventArgs e)
{
    Console.WriteLine(e.Exception);
}

public static async Task Test()
{
    throw new Exception();
}
</pre>
<p>We&rsquo;ll notice that the event is not raised immediately after the task is in a faulted state. The task instance must be collected before, which is why forcing a garbage collection will raise the event.</p>
<p>As a best practice, you should always add a handler to the UnobservedTaskException as a safety net to log those exceptions, and probably treat them as potential bugs.</p>
<p>&nbsp;</p>
<h2><em>async void</em> methods and Exceptions</h2>
<p>Now we&rsquo;ll be using the same code, but change the <em>async Task</em> to an <em>async void</em> method:</p>
<pre class="brush: c-sharp">static void Main(string[] args)
{
    Test();
    Console.ReadLine();
}

public static async void Test()
{
    throw new Exception();
}
</pre>
<p>First, you&rsquo;ll notice that the compiler warning that was present with <em>async Task</em> is no longer shown.</p>
<p>Second, when run, the whole process crashes.</p>
<p>The reason behind this is the Synchronization Context used by the AsyncVoidMethodBuilder, being none in this example. When there is no ambient Synchronization Context, any exception that is unhandled by the body of an <em>async void</em> method is rethrown on the ThreadPool.</p>
<p>While there is seemingly no other logical place where that kind of unhandled exception could be thrown, the unfortunate effect is that the process is being terminated, because unhandled exceptions on the <a href="http://msdn.microsoft.com/en-us/library/ms228965.aspx">ThreadPool effectively terminate the process since .NET 2.0</a>. You may intercept all unhandled exception using the <a href="http://msdn.microsoft.com/en-us/library/system.appdomain.unhandledexception.aspx">AppDomain.UnhandledException</a> event, but there is no way to recover the process from this event.</p>
<p>To make the matter a bit more difficult, this event is not available in Metro apps, because the AppDomain class has been removed.</p>
<p>&nbsp;</p>
<h2>Async lambdas</h2>
<p>Unless you consider yourself as reliable enough to never forget to place a try catch block in all <em>async void</em> methods, you should never use an <em>async void</em> method anywhere but in UI event handlers, such as behind button clicks and alike.</p>
<p>You could make a code review and search for all <em>async void</em> methods using a find and replace... but you won&rsquo;t find this one:</p>
<pre class="brush: c-sharp">new List&lt;int&gt;().ForEach(async i =&gt; { throw new Exception(); });</pre>
<p>The method signature for List&lt;T&gt;.ForEach is :</p>
<pre class="brush: c-sharp">public void ForEach(Action&lt;T&gt; action);</pre>
<p>for which it is valid to provide and <em>async void</em> lambda, putting the process at risk of termination.</p>
<p>This kind of code is very easy to write when moving stateful code to async-only APIs, and you may crash your process without being warned that you&rsquo;re writing dangerous code.</p>
<p>An other interesting fact about these <em>async void</em>&nbsp;lambdas is that the compiler, when facing these two overloads :</p>
<pre class="brush: c-sharp">public void ForEach(Action&lt;T&gt; action);
public void ForEach(Func&lt;T, Task&gt; action);</pre>
<p>with this call :</p>
<pre class="brush: c-sharp">ForEach(async i =&gt; { });</pre>
<p>Will automatically choose the Task returning Func&lt;&gt;.</p>
<p>&nbsp;</p>
<h2><em>async void</em> is for UI Event Handlers Only</h2>
<p>When writing UI event handlers, <em>async void</em> methods are somehow painless because exceptions are treated the same way found in non-async methods; they are thrown on the Dispatcher. There is a possibility to recover from such exceptions, with is more than correct for most cases.</p>
<p>Outside of UI event handlers however, <em>async void</em> methods are somehow dangerous to use&nbsp;and may not that easy to find.</p>
<p>This is why I&rsquo;ve been using a custom Static Analysis (fxcop) rule that prevents the use of the AsyncVoidMethodBuilder class in my apps, effectively banning both explicit and implicit <em>async void</em> methods.</p>
<p>Most of this code is using an MVVM approach where ICommand data-binding is used, therefore limiting the use of any <em>async void</em> methods.</p>
<p>&nbsp;</p>
<h2>What to Take Away</h2>
<p>Here is what to remember about <em>async Task/void</em> methods:</p>
<ul>
<li><em>async void</em> methods are only suited for UI event handlers</li>
<li><em>async void</em> methods outside the dispatcher raise unhandled exceptions on the thread pool, which terminate the process</li>
<li><em>async void</em> lambdas are difficult to find, but easy to write. You should be looking for any use of the AsyncVoidMethodBuilder in your generated IL</li>
<li>You should always be using or awaiting the return value of an async method (treat CS4014 as an error)</li>
<li>Always add a handler to <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.taskscheduler.unobservedtaskexception(v=vs.100).aspx">UnobservedTaskException</a> event, to log exceptions in case the previous point is not respected.</li>
</ul>
<p>I&rsquo;ll probably soon be publishing a simple code source for the custom static analysis rule that prevents the use AsyncVoidMethodBuilder class, to streamline the process of banning <em>async void</em> methods.&nbsp;</p>
{% include imported_disclaimer.html %}
