---
layout: post
title: "When declarativeness goes away for performance"
date: 2011-07-25 20:35:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows Phone Dev"]
redirect_from: ["/post/2011/07/25/When-declarativeness-goes-away-for-performance.aspx", "/post/2011/07/25/when-declarativeness-goes-away-for-performance.aspx"]
author: jay
---
<!-- more -->
<p><strong><em>TLDR: The creation a&nbsp;C# <a href="http://msdn.microsoft.com/en-us/library/bb397951.aspx">expression tree</a> is not cached, and using it to simulate a methodof keyword is terribly slow. This article is about reconsidering the use of this technique when performance is a concern.</em></strong></p>
<p>&nbsp;</p>
<p>During the development of my last projects, and it's been like that for a while, I've been used to look for ways to express programs in a more declarative or functional way.</p>
<p>LINQ is a pretty good tool to acheive that, as well as fluent interfaces, lamdbas and all those neat language features and tricks.</p>
<p>&nbsp;</p>
<h2>When the language is against you</h2>
<p>But there are some times when the language is not there to support patterns, like with the use of the <a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.aspx">IPropertyChanged</a> interface. The&nbsp;language (and the CLR for that matter) does not&nbsp;publicly support intercepting calls to properties or&nbsp;methods.&nbsp;That can actually be done through&nbsp;<a href="http://msdn.microsoft.com/en-us/library/system.runtime.remoting.proxies.realproxy.aspx">Transparent Proxies</a> or <a href="http://www.castleproject.org/dynamicproxy/index.html">dynamic proxy</a> generation, but these are not supported on Windows Phone 7. The latter somehow will on WP7.1.</p>
<p>Anyway, using that interface requires raising an event with a string containing the name of the property that has changed.</p>
<p>The use of metadata in the form of strings is unfortunately not refactoring friendly, and if you change your property name, you've got a bug on your hands.</p>
<p>Since there is no <a href="http://stackoverflow.com/questions/1213862/why-is-there-not-a-fieldof-or-methodof-operator-in-c">methodof</a> keyword <a href="http://blogs.msdn.com/b/ericlippert/archive/2009/05/21/in-foof-we-trust-a-dialogue.aspx">in C#</a>, you could say that the language is against you since there&nbsp;is no direct way to get the name of a property or method at compile time.</p>
<p>This can still be done through reflection&nbsp;with <a href="http://blog.decarufel.net/2010/03/how-to-use-strongly-typed-name-with.html">plumbing code</a>&nbsp;that uses Expression Trees to work around it, and it works pretty fine.</p>
<p>That way you can easily write nice wrappers like this one :</p>
<pre class="brush: c-sharp">public int MyProperty 
{
    get { return GetValue(() =&gt; MyProperty); }
    set { SetValue(() =&gt; MyProperty, value); }
}
</pre>
<p>That way, you get both the type safety and the refactoring friendly use of INotifyPropertyChanged.</p>
<p>&nbsp;</p>
<h2>Where declarativeness does not shine</h2>
<p>If you crack open the assembly for a property like this one with a disassembler, you get this :</p>
<pre class="brush: c-sharp">public int MyProperty
{
    get
    {
        return this.GetValue&lt;int&gt;(
            Expression.Lambda&lt;Func&lt;int&gt;&gt;(
               Expression.Property(
                  Expression.Constant(this, typeof(MainPage)
               ),
               (MethodInfo) methodof(MainPage.get_MyProperty)
            ),
        new ParameterExpression[0]));
    }
    set
    {
        this.SetValue&lt;int&gt;(
           Expression.Lambda&lt;Func&lt;int&gt;&gt;(
              Expression.Property(
                 Expression.Constant(this, typeof(MainPage)),
                 (MethodInfo) methodof(MainPage.get_MyProperty)
              ),
           new ParameterExpression[0]),
        value);
    }
}</pre>
<p>That's a lot of code !</p>
<p>And worse, that's not the end of it, because you'll have to traverse the expression just to get the "methodof", and then call the "Name" property to get the string. All this for a string that will never change after you've compiled your assembly.</p>
<p>&nbsp;</p>
<h2>When declarativeness goes away for performance</h2>
<p>But that would not be that bad if you executed that code once, or ran it on a desktop computer (or you don't care about performance). For desktop and server applications, where the cost of executing that kind of code is (almost) neglectible, you do not care much about that.</p>
<p>But if you execute that code a few million times, or run it on a Windows Phone 7 on the UI thread, you've got a problem. The expression is not cached, neither in a static variable or an instance variable, depending on the context. Sure, you could store it in an expression typed&nbsp;variable, and cache it manually that way, but you'd lose a bit declarativeness.</p>
<p>To make a small comparison, it takes <strong>13 seconds </strong>on my Samsung Focus to execute <strong>10,000 gets </strong>of the property using expressions, and takes <strong>0.2 milliseconds </strong>to do the same using a simple string.</p>
<p>Pretty easy to choose, isn't it ?</p>
<p>That's where you lose the declarativeness away for performance, and cringe&nbsp;a little bit about it when you know that you'll have to chase a bug in the future because of a lazy rename. Still, you can have debug-time only code that walks up the stack and checks that the property actually exists, but you have to execute that code to find the bug.</p>
<p>&nbsp;</p>
<h2>Mitigating</h2>
<p>Hopefully, there are tools like <a href="http://www.sharpcrafters.com/">Postsharp</a> that help in that regard, where a post processing step does the reflection once and for all, and injects that missing string. That's direction I'd rather not take, but since there's still&nbsp;no out of the box solution, that can be a good fit.</p>
<p>&nbsp;</p>
<p>We're so used to techniques that avoid us to write boring code, but when performance is a concern, it is necessary to reconsider all coding reflexes and think twice before using them.</p>
{% include imported_disclaimer.html %}
