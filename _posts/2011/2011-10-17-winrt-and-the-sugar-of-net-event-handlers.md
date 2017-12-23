---
layout: post
title: "WinRT and the syntactic sugar around .NET event handlers"
date: 2011-10-17 19:48:00 -0400
comments: true
category: Archive
tags: ["Windows 8"]
redirect_from: ["/post/2011/10/17/WinRT-and-the-sugar-of-NET-event-handlers", "/post/2011/10/17/winrt-and-the-sugar-of-net-event-handlers"]
author: jay
---
<!-- more -->
<p>If you've watched the <a href="http://channel9.msdn.com/events/BUILD/BUILD2011">great number of videos available</a> from the Build conference, you've probably noticed that the layer between .NET and WinRT is very thin.</p>
<p>So thin that in permeates through to C# 5.0, even though it's not immediately visible to the naked eye.</p>
<p>&nbsp;</p>
<p>Also, that Windows 8&nbsp;developer preview is pretty stable... I'm writing this blog post using it, and it's pretty good :) (lovin' the inline spell checker, everywhere&nbsp;!!)</p>
<p>&nbsp;</p>
<h2>What about WinRT ?</h2>
<p>The Windows Runtime has been explained <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh464942(v=VS.85).aspx">here</a>, <a href="http://geekswithblogs.net/lbugnion/archive/2011/09/17/my-thoughts-about-build-windows-8-winrt-xaml-and-silverlight.aspx">there</a> and&nbsp;by <a href="http://tirania.org/blog/archive/2011/Sep-15.html">Miguel de Icasa</a>&nbsp;(and <a href="http://julien.dollon.net/post/MicrosoftDev-Comprendre-Windows-8-WinRT.aspx">there too</a>&nbsp;by Julien Dollon), but to summarize in other words, WinRT is (at least for now) the new way to communicate with the Windows Core, with an improved developer experience. It's the new preferred (and only, as far as I know) way to develop Metro style applications, in many languages like C#/F#/VB, C++, JavaScript and more...</p>
<p>The API is oriented toward developing tablet applications, with the power and connectivity limitation that kind of platform has, plus the addition of what makes Windows Phone so interesting. That means Live Tiles, background agents, background transfers, XAML, background audio, social APIs, camera, sensors, location, and new features like sharing and search contracts, ...</p>
<p>My favorite part of all this is the new addition of a rule that make a LOT of sense : <strong>If an API call nominally takes more than 50ms to execute, then it's an asynchronous api call</strong>. But not using the ugly Begin/End pattern, rather through the nice <a href="http://blogs.msdn.com/b/ericlippert/archive/2010/10/29/asynchronous-programming-in-c-5-0-part-two-whence-await.aspx">async/await</a> pattern, WinRT style (I'll elaborate on that in a later post). I've even started to apply that rule to my existing development with the Reactive Extensions (And that's yet an other later post).</p>
<p>Microsoft has taken the approach of cleaning up the .NET framework with the ".NET Core" profile. For instance, the new <a href="http://msdn.microsoft.com/en-us/library/system.reflection.typeinfo(v=VS.110).aspx">TypeInfo</a> class now separates the introspection part from the type safety part that were historically merged in the System.Type type. This segregation limits the loading of type metadata only when necessary, and not when just doing a simple typeof(). Now, the System.Type type is fairly simple, and to get back all the known methods like GetMethods() or GetProperties() there's an extension method called Type.GetTypeInfo() in System.Reflection that gives back all the reflection side.</p>
<p>There are a lot of other differences, I'll discuss in a later post. (yeah, that's&nbsp;a lot to talk about !)</p>
<p>For the .NET developer, <strong>WinRT takes the form of *.winmd files that follow the .NET standard metadata format</strong> (kind of like <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/aa366757(v=vs.85).aspx">TLB files</a>&nbsp;on steroids, if you know what I mean...). These files can be directly referenced from .NET code like any other assembly, it's then very easy to call the underlying Windows platform. No more P/Invoke.</p>
<p>Just before you start freaking out, WinRT does not replace the standard .NET 4.5 full platform you already know, remember that. That's just a new profile, much like Windows Phone or Xbox 360 are profiles, but targeted at Metro style&nbsp;app<span style="text-decoration: line-through;">lication</span>s. (It's not applications anymore, it's apps :) just so you know...)</p>
<p>&nbsp;</p>
<h2>But how thin is the layer, really ?</h2>
<p>To accommodate all these languages, compromises had to be made and&nbsp;underneath, WinRT is native code. Native code means no garbage collection, limited value types, a&nbsp;pretty different exception handling (SEH), and so on.</p>
<p>The CLR and C# compiler teams have made a great job at trying to hide all this but there are still some corner cases where you can see those differences appear.</p>
<p>For instance, you'll find that there are two EventHandler types :&nbsp;the existing <a href="http://msdn.microsoft.com/en-us/library/system.eventhandler.aspx">System.EventHandler</a>, and the new <a href="http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.eventhandler">Windows.UI.Xaml.EventHandler</a>. What's the difference ? See for yourself :</p>
<pre class="brush: c-sharp">namespace System
{
    [ComVisible(true)]
    public delegate void EventHandler(object sender, EventArgs e);
}</pre>
<p>And the other one :</p>
<pre class="brush: c-sharp">namespace Windows.UI.Xaml
{
    // Summary:
    //     Represents a basic event handler method.
    [Version(100794368)]
    [WebHostHidden]
    [Guid(3817893849, 19739, 19144, 164, 60, 195, 185, 8, 116, 39, 152)]
    public delegate void EventHandler(object sender, object e);
}</pre>
<p>The difference is subtle, but it's there : the second parameter is an object. This is kind of troubling, and having to juggle between the two is going to be a bit messy. That's going to be the forced return of conditional compilation and the myriads of #if and #endif...</p>
<p>But the difference does not stop here though. Let's look at how the WinRT handler can be used :</p>
<pre class="brush: c-sharp">public class MyCommand : Windows.UI.Xaml.Input.ICommand
{
    public event Windows.UI.Xaml.EventHandler CanExecuteChanged;

    public bool CanExecute(object parameter) { }

    public void Execute(object parameter) { }
}</pre>
<p>Translates to this, after the compiler does its magic&nbsp;:</p>
<pre class="brush: c-sharp">using System.Runtime.InteropServices.WindowsRuntime;
public class MyCommand : Windows.UI.Xaml.Input.ICommand
{
    public event Windows.UI.Xaml.EventHandler CanExecuteChanged
    {
        add
        {
            return this.CanExecuteChanged.AddEventHandler(value);
        }
        remove
        {
            this.CanExecuteChanged.RemoveEventHandler(value);
        }
    }

    public bool CanExecute(object parameter) { }

    public void Execute(object parameter) { }

    public MyCommand()
    {
        this.CanExecuteChanged = 
           new EventRegistrationTokenTable();
    }
}</pre>
<p>The delegates are not stored in a multicast delegate instance like they used to be, but are now stored in an <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh138412(v=VS.110).aspx">EventRegistrationTokenTable</a> type instance, and <strong>provides a return value&nbsp;for the add handler</strong> ! Also, the remove handler "value" is a <a href="http://msdn.microsoft.com/en-us/library/system.runtime.interopservices.windowsruntime.eventregistrationtoken(v=VS.110).aspx">EventRegistrationToken</a> instance.</p>
<p>That construct is&nbsp;so new that even the intellisense engine is mistaken by this new syntax if you try to write it by yourself, but it compiles correctly.</p>
<p>The return value is of type EventRegistrationToken, and I'm guessing that it must be something WinRT must keep track of to call marshaled managed delegates.</p>
<p>The calling part is also very interesting, if you try to register to that event&nbsp;:</p>
<pre class="brush: c-sharp">// Before
MyCommand t = new MyCommand();
t.CanExecuteChanged += (s, e) =&gt; { };</pre>
<pre class="brush: c-sharp">// After
MyCommand t = new MyCommand();
WindowsRuntimeMarshal.AddEventHandler(
   new Func(t.add_CanExecuteChanged)
 , new Action(t.remove_CanExecuteChanged)
 , delegate(object s, object e) { }
);</pre>
<p>Quite different, isn't it ?</p>
<p>But this syntactic sugar seems only to be related to the fact that the WinRT EventHandler delegate type is exposed as a implemented interface member, like in ICommand. It does not appear if it is used somewhere else.</p>
<h2>&nbsp;</h2>
<h2>Cool. Why should care ?</h2>
<p>Actually, you may not care at all, unless you write ICommand implementations.</p>
<p>If you write a command, and particularly ICommand wrappers or proxies, you may want to write your own add/remove handlers and to be able to do so, you'll need to return that EventRegistrationToken too, and map that token to your delegate.</p>
<p>Here's what I came up with :</p>
<pre class="brush: c-sharp">public class MyCommand : Windows.UI.Xaml.Input.ICommand
{
    EventRegistrationTokenTable _table = new EventRegistrationTokenTable();
    Dictionary _reverseTable = new Dictionary();
        
    public event EventHandler CanExecuteChanged
    {
        add
        {
            var token = _table.AddEventHandler(value);
            _reverseTable[token] = value;

            // do something with value

            return token;
        }

        remove
        {
            // Unregister value 
            RemoveMyHandler(_reverseTable[value]);

            _table.RemoveEventHandler(value);
        }
    }
}</pre>
<p>All this because the EventRegistrationTokenTable does not expose a bi-directional mapping between event handlers and their tokens.</p>
<p>But remember, WinRT and Dev11 are in Developer Preview state. That's not even beta. This will probably change !</p>
{% include imported_disclaimer.html %}
